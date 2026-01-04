# 多助手模式：技术参考 (Multi-Agent Patterns: Technical Reference)

本文档提供多助手架构在不同框架下的实现细节。

## 主管模式 (Supervisor Pattern)

### LangGraph 主管实现
实现一个主管节点，负责路由到不同的 worker 节点：
- **逻辑**：主管根据任务（task）和历史记录（messages）决定下一步调用哪个 worker（researcher, writer, reviewer 等）。使用 `StateGraph` 构建工作流。

### AutoGen 主管实现
使用 `GroupChat` 模式实现主管：
- **逻辑**：定义专门的 `AssistantAgent`（researcher, writer）和一个 `supervisor` 助手。通过 `GroupChatManager` 管理对话轮次和协调。

## 群集模式 (Swarm Pattern)

### LangGraph 群集实现
实现点对点的交接（handoffs）：
- **逻辑**：每个助手节点在处理完后，判断是否需要“交接”给下一个助手。如果返回 `handoff`，工作流则流向目标助手；否则流程结束。

## 分层模式 (Hierarchical Pattern)

### CrewAI 风格的层级结构
- **逻辑**：`ManagerAgent` 负责分析任务需求，选择最合适的 worker 并分发任务（delegate），同时负责审核 worker 的产出（review_output）。

## 上下文隔离模式 (Context Isolation Patterns)

- **全上下文委派 (Full Context Delegation)**：将整个 planner 的上下文传递给子助手，适用于需要全局理解的复杂任务。
- **指令传递 (Instruction Passing)**：仅向子助手传递特定指令、约束、输入和输出 schema，适用于简单、定义明确的任务。
- **文件系统协调 (File System Coordination)**：助手通过读写工作区中的共享 JSON 文件来交换状态，使用锁（lock）机制防止冲突。

## 共识机制 (Consensus Mechanisms)

- **加权投票 (Weighted Voting)**：根据自信度（verbalized_confidence）和领域专业度（expertise）对助手投票进行加权汇总。
- **辩论协议 (Debate Protocol)**：多轮讨论。助手提出初始方案，互相进行批判（critique），然后整合批判意见更新方案，直到达成收敛。

## 故障恢复 (Failure Recovery)

- **熔断器 (Circuit Breaker)**：监控助手的失败次数。如果超过阈值（failure_threshold），则暂时停止调用该助手并抛出异常。
- **检查点与恢复 (Checkpoint and Resume)**：在每个步骤保存工作流状态（state），以便在崩溃后能从断点处恢复（resume）。
