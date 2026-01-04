---
name: agents-module
description: 用于“数字大脑”的自动化脚本和智能体助手。使用这些脚本进行定期任务、总结和维护。
---

# 智能体自动化 (Agent Automation)

用于维护和利用您的“数字大脑”的脚本与工作流。

## 可用脚本

| 脚本 | 用途 | 频率 |
|--------|---------|-----------|
| `weekly_review.py` | 根据数据生成周复盘 | 每周 |
| `content_ideas.py` | 从知识库中生成内容灵感 | 按需 |
| `stale_contacts.py` | 查找需要联络的联系人 | 每周 |
| `metrics_snapshot.py` | 汇总待跟踪的指标 | 每周 |
| `idea_to_draft.py` | 将灵感扩展为草稿 | 按需 |

## 如何使用

脚本位于 `agents/scripts/` 中。它们与您的“数字大脑”数据配合工作，并可由智能体根据需要运行。

### 运行脚本
```bash
# 智能体可以直接执行脚本
python agents/scripts/weekly_review.py

# 或者带有参数运行
python agents/scripts/content_ideas.py --pillar "ai_agents" --count 5
```

### 脚本输出
脚本将输出发送到标准输出 (stdout)，格式可由智能体处理。在适当的情况下，它们也可能写入文件（例如，生成复盘文档）。

## 智能体指令

<instructions>
使用自动化脚本时：

1. **每周复盘**：每周日运行，输出已填入数据的复盘模板。
2. **内容灵感**：当用户征求灵感时使用，旨在杠杆化知识库。
3. **沉睡联系人**：每周运行，展示需要关注的人际关系。
4. **指标快照**：每周运行，以追加到 metrics.jsonl。
5. **从灵感到草稿**：当用户想要开发特定的灵感时使用。

脚本会读取“数字大脑”文件并输出可操作的结果。
</instructions>

## 工作流自动化

### 周日周复盘
```
1. 运行 metrics_snapshot.py 以更新 metrics.jsonl。
2. 运行 stale_contacts.py 以确定联络需求。
3. 运行 weekly_review.py 以生成复盘文档。
4. 向用户展示摘要。
```

### 内容构思环节
```
1. 从 knowledge/bookmarks.jsonl 中读取最近的条目。
2. 在 content/ideas.jsonl 中查看尚未开发的灵感。
3. 运行 content_ideas.py 以获取新鲜的建议。
4. 与内容日历进行交叉引用。
```

### 会前准备
```
1. 在 network/contacts.jsonl 中查找联系人。
2. 从 network/interactions.jsonl 中提取最近的互动。
3. 检查任何涉及他们的待办事项。
4. 生成带有背景信息的简报。
```

## 自定义脚本开发

若要添加新脚本：
1. 在 `agents/scripts/` 中创建 Python 文件。
2. 遵循现有模式（读取 JSONL，输出结构化数据）。
3. 在本文档中进行记录。
4. 使用样本数据进行测试。
