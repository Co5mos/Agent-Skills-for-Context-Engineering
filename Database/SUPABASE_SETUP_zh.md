# Supabase 设置指南

本指南将引导您在 Supabase 上设置语义知识注册库（Semantic Knowledge Registry）。

## 前提条件

- Supabase 账号（免费版即可）
- OpenAI API Key（用于生成向量嵌入）

## 第一步：获取数据库连接字符串

1. 前往您的 Supabase 项目控制面板：https://supabase.com/dashboard/project/cppdscqtkzuiwqdiumdq

2. 点击左侧边栏的 **Settings** > **Database**

3. 滚动到 **Connection string**

4. 选择 **URI** 格式

5. 复制连接字符串。它看起来应该像这样：
   ```
   postgresql://postgres.[ref]:[YOUR-PASSWORD]@aws-0-us-west-1.pooler.supabase.com:6543/postgres
   ```

6. **重要提示**：将 `[YOUR-PASSWORD]` 替换为您真实的数据库密码。

## 第二步：启用 pgvector 扩展

1. 在 Supabase 控制面板中，前往 **Database** > **Extensions**

2. 搜索 "vector"

3. 在 `vector` 扩展上点击 **Enable**

4. 等待安装完成（通常是即时的）

## 第三步：配置本地环境

更新您的 `Database/.env` 文件：

```bash
# Supabase 连接串
DATABASE_URL=postgresql://postgres.[ref]:[YOUR-PASSWORD]@aws-0-us-west-1.pooler.supabase.com:6543/postgres

# OpenAI API key (已设置)
OPENAI_API_KEY=sk-proj-...

# 向量嵌入配置
EMBEDDING_MODEL=text-embedding-3-small
EMBEDDING_DIMENSION=1536

# 环境
ENVIRONMENT=production
```

## 第四步：初始化 Supabase 数据库

运行设置脚本：

```bash
cd Database
python -m scripts.setup_supabase
```

该脚本将：
- 连接到您的 Supabase 数据库
- 创建所有必需的表
- 设置 pgvector 索引
- 验证设置

## 第五步：为技能和文档建立索引

```bash
# 为所有内容生成嵌入并建立索引
python -m scripts.index --all
```

该脚本将：
- 为 `skills/` 目录下的所有 10 项技能建立索引
- 为 `docs/` 目录下的所有文档建立索引
- 生成用于语义搜索的向量嵌入
- 将文档链接到相应的技能

预期输出：
```
Indexed 10 skills
Indexed 7 documents
Total indexed: 17 items

Registry stats:
  Skills: 10 (10 with embeddings)
  Documents: 7 (7 with embeddings)
  Skill-Document links: 14
```

## 第六步：测试语义搜索

```bash
# 搜索技能
python -m scripts.search "how to design tools for AI agents" --threshold 0.5

# 搜索特定类型
python -m scripts.search "context window management" --type skills --threshold 0.4

# 获取 JSON 输出
python -m scripts.search "evaluation techniques" --json
```

## 同时使用本地和 Supabase

您可以通过更改 `DATABASE_URL` 在本地 Docker 数据库和 Supabase 之间切换：

**本地 Docker:**
```bash
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/agentskills
```

**Supabase:**
```bash
DATABASE_URL=postgresql://postgres.[ref]:[password]@aws-0-us-west-1.pooler.supabase.com:6543/postgres
```

## 验证设置

在 Supabase 控制面板中检查您的数据库：

1. 前往 **Table Editor**
2. 您应该能看到以下表格：`skills`, `documents`, `skill_sources`, `skill_versions`, `skill_references`
3. 点击 `skills` 表查看生成的索引数据

## 成本

- **Supabase**: 免费层包含 500MB 数据库，足够处理此用例。
- **OpenAI Embeddings**: 约 $0.10 / 1M tokens (索引 10 项技能大约只需 $0.01)。

## 故障排除

### 连接被拒绝 (Connection Refused)

- 检查 `DATABASE_URL` 中的密码是否正确。
- 确认您使用的是 **pooler** 连接字符串（端口 6543）。
- 检查 Supabase 控制面板，确认项目处于活跃状态。

### 未找到 pgvector 扩展

- 手动启用：Database > Extensions > vector > Enable
- 重新运行设置脚本。

### 嵌入错误 (Embedding Errors)

- 验证 `OPENAI_API_KEY` 是否设置正确。
- 检查您的账户是否有 API 额度。

### Schema 已存在

Schema 使用了 `IF NOT EXISTS`，因此可以安全地重新运行。如果您需要重置：

```sql
-- 在 Supabase SQL 编辑器中运行
DROP TABLE IF EXISTS skill_references CASCADE;
DROP TABLE IF EXISTS skill_versions CASCADE;
DROP TABLE IF EXISTS skill_sources CASCADE;
DROP TABLE IF EXISTS documents CASCADE;
DROP TABLE IF EXISTS skills CASCADE;
```

然后重新运行 `python -m scripts.setup_supabase`。

## 安全性

- **切勿提交** 您的 `.env` 文件（已在 `.gitignore` 中列出）。
- Supabase 的可公开 API Key 适用于 REST API，而非直接的数据库访问。
- 如果通过 Supabase API 暴露数据，请使用行级安全（RLS）策略。
- 对于此内部工具，直接数据库访问是安全的。
