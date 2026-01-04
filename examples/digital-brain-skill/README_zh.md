# 数字大脑 (Digital Brain)

> 一个为创始人、创作者和建设者打造的个人操作系统。属于 [上下文工程智能体技能集合](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) 的一部分。

## 概览

“数字大脑”是一个专为 AI 辅助个人生产力设计的结构化知识管理 system。它提供了一个完整的基于文件夹的架构，用于管理：

- **个人品牌** - 语气、定位、价值观
- **内容创作** - 灵感、草稿、发布流水线
- **知识库** - 书签、研究、学习
- **人脉网络** - 联系人、关系维护、引荐
- **运营系统** - 目标、任务、会议、指标

该系统遵循上下文工程原则：渐进式披露、仅追加式数据（append-only）以及模块分离，以优化 AI 智能体的交互体验。

## 架构

```
digital-brain/
├── SKILL.md                 # 主技能定义 (兼容 Claude Code)
├── SKILLS-MAPPING.md        # 上下文工程技能的应用方式
│
├── identity/                # 个人品牌与语气
│   ├── IDENTITY.md          # 模块指令
│   ├── voice.md             # 语气、风格、模式
│   ├── brand.md             # 定位、受众
│   ├── values.yaml          # 核心原则
│   ├── bio-variants.md      # 各平台个人简介
│   └── prompts/             # 生成模板
│
├── content/                 # 内容创作中心
│   ├── CONTENT.md           # 模块指令
│   ├── ideas.jsonl          # 内容灵感 (仅追加)
│   ├── posts.jsonl          # 已发布内容日志
│   ├── calendar.md          # 内容排期
│   ├── engagement.jsonl     # 收藏的灵感
│   ├── drafts/              # 进行中的草稿
│   └── templates/           # 帖子、通讯、推文模板
│
├── knowledge/               # 个人知识库
│   ├── KNOWLEDGE.md         # 模块指令
│   ├── bookmarks.jsonl      # 保存的资源
│   ├── learning.yaml        # 技能与目标
│   ├── competitors.md       # 市场版图
│   ├── research/            # 深度研究笔记
│   └── notes/               # 快速捕获
│
├── network/                 # 关系管理
│   ├── NETWORK.md           # 模块指令
│   ├── contacts.jsonl       # 联系人数据库
│   ├── interactions.jsonl   # 互动日志
│   ├── circles.yaml         # 关系圈分层
│   └── intros.md            # 引荐跟踪
│
├── operations/              # 生产力系统
│   ├── OPERATIONS.md        # 模块指令
│   ├── todos.md             # 任务列表 (P0-P3)
│   ├── goals.yaml           # OKR 目标
│   ├── meetings.jsonl       # 会议笔记
│   ├── metrics.jsonl        # 关键指标
│   └── reviews/             # 周回顾
│
├── agents/                  # 自动化
│   ├── AGENTS.md            # 脚本文档
│   └── scripts/
│       ├── weekly_review.py
│       ├── content_ideas.py
│       ├── stale_contacts.py
│       └── idea_to_draft.py
│
├── references/              # 详细文档
│   └── file-formats.md
│
└── examples/                # 使用工作流示例
    ├── content-workflow.md
    └── meeting-prep.md
```

## 技能集成

该示例展示了以下上下文工程技能的应用：

| 技能 | 应用场景 |
|-------|-------------|
| `context-fundamentals` | 渐进式披露、注意力预算 |
| `memory-systems` | JSONL 仅追加日志、结构化召回 |
| `tool-design` | 自包含的自动化脚本 |
| `context-optimization` | 模块分离、即时加载 |

查看 [SKILLS-MAPPING.md](./SKILLS-MAPPING.md) 了解每个技能如何指导系统设计的详细映射。

## 安装

### 作为 Claude Code 技能

```bash
# 全局安装
git clone https://github.com/muratcankoylan/digital-brain-skill.git \
  ~/.claude/skills/digital-brain

# 或针对特定项目安装
git clone https://github.com/muratcankoylan/digital-brain-skill.git \
  .claude/skills/digital-brain
```

### 作为独立模板

```bash
git clone https://github.com/muratcankoylan/digital-brain-skill.git ~/digital-brain
cd ~/digital-brain
```

## 快速上手

1. **定义你的语气** - 在 `identity/voice.md` 中填写你的语气和风格
2. **设定定位** - 在 `identity/brand.md` 中完善你的受众和支柱内容
3. **添加联系人** - 在 `network/contacts.jsonl` 中填入核心关系
4. **设定目标** - 在 `operations/goals.yaml` 中定义 OKR
5. **开始创作** - 让 AI “写一篇帖子”，观察它如何运用你的语气

## 文件格式约定

| 格式 | 使用场景 | 优势 |
|--------|----------|-----|
| `.jsonl` | 仅追加日志 | 对智能体友好，保留历史记录 |
| `.yaml` | 结构化配置 | 人类可读的分层结构 |
| `.md` | 叙述性内容 | 可编辑、丰富的格式 |
| `.xml` | 复杂提示词 | 为智能体提供清晰的结构 |

## 使用示例

### 内容创作
```
用户: "帮我写一篇关于 AI 智能体的 X (推文) 连载"

智能体处理过程:
1. 读取 identity/voice.md 获取语气模式
2. 检查 identity/brand.md - 确认 "ai_agents" 是核心支柱之一
3. 参考 content/posts.jsonl 获取成功的格式
4. 起草符合语气特性的连载推文
```

### 会议准备
```
用户: "帮我准备一下与 Sarah 的通话"

智能体处理过程:
1. 在 network/contacts.jsonl 中搜索 Sarah
2. 从 network/interactions.jsonl 中获取互动历史
3. 检查 operations/todos.md 中的待办项
4. 生成会前简报
```

### 周回顾
```
用户: "运行我的周回顾"

智能体处理过程:
1. 执行 agents/scripts/weekly_review.py
2. 从 operations/metrics.jsonl 汇总指标
3. 运行 agents/scripts/stale_contacts.py
4. 提交包含行动项的总结
```

## 自动化脚本

| 脚本 | 用途 | 运行频率 |
|--------|---------|---------------|
| `weekly_review.py` | 从数据生成周回顾 | 每周 |
| `content_ideas.py` | 基于知识库建议内容灵感 | 按需 |
| `stale_contacts.py` | 找出被冷落的人脉关系 | 每周 |
| `idea_to_draft.py` | 将灵感扩展为草稿框架 | 按需 |

```bash
# 直接运行
python agents/scripts/weekly_review.py

# 或带参数运行
python agents/scripts/content_ideas.py --pillar ai_agents --count 5
```

## 设计原则

1. **渐进式披露 (Progressive Disclosure)** - 仅加载当前任务所需的内容
2. **仅追加式数据 (Append-Only Data)** - 永不删除，保留历史数据用于模式分析
3. **模块分离 (Module Separation)** - 每个领域相互独立，无交叉污染
4. **语气优先 (Voice First)** - 在生成任何内容前务必先阅读 voice.md
5. **平台无关 (Platform Agnostic)** - 适用于 Claude Code、Cursor 及任何 AI 助手

---

**作者**: Muratcan Koylan
**版本**: 1.0.0
**上次更新**: 2025-12-29
