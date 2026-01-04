# 架构缩减：生产实践证据

本文档提供了针对智能体工具设计的架构缩减（Architectural Reduction）方法的详细证据和实现模式。

## 案例研究：Text-to-SQL 智能体

一个生产环境下的 Text-to-SQL 智能体基于架构缩减原则进行了重构。原始架构使用了大量的专用工具、繁重的提示词工程和精细的上下文管理。缩减后的架构仅使用了一个单一的 Bash 命令执行工具。

### 原始架构（大量专用工具）

原始系统包含：
- GetEntityJoins：查找实体间的关系
- LoadCatalog：加载数据目录信息
- RecallContext：检索之前的上下文
- LoadEntityDetails：获取实体规范
- SearchCatalog：搜索数据目录
- ClarifyIntent：澄清用户意图
- SearchSchema：搜索数据库架构
- GenerateAnalysisPlan：创建查询计划
- FinalizeQueryPlan：完成查询计划
- FinalizeNoData：处理无数据情况
- JoinPathFinder：查找连接路径
- SyntaxValidator：验证 SQL 语法
- FinalizeBuild：完成查询构建
- ExecuteSQL：执行 SQL 查询
- FormatResults：格式化查询结果
- VisualizeData：创建可视化
- ExplainResults：解释查询结果

每个工具都是为了解决团队预期模型会遇到的特定问题而设计的。当时的假设是模型会在复杂的架构中迷失、做出错误的连接或幻觉表名。

### 缩减后的架构（两个基础工具）

缩减后的系统包含：
- ExecuteCommand：在沙盒中运行任意 Bash 命令
- ExecuteSQL：对数据库运行 SQL 查询

智能体使用标准的 Unix 工具探索语义层：

```python
from vercel_sandbox import Sandbox

sandbox = Sandbox.create()
await sandbox.write_files(semantic_layer_files)

def execute_command(command: str):
    """在沙盒中执行任意 Bash 命令。"""
    result = sandbox.exec(command)
    return {
        "stdout": result.stdout,
        "stderr": result.stderr,
        "exit_code": result.exit_code
    }
```

智能体现在使用 `grep`、`cat`、`find` 和 `ls` 来浏览包含维度定义、度量计算和连接关系的 YAML、Markdown 和 JSON 文件。

### 结果对比

| 指标 | 原始（17 个工具） | 缩减（2 个工具） | 变化 |
|--------|---------------------|-------------------|--------|
| 平均执行时间 | 274.8s | 77.4s | 快 3.5 倍 |
| 成功率 | 80% (4/5) | 100% (5/5) | +20% |
| 平均 Token 使用量 | ~102k tokens | ~61k tokens | 减少 37% |
| 平均步数 | ~12 步 | ~7 步 | 减少 42% |

原始架构中最糟糕的情况：724 秒，100 步，145,463 个 Token，且失败。缩减后的架构以 141 秒、19 步和 67,483 个 Token 成功完成了相同的查询。

## 为什么缩减有效

### 文件系统是强大的抽象

文件系统经过了 50 多年的精炼。标准的 Unix 工具如 `grep` 文档齐全、可预测且被模型所理解。为 Unix 已经解决的问题构建自定义工具，只会增加复杂性而没有价值。

### 工具约束了推理

这些专用工具正在解决模型本来就能独立处理的问题：
- 预过滤模型可以自行导航的语音上下文
- 约束模型可以评估的选项
- 在模型不需要的验证逻辑中封装交互

每一个限制措施都变成了维护负担。每一次模型更新都需要重新校准约束。团队花费在维护这些支架上的时间比改进智能体本身的时间还要多。

### 优秀的文档取代了工具的复杂性

语义层本身就已经文档齐全：
- 结构化 YAML 中的维度定义
- 命名清晰的度量计算
- 可导航文件中的连接关系

自定义工具只是对本来就清晰的内容进行概括。模型需要的是直接阅读文档的权限，而不是建立在其之上的抽象。

## 实现模式

### 文件系统智能体

```python
from ai import ToolLoopAgent, tool
from sandbox import Sandbox

# 创建包含数据层的沙盒环境
sandbox = Sandbox.create()
await sandbox.write_files(data_layer_files)

# 单一的基础工具
def create_execute_tool(sandbox):
    return tool(
        name="execute_command",
        description="""
        在沙盒环境中执行 Bash 命令。
        
        使用标准的 Unix 工具来探索和理解数据层：
        - ls: 列出目录内容
        - cat: 读取文件内容
        - grep: 搜索模式
        - find: 定位文件
        
        沙盒包含语义层文档：
        - /data/entities/*.yaml: 实体定义
        - /data/measures/*.yaml: 度量计算
        - /data/joins/*.yaml: 连接关系
        - /docs/*.md: 额外文档
        """,
        execute=lambda command: sandbox.exec(command)
    )

# 最小化智能体
agent = ToolLoopAgent(
    model="claude-opus-4.5",
    tools={
        "execute_command": create_execute_tool(sandbox),
        "execute_sql": sql_tool,
    }
)
```

### 成功的先决条件

此模式在以下情况下有效：

1. **文档质量高**：文件结构良好、命名一致且包含清晰的定义。

2. **模型能力足够**：模型能够在没有过多引导的情况下进行复杂推理。

3. **安全约束允许**：沙盒限制了智能体可以访问和修改的内容。

4. **领域可导航**：问题空间可以通过文件检查来探索。

### 何时不使用

以下情况缩减会失效：

1. **数据层混乱**：过时的命名约定、未记录的连接、不一致的结构。模型会更快地生成错误的查询。

2. **需要专业知识**：无法在文件中记录的领域专业知识。

3. **安全要求严格限制**：出于安全或合规性考虑必须严格约束的操作。

4. **工作流程确实复杂**：受益于结构化编排的多步骤过程。

## 设计原则

### 削减即增加

最好的智能体可能是工具最少的智能体。每一个工具都是为模型做出的选择。有时，当模型被赋予基础能力而不是受限的工作流时，它会做出更好的选择。

### 信任模型推理

现代模型能够处理复杂性。因为不信任模型的推理能力而约束推理通常是适得其反的。在构建限制措施之前，先测试模型实际能做什么。

### 投资于上下文，而非工具

基础比聪明的工具更重要：
- 清晰的文件命名约定
- 结构良好的文档
- 一致的数据组织
- 易读的关系定义

### 为未来的模型构建

模型改进的速度超过了工具跟上的速度。针对当今模型局限性优化的架构对于明天的模型能力而言可能过度受限。构建能从模型改进中受益的极简架构。

## 评估框架

在考虑架构缩减时，请评估：

1. **维护开销**：花费在维护工具上的时间与改进结果的时间比例是多少？

2. **失败分析**：失败是由模型局限性还是工具约束引起的？

3. **文档质量**：如果给模型访问权限，它能直接导航你的数据层吗？

4. **约束必要性**：限制措施是在防范真实风险还是各种假设出来的担忧？

5. **模型能力**：自工具设计以来，模型能力是否有提升？

## 结论

架构缩减并非普遍适用，但这一原则挑战了一个普遍假设：即更复杂的工具会导致更好的结果。有时情况恰恰相反。从最简单的可能架构开始，仅在证明必要时才增加复杂性，并不断质疑工具是在赋能还是在约束模型的能力。

## 参考资料

- Vercel 工程团队：“我们移除了智能体 80% 的工具”（2025 年 12 月）
- AI SDK ToolLoopAgent 文档
- Vercel Sandbox 文档
