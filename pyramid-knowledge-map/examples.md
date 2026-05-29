# 触发示例

## 示例 1 · 一句话

**输入**：PyTorch 如何集成算子？算子如何实现？

**预期行为**

1. 定锚顶层命题：「在 PyTorch 中注册并执行自定义算子的完整路径」
2. P1 展开：torch.library、C++ Extension、CompositeImplicitAutograd、（可选）Triton
3. 主链走 C++ Extension → CUDA kernel → SIMT → SM → 峰值算力
4. WebSearch 核实 PyTorch 2.x 官方推荐路径
5. 输出 Mermaid + HTML 金字塔 + 节点总表
6. 用户说「铸成图」→ `ljg-card -w`

## 示例 2 · 知识点

**输入**：Warp Scheduler

**预期行为**

1. 消歧：SM 内 warp 调度器 vs CPU OS scheduler
2. P0：Warp Scheduler 在 GPU 执行模型中的角色
3. P1：与 Warp、Block、Pipeline 的关系
4. 向下至 P4：NVIDIA 白皮书中的 warp dispatch / issue 机制
5. 难度 ★5，依据 Whitepaper + Programming Guide

## 示例 3 · 歧义输入

**输入**：Shared Memory

**预期行为**

1. 列出读法 A：CUDA `__shared__`；读法 B：GPU SRAM 硬件
2. 询问或默认双路径各建子图，**不合并为一个模糊节点**
3. 标注两者容量、生命周期、可见范围差异

## 最小输出片段（格式参考）

```markdown
### 节点 N5 · P3 · Grid / Block / Thread / Warp

- **难度**：★★★★
- **定义**：CUDA 将 kernel 实例映射为 Grid of Blocks；每 Block 含 Thread；连续 32 Thread 构成一个 Warp，为 SM 调度最小单位。
- **消歧**：Warp 大小 32 为 NVIDIA GPU 现行规范；与 CPU thread 不同。
- **示例**：`<<<grid, block>>>`  launch `vecAdd<<<256, 256>>>` → 256 block × 256 thread/block。
- **依据**：CUDA C Programming Guide — SIMT Architecture（verified）
- **父链**：N0 → N1b → N2 → N4 → **N5**
```
