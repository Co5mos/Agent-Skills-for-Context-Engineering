# 如何根据新文档 Frontmatter 更新数据库

现在 `docs/` 中的所有文档都包含带有 `name`、`description` 和 `source_url` 字段的 Frontmatter，这与 Skills 的格式保持一致。

## 第一步：运行数据库迁移

首先，在您的数据库中添加新列（`description` 和 `source_url`）：

### 针对本地 Docker 数据库：

```bash
cd Database
PGPASSWORD=postgres psql -h localhost -U postgres -d agentskills -f schema/add_document_fields.sql
```

### 针对 Supabase：

```bash
cd Database

# 选项 1：直接使用 psql
psql "您的-supabase-连接-字符串" -f schema/add_document_fields.sql

# 选项 2：使用 Supabase SQL 编辑器
# 1. 前往：https://supabase.com/dashboard/project/cppdscqtkzuiwqdiumdq/sql/new
# 2. 复制并粘贴 schema/add_document_fields.sql 的内容
# 3. 点击 "Run" (运行)
```

## 第二步：重新对文档建立索引

运行迁移后，重新对所有文档建立索引，以便使用新的 Frontmatter 数据更新它们：

```bash
cd Database

# 仅重新对文档建立索引（速度较快）
python -m scripts.index --docs

# 或者重新对所有内容建立索引
python -m scripts.index --all
```

该操作将：
- 解析每个文档中的 Frontmatter
- 提取 `name`、`description` 和 `source_url`
- 使用新信息更新数据库记录
- 必要时重新生成向量嵌入

## 第三步：验证更新

检查文档是否已更新：

```bash
cd Database

# 检查特定文档
python -c "
from scripts.registry import SkillRegistry
registry = SkillRegistry()
doc = registry.get_document('docs/blogs.md')
if doc:
    print(f\"Title: {doc.get('title', 'N/A')}\")
    print(f\"Description: {doc.get('description', 'N/A')[:100]}...\")
    print(f\"Source URL: {doc.get('source_url', 'N/A')}\")
"

# 列出所有文档及其新字段
python -c "
from scripts.registry import SkillRegistry
from scripts.db import execute_query

results = execute_query('SELECT title, description, source_url FROM documents ORDER BY title')
print(f\"Found {len(results)} documents:\")
for r in results:
    print(f\"\\n  {r['title']}\")
    print(f\"    Description: {r.get('description', 'N/A')[:80]}...\")
    print(f\"    Source: {r.get('source_url', 'N/A')}\")
"
```

## 变更内容

### 文档 Frontmatter 格式

所有文档现在都以以下内容开头：

```yaml
---
name: document-name
description: Brief description of what the document covers
doc_type: blog|research|reference|case_study
source_url: URL or "No" if unknown
---
```

所有字段（`name`, `description`, `doc_type`, `source_url`）都从 Frontmatter 中读取并存储在数据库中，与 Skills 格式匹配。

### 已更新的文档

1. **blogs.md** → `context-engineering-blogs` (doc_type: blog)
2. **claude_research.md** → `production-grade-llm-agents` (doc_type: research)
3. **compression.md** → `context-compression-evaluation` (doc_type: research)
4. **gemini_research.md** → `advanced-agentic-architectures` (doc_type: research)
5. **netflix_context.md** → `netflix-context-compression` (doc_type: case_study)
6. **vercel_tool.md** → `vercel-tool-reduction` (doc_type: blog, 包含源 URL)
7. **anthropic_skills/agentskills.md** → `agent-skills-format` (doc_type: reference)

### 数据库 Schema 变更

在 `documents` 表中添加了两个新列：
- `description TEXT` - 来自 Frontmatter 的描述
- `source_url VARCHAR(512)` - 源 URL 或 "No"

## 故障排除

### 迁移失败

如果收到“列已存在” (column already exists) 错误，表示迁移已运行。您可以直接跳到第二步。

### 文档未更新

如果文档未更新，请检查：
1. Frontmatter 格式是否正确（处于 `---` 标记之间的 YAML）。
2. 数据库连接是否正常。
3. 文件路径是否正确。

### 需要强制重新建立索引

即使内容哈希未发生变化，如果要强制重新建立索引：

```bash
# 这将重新生成向量嵌入并更新所有字段
python -m scripts.index --docs --force
```

## 后续步骤

更新完成后：
- 文档在搜索结果中将拥有正确的名称和描述。
- 源 URL 将被追踪以进行归属说明。
- 通过改进的元数据获得更好的语义搜索结果。
