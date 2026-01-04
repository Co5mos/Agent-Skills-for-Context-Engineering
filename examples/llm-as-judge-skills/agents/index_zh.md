# 智能体索引 (Agents Index)

智能体是具备特定能力、工具和指令的可复用 AI 组件。

## 可用智能体 (Available Agents)

### 评估者智能体 (Evaluator Agent)
**路径**：`agents/evaluator-agent/evaluator-agent.md`
**目的**：评估 LLM 生成的响应质量。

**能力**：
- 对照评分量表进行直接评分。
- 对响应进行成对比较。
- 从任务描述中提取评估标准。
- 为评估生成评分量表。

**使用的工具**：
- `directScore`
- `pairwiseCompare`
- `extractCriteria`
- `generateRubric`

**最适用场景**：
- 内容流水线中的质量关口。
- 模型对比研究。
- RLHF 偏好数据的生成。
- 交付前的输出验证。

---

### 研究员智能体 (Research Agent)
**路径**：`agents/research-agent/research-agent.md`
**目的**：从多个来源收集、验证并综合信息。

**能力**：
- 网页搜索及结果分析。
- URL 内容提取。
- 断言 (Claim) 提取与验证。
- 研究成果综合。

**使用的工具**：
- `webSearch`
- `readUrl`
- `extractClaims`
- `verifyClaim`
- `synthesize`

**最适用场景**：
- 知识库构建。
- 事实核查。
- 市场研究。
- 技术文档编写。

---

### 编排者智能体 (Orchestrator Agent)
**路径**：`agents/orchestrator-agent/orchestrator-agent.md`
**目的**：针对复杂任务协调多智能体工作流。

**能力**：
- 任务分解与分配。
- 并行任务执行。
- 结果综合。
- 错误处理与恢复。

**使用的工具**：
- `delegateToAgent`
- `parallelExecution`
- `waitForCompletion`
- `synthesizeResults`
- `handleError`

**最适用场景**：
- 复杂的跨步骤任务。
- 跨能力的协同工作流。
- 质量保障流水线。
- 长时间运行的操作。

## 智能体交互模式

### 顺序流水线 (Sequential Pipeline)
```
输入 → 智能体 A → 智能体 B → 智能体 C → 输出
```
适用于每个步骤都依赖于前一步骤的情况。

### 并行扇出 (Parallel Fan-Out)
```
        ┌→ 智能体 A ─┐
输入 ──┼→ 智能体 B ──┼→ 结果综合 → 输出
        └→ 智能体 C ─┘
```
适用于可以并发运行的独立子任务。

### 迭代优化 (Iterative Refinement)
```
输入 → 生成智能体 → 评估智能体 ─┬→ 输出 (若通过)
                                 └→ 生成智能体 (若失败，带反馈)
```
适用于对质量要求极高的输出。

## 添加新智能体

1. 创建智能体目录：`agents/<智能体名称>/`
2. 创建主文件：`agents/<智能体名称>/<智能体名称>.md`
3. 定义以下内容：
   - 目的与角色
   - 系统指令
   - 工具分配
   - 配置选项
   - 使用示例
4. 更新此索引文件。
5. 若适用，将其向编排者 (Orchestrator) 注册。
