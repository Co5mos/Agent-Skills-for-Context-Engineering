# 示例：会议准备工作流 (Meeting Preparation Workflow)

使用“数字大脑”进行会议准备的完整演示。

---

## 场景设定

您在 30 分钟后将与 “Sarah Chen” 通话，需要一份快速简报。

---

## 第一步：智能体激活技能

**用户输入**：“为我和 Sarah Chen 的通话做准备。”

**智能体识别**：
- 触发词：“为……做准备” + 人名 → 会议准备。
- 动作：加载“数字大脑”技能，加载人脉模块。

---

## 第二步：查找联系人

智能体在 `network/contacts.jsonl` 中搜索 “Sarah Chen”：

```json
{
  "id": "contact_sarahchen",
  "name": "Sarah Chen",
  "handle": "@sarahchen_ai",
  "company": "AI Ventures",
  "role": "合伙人",
  "circle": "active (活跃圈子)",
  "how_met": "在我发布关于智能体系统的线程后，她主动联系了我",
  "relationship": "potential_investor (潜在投资者)",
  "topics": ["ai_agents (AI 智能体)", "investing (投资)", "founder_market_fit (创始人与市场匹配)"],
  "can_help_with": ["融资引荐", "市场进入策略建议"],
  "you_can_help_with": ["技术尽职调查", "智能体架构见解"],
  "notes": "作为 VC，她的技术非常强。曾在 Google 担任 ML 工程师。看重深度而非华而不实装饰。",
  "last_contact": "2024-12-15T00:00:00Z",
  "links": {
    "twitter": "https://twitter.com/sarahchen_ai",
    "linkedin": "https://linkedin.com/in/sarahchen"
  }
}
```

---

## 第三步：调取互动历史

智能体通过 `contact_id` 过滤 `network/interactions.jsonl`：

```json
// 最近一次互动：
{
  "id": "int_20241215_100000",
  "date": "2024-12-15T10:00:00Z",
  "contact_id": "contact_sarahchen",
  "type": "call",
  "context": "初次引荐通话。她对我们的智能体框架方法很感兴趣。",
  "key_points": [
    "她喜欢我们的上下文工程方法",
    "询问了护城河 —— 我解释了技能的可组合性",
    "她提到她的被投公司 BuildAI 可能需要我们的帮助"
  ],
  "follow_ups": [
    "发送带有技术架构的 PPT (deck)",
    "引荐 BuildAI 团队"
  ],
  "sentiment": "positive (积极)"
}

// 较早的互动：
{
  "id": "int_20241201_140000",
  "date": "2024-12-01T14:00:00Z",
  "contact_id": "contact_sarahchen",
  "type": "dm (私信)",
  "context": "在我发布关于智能体系统的线程后，她发了私信",
  "key_points": ["对技术深度印象深刻", "询问了投资阶段"],
  "follow_ups": ["安排初次引荐通话"],
  "sentiment": "positive (积极)"
}
```

---

## 第四步：检查待办事项

智能体扫描 `operations/todos.md` 中与 Sarah 相关的项：

```markdown
### 等待中
| 任务 | 等待对象 | 起始日期 | 后续跟进 |
|------|------------|-------|-----------|
| BuildAI 引荐 | Sarah Chen | 2024-12-15 | 2024-12-29 |

### P1 - 本周要完成
- [ ] 发送技术 PPT (deck) 给 Sarah Chen
```

**洞察**：后续事项已经逾期了！

---

## 第五步：生成简报

智能体汇编带上下文的简报：

```markdown
# 会议简报：Sarah Chen

**时间**：今天，[时间]
**访谈对象**：Sarah Chen，AI Ventures 合伙人
**关系**：潜在投资者 (活跃圈子)
**自上次联系以来的天数**：14 天

---

## 快速背景
- 前 Google ML 工程师（技术背景强）
- 在你发布智能体系统线程后主动联系
- 看重深度，而非表面功夫

---

## 上次通话记录 (12月15日)
- 初次引荐通话 —— 情感倾向：积极
- 她喜欢上下文工程方法
- 询问了护城河 → 解释了技能的可组合性
- 提到 BuildAI (被投公司) 可能需要帮助

---

## 待处理后续事项 ⚠️
1. **发送技术 PPT** —— 仍待处理 (P1)
2. **BuildAI 引荐** —— 正在等她，今天需跟进

---

## 她的兴趣点
- 智能体架构
- 上下文工程
- 技术尽职调查

---

## 价值交换
- **她可以提供**：融资引荐、市场进入策略
- **你可以提供**：技术尽职调查、智能体架构见解

---

## 建议谈话要点
1. 为 PPT 延迟发送道歉，提议通过在线演示 (walk through) 进行讲解
2. 询问 BuildAI 引荐的进展情况
3. 分享关于 [相关功能] 的最新进展
4. 询问她的投资逻辑及时间线
5. [可选] 展示我们的 context-fundamentals 模式

---

## 链接
- Twitter: @sarahchen_ai
- LinkedIn: linkedin.com/in/sarahchen
```

---

## 第六步：会后记录

通话结束后，智能体协助将其记录到 `network/interactions.jsonl`：

**用户**：“刚和 Sarah 谈完。她正着手引荐 BuildAI，并希望在周五前看到我们的 PPT。”

智能体追加记录：

```json
{
  "id": "int_20241229_143000",
  "date": "2024-12-29T14:30:00Z",
  "contact_id": "contact_sarahchen",
  "type": "call",
  "context": "跟进通话。两方面都有积极进展。",
  "key_points": [
    "她正着手进行 BuildAI 的引荐",
    "希望在周五前拿到技术 PPT",
    "提到 A 轮融资时间线在第一季度"
  ],
  "follow_ups": [
    "在12月31日周五前发送 PPT",
    "预计下周进行 BuildAI 的对接"
  ],
  "sentiment": "positive"
}
```

智能体更新 `operations/todos.md`：

```markdown
### P0 - 今日必做
- [ ] 发送技术 PPT (deck) 给 Sarah Chen (截止日期：周五)

### 等待中
| 任务 | 等待对象 | 起始日期 | 后续跟进 |
|------|------------|-------|-----------|
| BuildAI 引荐 | Sarah Chen | 2024-12-29 | 2025-01-05 |
```

智能体更新联系人的 `last_contact`：

```json
{
  "id": "contact_sarahchen",
  ...
  "last_contact": "2024-12-29T14:30:00Z"
}
```

---

## 涉及文件

| 文件 | 用途 | Token 消耗 (约) |
|------|---------|------------|
| `SKILL.md` | 路由 | 50 |
| `network/NETWORK.md` | 模块指令 | 60 |
| `network/contacts.jsonl` | 查找联系人 | 80 |
| `network/interactions.jsonl` | 历史记录 | 150 |
| `operations/todos.md` | 待处理项 | 50 |

**总计**：约 390 tokens

---

## 核心收获

1. **上下文组装**：智能体从多个来源提取信息。
2. **可操作的输出**：简报重点突出了要讨论的内容。
3. **后续事项跟踪**：主动呈现待处理项。
4. **闭环管理**：会议 → 记录 → 更新待办事项 → 更新联系人。
5. **人脉情报**：记录的笔记指导了会议的沟通方式。
