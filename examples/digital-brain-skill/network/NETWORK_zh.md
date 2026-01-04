---
name: network-module
description: 人脉与联系人管理——包含认识的人、互动历史和社交笔记。用于会议前、引荐他人或管理关系。
---

# 人脉模块 (Network Module)

您的个人 CRM，用于维护有意义的人际关系。

## 模块内的文件

| 文件 | 格式 | 用途 |
|------|--------|---------|
| `contacts.jsonl` | JSONL | 联系人数据库 |
| `interactions.jsonl` | JSONL | 会议/对话日志 |
| `circles.yaml` | YAML | 关系圈分层与分组 |
| `intros.md` | Markdown | 待处理/已完成的引荐记录 |

## 数据 Schema

### 联系人条目 (Contact Entry)
```json
{
  "id": "contact_[唯一标识]",
  "created": "ISO8601",
  "updated": "ISO8601",
  "name": "全名",
  "handle": "@twitter账号",
  "email": "email@domain.com",
  "company": "公司名称",
  "role": "职位",
  "location": "城市, 国家",
  "circle": "inner (核心)|active (活跃)|network (普通)|dormant (沉睡)",
  "how_met": "结识方式",
  "relationship": "friend (朋友)|mentor (导师)|peer (同行)|collaborator (合作伙伴)|investor (投资者)|customer (客户)",
  "topics": ["话题1", "话题2"],
  "can_help_with": ["他们能为你提供什么帮助"],
  "you_can_help_with": ["你能为他们提供什么帮助"],
  "notes": "个人笔记",
  "last_contact": "ISO8601",
  "links": {
    "twitter": "url",
    "linkedin": "url",
    "website": "url"
  }
}
```

### 互记录条目 (Interaction Entry)
```json
{
  "id": "int_YYYYMMDD_HHMMSS",
  "date": "ISO8601",
  "contact_id": "contact_[id]",
  "type": "call (通话)|coffee (面谈)|dm (私信)|email (邮件)|event (活动)|collab (协作)",
  "context": "讨论内容",
  "key_points": ["要点1", "要点2"],
  "follow_ups": ["后续事项1", "后续事项2"],
  "sentiment": "positive (积极)|neutral (中性)|needs_attention (需关注)"
}
```

## 工作流

### 会议前
1. 在 `contacts.jsonl` 中查找联系人。
2. 在 `interactions.jsonl` 中查看最近的互动记录。
3. 检查 `circles.yaml` 以了解关系背景。
4. 记录任何待处理的后续事项或引荐。

### 会议后
1. 在 `interactions.jsonl` 中记录互动情况。
2. 在 `contacts.jsonl` 中更新 `last_contact` (最近联系时间)。
3. 将任何后续事项添加到 `operations/todos.md`。
4. 必要时更新关系笔记。

### 进行引荐
1. 在 `contacts.jsonl` 中检查双方联系人。
2. 确保互惠互助 (检查 `can_help_with` 字段)。
3. 在 `intros.md` 中记录。
4. 跟踪后续进度。

## 智能体指令

<instructions>
在管理关系时：

1. **查找联系人**：按姓名、账号、公司或话题进行搜索。
2. **会前准备**：汇总联系人信息 + 最近互动情况 + 共同兴趣。
3. **记录互动**：始终包含日期、类型、内容和后续事项。
4. **引荐匹配**：交叉引用 `can_help_with` 字段。
5. **关系维护**：标记那些 `last_contact` 日期过久的联系人。

关系圈 (Circle) 定义：
- inner: 亲密关系，定期联系。
- active: 当前合作伙伴，互动频繁。
- network: 认识的联系人，定期触达。
- dormant: 历史联系人，可能重新激活。
</instructions>

## 社交原则

```yaml
社交哲学:
  - "先给予，再索取"
  - "质量重于数量"
  - "后续跟进代表一切"
  - "始终真诚地提供帮助"
  - "进行有温度的引荐，拒绝冷启动"
```
