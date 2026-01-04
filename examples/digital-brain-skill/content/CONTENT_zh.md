---
name: content-module
description: 内容创作中心——包含灵感、草稿、日历和已发布的帖子。用于内容规划、创作和跟踪。
---

# 内容中心 (Content Hub)

您的内容创作与管理系统。

## 模块内的文件

| 文件 | 格式 | 用途 |
|------|--------|---------|
| `ideas.jsonl` | JSONL | 原始内容灵感 (仅追加) |
| `posts.jsonl` | JSONL | 已发布内容的日志 |
| `calendar.md` | Markdown | 内容排期表 |
| `drafts/` | 文件夹 | 进行中的内容草稿 |
| `templates/` | 文件夹 | 可重用的内容格式模板 |
| `engagement.jsonl` | JSONL | 保存的用于获取灵感的帖子/线程 |

## 工作流

### 捕获灵感
```bash
# 带有时间戳并追加到 ideas.jsonl
{
  "id": "idea_YYYYMMDD_HHMMSS",
  "created": "ISO8601",
  "idea": "内容描述",
  "source": "来源",
  "pillar": "内容支柱",
  "status": "raw (原始)|developing (开发中)|ready (就绪)",
  "priority": "high (高)|medium (中)|low (低)"
}
```

### 内容创作流水线
```
1. ideas.jsonl (捕获)
      ↓
2. drafts/draft_[topic].md (开发)
      ↓
3. 对照 voice.md 进行审核
      ↓
4. 发布
      ↓
5. posts.jsonl (存档并记录指标)
```

### 每周内容复盘
1. 审核 `ideas.jsonl`——晋升优质想法或存档过时想法。
2. 检查 `calendar.md`——规划下一周。
3. 审核 `posts.jsonl`——分析哪些内容行之有效。
4. 更新 `engagement.jsonl`——保存激发灵感的内容。

## 智能体指令

<instructions>
在处理内容时：

1. **捕获灵感**：始终追加到 ideas.jsonl，绝不覆盖。
2. **创建草稿**：使用 templates/ 中的模板作为起点。
3. **撰写内容**：**务必**先阅读 identity/voice.md。
4. **发布**：将带有完整元数据的记录记入 posts.jsonl。
5. **分析**：参考 posts.jsonl 了解表现模式。

优先级打分：
- 高：时效性强、价值高、符合当前目标
- 中：不错的想法，无紧迫性
- 低：值得捕获，稍后开发
</instructions>

## 待跟踪的内容指标

```yaml
互动指标:
  - impressions (展示量)
  - likes (点赞数)
  - comments (评论数)
  - reposts (转发数)
  - saves (保存数)
  - link_clicks (链接点击数)

质量指标:
  - 评论质量: "有意义的讨论 vs. 表情符号回应"
  - 分享背景: "人们分享时说了什么"
  - 粉丝转化: "帖子带来的粉丝增长"
```
