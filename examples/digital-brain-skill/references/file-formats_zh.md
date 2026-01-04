# 文件格式参考 (File Format Reference)

“数字大脑”中使用的各种文件格式的详细规范。

---

## JSONL 文件

### Schema 规范

每个 JSONL 文件都以 Schema 定义行开始：

```json
{"_schema": "schema_name", "_version": "1.0", "_description": "该文件的用途"}
```

该行在数据处理过程中会被跳过，但用于记录预期的结构。

### 通用字段

所有数据条目应包含：

```json
{
  "id": "type_YYYYMMDD_HHMMSS",  // 唯一标识符
  "created": "ISO8601",           // 创建时间戳
  "updated": "ISO8601"            // 最近修改时间 (可选)
}
```

### ideas.jsonl (灵感日志)

```json
{
  "id": "idea_20241229_143022",
  "created": "2024-12-29T14:30:22Z",
  "idea": "灵感内容",
  "source": "observation (观察)|conversation (对话)|reading (阅读)|shower_thought (灵光一现)",
  "pillar": "内容支柱名称",
  "status": "raw (原始)|developing (开发中)|ready (就绪)|published (已发布)|archived (已存档)",
  "priority": "high (高)|medium (中)|low (低)",
  "notes": "额外背景",
  "tags": ["标签1", "标签2"]
}
```

### posts.jsonl (已发布内容日志)

```json
{
  "id": "post_20241229_160000",
  "published": "2024-12-29T16:00:00Z",
  "platform": "twitter|linkedin|newsletter|blog|youtube",
  "type": "post|thread|article|video|podcast",
  "content": "全文或摘要",
  "url": "https://...",
  "pillar": "内容支柱名称",
  "metrics": {
    "impressions": 0,
    "likes": 0,
    "comments": 0,
    "reposts": 0,
    "saves": 0
  },
  "metrics_updated": "2024-12-29T20:00:00Z",
  "notes": "哪些行之有效/无效",
  "tags": ["标签1", "标签2"]
}
```

### contacts.jsonl (联系人数据库)

```json
{
  "id": "contact_johndoe",
  "created": "2024-01-15T00:00:00Z",
  "updated": "2024-12-29T00:00:00Z",
  "name": "John Doe",
  "handle": "@johndoe",
  "email": "john@example.com",
  "company": "Acme Inc",
  "role": "CEO",
  "location": "San Francisco, USA",
  "circle": "inner|active|network|dormant",
  "how_met": "在会议上认识",
  "relationship": "friend|mentor|peer|collaborator|investor|customer",
  "topics": ["ai", "startups"],
  "can_help_with": ["引荐 VC"],
  "you_can_help_with": ["技术建议"],
  "notes": "个人背景",
  "last_contact": "2024-12-15T00:00:00Z",
  "links": {
    "twitter": "https://twitter.com/johndoe",
    "linkedin": "https://linkedin.com/in/johndoe",
    "website": "https://johndoe.com"
  }
}
```

### interactions.jsonl (互动记录)

```json
{
  "id": "int_20241229_100000",
  "date": "2024-12-29T10:00:00Z",
  "contact_id": "contact_johndoe",
  "type": "call|coffee|dm|email|event|collab",
  "context": "讨论了合作伙伴机会",
  "key_points": ["要点 1", "要点 2"],
  "follow_ups": ["发送提案", "引荐给 Sarah"],
  "sentiment": "positive|neutral|needs_attention"
}
```

### bookmarks.jsonl (书签记录)

```json
{
  "id": "bm_20241229_120000",
  "saved_at": "2024-12-29T12:00:00Z",
  "url": "https://example.com/article",
  "title": "文章标题",
  "source": "article|video|podcast|tool|tweet|paper",
  "category": "ai_agents|building|growth|productivity|leadership|industry|personal",
  "summary": "1-2 句话摘要",
  "key_insights": ["见解 1", "见解 2"],
  "status": "unread|read|reviewed|archived",
  "rating": 1-5,
  "tags": ["标签1", "标签2"]
}
```

### meetings.jsonl (会议记录)

```json
{
  "id": "mtg_20241229_140000",
  "date": "2024-12-29T14:00:00Z",
  "title": "会议标题",
  "type": "1on1|team|external|interview|pitch|advisory",
  "attendees": ["John Doe", "Jane Smith"],
  "duration_mins": 30,
  "agenda": ["话题 1", "话题 2"],
  "notes": "讨论摘要",
  "decisions": ["做出的决定"],
  "action_items": [
    {"task": "任务描述", "owner": "John", "due": "2024-12-31"}
  ],
  "follow_up": "后续步骤"
}
```

### metrics.jsonl (指标日志)

```json
{
  "id": "metrics_20241229",
  "week_of": "2024-12-23",
  "recorded_at": "2024-12-29T00:00:00Z",
  "audience": {
    "twitter_followers": 5000,
    "newsletter_subscribers": 1200,
    "linkedin_connections": 3000,
    "youtube_subscribers": 500
  },
  "engagement": {
    "avg_impressions": 10000,
    "avg_engagement_rate": 0.05,
    "newsletter_open_rate": 0.45
  },
  "content": {
    "posts_published": 7,
    "threads_published": 2,
    "newsletters_sent": 1
  },
  "business": {
    "revenue": 0,
    "mrr": 0,
    "customers": 0,
    "leads": 5
  },
  "personal": {
    "deep_work_hours": 25,
    "exercise_sessions": 4,
    "books_read": 0.5
  },
  "notes": "内容表现强劲的一周"
}
```

---

## YAML 文件

### values.yaml (价值观)

```yaml
core_values:
  - name: "价值观名称"
    description: "含义"
    in_practice: "如何体现"

beliefs:
  - "信念陈述"

contrarian_views:
  - view: "不同见解"
    reasoning: "持此见解的原因"

non_negotiables:
  - "不可逾越的底线"

principles:
  content_creation:
    - "原则"
  business:
    - "原则"
```

### goals.yaml (目标)

```yaml
current_period:
  quarter: "2025 Q1"
  theme: "聚焦增长"

objectives:
  - objective: "目标描述"
    why: "为什么重要"
    key_results:
      - description: "关键结果描述"
        target: 100
        current: 25
        unit: "粉丝"
        status: "on_track (正常)|at_risk (有风险)|behind (滞后)|completed (已完成)"

north_star:
  metric: "北极星指标"
  current: 1000
  target: 10000
  why: "为什么这个最重要"
```

### learning.yaml (学习)

```yaml
current_focus:
  skill: "技能名称"
  why: "为什么要学这个"
  target_level: "目标熟练度"
  deadline: "2025-03-31"

skills:
  - name: "技能名称"
    category: "technical (技术)|creative (创意)|business (商业)|personal (个人)"
    current_level: "beginner (初学者)|intermediate (中级)|advanced (高级)|expert (专家)"
    target_level: "目标"
    status: "learning (学习中)|practicing (练习中)|maintaining (保持中)"
    resources:
      - type: "course (课程)|book (书)|tutorial (教程)|project (项目)"
        title: "资源名称"
        url: "https://..."
        status: "not_started (未开始)|in_progress (进行中)|completed (已完成)"
    milestones:
      - "里程碑描述"
    last_practiced: "2024-12-29"
```

### circles.yaml (关系圈)

```yaml
circles:
  inner:
    description: "核心圈子/亲密关系"
    touchpoint_frequency: "每周"
    members:
      - "姓名 - 关系背景"

  active:
    description: "活跃的合作伙伴"
    touchpoint_frequency: "两周一次"
    members:
      - "姓名 - 关系背景"

groups:
  founders:
    description: "创始人同行"
    members:
      - "姓名"

goals:
  this_quarter:
    - "关系目标"
```

---

## Markdown 文件

### 结构规范

所有 Markdown 文件遵循以下结构：

```markdown
# 标题

简要描述。

---

## 章节 1

内容...

---

## 章节 2

内容...

---

*最近更新时间：[日期]*
```

### 占位符规范

使用 `[PLACEHOLDER: 描述]` 以表示用户可填写的字段：

```markdown
### 您的故事
```
[PLACEHOLDER: 在此写下您的创始人旅程]
```
```

---

## XML 文件

### 提示词模板结构

```xml
<?xml version="1.0" encoding="UTF-8"?>
<prompt name="prompt-name" version="1.0">
  <description>
    该提示词的作用
  </description>

  <instructions>
    <context>
      任务背景
    </context>

    <guidelines>
      需遵循的规则
    </guidelines>

    <output_requirements>
      预期的输出格式
    </output_requirements>
  </instructions>

  <examples>
    输入/输出示例
  </examples>
</prompt>
```

---

## ID 生成

### 规范

`{类型}_{YYYYMMDD}_{HHMMSS}` 或 `{类型}_{唯一标识词}`

示例：
- `idea_20241229_143022`
- `contact_johndoe`
- `post_20241229_160000`
- `bm_20241229_120000`

### 唯一性

ID 在其所在文件内必须唯一。基于时间戳的 ID 能够确保时间序列数据的唯一性。
