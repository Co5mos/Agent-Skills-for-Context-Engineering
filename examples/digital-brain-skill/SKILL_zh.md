---
name: digital-brain
description: 当用户要求“写一篇帖子”、“检查我的语气”、“查找联系人”、“准备会议”、“周复盘”、“跟踪目标”，或提到个人品牌、内容创作、网络管理或语气一致性时，应使用此技能。
version: 1.0.0
---

# 数字大脑 (Digital Brain)

一个结构化的个人操作系统，旨在 AI 的协助下管理数字存在、知识、关系和目标。专为在公开场合进行建设的创始人、希望扩大受众的内容创作者，以及寻求 AI 辅助个人管理的资深专业人士设计。

**重要提示**：此技能使用渐进式披露。模块特定的指令位于每个子目录的 `.md` 文件中。请仅加载当前任务所需的内容。

## 何时激活

当用户进行以下操作时激活此技能：

- 要求创作内容（帖子、线程、时事通讯）——**请先加载 `identity/voice.md`**
- 寻求有关个人品牌或定位的帮助
- 需要查找或管理联系人/关系
- 需要捕获或开发内容感
- 要求进行会议准备或后续跟进
- 要求进行周复盘或目标跟踪
- 需要保存或检索书签资源
- 想要组织研究或学习材料

**触发词**：“写一篇短文”、“我的声音”、“内容感”、“[姓名]是谁”、“准备会议”、“周复盘”、“保存这个”、“我的目标”

## 核心概念

### 渐进式披露架构 (Progressive Disclosure)

数字大脑遵循三层加载模式：

| 级别 | 何时加载 | 内容 |
|-------|-------------|---------|
| **L1: 元数据** | 始终加载 | 本 `SKILL.md` 概览 |
| **L2: 模块指令** | 按需加载 | `[module]/[MODULE].md` 文件 |
| **L3: 数据文件** | 必要时加载 | `.jsonl`, `.yaml`, `.md` 数据 |

### 文件格式策略

选择了最适合智能体解析的格式：

- **JSONL** (`.jsonl`)：仅追加日志——灵感、帖子、联系人、互动
- **YAML** (`.yaml`)：结构化配置——目标、价值观、圈子
- **Markdown** (`.md`)：叙述性内容——语气、品牌、日历、待办事项
- **XML** (`.xml`)：复杂提示词——内容生成模板

### 仅追加的数据完整性

JSONL 文件是**仅追加**的。切勿删除条目：
- 使用 `"status": "archived"` 标记而非删除
- 为模式分析保留完整历史记录
- 支持对“哪些行之有效”进行回顾性分析

## 详细主题

### 模块概览

```
digital-brain/
├── identity/     → 语气、品牌、价值观 (创作内容前必读)
├── content/      → 灵感、草稿、帖子、日历
├── knowledge/    → 书签、研究、学习
├── network/      → 联系人、互动、引荐
├── operations/   → 待办事宜、目标、会议、指标
└── agents/       → 自动化脚本
```

### 身份模块 (Identity Module - 对创作至关重要)

**在生成任何内容之前，请务必阅读 `identity/voice.md`。**

该模块包含：
- `voice.md` - 语气、风格、词汇、模式
- `brand.md` - 定位、受众、内容支柱
- `values.yaml` - 核心信念和原则
- `bio-variants.md` - 特定平台的个人介绍
- `prompts/` - 可重用的生成模板

### 内容模块 (Content Module)

流程：`ideas.jsonl` → `drafts/` → `posts.jsonl`

- 立即将想法捕获到 `ideas.jsonl`
- 使用 `templates/` 在 `drafts/` 中进行开发
- 将已发布的内容及其指标记录到 `posts.jsonl`
- 在 `calendar.md` 中进行规划

### 人脉模块 (Network Module)

具有关系分层的个人 CRM：
- `inner` - 每周触达
- `active` - 每两周触达
- `network` - 每月触达
- `dormant` - 每季度重新激活检查

### 运营模块 (Operations Module)

具有优先级划分的生产力系统：
- P0: 今天完成，阻塞项
- P1: 本周完成，重要项
- P2: 本月完成，有价值项
- P3: 暂存，锦上添花项

## 实践指南

### 内容创作工作流

```
1. 阅读 identity/voice.md (必需)
2. 检查 identity/brand.md 以确保主题一致性
3. 参考 content/posts.jsonl 了解成功的模式
4. 使用 content/templates/ 作为起始结构
5. 起草符合声音特质的内容
6. 发布后记录到 posts.jsonl
```

### 会前准备

```
1. 查找联系人：network/contacts.jsonl
2. 获取历史记录：network/interactions.jsonl
3. 检查待办项：operations/todos.md
4. 生成带有背景信息的简报
```

### 周复盘流程

```
1. 运行：python agents/scripts/weekly_review.py
2. 在 operations/metrics.jsonl 中查看指标
3. 检查陈旧联系人：agents/scripts/stale_contacts.py
4. 在 operations/goals.yaml 中更新目标进度
5. 在 content/calendar.md 中规划下一周
```

## 示例

### 示例：撰写 X (Twitter) 帖子

**输入**：“帮我写一篇关于 AI 智能体的帖子”

**过程**：
1. 阅读 `identity/voice.md` → 提取声音特质
2. 检查 `identity/brand.md` → 确认 “ai_agents” 是内容支柱
3. 参考 `content/posts.jsonl` → 寻找类似的成功案例
4. 起草符合声音模式的帖子
5. 如果不立即发布，建议添加到 `content/ideas.jsonl`

**输出**：符合用户真实语气且格式适配平台的帖子草稿。

### 示例：联系人查找

**输入**：“为我与 Sarah Chen 的通话做准备”

**过程**：
1. 在 `network/contacts.jsonl` 中搜索 “Sarah Chen”
2. 从 `network/interactions.jsonl` 获取最近的记录
3. 检查 `operations/todos.md` 中与 Sarah 相关的待办项
4. 汇总简报：角色、背景、上次讨论的内容、后续事项

**输出**：带有关系背景信息的会前简报。

## 指南板

1. **声音优先**：在生成任何内容之前，务必先阅读 `identity/voice.md`。
2. **仅追加**：切勿删除 JSONL 文件中的条目——改为存档。
3. **更新时间戳**：修改跟踪数据时设置 `updated` 字段。
4. **交叉引用**：知识库指导内容创作，人脉网络指导业务运营。
5. **记录互动**：始终将会议/通话记录到 `interactions.jsonl`。
6. **保留历史**：`posts.jsonl` 中的历史内容将为未来的表现提供参考。

## 集成

此技能集成了上下文工程原则：

- **context-fundamentals** - 渐进式披露、注意力预算管理
- **memory-systems** - 用于持久化记忆和结构化召回的 JSONL
- **tool-design** - `agents/scripts/` 中的脚本遵循工具设计原则
- **context-optimization** - 模块分离防止上下文膨胀

## 参考资料

内部参考：
- [身份模块](./identity/IDENTITY_zh.md) - 语气与品牌详情
- [内容模块](./content/CONTENT_zh.md) - 内容流水线文档
- [人脉模块](./network/NETWORK_zh.md) - CRM 文档
- [运营模块](./operations/OPERATIONS_zh.md) - 生产力系统
- [智能体脚本](./agents/AGENTS_zh.md) - 自动化文档

外部资源：
- [Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering)
- [Anthropic 上下文工程指南](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents)

---

## 技能元数据

**创建日期**: 2024-12-29
**最近更新**: 2024-12-29
**作者**: Murat Can Koylan
**版本**: 1.0.0
