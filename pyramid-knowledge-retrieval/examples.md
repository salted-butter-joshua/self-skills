# 输出样例（节选）

输入：**PyTorch 算子在 NVIDIA GPU 上如何调度**

以下展示 **§0 前半 + §3 的 L1/L2 完整写法**。L3/L4 同模板续写。

---

# 金字塔检索 · PyTorch 算子在 NVIDIA GPU 上的调度

## 0｜概览与完整链路

| 顶层问句 | 推理模式 | 边界 |
|----------|----------|------|
| `torch` 算子从 Python 调用到 GPU 执行，调度链如何传递？ | 演绎 | eager；不展开 compile/NCCL |

**完整链路叙事**

用户在 Python 调用 `torch.add(x,y)` 时，请求先进入 ATen 算子入口，由 **Dispatcher** 根据张量上的 **分派键（DispatchKey）** 选出该走 Autograd 包装还是直达 CUDA 实现——这是 CPU 侧的「选路」。若需要训练，Autograd 内核会先 **构建计算图节点** 并 **二次分派** 到不含 Autograd 的 CUDA 内核做实际加法。CUDA 内核侧用 **TensorIterator** 准备内存视图，再通过 **gpu_kernel** 把 kernel **发射到当前 CUDA Stream** 队列；此时 CPU 即可返回，GPU 驱动按流内顺序排队执行。最后 GPU **Block Scheduler** 把 thread block 分给各 SM，SM 内 **Warp Scheduler** 在 32 线程一组的 Warp 间切换指令，完成 SIMT 执行。整条链是：选路 →（可选）记图 → 发射入流 → 硬件执行。

**有序主链表**

| 步骤 | 环节 | 输入 | 输出 | 一句话 |
|------|------|------|------|--------|
| 1 | Python→ATen 入口 | `torch.add` 调用 | `aten::add` C++ 调用 | 进入算子库 |
| 2 | Dispatcher 分派 | 张量 DispatchKeySet | 选中内核函数指针 | CPU 选路 |
| 3 | Autograd（条件） | 需梯度张量 | 计算图节点 + 重分派 | 记图后再算 |
| 4 | CUDA 实现 | 张量数据指针 | launch 参数 | 准备 kernel |
| 5 | Stream 入队 | grid/block/stream | 驱动队列中的 job | 异步排队 |
| 6 | SM/Warp 执行 | grid | 写回显存 | 硬件算完 |

**待回答疑问清单**

| # | 疑问 | 计划层 |
|---|------|--------|
| 1 | 为什么需要 Dispatcher，不能 if/else 吗？ | L1 |
| 2 | 分派键是什么、存在哪？ | L2 |
| 3 | 为什么需要分派键？ | L1 |
| 4 | 分派器与分派键什么关系？ | L2 |
| 5 | 分派如何实现？ | L3 |
| 6 | 计算图是什么？ | L2 |
| 7 | 什么情况需要图？ | L1 |
| 8 | 图如何构建？ | L3 |
| 9 | 为什么二次分派？ | L3 |
| 10 | CUDA Stream 是什么、为何需要、哪一步、如何实现？ | L2/L3 |
| 11 | Stream 与 Warp 调度区别？ | L1 |
| 12 | Block 如何分到 SM？ | L3 |

---

## 3｜逐层深入（节选 L0–L2）

## L1 · 各段为什么存在

**本层在整条链中的位置**  
L0 四段边界 → **【论证 Dispatcher / 图 / Stream 的必要性】** → L2 关键对象定义

**本层要解决的疑问**  
1. 为什么需要分派？ ✅ 步骤 1  
2. 为什么需要分派键？ ✅ 步骤 2  
3. 什么情况需要计算图？ ✅ 步骤 3  
4. 为什么需要 CUDA Stream？ ✅ 步骤 4  

---

### 步骤 1 · 为什么需要 Dispatcher（分派）

**为什么需要**  
PyTorch 有 1600+ 算子 × 多后端 × 横切能力（Autograd、Autocast…）。每算子写巨型 `switch(device)` 无法扩展。Dispatcher 把「选哪个实现」剥离成一张表。

**是什么**  
中央路由：每个算子名对应 dispatch table（分派键 → 函数指针）。

**如何做到**  
编译期 `TORCH_LIBRARY_IMPL` 注册；运行期 `Dispatcher::call` 查表跳转；DispatchKeySet 位运算合并后 O(1) 索引。

**接上一步 / 交下一步**  
- 上一步：知有「软件分派」段  
- 本步：知分派为多后端+横切扩展  
- 下一步：L2 定义分派键载体  

**例子**  
自定义算子注册 `CPU` 与 `CUDA` 两个块，无需改 `add` 源码。

**依据**  
PyTorch [Registering a Dispatched Operator](https://docs.pytorch.org/tutorials/advanced/dispatcher.html)

---

### 步骤 2 · 为什么需要分派键

**为什么需要**  
同一次 `aten::add`，在不同上下文要选不同内核（CUDA / Autograd / Autocast）。需要运行时可变的路由标签。

**是什么**  
枚举标签（`CUDA`、`AutogradCUDA`…），附在 `TensorImpl` 或 TLS 上。

**如何做到**  
`TensorImpl::key_set_` 位集；调用时输入张量键并集 + TLS include/exclude → 最高优先级键查表。

**例子**  
`requires_grad=True` → 带 `AutogradCUDA`，优先级高于 `CUDA`，先进 Autograd 内核。

**依据**  
Dispatcher Walkthrough

---

### 步骤 3 · 什么情况需要计算图

**为什么需要**  
反向传播需知前向算子拓扑才能链式求导；无数值图只能差分，无法高效训练。

**是什么**  
Autograd DAG：节点 = `grad_fn`（如 `AddBackward0`），不是 CUDA Graph。

**如何做到**  
Autograd 键内核：创建 `grad_fn`、登记边 → 再 redispatch 到 CUDA 核做实际计算。

**例子**  
`z=x+y` 需梯度：前向挂 `AddBackward0`；`z.backward()` 再分派 backward CUDA 核。

**依据**  
Dispatcher 教程；Walkthrough `add_Tensor`

---

### 步骤 4 · 为什么需要 CUDA Stream

**为什么需要**  
避免每次 launch 同步等 GPU；同流有序保证 add 后再 relu 无需手动 barrier。

**是什么**  
单 GPU 上 FIFO 工作队列。

**如何做到**  
`getCurrentCUDAStream()` → `kernel<<<g,b,0,stream>>>`；驱动保证同 stream 顺序、相对 host 异步。

**在链路哪一步**  
主链步骤 5（launch 入队）。

**依据**  
CUDA PG §2.5

---

**本层串讲**  
Dispatcher+键解决选核；图在需梯度时记录 DAG；Stream 在 launch 环节异步有序交 GPU。三者分别在选路、训练、入队环节不可或缺。

---

## L2 · 分派器与分派键的关系（单步骤示例）

### 步骤 2 · Dispatcher 与分派键的关系

**为什么需要**  
键是条件，Dispatcher 是用条件查表的机制，不可混为一物。

**是什么**  
Dispatcher = 单例 + 每算子一张表；键合并结果决定跳哪一行函数指针。

**如何做到**  
`m.impl("add", add_cuda)` 绑定 `CUDA` 行；`op.call` → `lookup(keySet)` → 间接调用；Autograd 行内部再 `redispatch` exclude Autograd。

**例子**  
表意：`dispatch["aten::add"][CUDA]=native::add_cuda`；`[AutogradCUDA]=VariableType::add`。

**依据**  
TORCH_LIBRARY_IMPL 示例

---

## 5｜贯穿推演（节选）

`z=torch.add(x,y)`，4096²，cuda，x 需梯度：

1. Python → `aten::add`  
2. 键合并 → `AutogradCUDA` → `VariableType::add`  
3. 建 `AddBackward0`；TLS exclude Autograd  
4. 二次 `at::add` → `CUDA` → `native::add_cuda`  
5. Iterator + `gpu_kernel` + current stream launch  
6. CPU 返回；GPU 流内执行；SM/Warp 写显存  

**5.2** §0 清单 12 条均在 L1/L2/5.1 标 ✅。
