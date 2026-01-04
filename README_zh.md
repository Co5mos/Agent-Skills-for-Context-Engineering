[English](README.md) | 中文版
# Agent Skills for Context Engineering

一个全方位的、开放的 Agent Skills（助手技能）集合，专注于构建生产级 AI 助手系统的上下文工程原则。这些技能教授策划上下文的艺术与科学，以在任何助手平台上最大化助手的效能。

## 什么是上下文工程？

上下文工程（Context Engineering）是管理语言模型上下文窗口的学科。与专注于编写有效指令的提示词工程（Prompt Engineering）不同，上下文工程处理的是进入模型有限注意力预算的所有信息的整体策划：系统提示词、工具定义、检索到的文档、消息历史和工具输出。

根本挑战在于，上下文窗口受到的限制并非原始的 Token 容量，而是注意力机制。随着上下文长度的增加，模型会表现出可预测的退化模式：“迷失在中间”（lost-in-the-middle）现象、U 型注意力曲线和注意力稀缺。有效的上下文工程意味着寻找尽可能小的高信号 Token 集合，以最大化实现预期结果的可能性。

## 技能概览

### 基础技能

这些技能建立了所有后续上下文工程工作所需的基础理解。

| 技能 | 描述 |
|-------|-------------|
| [context-fundamentals](skills/context-fundamentals/) | 了解什么是上下文，为什么它很重要，以及助手系统中上下文的剖析 |
| [context-degradation](skills/context-degradation/) | 识别上下文失败的模式：迷失在中间、毒化、干扰和冲突 |
| [context-compression](skills/context-compression/) | 为长时间运行的会话设计和评估压缩策略 |

### 架构技能

这些技能涵盖了构建有效助手系统的模式和结构。

| 技能 | 描述 |
|-------|-------------|
| [multi-agent-patterns](skills/multi-agent-patterns/) | 掌握编排者（Orchestrator）、点对点（Peer-to-Peer）和分层（Hierarchical）多代理架构 |
| [memory-systems](skills/memory-systems/) | 设计短期、长期和基于图的记忆架构 |
| [tool-design](skills/tool-design/) | 构建助手可以有效使用的工具 |

### 运营技能

这些技能解决了助手系统的持续运行和优化。

| 技能 | 描述 |
|-------|-------------|
| [context-optimization](skills/context-optimization/) | 应用压缩、屏蔽和缓存策略 |
| [evaluation](skills/evaluation/) | 为助手系统构建评估框架 |
| [advanced-evaluation](skills/advanced-evaluation/) | 掌握 LLM-as-a-Judge 技术：直接评分、成对比较、评分标准生成和偏差缓解 |

### 开发方法论

这些技能涵盖了构建 LLM 驱动项目的元级实践。

| 技能 | 描述 |
|-------|-------------|
| [project-development](skills/project-development/) | **新增** 设计和构建从构思到部署的 LLM 项目，包括任务-模型适配分析、流水线架构和结构化输出设计 |

## 设计理念

### 渐进式披露

每个技能都为高效的上下文使用而结构化。在启动时，助手仅加载技能名称和描述。只有当技能为相关任务而激活时，才会加载全文内容。

### 平台无关性

这些技能专注于可迁移的原则，而非特定供应商的实现。这些模式适用于 Claude Code、Cursor 以及任何支持技能或允许自定义指令的助手平台。

### 理论基础与实践案例

脚本和示例使用 Python 伪代码演示概念，这些代码可以在各种环境中运行，无需安装特定的依赖项。

## 使用方法

### 在 Claude Code 中使用

本仓库是一个 **Claude Code 插件市场**，包含 Claude 根据您的任务上下文自动发现并激活的上下文工程技能。

### 安装

**第一步：添加市场**

在 Claude Code 中运行此命令以将此仓库注册为插件源：

```
/plugin marketplace add muratcankoylan/Agent-Skills-for-Context-Engineering
```

**第二步：浏览并安装**

选项 A - 浏览可用插件：
1. 选择 `Browse and install plugins`
2. 选择 `context-engineering-marketplace`
3. 选择一个插件（例如 `context-engineering-fundamentals`, `agent-architecture`）
4. 选择 `Install now`

选项 B - 通过命令直接安装：

```
/plugin install context-engineering-fundamentals@context-engineering-marketplace
/plugin install agent-architecture@context-engineering-marketplace
/plugin install agent-evaluation@context-engineering-marketplace
/plugin install agent-development@context-engineering-marketplace
```

### 可用插件

| 插件 | 包含的技能 |
|--------|-----------------|
| `context-engineering-fundamentals` | context-fundamentals, context-degradation, context-compression, context-optimization |
| `agent-architecture` | multi-agent-patterns, memory-systems, tool-design |
| `agent-evaluation` | evaluation, advanced-evaluation |
| `agent-development` | project-development |

### 技能触发器

| 技能 | 触发词 |
|-------|-------------|
| `context-fundamentals` | "understand context", "explain context windows", "design agent architecture" |
| `context-degradation` | "diagnose context problems", "fix lost-in-middle", "debug agent failures" |
| `context-compression` | "compress context", "summarize conversation", "reduce token usage" |
| `context-optimization` | "optimize context", "reduce token costs", "implement KV-cache" |
| `multi-agent-patterns` | "design multi-agent system", "implement supervisor pattern" |
| `memory-systems` | "implement agent memory", "build knowledge graph", "track entities" |
| `tool-design` | "design agent tools", "reduce tool complexity", "implement MCP tools" |
| `evaluation` | "evaluate agent performance", "build test framework", "measure quality" |
| `advanced-evaluation` | "implement LLM-as-judge", "compare model outputs", "mitigate bias" |
| `project-development` | "start LLM project", "design batch pipeline", "evaluate task-model fit" |

<img width="1014" height="894" alt="Screenshot 2025-12-26 at 12 34 47 PM" src="https://github.com/user-attachments/assets/f79aaf03-fd2d-4c71-a630-7027adeb9bfe" />

### 适用于 Cursor & Codex & IDE

将技能内容复制到 `.rules` 或创建项目特定的技能文件夹。这些技能提供了助手进行有效上下文工程和助手设计所需的背景和指南。

### 适用于自定义实现

从任何技能中提取原则和模式，并在您的助手框架中实现。这些技能特意设计为平台无关的。

## 示例

[examples](examples/) 文件夹包含完整的系统设计，演示了多种技能如何在实践中协同工作。

| 示例 | 描述 | 应用的技能 |
|---------|-------------|----------------|
| [digital-brain-skill](examples/digital-brain-skill/) | **新增** 适用于创业者和创作者的个人操作系统。包含 6 个模块、4 个自动化脚本的完整 Claude Code 技能 | context-fundamentals, context-optimization, memory-systems, tool-design, multi-agent-patterns, evaluation, project-development |
| [x-to-book-system](examples/x-to-book-system/) | 监控 X 账号并生成每日综合书籍的多代理系统 | multi-agent-patterns, memory-systems, context-optimization, tool-design, evaluation |
| [llm-as-judge-skills](examples/llm-as-judge-skills/) | 具备 TypeScript 实现、19 个测试通过的生产级 LLM 评估工具 | advanced-evaluation, tool-design, context-fundamentals, evaluation |
| [book-sft-pipeline](examples/book-sft-pipeline/) | 训练模型以任何作者的风格写作。包含 Gertrude Stein 案例研究，在 Pangram 上人工评分达 70%，总成本 2 美元 | project-development, context-compression, multi-agent-patterns, evaluation |

每个示例包括：
- 包含架构决策的完整 PRD
- 显示哪些概念指导了每项决策的技能映射
- 实现指南

### 个人大脑技能示例

[digital-brain-skill](examples/digital-brain-skill/) 示例是一个完整的个人操作系统，演示了技能的综合应用：

- **渐进式披露**：3 级加载（SKILL.md → MODULE.md → 数据文件）
- **模块隔离**：6 个独立模块（身份、内容、知识、人脉、运营、助手）
- **仅追加记忆**：采用 Schema 优先行的 JSONL 文件，便于助手解析
- **自动化脚本**：4 个整合工具（周复盘、内容灵感、陈旧联系人、想法转草案）

在 [HOW-SKILLS-BUILT-THIS.md](examples/digital-brain-skill/HOW-SKILLS-BUILT-THIS.md) 中包含详细的可追溯性，将每个架构决策映射到特定的技能原则。

### LLM-as-Judge 技能示例

[llm-as-judge-skills](examples/llm-as-judge-skills/) 示例是一个完整的 TypeScript 实现，演示了：

- **直接评分**：根据带权重的标准在评分标准支持下评估响应
- **成对比较**：在缓解位置偏差的情况下比较响应
- **评分标准生成**：创建特定领域的评估标准
- **EvaluatorAgent**：结合了所有评估能力的高级助手

### 书籍 SFT 流水线示例

[book-sft-pipeline](examples/book-sft-pipeline/) 示例演示了如何训练小型模型（8B）以任何作者的风格写作：

- **智能分段**：带有重叠的两层分段，以获得最大的训练样本
- **提示词多样性**：15 个以上的模板，防止记忆并强制学习风格
- **Tinker 集成**：完整的 LoRA 训练工作流，总成本 2 美元
- **验证方法论**：现代场景测试证明了风格迁移而非内容记忆

与上下文工程技能集成：project-development, context-compression, multi-agent-patterns, evaluation。

## Star 历史
<img width="1832" height="1324" alt="star-history-20251229" src="https://github.com/user-attachments/assets/153f691a-12db-4fb9-8960-53bc431b202a" />

## 结构

每个技能都遵循 Agent Skills 规范：

```
skill-name/
├── SKILL.md              # 必需：指令 + 元数据
├── scripts/              # 可选：演示概念的可执行代码
└── references/           # 可选：额外的文档和资源
```

有关规范的技能结构，请参阅 [template](template/) 文件夹。

## 贡献

本仓库遵循 Agent Skills 开放开发模式。欢迎来自更广泛生态系统的贡献。贡献时：

1. 遵循技能模板结构
2. 提供清晰、可操作的指令
3. 在适当的情况下包含工作示例
4. 记录权衡和潜在问题
5. 保持 SKILL.md 在 500 行以内以获得最佳性能

如有协作机会或任何垂询，请随时联系 [Muratcan Koylan](https://x.com/koylanai)。

[![通过 GitHub 赞助](https://img.shields.io/badge/Sponsor-muratcankoylan-pink?logo=github-sponsors&style=for-the-badge)](https://github.com/sponsors/muratcankoylan)

## 许可证

MIT 许可证 - 详情请参阅 LICENSE 文件。

## 参考资料

这些技能中的原则源自领先 AI 实验室和框架开发者的研究和生产经验。每个技能都包含对其建议所依据的基础研究和案例研究的引用。
