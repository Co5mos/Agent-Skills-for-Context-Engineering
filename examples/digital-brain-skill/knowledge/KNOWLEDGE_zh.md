---
name: knowledge-module
description: 个人知识库——包含研究记录、书签、学习资源和笔记。用于信息检索、研究组织和学习跟踪。
---

# 知识库 (Knowledge Base)

您的“第二大脑”，用于研究、学习和信息组织。

## 模块内的文件

| 文件 | 格式 | 用途 |
|------|--------|---------|
| `bookmarks.jsonl` | JSONL | 保存的链接和资源 |
| `learning.yaml` | YAML | 技能和学习目标 |
| `competitors.md` | Markdown | 竞争格局分析 |
| `research/` | 文件夹 | 深度研究笔记 |
| `notes/` | 文件夹 | 灵感快速捕捉笔记 |

## 数据 Schema

### 书签条目 (Bookmark Entry)
```json
{
  "id": "bm_YYYYMMDD_HHMMSS",
  "saved_at": "ISO8601",
  "url": "https://...",
  "title": "页面标题",
  "source": "article (文章)|video (视频)|podcast (播客)|tool (工具)|tweet (推文)|paper (论文)",
  "category": "类别名称",
  "summary": "1-2 句话摘要",
  "key_insights": ["见解1", "见解2"],
  "status": "unread (未读)|read (已读)|reviewed (已复评审)|archived (存档)",
  "rating": 1-5,
  "tags": ["标签1", "标签2"]
}
```

## 工作流

### 保存资源
1. 带着 "unread" 状态追加到 `bookmarks.jsonl`。
2. 添加类别和初始标签。
3. 稍后：阅读、总结并更新状态。

### 研究项目
1. 为深度研究创建 `research/[主题].md`。
2. 链接相关书签。
3. 综合见解。
4. 提取内容想法。

### 学习跟踪
1. 在 `learning.yaml` 中定义技能。
2. 将资源链接到技能。
3. 跟踪进度和里程碑。
4. 每季度复盘。

## 智能体指令

<instructions>
管理知识时：

1. **保存链接**：务必捕获 URL、标题和初始类别。
2. **组织**：使用一致的类别和标签。
3. **检索**：通过类别、标签或关键词搜索 `bookmarks.jsonl`。
4. **综合**：当被问及某个话题时，先检查 `research/` 文件夹。
5. **学习更新**：完成资源学习后更新 `learning.yaml`。

可使用的类别：
- ai_agents: AI、智能体、自动化
- building: 初创公司、产品、工程
- growth: 营销、受众、内容
- productivity: 系统、工具、工作流
- leadership: 管理、团队、文化
- industry: 市场趋势、竞争对手
- personal: 健康、人际关系、个人生活
</instructions>

## 知识图谱提示

检索信息时，请考虑以下关联：
- 书签 → 内容想法
- 研究 → 具有权威性的作品
- 学习 → 需要在品牌中强调的技能
- 竞争对手 → 差异化切入点
