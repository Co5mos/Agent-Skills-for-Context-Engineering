# 上下文优化参考

本文档提供了有关上下文优化技术和策略的详细技术参考。

## 压缩策略

### 基于总结的压缩 (Summary-Based Compaction)

基于总结的压缩通过简洁的总结替换冗长的内容，同时保留关键信息。这种方法通过识别可以压缩的部分，生成捕获要点的摘要，并用摘要替换完整内容。

压缩的有效性取决于保留了哪些信息。关键决策、用户偏好和当前任务状态**绝不应被压缩**。中间结果和支持性证据可以更激进地进行总结。样板信息、重复信息和探索性推理通常可以完全移除。

### Token 预算分配

有效的上下文预算需要了解不同上下文组件如何消耗 Token 并进行战略性分配：

| 组件 | 典型范围 | 备注 |
|-----------|---------------|-------|
| 系统提示词 | 500-2000 tokens | 跨会话稳定 |
| 工具定义 | 每个工具 100-500 | 随工具数量增长 |
| 检索到的文档 | 不固定 | 通常是最大的消耗者 |
| 消息历史记录 | 不固定 | 随对话进行而增长 |
| 工具输出 | 不固定 | 可能占据上下文主导地位 |

### 压缩阈值

在适当的阈值触发压缩以维持性能：

- 有效上下文限制的 70%：警告阈值。
- 有效上下文限制的 80%：触发压缩。
- 有效上下文限制的 90%：执行激进压缩。

确切的阈值取决于模型行为和任务特征。某些模型表现出优雅降级，而另一些模型则会出现性能骤降。

## 观察遮蔽模式 (Observation Masking Patterns)

### 选择性遮蔽

并非所有观察结果都应被同等对待。考虑遮蔽那些已完成使命且不再为活跃推理所需要的观察结果。保留对于当前任务核心的观察、最近一轮的观察，以及可能被再次引用的观察。

### 遮蔽实现

```python
def selective_mask(observations: List[Dict], current_task: Dict) -> List[Dict]:
    """
    根据相关性选择性地遮蔽观察结果。
    
    返回带有 mask 字段的观察结果，指示内容是否被遮蔽。
    """
    masked = []
    
    for obs in observations:
        relevance = calculate_relevance(obs, current_task)
        
        if relevance < 0.3 and obs["age"] > 3:
            # 相关性低且较旧 - 遮蔽
            masked.append({
                **obs,
                "masked": True,
                "reference": store_for_reference(obs["content"]),
                "summary": summarize_content(obs["content"])
            })
        else:
            masked.append({
                **obs,
                "masked": False
            })
    
    return masked
```

## KV 缓存优化 (KV-Cache Optimization)

### 前缀稳定性

KV 缓存命中率取决于前缀的稳定性。稳定的前缀允许跨请求重用缓存。动态前缀会使缓存失效并迫使重新计算。

应保持稳定的元素包括系统提示词、工具定义和频繁使用的模板。可能变化的元素包括时间戳、会话标识符和特定查询内容。

### 缓存友好型设计

设计提示词以最大化缓存命中率：

1. 将稳定内容放在开头。
2. 跨请求使用一致的格式。
3. 尽可能避免在提示词中使用动态内容。
4. 为动态内容使用占位符。

```python
# 缓存不友好：提示词中包含动态时间戳
system_prompt = f"""
当前时间: {datetime.now().isoformat()}
你是一个得力的助手。
"""

# 缓存友好：稳定的提示词，时间作为单独变量提供
system_prompt = """
你是一个得力的助手。
当前时间在相关时单独提供。
"""
```

## 上下文分区策略 (Context Partitioning Strategies)

### 子智能体隔离

将工作划分到多个子智能体中，以防止任何单个上下文变得过大。每个子智能体在一个专注于其子任务的干净上下文中运行。

### 分区规划

```python
def plan_partitioning(task: Dict, context_limit: int) -> Dict:
    """
    根据上下文限制规划如何拆分任务。
    
    返回分区策略和子任务定义。
    """
    estimated_context = estimate_task_context(task)
    
    if estimated_context <= context_limit:
        return {
            "strategy": "single_agent",
            "subtasks": [task]
        }
    
    # 规划多智能体方法
    subtasks = decompose_task(task)
    
    return {
        "strategy": "multi_agent",
        "subtasks": subtasks,
        "coordination": "hierarchical"
    }
```

## 优化决策框架

### 何时优化

当上下文利用率超过 70%、随着对话延长响应质量下降、长上下文导致成本增加或延迟增加时，应考虑上下文优化。

### 应用哪种优化

根据上下文组成选择优化策略：

- 如果工具输出占据主导，应用**观察遮蔽**。
- 如果检索出的文档占据主导，应用**总结**或**分区**。
- 如果消息历史记录占据主导，应用**带总结的压缩**。
- 如果由多个组件共同造成，则**组合使用**上述策略。

### 优化的评估

在应用优化后，评估其有效性：

- 衡量实现的 Token 减少。
- 衡量质量保持情况（输出质量不应下降）。
- 衡量延迟改善。
- 衡量成本降低。

根据评估结果迭代优化策略。

## 常见陷阱

### 过度压缩

过于激进的压缩可能会移除关键信息。务必保留任务目标、用户偏好和最近的对话上下文。在不断增加的压缩强度下进行测试，以找到最佳平衡点。

### 遮蔽了关键观察

遮蔽仍需要的观察结果会导致错误。跟踪观察的使用情况，仅遮蔽不再被引用的内容。考虑保留已被遮蔽内容的引用，以便在需要时重新检索。

### 忽视注意力分布

“遗忘中段”（Lost-in-middle）现象意味着信息的放置位置非常重要。将关键信息放在注意力青睐的位置（上下文的开头和结尾）。使用显式标记突出显示重要内容。

### 过早优化

并非所有上下文都需要优化。增加优化机制会带来开销。只有当上下文限制确实制约了智能体性能时再进行优化。

## 监控与告警

### 关键指标

跟踪这些指标以了解优化需求：

- 随时间变化的上下文 Token 计数。
- 重复模式的缓存命中率。
- 按上下文大小划分的响应质量指标。
- 按上下文长度划分的单次对话成本。
- 随上下文大小变化的延迟。

### 告警阈值

为以下情况设置告警：

- 上下文利用率高于 80%。
- 缓存命中率低于 50%。
- 质量分数下降超过 10%。
- 成本超出基准线。

## 集成模式

### 与智能体框架集成

将优化整合进智能体工作流：

```python
class OptimizingAgent:
    def __init__(self, context_limit: int = 80000):
        self.context_limit = context_limit
        self.optimizer = ContextOptimizer()
    
    def process(self, user_input: str, context: Dict) -> Dict:
        # 检查是否需要优化
        if self.optimizer.should_compact(context):
            context = self.optimizer.compact(context)
        
        # 使用优化后的上下文进行处理
        response = self._call_model(user_input, context)
        
        # 记录指标
        self.optimizer.record_metrics(context, response)
        
        return response
```

### 与记忆系统集成

连接优化与记忆系统：

```python
class MemoryAwareOptimizer:
    def __init__(self, memory_system, context_limit: int):
        self.memory = memory_system
        self.limit = context_limit
    
    def optimize_context(self, current_context: Dict, task: str) -> Dict:
        # 检查信息是否在记忆中
        relevant_memories = self.memory.retrieve(task)
        
        # 如果上下文中不需要，则将信息移动到记忆中
        for mem in relevant_memories:
            if mem["importance"] < threshold:
                current_context = remove_from_context(current_context, mem)
                # 保留该记忆可检索的引用
        
        return current_context
```

## 性能基准

### 压缩性能

压缩应在保证质量的同时减少 Token 计数。目标：

- 激进压缩实现 50-70% 的 Token 减少。
- 质量由于压缩产生的下降应小于 5%。
- 压缩开销带来的延迟增加应小于 10%。

### 遮蔽性能

观察遮蔽应显著减少 Token 计数：

- 被遮蔽观察结果的 Token 减少 60-80%。
- 遮蔽对质量的影响应小于 2%。
- 几乎为零的延迟开销。

### 缓存性能

KV 缓存优化应改善成本和延迟：

- 稳定工作负载下，缓存命中率达到 70% 以上。
- 缓存命中带来 50% 以上的成本降低。
- 缓存命中带来 40% 以上的延迟改善。
