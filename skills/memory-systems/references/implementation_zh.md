# 记忆系统：技术参考 (Memory Systems: Technical Reference)

本文档提供记忆系统组件的实现细节。

## 向量数据库实现 (Vector Store Implementation)

### 基础向量数据库
```python
class VectorStore:
    # 逻辑：添加文档并生成嵌入向量。支持余弦相似度搜索。
    # search 函数支持元数据过滤。
```

### 增强型元数据向量数据库
```python
class MetadataVectorStore(VectorStore):
    # 逻辑：在基础向量库之上增加了按实体（entity）和时间范围（time_index）的分级索引。
    # 允许直接通过特定的 entity identifier 进行检索。
```

## 知识图谱实现 (Knowledge Graph Implementation)

### 属性图存储
```python
class PropertyGraph:
    # 逻辑：管理节点（id, label, properties）和边（id, source, target, type, properties）。
    # 实现了一个简易的类 Cypher 模式匹配器（MATCH pattern）。
```

## 时序知识图谱 (Temporal Knowledge Graph)

```python
class TemporalKnowledgeGraph(PropertyGraph):
    # 逻辑：为边增加了有效时间范围（valid_from, valid_until）。
    # 支持 query_at_time：仅检索在给定时间点有效的事实和关系。
```

## 记忆整合 (Memory Consolidation)

```python
class MemoryConsolidator:
    # 逻辑：当记忆达到阈值时触发整合。
    # 1. 识别重复事实。
    # 2. 合并相关事实组（merge_fact_group），保留置信度最高的一个。
    # 3. 更新有效期并重建索引。
```

## 记忆-上下文整合 (Memory-Context Integration)

```python
class MemoryContextIntegrator:
    # 逻辑：根据当前任务从记忆系统中提取相关实体的记忆，并将其格式化注入到上下文。
    # 包含 token 数量估算和截断逻辑，以确保不超出上下文窗口限制。
```
