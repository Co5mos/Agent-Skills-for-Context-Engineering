---
name: agent-skills-format
description: Agent Skills 格式官方文档 —— 一种轻量级、开放的标准，用于通过专业知识和工作流扩展 AI 助手的各方面能力。
doc_type: reference
source_url: No
---

# 概览

一种简单、开放的格式，赋予助手新的能力和专业知识。

Agent Skills（助手技能）是包含指令、脚本和资源的文件夹，助手可以发现并使用它们，从而更准确、高效地完成任务。

## 为什么需要 Agent Skills？
助手的能力日益增强，但往往缺乏可靠地进行实际工作所需的上下文。技能通过赋予助手加载程序性知识以及特定于公司、团队和用户的上下文（按需加载）来解决这个问题。拥有技能访问权限的助手可以根据正在处理的任务扩展其能力。

- **对于技能作者**：只需构建一次能力，即可部署到多个助手产品中。
- **对于兼容的助手**：支持技能让终端用户能够开箱即用地赋予助手新能力。
- **对于团队和企业**：以可移植、版本受控的包形式获取组织知识。

## Agent Skills 可以实现什么？
- **领域专业知识**：将专业知识打包成可重用的指令，从法律审查流程到数据分析流水线。
- **新能力**：赋予助手新能力（例如创建演示文稿、构建 MCP 服务器、分析数据集）。
- **可重复的工作流**：将多步任务转化为一致且可审计的工作流。
- **互操作性**：在不同的兼容技能的助手产品中重复使用相同的技能。

## 采用情况
Agent Skills 受到领先的 AI 开发工具的支持：
- OpenCode
- Cursor
- Amp
- Letta
- Goose
- GitHub
- VS Code
- Claude Code
- Claude
- OpenAI Codex

## 开放开发
Agent Skills 格式最初由 Anthropic 开发，作为开放标准发布，并已被越来越多的助手产品采用。该标准对来自更广泛生态系统的贡献持开放态度。

# 什么是技能？

Agent Skills 是一种轻量级、开放的格式，用于通过专业知识和工作流扩展 AI 助手的能力。

核心而言，一个技能是一个包含 `SKILL.md` 文件的文件夹。该文件包含元数据（至少包含名称和描述）以及告诉助手如何执行特定任务的指令。技能还可以捆绑脚本、模板和参考材料。

```
my-skill/
├── SKILL.md          # 必需：指令 + 元数据
├── scripts/          # 可选：可执行代码
├── references/       # 可选：文档
└── assets/           # 可选：模板、资源
```

## 技能如何运作
技能使用**渐进式披露**来高效管理上下文：
1. **发现**：启动时，助手仅加载每个可用技能的名称和描述，足以知道何时可能相关。
2. **激活**：当任务与技能描述匹配时，助手将完整的 `SKILL.md` 指令读取到上下文中。
3. **执行**：助手遵循指令，根据需要选择性地加载引用的文件或执行捆绑的代码。
这种方法在保持助手快速响应的同时，让他们能够按需访问更多上下文。

## SKILL.md 文件
每个技能都始于一个包含 YAML frontmatter 和 Markdown 指令的 `SKILL.md` 文件：

```yaml
---
name: pdf-processing
description: 从 PDF 文件中提取文本和表格，填写表单，合并文档。
---
```

# PDF 处理

## 何时使用此技能
当用户需要处理 PDF 文件时使用此技能...

## 如何提取文本
1. 使用 pdfplumber 进行文本提取...

## 如何填写表单
...

`SKILL.md` 顶部需要以下 frontmatter：
- `name`：短标识符
- `description`：何时使用此技能

Markdown 正文包含实际指令，对结构或内容没有特定限制。

这种简单的格式具有一些关键优势：
- **自描述性**：技能作者或用户可以阅读 `SKILL.md` 并了解其功能，使技能易于审计和改进。
- **可扩展性**：技能的复杂度可以从仅包含文本指令到包含可执行代码、资产和模板。
- **可移植性**：技能只是文件，因此易于编辑、版本化和共享。

## 后续步骤
- 查看规范以了解完整格式。
- 为您的助手添加技能支持，以构建兼容的客户端。
- 在 GitHub 上查看示例技能。
- 阅读编写有效技能的作者最佳实践。
- 使用参考库验证技能并生成提示词 XML。

# 规范

Agent Skills 的完整格式规范。

本文档定义了 Agent Skills 格式。

## 目录结构
一个技能是一个至少包含 `SKILL.md` 文件的目录：
```
skill-name/
└── SKILL.md          # 必需
```
您可以选择包含额外的目录，如 `scripts/`、`references/` 和 `assets/` 来支持您的技能。

## SKILL.md 格式
`SKILL.md` 文件必须包含 YAML frontmatter，后面跟着 Markdown 内容。

### Frontmatter (必需)
```yaml
---
name: skill-name
description: 描述此技能的功能以及何时使用它。
---
```
可选字段：
```yaml
---
name: pdf-processing
description: 从 PDF 文件中提取文本和表格，填写表单，合并文档。
license: Apache-2.0
metadata:
  author: example-org
  version: "1.0"
---
```

| 字段 | 必需 | 约束 |
| :--- | :--- | :--- |
| `name` | 是 | 最多 64 个字符。仅限小写字母、数字和连字符。不能以连字符开头或结尾。 |
| `description` | 是 | 最多 1024 个字符。非空。描述技能的功能以及何时使用它。 |
| `license` | 否 | 许可证名称或对捆绑许可证文件的引用。 |
| `compatibility` | 否 | 最多 500 个字符。指示环境要求（预期产品、系统包、网络访问等）。 |
| `metadata` | 否 | 附加元数据的任意键值映射。 |
| `allowed-tools` | 否 | 技能可能使用的预批准工具的空格分隔列表。（实验性） |

### name 字段
必需的 `name` 字段：
- 必须为 1-64 个字符
- 只能包含 unicode 小写字母数字字符和连字符（a-z 和 -）
- 不能以 `-` 开头或结尾
- 不能包含连续的连字符 (`--`)
- 必须与父目录名称匹配

有效示例：
- `name: pdf-processing`
- `name: data-analysis`
- `name: code-review`

无效示例：
- `name: PDF-Processing` # 不允许大写
- `name: -pdf` # 不能以连字符开头
- `name: pdf--processing` # 不允许连续连字符

### description 字段
必需的 `description` 字段：
- 必须为 1-1024 个字符
- 应同时描述技能的功能以及何时使用它
- 应包含帮助助手识别相关任务的特定关键词

最佳示例：
`description: Extracts text and tables from PDF files, fills PDF forms, and merges multiple PDFs. Use when working with PDF documents or when the user mentions PDFs, forms, or document extraction.`

糟糕示例：
`description: Helps with PDFs.`

### license 字段
可选的 `license` 字段：
- 指定应用于技能的许可证
- 我们建议保持简短（许可证名称或捆绑许可证文件的名称）

示例：
`license: Proprietary. LICENSE.txt has complete terms`

### compatibility 字段
可选的 `compatibility` 字段：
- 如果提供，必须为 1-500 个字符
- 仅当您的技能有特定的环境要求时才应包含
- 可以指示预期产品、所需的系统包、网络访问需求等

示例：
- `compatibility: Designed for Claude Code (or similar products)`
- `compatibility: Requires git, docker, jq, and access to the internet`
大多数技能不需要 `compatibility` 字段。

### metadata 字段
可选的 `metadata` 字段：
- 从字符串键到字符串值的映射
- 客户端可以使用它来存储 Agent Skills 规范未定义的附加属性
- 我们建议让您的键名保持合理的唯一性，以避免意外冲突

示例：
```yaml
metadata:
  author: example-org
  version: "1.0"
```

### allowed-tools 字段
可选的 `allowed-tools` 字段：
- 预先批准运行的工具的空格分隔列表
- 实验性。不同助手实现对此字段的支持可能有所不同。

示例：
`allowed-tools: Bash(git:*) Bash(jq:*) Read`

## 正文内容
frontmatter 之后的 Markdown 正文包含技能指令。没有格式限制。编写任何有助于助手有效执行任务的内容。

推荐章节：
- 逐步说明
- 输入和输出示例
- 常见边缘情况

请注意，一旦助手决定激活技能，它将加载整个文件。考虑将较长的 `SKILL.md` 内容拆分为引用的文件。

## 可选目录

### scripts/
包含助手可以运行的可执行代码。脚本应：
- 自包含或清晰记录依赖项
- 包含有用的错误消息
- 优雅地处理边缘情况
支持的语言取决于助手实现。常见选项包括 Python、Bash 和 JavaScript。

### references/
包含助手在需要时可以阅读的附加文档：
- `REFERENCE.md` - 详细的技术参考
- `FORMS.md` - 表单模板或结构化数据格式
- 领域特定文件（`finance.md`、`legal.md` 等）
保持单个参考文件聚焦。助手按需加载这些文件，因此较小的文件意味着占用更少的上下文。

### assets/
包含静态资源：
- 模板（文档模板、配置模板）
- 图像（图表、示例）
- 数据文件（查询表、Schema）

## 渐进式披露
技能应结构化以高效利用上下文：
- **元数据** (~100 tokens)：启动时为所有技能加载名称和描述字段
- **指令** (建议 < 5000 tokens)：激活技能时加载完整的 `SKILL.md` 正文
- **资源** (按需)：仅在需要时加载文件（例如 `scripts/`、`references/` 或 `assets/` 中的文件）
保持主 `SKILL.md` 在 500 行以内。将详细的参考资料移至单独的文件。

## 文件引用
在引用技能中的其他文件时，请使用相对于技能根目录的路径：
`See [the reference guide](references/REFERENCE.md) for details.`

运行提取脚本：
`scripts/extract.py`
保持文件引用距离 `SKILL.md` 一层深。避免深度嵌套的引用链。

## 验证
使用 `skills-ref` 参考库来验证您的技能：
`skills-ref validate ./my-skill`
这将检查您的 `SKILL.md` frontmatter 是否有效并遵循所有命名规范。

# 将技能集成到您的助手中

如何为您的助手或工具添加 Agent Skills 支持。

本指南说明了如何为 AI 助手或开发工具添加技能支持。

## 集成方法
集成技能的两种主要方法是：
1. **基于文件系统的助手**：在计算机环境（bash/unix）内操作，代表了能力最强的选项。当模型发出 shell 命令（如 `cat /path/to/my-skill/SKILL.md`）时，技能就会被激活。捆绑资源通过 shell 命令访问。
2. **基于工具的助手**：在没有专用计算机环境的情况下运行。相反，它们实现工具，允许模型触发技能并访问捆绑资产。具体的工具实现由开发人员决定。

## 概览
兼容技能的助手需要：
1. 在配置的目录中发现技能
2. 启动时加载元数据（名称和描述）
3. 将用户任务匹配到相关技能
4. 通过加载完整指令激活技能
5. 根据需要执行脚本并访问资源

## 技能发现
技能是包含 `SKILL.md` 文件的文件夹。您的助手应扫描配置的目录以查找有效技能。

## 加载元数据
启动时，仅解析每个 `SKILL.md` 文件的 frontmatter。这可以保持初始上下文使用量较低。

### 解析 frontmatter
```javascript
function parseMetadata(skillPath) {
    const content = readFile(skillPath + "/SKILL.md");
    const frontmatter = extractYAMLFrontmatter(content);

    return {
        name: frontmatter.name,
        description: frontmatter.description,
        path: skillPath
    };
}
```

## 注入上下文
在系统提示词中包含技能元数据，以便模型了解有哪些可用技能。
遵循您平台的系统提示词更新指南。例如，对于 Claude 模型，推荐格式使用 XML：

```xml
<available_skills>
  <skill>
    <name>pdf-processing</name>
    <description>Extracts text and tables from PDF files, fills forms, merges documents.</description>
    <location>/path/to/skills/pdf-processing/SKILL.md</location>
  </skill>
  <skill>
    <name>data-analysis</name>
    <description>Analyzes datasets, generates charts, and creates summary reports.</description>
    <location>/path/to/skills/data-analysis/SKILL.md</location>
  </skill>
</available_skills>
```
对于基于文件系统的助手，包含指向 `SKILL.md` 文件的绝对路径的 `location` 字段。对于基于工具的助手，可以省略 `location`。
保持元数据简明。每个技能应为上下文增加大约 50-100 个 token。

## 安全考量
脚本执行会引入安全风险。考虑：
- **沙盒化**：在隔离环境中运行脚本
- **白名单**：仅执行来自信任技能的脚本
- **确认**：在运行潜在危险操作前询问用户
- **日志记录**：记录所有脚本执行以供审计

## 参考实现
`skills-ref` 库提供了用于处理技能的 Python 工具和 CLI。
例如：
- 验证技能目录：`skills-ref validate <path>`
- 为助手提示词生成 `<available_skills>` XML：`skills-ref to-prompt <path>...`
将库源代码作为参考实现。

# 技能编写最佳实践

学习如何编写 Claude 能够成功发现并使用的有效技能。
好的技能应当简明、结构良好，并在实际使用中经过测试。本指南提供了实际的编写决策，帮助您编写 Claude 能够有效发现并使用的技能。

有关技能运作机制的概念背景，请参阅“技能概览”。

## 核心原则

### 简明是关键
上下文窗口是公共资源。您的技能与 Claude 需要知道的其他所有内容共享上下文窗口，包括：
- 系统提示词
- 对话历史
- 其他技能的元数据
- 您的实际请求

并非技能中的每个 token 都有即时成本。启动时，仅预加载所有技能的元数据（名称和描述）。Claude 只有在技能变得相关时才会读取 `SKILL.md`，并仅在需要时读取额外文件。然而，在 `SKILL.md` 中保持简明仍然很重要：一旦 Claude 加载了它，每个 token 都会与对话历史和其他上下文竞争。

### 默认假设：Claude 已经非常聪明
仅添加 Claude 尚未拥有的上下文。挑战每一条信息：
- “Claude 真的需要这个解释吗？”
- “我可以假设 Claude 知道这个吗？”
- “这一段话值得它的 token 成本吗？”

**佳例：简明（约 50 token）**：
```markdown
## 提取 PDF 文本
使用 pdfplumber 提取文本：
import pdfplumber
with pdfplumber.open("file.pdf") as pdf:
    text = pdf.pages[0].extract_text()
```

**坏例：太啰嗦（约 150 token）**：
```markdown
## 提取 PDF 文本
PDF (Portable Document Format) 格式是一种包含文本、图像和其他内容的常见文件格式。
要从 PDF 中提取文本，您需要使用一个库。PDF 处理有很多库，但我们推荐 pdfplumber，
因为它易于使用且能很好地处理大多数情况。首先，您需要使用 pip 安装它。
然后您可以使用下面的代码...
```
简明版本假设 Claude 知道什么是 PDF 以及库是如何运作的。

### 设定适当的自由度
将具体程度与任务的脆弱性和可变性相匹配。

**高自由度（基于文本的指令）**：
适用于：
- 多种方法都有效
- 决策取决于上下文
- 启发式方法引导流程
示例：
```markdown
## 代码审查流程
1. 分析代码结构和组织
2. 检查潜在的错误或边缘情况
3. 提出关于可读性和可维护性的改进建议
4. 验证是否符合项目规范
```

**中等自由度（带参数的伪代码或脚本）**：
适用于：
- 存在首选模式
- 一定程度的变化是可以接受的
- 配置会影响行为
示例：
```markdown
## 生成报告
使用此模板并根据需要进行自定义：
def generate_report(data, format="markdown", include_charts=True):
    # 处理数据
    # 生成指定格式的输出
    # 可选包含可视化
```

**低自由度（具体的脚本，很少或没有参数）**：
适用于：
- 操作脆弱且容易出错
- 一致性至关重要
- 必须遵循特定的顺序
示例：
```markdown
## 数据库迁移
确切运行此脚本：
python scripts/migrate.py --verify --backup
不要修改命令或添加额外的参数。
```

**比喻**：将 Claude 想象成一个在路径上探索的机器人：
- **两边都是悬崖的窄桥**：只有一条安全的道路。提供具体的护栏和确切的指令（低自由度）。例如：必须按确切顺序运行的数据库迁移。
- **没有危险的开阔场地**：条条大路通罗马。给出总体方向，相信 Claude 能找到最佳路线（高自由度）。例如：上下文决定最佳方法的代码审查。

### 测试所有你计划使用的模型
技能是模型的补充，因此效能取决于底层模型。使用所有你计划使用的模型来测试你的技能。

**不同模型的测试考量**：
- **Claude Haiku (快速、经济)**：技能是否提供了足够的引导？
- **Claude Sonnet (平衡)**：技能是否清晰且高效？
- **Claude Opus (强大的推理)**：技能是否避免了过度解释？

在 Opus 上完美运行的内容在 Haiku 上可能需要更多细节。如果你打算在多个模型上使用你的技能，目标是编写在所有模型上都表现良好的指令。

## 技能结构
YAML Frontmatter：`SKILL.md` 的 frontmatter 需要两个字段：

- **name**：
  - 最多 64 个字符
  - 只能包含小写字母、数字和连字符
  - 不能包含 XML 标签
  - 不能包含保留词：“anthropic”, “claude”
- **description**：
  - 不能为空
  - 最多 1024 个字符
  - 不能包含 XML 标签
  - 应当描述技能的功能以及何时使用它

有关完整的技能结构细节，请参阅“技能概览”。

## 命名规范
使用一致的命名模式使技能更易于引用和讨论。我们建议使用动名词形式（动词 + `-ing`）来命名技能，因为这能清晰地描述技能提供的活动或能力。

请记住，`name` 字段只能使用小写字母、数字和连字符。

**好的命名示例（动名词形式）**：
- `processing-pdfs`
- `analyzing-spreadsheets`
- `managing-databases`
- `testing-code`
- `writing-documentation`

**可接受的替代方案**：
- 名词短语：`pdf-processing`, `spreadsheet-analysis`
- 行动导向：`process-pdfs`, `analyze-spreadsheets`

**避免**：
- 模糊名称：`helper`, `utils`, `tools`
- 过于通用：`documents`, `data`, `files`
- 保留词：`anthropic-helper`, `claude-tools`
- 技能集合内模式不一致

一致的命名使得以下操作更容易：
- 在文档和对话中引用技能
- 一眼识破技能的功能
- 组织和搜索多个技能
- 维护专业、有凝聚力的技能库

## 编写有效的描述
描述字段实现了技能的发现，应当包含技能的功能以及何时使用它。

**始终使用第三人称。**描述会被注入系统提示词，不一致的人称视角会导致发现问题。
- 正确：“分析 Excel 文件并生成报告”
- 避免：“我可以帮你处理 Excel 文件”
- 避免：“你可以用它来处理 Excel 文件”

**具体并包含关键术语。**包含技能的功能以及特定的触发词/上下文。

每个技能正好有一个描述字段。描述对于技能选择至关重要：Claude 使用它从可能存在的 100 多个可用技能中选择正确的技能。您的描述必须提供足够的细节以便 Claude 决定何时选择此技能，而 `SKILL.md` 的其余部分则提供实现细节。

**有效的示例**：
- **PDF 处理技能**：`description: Extract text and tables from PDF files, fill forms, merge documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.`
- **Excel 分析技能**：`description: Analyze Excel spreadsheets, create pivot tables, generate charts. Use when analyzing Excel files, spreadsheets, tabular data, or .xlsx files.`
- **Git Commit 助手技能**：`description: Generate descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.`

**避免模糊的描述**：
- `description: Helps with documents`
- `description: Processes data`
- `description: Does stuff with files`

## 渐进式披露模式
`SKILL.md` 作为一个概览，在需要时引导 Claude 查阅详细材料，就像入职指南中的目录一样。有关渐进式披露的工作原理，请参阅概览中的“技能如何运作”。

**实际指导**：
- 保持 `SKILL.md` 正文在 500 行以内以获得最佳性能
- 在接近此限制时将内容拆分为独立文件
- 使用下面的模式有效地组织指令、代码和资源

**视觉概览：从简单到复杂**
一个基础技能仅包含一个 `SKILL.md` 文件：
[此处通常有一个 SKILL.md 的视觉表示]

随着技能的增长，您可以捆绑 Claude 仅在需要时加载的额外内容：
[此处通常有一个捆绑额外参考文件的视觉表示]

完整的技能目录结构可能如下所示：
```
pdf/
├── SKILL.md              # 主指令 (触发后加载)
├── FORMS.md              # 表单填写指南 (按需加载)
├── reference.md          # API 参考 (按需加载)
├── examples.md           # 使用示例 (按需加载)
└── scripts/
    ├── analyze_form.py   # 工具脚本 (执行，不加载)
    ├── fill_form.py      # 表单填写脚本
    └── validate.py       # 验证脚本
```

### 模式 1：带参考的高级指南
```yaml
---
name: pdf-processing
description: Extracts text and tables from PDF files, fills forms, and merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
# PDF 处理

## 快速上手
使用 pdfplumber 提取文本...
## 高级功能
表单填写：参见 [FORMS.md](FORMS.md) 了解完整指南
API 参考：参见 [REFERENCE.md](REFERENCE.md) 了解所有方法
示例：参见 [EXAMPLES.md](EXAMPLES.md) 了解常见模式
```
Claude 只有在需要时才会加载 `FORMS.md`、`REFERENCE.md` 或 `EXAMPLES.md`。

### 模式 2：特定领域的组织
对于具有多个领域的技能，按领域组织内容以避免加载无关的上下文。当用户询问销售指标时，Claude 只需要读取与销售相关的 Schema，而不是财务或营销数据。

```
bigquery-skill/
├── SKILL.md (概览与导航)
└── reference/
    ├── finance.md (收入、计费指标)
    ├── sales.md (机会、流水线)
    ├── product.md (API 使用、功能)
    └── marketing.md (活动、归因)
```

### 模式 3：条件性细节
显示基础内容，链接到高级内容：
```markdown
# DOCX 处理
## 创建文档
使用 docx-js 创建新文档。参见 [DOCX-JS.md](DOCX-JS.md)。
## 编辑文档
对于简单修改，直接修改 XML。
**对于修订追踪**：参见 [REDLINING.md](REDLINING.md)
**对于 OOXML 细节**：参见 [OOXML.md](OOXML.md)
```
Claude 只有在用户需要这些功能时才会读取 `REDLINING.md` 或 `OOXML.md`。

### 避免深度嵌套的引用
当文件从其他引用文件中再次引用时，Claude 可能会部分读取文件。当遇到嵌套引用时，Claude 可能会使用 `head -100` 之类的命令来预览内容而不是读取整个文件，导致信息不完整。

保持引用距离 `SKILL.md` 只有一层深。所有参考文件应当直接从 `SKILL.md` 链接，以确保 Claude 在需要时能读取完整文件。

**坏例：太深**：
`SKILL.md` -> `advanced.md` -> `details.md`

**佳例：一层深**：
`SKILL.md` -> `advanced.md`
`SKILL.md` -> `reference.md`
`SKILL.md` -> `examples.md`

### 为较长的参考文件构建目录
对于超过 100 行的参考文件，请在顶部包含目录。这确保了 Claude 即使在部分预览时也能看到可用信息的全貌。
示例：
```markdown
# API 参考
## 目录
- 身份验证与设置
- 核心方法
- 高级功能
- 错误处理模式
- 代码示例
```
Claude 随后可以读取完整文件或根据需要跳至特定章节。

有关这种基于文件系统的架构如何实现渐进式披露的细节，请参阅下面高级章节中的“运行时环境”。


选择一个术语并在整个技能中始终使用它：
- **一致（好）**：始终使用“API endpoint”、“field”、“extract”
- **不一致（不好）**：混合使用“API endpoint”、“URL”、“API route”、“path”；混合使用“field”、“box”、“element”、“control”；混合使用“extract”、“pull”、“get”、“retrieve”
一致性有助于 Claude 理解并遵循指令。

## 常见模式

### 模板模式
为输出格式提供模板。根据您的需求调整严格程度。

对于严格要求（如 API 响应或数据格式）：
```markdown
## 报告结构
始终使用此确切的模板结构：
# [分析标题]
## 执行摘要
[关键发现的一段概览]
## 关键发现
- 带有支持数据的发现 1
- 带有支持数据的发现 2
## 建议
1. 具体的、可操作的建议
```

对于灵活引导：
```markdown
## 报告结构
这是一个合理的默认格式，但请根据分析情况自行判断：
# [分析标题]
## 执行摘要
[概览]
## 关键发现
[根据您的发现调整章节]
## 建议
[针对特定场景进行定制]
```

### 示例模式
对于输出质量依赖于参考范例的技能，提供像常规提示词一样的输入/输出对：

```markdown
## Commit 消息格式
参考以下示例生成 commit 消息：
**示例 1：**
输入：Added user authentication with JWT tokens
输出：feat(auth): implement JWT-based authentication
**示例 2：**
输入：Fixed bug where dates displayed incorrectly in reports
输出：fix(reports): correct date formatting in timezone conversion
```

### 条件工作流模式
引导 Claude 经过决策点：

```markdown
## 文档修改工作流
1. 确定修改类型：
   **创建新内容？** → 遵循下方的“创建工作流”
   **编辑现有内容？** → 遵循下方的“编辑工作流”
2. 创建工作流：...
3. 编辑工作流：...
```
如果工作流包含许多步骤且变得庞大复杂，请考虑将其放入独立文件，并告诉 Claude 根据当前任务阅读相应文件。

## 评估与迭代

### 先构建评估
在编写大量文档之前创建评分标准。这确保了您的技能是在解决实际问题，而不是在记录想象中的问题。

**评估驱动开发**：
1. **识别差距**：在没有技能的情况下让 Claude 运行代表性任务。记录具体的失败或缺失的上下文。
2. **创建评估**：构建三个测试这些差距的场景。
3. **建立基准**：衡量 Claude 在没有技能时的表现。
4. **编写最简指令**：创建刚好够填补差距并通过评估的内容。
5. **迭代**：执行评估，与基准对比，并不断完善。

### 与 Claude 协同开发技能
最有效的技能开发过程涉及 Claude 本身。使用一个 Claude 实例（“Claude A”）来创建一个由其他实例（“Claude B”）使用的技能。Claude A 帮助您设计和完善指令，而 Claude B 在实际任务中测试它们。

1. **在没有技能的情况下完成一项任务**：通过普通提示词让 Claude A 解决一个问题。在这个过程中，你会自然地提供上下文、偏好和程序性知识。记录你重复提供的信息。
2. **识别可重用的模式**：任务完成后，识别哪些上下文对未来的类似任务有用。
3. **让 Claude A 创建技能**：“创建一个技能，捕捉我们刚才使用的 BigQuery 分析模式。包含表 Schema、命名规范和过滤测试账号的规则。”
4. **审查简明性**：检查 Claude A 是否添加了不必要的解释。
5. **改进信息架构**：让 Claude A 更有效地组织内容。例如：“将表 Schema 放在单独的参考文件中。”
6. **在类似任务上测试**：将技能交给全新的 Claude B 实例。观察它是否能找到正确信息并成功处理任务。
7. **根据观察结果迭代**：如果 Claude B 遇到困难，带上细节回到 Claude A 进行改进。

### 关于团队反馈
与队友分享技能并观察他们的使用情况。询问：技能是否按预期激活？指令是否清晰？缺少了什么？

## 观察 Claude 如何导航技能
在迭代技能时，关注 Claude 在实践中是如何使用它们的：
- **意外的探索路径**：Claude 是否按你没预料到的顺序读取文件？这可能意味着你的结构不够直观。
- **缺失的连接**：Claude 是否未能跟随重要文件的引用？你的链接可能需要更显眼。
- **过度依赖某些章节**：如果 Claude 反复读取同一个文件，考虑该内容是否应放在主 `SKILL.md` 中。
- **被忽略的内容**：如果 Claude 从不访问捆绑文件，它可能是多余的或信号不够强。

## 应避免的反模式

### 避免 Windows 风格路径
始终在文件路径中使用正斜杠，即使在 Windows 上：
- ✓ **推荐**：`scripts/helper.py`, `reference/guide.md`
- ✗ **不推荐**：`scripts\helper.py`, `reference\guide.md`
Unix 风格路径适用于所有平台。

### 避免提供过多选项
除非必要，否则不要展示多种方法：
- **坏例**：提供 5 个不同的库供选择。
- **佳例**：提供一个默认方案，并附带备选方案。

# 高级：包含可执行代码的技能

## 解决问题，不要推卸
在为技能编写脚本时，要处理错误情况，而不是推卸给 Claude。

**佳例：明确处理错误**：
```python
def process_file(path):
    try:
        with open(path) as f:
            return f.read()
    except FileNotFoundError:
        # 创建文件而不是报错
        print(f"File {path} not found, creating default")
        return ''
```

### 提供工具脚本
即使 Claude 能够编写脚本，预制脚本也有其优势：
- 比生成的代码更可靠
- 节省 token（无需将代码包含在上下文中）
- 节省时间
- 确保一致性

在指令中明确告知 Claude 应当**执行**脚本（最常见）还是将其作为参考**阅读**。

### 使用视觉分析
当输入可以渲染为图像时，让 Claude 分析它们。Claude 的视觉能力有助于理解布局和结构。

### 创建可验证的中间输出
当执行复杂、开放式的任务时，使用“计划-验证-执行”模式。让 Claude 先以结构化格式创建一个计划，然后在执行前用脚本验证该计划。

## 软件包依赖
技能在具有平台限制的代码执行环境中运行。在 `SKILL.md` 中列出所需的软件包，并验证它们在执行工具中是否可用。

# 运行时环境

## 这如何影响您的编写：

1. **元数据预加载**：启动时，所有技能的 YAML frontmatter（名称和描述）都会加载到系统提示词中。
2. **文件按需读取**：Claude 使用 bash 读取工具在需要时访问 `SKILL.md` 和其他文件。
3. **脚本高效执行**：工具脚本可以通过 bash 执行，只有脚本输出会消耗 token。
4. **大文件没有上下文惩罚**：参考文件、数据或文档只有在实际读取时才消耗 token。
5. **名字要有描述性**：使用指示内容的名称：`form_validation_rules.md` 而非 `doc2.md`。
6. **为发现而组织**：按领域或功能结构化目录。
7. **捆绑综合资源**：可以包含完整的 API 文档、大型数据集；在访问前没有上下文惩罚。

## MCP 工具引用
如果您的技能使用 MCP (Model Context Protocol) 工具，始终使用完全限定的工具名称。
格式：`ServerName:tool_name`
例如：`BigQuery:bigquery_schema`

# 编写有效技能的检查清单

在分享技能前，请核实：

### 核心质量
- [ ] 描述具体且包含关键术语
- [ ] 描述包含功能和使用场景
- [ ] `SKILL.md` 正文在 500 行以内
- [ ] 额外细节在独立文件中
- [ ] 无时效性信息
- [ ] 术语始终一致
- [ ] 示例具体而非抽象
- [ ] 文件引用距离一层深

### 代码与脚本
- [ ] 脚本直接解决问题
- [ ] 错误处理明确有用
- [ ] 无盲目设置的常量
- [ ] 列出了所需的软件包
- [ ] 脚本有清晰文档
- [ ] 统一使用正斜杠路径
- [ ] 关键操作有验证步骤

### 测试
- [ ] 至少创建了三个评估
- [ ] 使用多个模型进行了测试
- [ ] 经过了实际使用场景测试

https://github.com/anthropics/skills

