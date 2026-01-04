# PRD: X-to-Book 多智能体系统

## 概览 (Overview)

一个多智能体（Multi-agent）系统，每天监控目标 X（Twitter）账号，综合其内容，并根据积累的见解生成结构化的书籍。该系统使用上下文工程（Context Engineering）原则来处理海量的社交数据，同时保持长篇输出的连贯性。

## 问题陈述 (Problem Statement)

手动从 X 账号中整理见解耗时且不连贯。现有工具只是在没有综合的情况下堆砌原始数据。我们需要一个能够实现以下功能的系统：
- 持续监控指定的 X 账号
- 跨时间提取有意义的模式和见解
- 产出结构化、连贯的每日书籍
- 保持对叙事演变的跨时间意识

## 架构 (Architecture)

### 多智能体模式选择：主管/协调者 (Supervisor/Orchestrator)

基于上下文工程模式，我们采用 **主管架构**，理由如下：
1. 书籍制作有明显的顺序阶段（抓取、分析、综合、撰写、编辑）
2. 质量关卡需要中央协调
3. 人工监督点定义明确
4. 每个阶段的上下文隔离可防止注意力饱和

```
用户配置 -> 协调者 -> [抓取器, 分析器, 综合器, 撰写器, 编辑器] -> 每日书籍
```

### 智能体定义

#### 1. 协调者智能体 (Orchestrator Agent)
**用途**：管理工作流、维护状态、路由到专家的中央协调者。

**上下文预算**：预留用于任务拆解、质量关卡和综合协调。不携带原始推文数据。

**职责**：
- 将每日写书任务拆解为子任务
- 路由到相应的专家智能体
- 为长时间运行的操作实现检查点/恢复机制
- 在不转述的情况下聚合结果（避免信息传递失真问题）

```python
class OrchestratorState(TypedDict):
    target_accounts: List[str]
    current_phase: str
    phase_outputs: Dict[str, Any]
    quality_scores: Dict[str, float]
    book_outline: str
    checkpoints: List[Dict]
```

#### 2. 抓取器智能体 (Scraper Agent)
**用途**：从目标 X 账号获取并标准化内容。

**上下文预算**：极小。一次操作一个账号，输出到文件系统。

**工具**：
- `fetch_timeline(account_id, since_date, until_date)` - 获取日期范围内的推文
- `fetch_thread(tweet_id)` - 展开完整的推文线程上下文
- `fetch_engagement_metrics(tweet_ids)` - 获取点赞/转发/回复等指标
- `write_to_store(account_id, data)` - 持久化到文件系统

**输出**：每个账号的结构化 JSON，写入文件系统（不通过上下文传递）。

#### 3. 分析器智能体 (Analyzer Agent)
**用途**：从原始内容中提取模式、主题和见解。

**上下文预算**：中等。通过读取文件系统，一次处理一个账号的数据。

**职责**：
- 主题提取和聚类
- 随时间变化的情感分析
- 关键见解识别
- 线程叙事提取
- 争议/辩论识别

**输出**：每个账号的结构化分析，包含：
- 热门主题（按频率和互动度排序）
- 著名引用（带上下文）
- 叙述弧线（多推文线程）
- 时间模式（发布时间、响应模式）

#### 4. 综合器智能体 (Synthesizer Agent)
**用途**：跨账号的模式识别和主题整合。

**上下文预算**：高。接收来自所有分析账号的摘要。

**职责**：
- 识别跨账号主题
- 检测共识/分歧模式
- 构建叙事连接
- 生成带章节结构的书籍大纲

**输出**：书籍大纲，包含：
- 章节结构
- 每个章节的主题分配
- 来源归属图谱
- 建议的叙述流

#### 5. 撰写器智能体 (Writer Agent)
**用途**：根据大纲和原始资料生成书籍内容。

**上下文预算**：按章节分配。一次撰写一个章节。

**职责**：
- 遵循大纲起草章节内容
- 整合引用并进行正确的归属说明
- 保持一致的语气和风格
- 处理主题之间的过渡

**输出**：Markdown 格式的章节草案。

#### 6. 编辑器智能体 (Editor Agent)
**用途**：质量保证和润色。

**上下文预算**：按章节。一次审核一个章节。

**职责**：
- 根据原始资料进行事实核查
- 验证引用的准确性
- 检查叙事连贯性
- 为人工审计标记潜在问题

**输出**：带有修改说明的已编辑章节。

---

## 记忆系统设计 (Memory System Design)

### 架构：时态知识图谱 (Temporal Knowledge Graph)

基于 memory-systems 技能，我们需要一个 **时态知识图谱**，原因如下：
- 关于账号的事实会随时间变化（观点转变、主题演变）
- 我们需要时空旅行查询（“@account 在 1 月份对 X 的立场是什么？”）
- 跨账号关系需要图遍历
- 简单的向量库会丢失关系结构

### 实体类型 (Entity Types)

```python
entities = {
    "Account": {
        "properties": ["handle", "display_name", "bio", "follower_count", "following_count"]
    },
    "Tweet": {
        "properties": ["content", "timestamp", "engagement_score", "thread_id"]
    },
    "Theme": {
        "properties": ["name", "description", "first_seen", "last_seen"]
    },
    "Book": {
        "properties": ["date", "title", "chapter_count", "word_count"]
    },
    "Chapter": {
        "properties": ["title", "theme", "word_count", "source_accounts"]
    }
}
```

### 关系类型 (Relationship Types)

```python
relationships = {
    "POSTED": {
        "from": "Account",
        "to": "Tweet",
        "temporal": True
    },
    "DISCUSSES": {
        "from": "Tweet",
        "to": "Theme",
        "temporal": True,
        "properties": ["sentiment", "stance"]
    },
    "RESPONDS_TO": {
        "from": "Tweet",
        "to": "Tweet"
    },
    "AGREES_WITH": {
        "from": "Account",
        "to": "Account",
        "temporal": True,
        "properties": ["on_theme"]
    },
    "DISAGREES_WITH": {
        "from": "Account",
        "to": "Account",
        "temporal": True,
        "properties": ["on_theme"]
    },
    "CONTAINS": {
        "from": "Book",
        "to": "Chapter"
    },
    "SOURCES": {
        "from": "Chapter",
        "to": "Tweet"
    }
}
```

### 记忆检索模式

```python
# @account 在过去 30 天内说了哪些关于 AI 的内容？
query_account_theme_temporal(account_id, theme="AI", days=30)

# 哪些账号在加密货币上存在分歧？
query_disagreement_network(theme="crypto")

# 今天的书中关于监管的内容应该包含哪些引用？
query_quotable_content(theme="regulation", min_engagement=100)
```

---

## 上下文优化策略 (Context Optimization Strategy)

### 挑战
X 的数据量非常大。目标账号每天发布 20 条推文，跨 10 个账号 = 每天 200 条推文。每条推文配合线程上下文平均 500 个 token。分析前的每日原始上下文 = 10 万个 token。

### 优化技术

#### 1. 观察掩码 (Observation Masking)
原始推文数据由抓取器处理并写入文件系统，绝不通过协调者的上下文传递。

```python
# 不在上下文中传递原始推文
# 抓取器写入文件系统
scraper.write_to_store(account_id, raw_tweets)

# 分析器从文件系统读取
raw_data = analyzer.read_from_store(account_id)
```

#### 2. 压缩触发器 (Compaction Triggers)

```python
COMPACTION_THRESHOLD = 0.7  # 70% 上下文利用率

if context_utilization > COMPACTION_THRESHOLD:
    # 总结旧阶段的输出
    phase_outputs = compact_phase_outputs(phase_outputs)
```

#### 3. 渐进式披露 (Progressive Disclosure)
书籍大纲首先加载（轻量级）。只有当撰写器处理该章节时，才加载完整的章节内容。

```python
# 第 1 层：仅大纲
book_outline = {
    "chapters": [
        {"title": "Chapter 1", "themes": ["AI", "Regulation"], "word_count_target": 2000}
    ]
}

# 第 2 层：完整的章节上下文（仅在撰写时）
chapter_context = load_chapter_context(chapter_id)
```

#### 4. KV-Cache 优化
系统提示词和工具定义在运行期间保持稳定。为缓存命中结构化上下文：

```python
context_order = [
    system_prompt,       # 稳定，可缓存
    tool_definitions,    # 稳定，可缓存
    account_config,      # 半稳定
    daily_outline,       # 每天变化
    current_task         # 每次调用变化
]
```

---

## 工具设计 (Tool Design)

### 整合原则的应用
我们不使用多个狭窄的工具，而是为每个领域实现综合工具：

#### X 数据工具（已整合）

```python
def x_data_tool(
    action: Literal["fetch_timeline", "fetch_thread", "fetch_engagement", "search"],
    account_id: Optional[str] = None,
    tweet_id: Optional[str] = None,
    query: Optional[str] = None,
    since_date: Optional[str] = None,
    until_date: Optional[str] = None,
    format: Literal["concise", "detailed"] = "concise"
) -> Dict:
    """
    统一的 X 数据获取工具。
    
    使用场景：
    - 获取时间轴以监控目标账号
    - 展开线程上下文以获取完整对话
    - 获取互动指标以进行内容优先级排序
    - 在账号间搜索特定主题
    
    操作：
    - fetch_timeline: 在日期范围内获取账号推文
    - fetch_thread: 从单条推文展开完整线程
    - fetch_engagement: 获取点赞/转发/回复
    - search: 跨账号搜索查询
    
    返回：
    - concise: tweet_id, content_preview, timestamp, engagement_score
    - detailed: 完整内容、线程上下文、所有互动指标、回复预览
    
    错误：
    - RATE_LIMITED: 请在 {retry_after} 秒后重试
    - ACCOUNT_PRIVATE: 无法访问私密账号
    - NOT_FOUND: 推文/账号不存在
    """
```

#### 记忆工具（已整合）

```python
def memory_tool(
    action: Literal["store", "query", "update_validity", "consolidate"],
    entity_type: Optional[str] = None,
    entity_id: Optional[str] = None,
    relationship_type: Optional[str] = None,
    query_params: Optional[Dict] = None,
    as_of_date: Optional[str] = None
) -> Dict:
    """
    统一的记忆系统工具。
    
    使用场景：
    - 存储从 X 数据中发现的新事实
    - 查询关于账号/主题的历史信息
    - 当事实发生变化时更新有效期
    - 运行整合以合并重复事实
    
    操作：
    - store: 添加新实体或关系
    - query: 检索匹配参数的实体/关系
    - update_validity: 用 valid_until 标记事实已过期
    - consolidate: 合并重复项并清理
    
    返回实体/关系数据或查询结果。
    """
```

#### 撰写工具（已整合）

```python
def writing_tool(
    action: Literal["draft", "edit", "format", "export"],
    content: Optional[str] = None,
    chapter_id: Optional[str] = None,
    style_guide: Optional[str] = None,
    output_format: Literal["markdown", "html", "pdf"] = "markdown"
) -> Dict:
    """
    统一的书籍撰写工具。
    
    使用场景：
    - 起草新章节内容
    - 为了质量编辑现有内容
    - 为输出格式化内容
    - 导出最终的书籍
    
    操作：
    - draft: 创建初始章节草案
    - edit: 对现有内容应用修订
    - format: 应用样式和格式化
    - export: 生成最终输出文件
    """
```

---

## 评估框架 (Evaluation Framework)

### 多维评分标准 (Multi-Dimensional Rubric)

基于评估技能，我们定义了质量维度：

| 维度 | 权重 | 优秀 | 合格 | 失败 |
|-----------|--------|-----------|------------|--------|
| 来源准确性 | 30% | 所有引用均已验证，归属说明正确 | 存在细微归属错误 | 伪造引用 |
| 主题连贯性 | 25% | 叙事脉络清晰，逻辑流畅 | 存在部分脱节的章节 | 无连贯叙事 |
| 完整性 | 20% | 涵盖了来源中的所有主要主题 | 遗漏了部分主题 | 存在重大缺漏 |
| 见解质量 | 15% | 跨来源的新颖综合 | 重申显而易见的观点 | 无综合见解 |
| 可读性 | 10% | 引人入胜、结构良好的散文 | 合格但枯燥 | 无法阅读 |

### 自动化评估流水线

```python
def evaluate_daily_book(book: Book, source_data: Dict) -> EvaluationResult:
    scores = {}
    
    # 来源准确性：根据原始推文验证引用
    scores["source_accuracy"] = verify_quotes(book.chapters, source_data)
    
    # 主题连贯性：LLM 即评委 (LLM-as-judge) 评估叙事流
    scores["thematic_coherence"] = judge_coherence(book)
    
    # 完整性：检查主题覆盖率
    scores["completeness"] = calculate_theme_coverage(book, source_data)
    
    # 见解质量：LLM 即评委评估综合能力
    scores["insight_quality"] = judge_insights(book, source_data)
    
    # 可读性：自动化指标 + LLM 评委
    scores["readability"] = assess_readability(book)
    
    overall = weighted_average(scores, DIMENSION_WEIGHTS)
    
    return EvaluationResult(
        passed=overall >= 0.7,
        scores=scores,
        overall=overall,
        flagged_issues=identify_issues(scores)
    )
```

### 人工审核触发器

- 综合评分 < 0.7
- 来源准确性 < 0.8
- 检测到任何伪造引用
- 添加了新账号（第一本书需要审核）
- 检测到争议性话题

---

## 数据流 (Data Flow)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                               每日流水线 (DAILY PIPELINE)                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 1. 抓取阶段 (SCRAPE PHASE)                                                   │
│    抓取智能体 → X API → 文件系统 (raw_data/{account}/{date}.json)            │
│    上下文：极小（仅限工具调用）                                              │
│    输出：持久化到文件系统的原始推文数据                                      │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 2. 分析阶段 (ANALYZE PHASE)                                                  │
│    分析智能体 → 文件系统 → 记忆库                                            │
│    上下文：一次处理一个账号                                                  │
│    输出：每个账号的结构化分析 + 知识图谱更新                                 │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 3. 综合阶段 (SYNTHESIZE PHASE)                                               │
│    综合智能体 → 分析摘要 → 书籍大纲                                          │
│    上下文：来自所有账号的摘要（已压缩）                                      │
│    输出：带有章节结构的书籍大纲                                              │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 4. 撰写阶段 (WRITE PHASE)                                                    │
│    撰写智能体 → 大纲 + 相关来源 → 章节草案                                   │
│    上下文：一次写一个章节（渐进式披露）                                      │
│    输出：Markdown 格式的章节草案                                             │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 5. 编辑阶段 (EDIT PHASE)                                                     │
│    编辑智能体 → 草案 + 来源 → 最终章节                                       │
│    上下文：一次审核一个章节                                                  │
│    输出：带有修订说明的已编辑章节                                            │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 6. 评估阶段 (EVALUATE PHASE)                                                 │
│    评估流水线 → 最终书稿 → 质量报告                                          │
│    输出：带评分的通过/失败结果，以及标记的问题                               │
└─────────────────────────────────────────────────────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│ 7. 发布（若通过）或 人工审核（若标记问题）                                   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 失败模式与缓解措施 (Failure Modes and Mitigations)

### 失败：协调者上下文饱和
**症状**：协调者积累了各阶段输出，导致路由决策质量下降。
**缓解**：阶段输出存储在文件系统中，协调者仅接收摘要。实现检查点以持久化状态。

### 失败：X API 速率限制
**症状**：抓取器触发布速率限制，导致数据不完整。
**缓解**：
- 实现带有指数退避的断路器
- 为续传记录部分抓取的检查点
- 在不同时间窗口调度抓取任务

### 失败：引用幻觉
**症状**：撰写器生成了原始资料中不存在的引用。
**缓解**：
- 在撰写提示词中要求严格的来源归属
- 编辑智能体根据来源验证所有引用
- 在评估中加入自动化引用验证

### 失败：主题偏移
**症状**：书籍主题偏离了真实的来源内容。
**缓解**：
- 综合器仅接收有据可查的摘要
- 撰写工具包含来源验证步骤
- 评估环节检查主题与来源的一致性

### 失败：协调开销过大
**症状**：智能体通信延迟超过了内容价值。
**缓解**：
- 批量处理阶段输出
- 使用文件系统处理智能体间数据（大负载不通过上下文传递）
- 尽可能并行化（抓取器可以按账号并行运行）

---

## 配置 (Configuration)

```yaml
# config.yaml
target_accounts:
  - handle: "@account1"
    priority: high
    themes_of_interest: ["AI", "startups"]
  - handle: "@account2"
    priority: medium
    themes_of_interest: ["regulation", "policy"]

schedule:
  scrape_time: "06:00"  # UTC
  publish_time: "08:00"
  timezone: "UTC"

book_settings:
  target_word_count: 5000
  min_chapters: 3
  max_chapters: 7
  style: "analytical"  # analytical | narrative | summary

quality_thresholds:
  min_overall_score: 0.7
  min_source_accuracy: 0.8
  require_human_review_below: 0.75

memory:
  retention_days: 90
  consolidation_frequency: "weekly"
  
context_limits:
  orchestrator: 50000
  scraper: 20000
  analyzer: 80000
  synthesizer: 100000
  writer: 80000
  editor: 60000
```

---

## 实现阶段 (Implementation Phases)

### 第一阶段：核心流水线（第 1-2 周）
- 具备基础路由能力的协调者
- 具备 X API 集成的抓取器
- 文件系统存储
- 产出 Markdown 输出的基础撰写器

### 第二阶段：分析层（第 3-4 周）
- 具备主题提取能力的分析智能体
- 具备跨账号模式识别能力的综合器
- 书籍大纲生成

### 第三阶段：记忆系统（第 5-6 周）
- 时态知识图谱实现
- 实体与关系存储
- 用于历史背景查询的时态查询

### 第四阶段：质量层（第 7-8 周）
- 编辑智能体
- 评估流水线
- 人工审核界面

### 第五阶段：生产环境加固（第 9-10 周）
- 检查点/恢复
- 断路器
- 监控与告警
- 整合任务

---

## 技术栈（推荐）

| 组件 | 技术 | 理由 |
|-----------|------------|-----------|
| 智能体框架 | LangGraph | 带有明确节点/边的基于图的状态机 |
| 知识图谱 | Neo4j 或 Memgraph | 原生时态查询，关系遍历 |
| 向量库 | Weaviate 或 Pinecone | 混合搜索（语义 + 元数据过滤） |
| X API | 官方 API 或 爬虫兜底 | 速率限制需要精心管理 |
| 存储 | PostgreSQL + S3 | 结构化数据 + 内容对象存储 |
| 编排 | Temporal.io | 带有检查点/恢复的持久化工作流 |

---

## 开放性问题

1. **X API 访问**：官方 API 还是爬虫？官方 API 的速率限制很严格。爬虫存在法律/服务条款方面的考量。
2. **书籍格式**：纯文字还是混合媒介（包括原始推文嵌入）？
3. **归属模型**：账号归属说明应该多显著？带有 ID 的完整引用还是经过转述的见解？
4. **变现**：如果书籍用于出售，综合公开推文在知识产权方面有哪些影响？
5. **人机协作 (Human-in-the-Loop)**：需要多少编辑控制？全文审核还是基于异常的审核？

---

## 参考资料

- [Agent Skills for Context Engineering](https://github.com/muratcankoylan/Agent-Skills-for-Context-Engineering) - 上下文工程模式
- 多智能体模式技能 - 主管架构选择
- 记忆系统技能 - 时态知识图谱设计
- 上下文优化技能 - 观察掩码与压缩策略
- 工具设计技能 - 工具整合原则
- 评估技能 - 多维评分标准
