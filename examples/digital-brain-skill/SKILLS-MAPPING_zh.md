# 技能映射：数字大脑 (Digital Brain)

本文档映射了 [Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) 中的原则是如何应用于数字大脑（Digital Brain）实现的。

---

## 应用的上下文工程原则

### 1. 上下文基础 (Context Fundamentals)

| 概念 | 来源技能 | 数字大脑应用方案 |
|---------|--------------|---------------------------|
| **注意力预算** | context-fundamentals | 模块分离确保仅加载相关内容。内容任务加载声音文件（约 200 行）；网络任务加载联系人文件。绝不加载全部内容。 |
| **渐进式披露** | context-fundamentals | 三层架构：第一层 (SKILL.md 元数据)、第二层 (模块指令)、第三层 (数据文件)。每一层仅在需要时加载。 |
| **高信号 Token** | context-fundamentals | JSONL Schema 仅包含核心字段。声音配置文件关注模式，而非详尽的规则。 |

**设计决策**：
> “寻找尽可能小的高信号 Token 集合，以最大化实现某种预期结果的可能性。”

**应用**：保持 `voice.md` 专注于独特的模式（标志性短语、反模式），而不是 Claude 已经知道的通用写作建议。

---

### 2. 记忆系统 (Memory Systems)

| 概念 | 来源技能 | 数字大脑应用方案 |
|---------|--------------|---------------------------|
| **仅追加日志** | memory-systems | 所有 `.jsonl` 文件均为仅追加。状态更改通过 `"status": "archived"` 实现，从不删除。保留完整历史记录。 |
| **结构化召回** | memory-systems | 各文件间一致的 Schema 可实现模式匹配。通过 `contact_id` 将 `contacts.jsonl` 关联至 `interactions.jsonl`。 |
| **情境记忆** | memory-systems | `interactions.jsonl` 捕获离散事件。`posts.jsonl` 记录带有性能指标的内容，用于回顾性分析。 |
| **语义记忆** | memory-systems | 带有类别和标签的 `knowledge/bookmarks.jsonl` 可实现基于主题的检索。 |

**设计决策**：
> “智能体维护持久化记忆文件，以跟踪复杂序列的进度。”

**应用**：在 `operations/metrics.jsonl` 中积累每周快照，支持趋势分析而无需从原始数据重新计算。

---

### 3. 工具设计 (Tool Design)

| 概念 | 来源技能 | 数字大脑应用方案 |
|---------|--------------|---------------------------|
| **自包含工具** | tool-design | `agents/scripts/` 中的脚本是独立的 Python 文件。每个脚本只做一件事：`weekly_review.py` 生成复盘，`stale_contacts.py` 查找被忽视的关系。 |
| **清晰的输入/输出** | tool-design | 脚本从已知路径读取，将结构化文本输出到 stdout。除非明确记录，否则无副作用。 |
| **Token 效率** | tool-design | 脚本处理数据并返回摘要。智能体接收结果，而非原始数据处理逻辑。 |

**设计决策**：
> “工具应该是自包含的、明确的，并提升 Token 效率。”

**应用**：让 `content_ideas.py` 在内部分析书签和过去的帖子，仅返回可操作的建议，而非原始分析数据。

---

### 4. 上下文优化 (Context Optimization)

| 概念 | 来源技能 | 数字大脑应用方案 |
|---------|--------------|---------------------------|
| **模块分离** | context-optimization | 六个独特的模块 (`identity/`, `content/`, `knowledge/`, `network/`, `operations/`, `agents/`) 防止交叉污染。内容创作绝不需要加载网络数据。 |
| **即时加载** | context-optimization | 模块指令文件 (`IDENTITY.md`, `CONTENT.md` 等) 仅在相关模块启用时加载。 |
| **引用深度** | context-optimization | 主 `SKILL.md` 链接到模块文档，模块文档链接到数据文件。获取任何信息的路径最大为两跳。 |

**设计决策**：
> “与其预加载所有数据，不如维护轻量级标识符，并在运行时动态加载数据。”

**应用**：在人脉网络模块中，智能体首先扫描 `contacts.jsonl` 查找匹配的名称，然后仅针对该联系人加载特定的 `interactions.jsonl` 条目。

---

### 5. (缓解) 上下文退化 (Context Degradation)

| 风险 | 来源技能 | 数字大脑缓解方案 |
|------|--------------|--------------------------|
| **上下文腐化** | context-degradation | 模块分离限制了单次加载量。声音文件保持在 300 行以内。数据文件通过 JSONL 流式传输（按行读取）。 |
| **过时上下文** | context-degradation | 联系人中的 `last_contact` 时间戳。`stale_contacts.py` 主动呈现需要关注的关系。 |
| **冲突的指令** | context-degradation | 每个领域唯一的真相来源。声音仅在 `voice.md`。目标仅在 `goals.yaml`。无重复。 |

**设计决策**：
> “随着上下文长度增加，模型在准确性和召回率方面的收益会递减。”

**应用**：保持 `SKILL.md` 在 200 行以内，每个模块指令文件在 100 行以内，并使用外部文件存储数据而非内联内容。

---

## 架构决策

### 为什么日志使用 JSONL？

```
✓ 按设计即为仅追加 (Append-only)
✓ 对流友好（无需解析整个文件）
✓ 每行一个 Schema（首行记录结构）
✓ 对智能体友好（标准的 JSON 解析）
✓ 兼容 Grep 以实现快速搜索

✗ 不易于人类编辑（配置请使用 YAML/MD）
✗ 无事务处理（对于个人数据是可以接受的）
```

### 为什么叙事性内容使用 Markdown？

```
✓ 人类可读且易于编辑
✓ 丰富的格式（表格、列表、代码）
✓ Git 友好的 diff
✓ 通用渲染

用于：声音、品牌、日历、待办事项、模板
```

### 为什么配置使用 YAML？

```
✓ 层级结构
✓ 人类可读
✓ 支持注释
✓ 嵌套数据的简洁语法

用于：目标、价值观、圈子、学习
```

### 为什么提示词使用 XML？

```
✓ 为智能体提供清晰的结构
✓ 命名的分节（指令、上下文、输出）
✓ 变量占位符
✓ 易于验证

用于：内容生成模板、复杂提示词
```

---

## 工作流映射

### 内容创作 → 应用的技能

```
用户：“写一篇关于公开建设 (building in public) 的帖子”

技能链：
1. context-fundamentals → 仅加载 identity 模块
2. memory-systems → 从 voice.md 检索声音模式
3. context-optimization → 不加载 network/operations
4. tool-design → 使用内容模板作为结构化支架

加载的文件：
- SKILL.md (50 tokens) - 路由
- identity/IDENTITY.md (80 tokens) - 模块指令
- identity/voice.md (200 tokens) - 声音模式
- identity/brand.md (扫描支柱) - 主题验证

总计：约 400 tokens，而加载整个“大脑”需要约 5000 tokens
```

### 人脉关系管理 → 应用的技能

```
用户：“为我与 Alex 的通话做准备”

技能链：
1. context-fundamentals → 仅加载 network 模块
2. memory-systems → 查询联系人，然后查询互动情况
3. context-optimization → 即时加载特定联系人
4. tool-design → 结构化输出（简报格式）

加载的文件：
- SKILL.md (50 tokens) - 路由
- network/NETWORK.md (60 tokens) - 模块指令
- network/contacts.jsonl (扫描 Alex) - 联系人数据
- network/interactions.jsonl (按 contact_id 过滤) - 历史记录

总计：约 300 tokens，仅包含相关上下文
```

---

## 权衡与权宜之计

| 方案 | 权衡 | 理由 |
|----------|-----------|-----------|
| 分离模块 | 需要导航更多文件 | 防止上下文膨胀；支持针对性加载 |
| 数据使用 JSONL | 对人类不太友好 | 针对智能体解析和追加操作进行了优化 |
| 无数据库 | 无查询语言 | 简单；离线工作；无依赖 |
| Python 脚本 | 需要 Python 运行时 | 通用；可读；易于扩展 |
| 占位符而非示例 | 用户必须自行填写 | 避免“AI 废话”；强制进行个性化设置 |

---

## 验证清单

在扩展“数字大脑”时，请验证：

- [ ] 新文件遵循格式规范 (JSONL/YAML/MD/XML)
- [ ] 模块指令文件保持在 100 行以内
- [ ] JSONL 文件将 Schema 行作为第一项
- [ ] 跨模块引用保持最小化
- [ ] 脚本是自包含的且具有清晰的 I/O
- [ ] 无重复的真相来源

---

## 相关技能

本实现借鉴了集合中的以下技能：

| 技能 | 主要应用 |
|-------|---------------------|
| `context-fundamentals` | 整体架构、渐进式披露 |
| `context-degradation` | 缓解策略、文件大小限制 |
| `context-optimization` | 模块分离、即时加载 |
| `memory-systems` | JSONL 设计、仅追加模式 |
| `tool-design` | 智能体脚本、I/O 模式 |
| `multi-agent-patterns` | 未来：授权给专门的子智能体 |

---

*本映射展示了理论上的上下文工程原则是如何转化为实际的系统设计的。*
