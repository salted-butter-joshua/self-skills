# self-skills

面向 [Cursor](https://cursor.com) 的 Agent Skills 集合，围绕**知识学习与检索**场景：从网页提取、概念深读、金字塔拆解到可核查的知识图谱与链路推演。

仓库地址：[github.com/salted-butter-joshua/self-skills](https://github.com/salted-butter-joshua/self-skills)

## 技能一览

| Skill | 一句话 | 典型输入 | 主要输出 |
|-------|--------|----------|----------|
| [web-page-knowledge](./web-page-knowledge/) | 从网页提取结构化知识与记忆卡片 | URL | Markdown 报告 + 是什么/为什么/怎么做卡片 |
| [onion-pyramid-learn](./onion-pyramid-learn/) | 单概念由外向内「剥洋葱」深读 | 概念或段落 | 分层知识点 + 真实案例 |
| [pyramid-knowledge-map](./pyramid-knowledge-map/) | 多路径金字塔 + 可核查知识图谱 | 知识点或问题 | Mermaid 图 + 节点表 + 可选 HTML/PNG |
| [pyramid-knowledge-retrieval](./pyramid-knowledge-retrieval/) | 疑问驱动的有序链路，讲到为什么/如何实现 | 知识点或一句话 | 术语地基 + 逐步推演 + 疑问闭合 |

### 怎么选？

```
想读一篇网页并做笔记卡片     → web-page-knowledge
想弄懂一个概念、要案例落地     → onion-pyramid-learn
想看清体系结构、要图谱可视化   → pyramid-knowledge-map
想跟完整条技术链路、消灭疑问   → pyramid-knowledge-retrieval
```

四个 skill 都涉及「分层」，但侧重点不同：

- **onion-pyramid-learn**：纵向剥概念，重直觉与案例，适合人文/社科/通识概念。
- **pyramid-knowledge-map**：横向展开多路径，重图谱与依据，适合软件/硬件全栈拆解。
- **pyramid-knowledge-retrieval**：按调用顺序讲清为什么/如何实现，适合「这个过程到底怎么串起来」。
- **web-page-knowledge**：唯一面向**网页 URL**，以页面原文为唯一事实来源。

---

## 安装

Cursor Agent Skills 的标准目录结构为「每个 skill 一个文件夹，内含 `SKILL.md`」。本仓库每个顶层目录即一个 skill。

### 方式一：复制到个人 skills 目录（推荐）

安装后**所有项目**均可使用。

**Windows（PowerShell）**

```powershell
git clone https://github.com/salted-butter-joshua/self-skills.git
$skillsDir = "$env:USERPROFILE\.cursor\skills"
New-Item -ItemType Directory -Force -Path $skillsDir | Out-Null
Copy-Item -Recurse -Force ".\self-skills\web-page-knowledge" $skillsDir
Copy-Item -Recurse -Force ".\self-skills\onion-pyramid-learn" $skillsDir
Copy-Item -Recurse -Force ".\self-skills\pyramid-knowledge-map" $skillsDir
Copy-Item -Recurse -Force ".\self-skills\pyramid-knowledge-retrieval" $skillsDir
```

**macOS / Linux**

```bash
git clone https://github.com/salted-butter-joshua/self-skills.git
mkdir -p ~/.cursor/skills
cp -r self-skills/web-page-knowledge ~/.cursor/skills/
cp -r self-skills/onion-pyramid-learn ~/.cursor/skills/
cp -r self-skills/pyramid-knowledge-map ~/.cursor/skills/
cp -r self-skills/pyramid-knowledge-retrieval ~/.cursor/skills/
```

### 方式二：复制到项目 skills 目录

仅当前仓库/项目可用。将对应文件夹复制到项目根目录：

```
your-project/.cursor/skills/<skill-name>/
```

### 方式三：符号链接（便于开发本仓库）

```bash
# macOS / Linux 示例
ln -s /path/to/self-skills/web-page-knowledge ~/.cursor/skills/web-page-knowledge
```

### 验证安装

1. 重启 Cursor 或新开 Agent 对话。
2. 在对话中用 `@` 引用 skill 名称，或直接使用下方「触发方式」中的关键词。
3. 若 Agent 能按 skill 规定的输出结构回复，即安装成功。

> **注意**：请勿将 skill 放入 `~/.cursor/skills-cursor/`，该目录为 Cursor 内置 skill 保留路径。

---

## 使用方法（通用）

1. **自动触发**：在对话中描述任务，Agent 会根据各 skill `SKILL.md`  frontmatter 中的 `description` 判断是否加载。
2. **手动附加**：在 Cursor 聊天里用 `@` 选择对应 skill（如 `@pyramid-knowledge-retrieval`）。
3. **关键词触发**：各 skill 支持的简称见下文「触发方式」一节。

安装后无需额外配置；`web-page-knowledge` 与 `pyramid-knowledge-map` 在联网环境下效果更好（需 Agent 使用 WebFetch / 搜索核实依据）。

---

## 各 Skill 说明

### web-page-knowledge

**技能点**

- 抓取 HTTP(S) 网页并清洗正文（去导航、页脚、重复块）
- 按标题层级（h1–h6）构建大纲与分块计划
- 分块精读总结，全程以**页面原文为唯一事实来源**，禁止臆造
- 生成「是什么 / 为什么 / 怎么做」记忆卡片
- 可选与 `ljg-card` 协作输出视觉卡片

**触发方式**

- 分享网页链接，或说：爬取、总结网页、提取知识、记忆卡片、知识卡片
- 英文：`summarize page`、`extract knowledge from URL`

**示例提示词**

```
请用 web-page-knowledge 处理这个链接：https://docs.pytorch.org/docs/stable/notes/cuda.html
```

**输出与保存**

- 默认保存：`~/Documents/notes/web-knowledge/{YYYYMMDD}--web-knowledge-{标题}.md`
- 附加文档：[reference.md](./web-page-knowledge/reference.md)、[examples.md](./web-page-knowledge/examples.md)

---

### onion-pyramid-learn

**技能点**

- 对**单个概念或一段文字**做洋葱式分层（表象 → 机制 → 原理 → 内核）
- 每层输出可独立记忆的知识点 + 连贯「内容介绍」
- 层间用「进入下一层的裂缝」制造递进感
- 附 2–3 个**可核查的真实案例**（非小明小红虚构故事）
- ASCII 层间关系图 +「若只记三件事」收束

**触发方式**

- 洋葱学习、金字塔学习、层层深入、剥洋葱、知识洋葱
- `/onion-learn`、`/pyramid-learn`
- 英文：`onion learning`、`pyramid knowledge`

**示例提示词**

```
用洋葱学习帮我弄懂「边际效用」
```

```
剥洋葱：这段话在讲什么？[粘贴一段文字]
```

**输出与保存**

- 默认输出到对话；保存路径：`~/Documents/notes/{YYYYMMDD}--onion-{主题}.md`
- 样例：[examples.md](./onion-pyramid-learn/examples.md)

---

### pyramid-knowledge-map

**技能点**

- 从知识点或一句话构建**由抽象到具体**的金字塔（P0–P4）
- **先广后深**：每层穷尽主要正交路径（如多种算子集成方式）
- 每个节点强制：定义、难度、依据、示例、消歧；未核实标 `needs_review`
- 输出 **Mermaid 流程图** + 节点总表 + 知识主链叙事
- 可生成交互 HTML 金字塔；用户要求时可配合 `ljg-card` 出 PNG
- 技术类主题须 WebSearch/WebFetch 交叉核实

**触发方式**

- 金字塔知识图、知识图谱、知识体系、由抽象到具体、知识链
- `/pyramid-map`
- 英文：`pyramid knowledge map`、`knowledge graph`

**示例提示词**

```
用 pyramid-knowledge-map 把「PyTorch 算子如何集成到 CUDA」拆到 SM 原理层，要有依据和 Mermaid 图
```

**输出与保存**

- Markdown：`~/Documents/notes/{YYYYMMDD}--pyramid-{主题}.md`
- HTML 图：同目录 `{YYYYMMDD}--pyramid-{主题}.html`
- 参考：[reference.md](./pyramid-knowledge-map/reference.md)、[examples.md](./pyramid-knowledge-map/examples.md)

---

### pyramid-knowledge-retrieval

**技能点**

- **疑问驱动**：事先列出 10–20 个读者会问的问题，逐层闭合
- 固定六段输出：概览与完整链路 → 术语地基 → 路径对比 → 逐层深入 → 支链 → 贯穿推演与疑问闭合
- 每个环节写清：**为什么需要 / 是什么 / 如何做到 / 与上下游如何衔接**
- 术语六要素：是什么、不是什么、为什么需要、链路哪一步、如何实现、依据
- 多实现路径对比表（如 ATen 自写核 vs cuBLAS vs cuDNN）
- 300–500 字「走调试器」式完整推演 + 疑问闭合表

**触发方式**

- 金字塔知识检索、知识体系检索、层层深入、知识链、由表及里、为什么需要
- `/pyramid-retrieval`
- 英文：`pyramid retrieval`、`knowledge decomposition`

**示例提示词**

```
按 pyramid-knowledge-retrieval 拆解：PyTorch 算子在 NVIDIA GPU 上的调度过程，要有依据和例子
```

**输出与保存**

- 默认输出到对话；保存路径：`~/Documents/notes/{YYYYMMDD}--pyramid-retrieval-{主题}.md`
- 样例：[examples.md](./pyramid-knowledge-retrieval/examples.md)

---

## 目录结构

```
self-skills/
├── README.md
├── web-page-knowledge/
│   ├── SKILL.md          # 必需：skill 元数据与主指令
│   ├── reference.md      # 分块阈值、卡片模板
│   └── examples.md
├── onion-pyramid-learn/
│   ├── SKILL.md
│   └── examples.md
├── pyramid-knowledge-map/
│   ├── SKILL.md
│   ├── reference.md      # Mermaid/HTML 模板、节点 schema
│   └── examples.md
└── pyramid-knowledge-retrieval/
    ├── SKILL.md
    └── examples.md
```

每个 skill 目录至少包含 `SKILL.md`。文件顶部 YAML frontmatter 示例：

```yaml
---
name: skill-folder-name
description: 第三人称描述 skill 做什么、何时使用（供 Agent 发现）
---
```

---

## 依赖与协作

| 依赖 | 说明 |
|------|------|
| Cursor IDE | 支持 Agent Skills 的 Cursor 版本 |
| 网络 | `web-page-knowledge`、`pyramid-knowledge-map`、`pyramid-knowledge-retrieval` 核实技术内容时建议联网 |
| `ljg-card`（可选） | 非本仓库 skill；`web-page-knowledge` / `pyramid-knowledge-map` 在用户要 PNG 视觉卡片时可配合使用 |

---

## 更新 skill

```bash
cd self-skills
git pull
# 重新复制或同步符号链接到 ~/.cursor/skills/
```

---

## 贡献

欢迎通过 Issue 或 Pull Request 改进 skill 指令、样例与文档。修改 `SKILL.md` 时请保持：

- `name` 与文件夹名一致（小写、连字符）
- `description` 用第三人称，包含 **做什么** 与 **何时触发**
- 主 `SKILL.md` 建议控制在 500 行以内，详细模板放入 `reference.md` / `examples.md`

---

## License

尚未指定开源协议。推送公开仓库前请自行添加 `LICENSE` 文件。
