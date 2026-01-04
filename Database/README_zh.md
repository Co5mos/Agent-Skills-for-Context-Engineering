# 语义知识注册表 (Semantic Knowledge Registry)

一个为 Agent Skills 仓库设计的 PostgreSQL + pgvector 数据库层。它支持对技能和文档进行语义搜索，使助手能够通过含义而非精确的文本匹配来查找相关知识。

## 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    语义知识注册表 (Semantic Knowledge Registry)    │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  ┌──────────────────┐     ┌──────────────────┐                  │
│  │     技能         │     │     文档         │                  │
│  │  (SKILL.md)      │◄────│   (docs/*.md)    │                  │
│  │                  │     │                  │                  │
│  │  - 元数据         │     │  - 内容          │                  │
│  │  - 嵌入向量       │     │  - 嵌入向量       │                  │
│  │  - 版本           │     │  - 哈希值         │                  │
│  └────────┬─────────┘     └────────┬─────────┘                  │
│           │                        │                            │
│           └────────┬───────────────┘                            │
│                    ▼                                            │
│           ┌────────────────┐                                    │
│           │  skill_sources │  (关联表)                          │
│           │                │                                    │
│           │  追踪哪些文档   │                                    │
│           │  支持了哪些技能 │                                    │
│           └────────────────┘                                    │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## 为什么选择 Postgres + pgvector

1. **统一的系统记录**：在一个数据库中存储结构化元数据和向量嵌入。
2. **生产就绪**：经过实战检验的技术栈，易于扩展。
3. **高性价比**：Neon/Supabase 提供的免费层对种子阶段非常友好。
4. **无供应商锁定**：开源、标准 SQL。

## 前提条件

- Docker 和 Docker Compose (用于本地开发)
- Python 3.9+
- OpenAI API Key (用于生成嵌入向量) 或兼容的嵌入提供商

## 快速上手

### 1. 启动本地数据库

```bash
cd Database
docker-compose up -d
```

这会启动：
- 带有 pgvector 扩展的 PostgreSQL 16
- Adminer (数据库 UI)，访问地址：http://localhost:8080

### 2. 安装 Python 依赖

```bash
pip install -r requirements.txt
```

### 3. 初始化 Schema

```bash
python scripts/init_db.py
```

### 4. 为技能和文档建立索引

```bash
# 为所有技能建立索引
python scripts/index.py --skills

# 为所有文档建立索引
python scripts/index.py --docs

# 为所有内容建立索引
python scripts/index.py --all
```

### 5. 搜索

```bash
# 语义搜索
python scripts/search.py "如何为助手设计工具"

# 带过滤条件的搜索
python scripts/search.py "上下文窗口优化" --type skills --limit 5
```

## 配置

设置环境变量或创建一个 `.env` 文件：

```bash
# 数据库连接
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/agentskills

# 嵌入向量提供商
OPENAI_API_KEY=sk-...

# 可选：使用不同的嵌入模型
EMBEDDING_MODEL=text-embedding-3-small
EMBEDDING_DIMENSION=1536
```

## Schema 概览

### skills (技能)

助手技能的主要注册表。

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| id | UUID | 主键 |
| name | VARCHAR(255) | 技能标识符 (例如 "tool-design") |
| description | TEXT | 来自 frontmatter 的简要描述 |
| content | TEXT | SKILL.md 的全文内容 |
| path | VARCHAR(512) | 文件系统路径 |
| version | VARCHAR(50) | 语义版本号 |
| embedding | vector(1536) | 语义嵌入向量 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 最后更新时间 |

### documents (文档)

用于构建技能的源文档。

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| id | UUID | 主键 |
| title | VARCHAR(512) | 文档标题 |
| content | TEXT | 文档全文内容 |
| path | VARCHAR(512) | 文件系统路径 |
| content_hash | VARCHAR(64) | SHA-256 哈希值，用于检测更改 |
| embedding | vector(1536) | 语义嵌入向量 |
| created_at | TIMESTAMP | 创建时间 |
| updated_at | TIMESTAMP | 最后更新时间 |

### skill_sources (技能来源)

连接技能与其源文档的关联表。

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| skill_id | UUID | 指向 skills 的外键 |
| document_id | UUID | 指向 documents 的外键 |
| relevance | FLOAT | 相关度 (0-1) |

### skill_versions (技能版本)

技能的版本历史。

| 列名 | 类型 | 描述 |
|--------|------|-------------|
| id | UUID | 主键 |
| skill_id | UUID | 指向 skills 的外键 |
| version | VARCHAR(50) | 版本字符串 |
| content | TEXT | 该版本下的内容 |
| created_at | TIMESTAMP | 此版本创建的时间 |

## Python API

```python
from scripts.registry import SkillRegistry

# 初始化
registry = SkillRegistry()

# 语义搜索技能
results = registry.search_skills("如何设计有效的助手工具", limit=5)
for skill in results:
    print(f"{skill['name']}: {skill['similarity']:.3f}")

# 搜索文档
docs = registry.search_documents("上下文窗口管理", limit=10)

# 为新文档查找相关技能
related = registry.find_related_skills(new_doc_content, threshold=0.7)

# 注册新技能
registry.upsert_skill(
    name="new-skill",
    description="一个新技能",
    content=skill_content,
    path="skills/new-skill/SKILL.md",
    version="1.0.0"
)

# 将技能链接到源文档
registry.link_skill_to_document(skill_id, document_id, relevance=0.9)
```

## 助手集成

当助手收到一份新文档时：

```python
from scripts.registry import SkillRegistry

registry = SkillRegistry()

def process_new_document(content: str, path: str):
    """处理新文档并确定技能更新。"""
    
    # 1. 为新文档建立索引
    doc_id = registry.upsert_document(
        title=extract_title(content),
        content=content,
        path=path
    )
    
    # 2. 查找语义相关的技能
    related_skills = registry.find_related_skills(content, threshold=0.7)
    
    # 3. 确定操作
    if not related_skills:
        # 没有相关技能 - 可能需要创建一个新技能
        return {"action": "create_skill", "document_id": doc_id}
    else:
        # 存在相关技能 - 可能需要更新
        return {
            "action": "update_skills",
            "document_id": doc_id,
            "related_skills": related_skills
        }
```

## 生产环境部署

在生产环境中，请使用支持 pgvector 的托管 PostgreSQL 服务：

- **Neon**：免费层包含 0.5GB 存储空间，包含 pgvector
- **Supabase**：免费层包含 500MB 存储空间，包含 pgvector
- **Railway**：每月 5 美元，包含 pgvector
- **AWS RDS**：可使用 pgvector 扩展

更新 `DATABASE_URL` 以指向您的生产实例。

## 维护

### 在 Schema 更改后重新索引

```bash
python scripts/index.py --all --force
```

### 检查嵌入向量覆盖率

```bash
python scripts/check_coverage.py
```

### 备份

```bash
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql
```
