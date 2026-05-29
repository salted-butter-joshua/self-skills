---
name: pyramid-knowledge-map
description: >-
  Builds a verified top-down knowledge pyramid and graph from a topic or sentence:
  branches into sub-concepts from abstract to concrete until principle-level content,
  rates difficulty, links the chain with examples, and outputs Mermaid graph plus
  optional PNG. Use when the user asks for 金字塔知识图, 知识图谱, 知识体系,
  由抽象到具体, 知识链, pyramid knowledge map, knowledge graph, /pyramid-map,
  or wants a complete decomposed path (e.g. PyTorch operator integration down to
  CUDA SM architecture).
---

# 金字塔知识图谱

从一个知识点或一句话出发，**由抽象到具体**层层展开关联子概念，直到**原理层**为止；输出可核查的知识链 + 图谱 + 可选图片。

与 `onion-pyramid-learn` 的区别：本 skill **横向展开同类路径**（如多种集成方式），并强制**有据可查、术语消歧**；默认产出**知识图谱**而非仅文字分层。

## 输入

- **知识点**：术语、问题（如「PyTorch 如何集成算子」）
- **一句话**：先提炼为**顶层命题**（≤30 字，可检验对错）

输入含糊时：列出 2 个可能读法 + 选定读法及理由，再继续；**不把猜测当事实写进图谱**。

## 工作流

```
- [ ] 1. 定锚：顶层命题、边界、停止条件（原理层定义）
- [ ] 2. 检索：官方文档 / 规范 / 白皮书（见 accuracy 协议）
- [ ] 3. 规划金字塔：层数 + 每层分支（breadth-first 展开）
- [ ] 4. 填节点：定义、难度、依据、示例、父链
- [ ] 5. 串链：主路径 + 备选路径
- [ ] 6. 输出图谱（Mermaid）+ 节点表
- [ ] 7. 生成图片（HTML 金字塔 或 ljg-card）
- [ ] 8. 自检（accuracy 清单）
```

## 金字塔分层（抽象 → 具体 → 原理）

| 层 | 代号 | 抽象度 | 回答什么 | 停止信号 |
|----|------|--------|----------|----------|
| P0 | 顶 | 最高 | 用户问的整体问题 | — |
| P1 | 路径 | 高 | 有哪些**正交路径/范式**（并列方法，须穷尽已知主类） | 再拆只是重复 |
| P2 | 模块 | 中高 | 每条路径上的**关键模块、接口、产物** | 到可命名组件 |
| P3 | 机制 | 中低 | **如何运行**：调度、数据流、执行模型 | 到可描述机制 |
| P4 | 原理 | 最低 | **硬件/数学/体系结构公理**；不可再拆而不丢学科意义 | 触达公理/架构规格 |

**层数**：通常 4–6 层（含 P0）。复杂主题（如 CUDA 栈）可到 P5，但 P4 必须出现原理层。

**展开规则**

1. **每层先 breadth 后 depth**：P1 必须列出该主题下**所有主要路径**（如 PyTorch 集成算子：Python/custom autograd、C++ Extension、torch.library、CUDA/CPU kernel 等），再择一条或用户指定路径向下。
2. **一条主链 + 标注支链**：全文给一条**推荐主路径**串到底；其余路径在 P1/P2 节点保留，用虚线或「支链」表标注。
3. **向下条件**：当前节点还能回答「它内部靠什么实现？」→ 继续；只能回答「为什么物理/数学上必然如此？」→ 进 P4 并停止。

## 节点规范（零歧义）

每个节点 **N** 必须包含：

```yaml
id: N-unique-slug
label: 中文名（英文缩写仅首次括号）
layer: P0|P1|P2|P3|P4
parent: 父节点 id（P0 无）
paths: [main] | [branch: 路径名]
difficulty: 1-5          # 1入门 3中级 5专家
definition: ≤2 句，边界清晰
disambiguation: 易混概念区分（无则写「无」）
depends_on: [前置节点 id]
leads_to: [子节点 id]
example: 可验证的具体例子（版本/型号/命令/场景）
evidence: 依据类型 + 出处（见下）
status: verified | needs_review  # 未核实不得标 verified
```

**术语铁律**

- 禁止模棱两可：如「算子」须标明 **ATen op / custom op / CUDA kernel** 之一或关系。
- 同名异义必须拆节点（如 **Shared Memory** 在 CUDA C++ vs GPU 硬件 SRAM）。
- 数字、架构名、算力公式须带**适用对象**（如 A100 SM 数 108、FP16 Tensor Core peak 需注明稀疏/稠密与 GPU Boost 条件）。
- 推不出处 → `status: needs_review`，**不写入 Mermaid 实线主链**。

## 依据协议（必须检索）

**优先级**

1. 官方文档（PyTorch、NVIDIA CUDA Programming Guide、硬件 Whitepaper）
2. 标准 / 规范（IEEE、ISO、语言标准）
3. 厂商规格书（datasheet、架构白皮书）
4. 经典教材 / 原始论文（注明年份）

**工具**：`WebSearch` + `WebFetch` 核实关键节点；技术栈类主题**至少 2 个权威来源**交叉确认再标 `verified`。

**禁止**：凭训练记忆填具体数字；「一般来说」「可能」式表述；无出处的性能对比。

## 难度标定

| 等级 | 读者背景 | 节点特征 |
|------|----------|----------|
| 1 | 会用 API | 概念、调用方式 |
| 2 | 写过脚本 | 配置、常见模式 |
| 3 | 读源码 | 模块交互、扩展点 |
| 4 | 写 kernel / 扩展 | 内存、调度、ISA 级 |
| 5 | 架构 / 体系 | 微架构、峰值公式、演进史 |

每层节点标注难度；**主链难度应单调不降**（允许同级并列）。

## 知识链

输出 **主链叙事**（800 字内）：

```markdown
## 知识主链

P0 → P1a → P2 → … → P4

{用箭头串起节点 label，每步 1 句「为何往下走」}

**支链摘要**：P1b …（各 1 句，指向节点 id）
```

## 图谱输出

### 1. Mermaid（必出）

使用 `flowchart TB`，层级用 subgraph。模板见 [reference.md](reference.md#mermaid-模板)。

- 实线：已 verified 的主链/支链
- 虚线：`needs_review` 或可选扩展
- 节点文本：`label` + `难度★`

### 2. 节点总表（必出）

Markdown 表格：id | 层 | 名称 | 难度 | 一句话定义 | 依据 | 示例

### 3. 图片（默认出 HTML 金字塔；用户要 PNG 时铸图）

**默认**：生成交互/static HTML 金字塔（见 [reference.md](reference.md#html-金字塔)），保存到输出目录。

**PNG**：用户说「铸」「做成图」「PNG」→ 将 **节点总表 + Mermaid 结构** 交给 `ljg-card -w`（白板框图）或 `-i`（信息图）；或对本 skill 生成的 HTML 运行 `ljg-card` 的 capture 流程。

## 全文结构

```markdown
# 金字塔知识图谱 · {主题}

> 顶层命题：…  
> 主路径：…  
> 构建时间：…

## 学习地图（各层一览表）

## 知识主链

## 逐层节点（P0…P4，每节点按规范）

## 图谱（Mermaid）

## 节点总表

## 举例说明（2–3 个跨层案例，回扣节点 id）

## 自检清单
```

## 保存

- Markdown：`~/Documents/notes/{YYYYMMDD}--pyramid-{主题}.md`
- HTML 图：同目录 `{YYYYMMDD}--pyramid-{主题}.html`
- 告知用户路径

## 与相近 skill

| Skill | 何时用 |
|-------|--------|
| `onion-pyramid-learn` | 单概念由外向内剥、重直觉与案例 |
| **pyramid-knowledge-map** | 体系化展开、多路径、图谱、硬件/软件全栈、要依据 |
| `ljg-card` | 仅要视觉 PNG 时配合 |

## 附加资源

- 节点 schema、Mermaid/HTML 模板、PyTorch→CUDA 样例骨架：[reference.md](reference.md)
- 简短触发示例：[examples.md](examples.md)
