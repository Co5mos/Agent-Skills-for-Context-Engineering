# LLM 项目流水线模式 (Pipeline Patterns for LLM Projects)

本参考文档提供构建 LLM 处理流水线的详细模式，适用于批处理、数据分析、内容生成等工作负载。

## 标准流水线结构 (The Canonical Pipeline)

`获取 (Acquire) → 准备 (Prepare) → 处理 (Process) → 解析 (Parse) → 渲染 (Render)`

| 阶段 | 确定性 | 昂贵 | 可并行 | 幂等性 |
| :--- | :--- | :--- | :--- | :--- |
| **获取 (Acquire)** | 是 | 低 | 是 | 是 |
| **准备 (Prepare)** | 是 | 低 | 是 | 是 |
| **处理 (Process)** | **否 (LLM 调用)** | **高** | **是** | **是（需缓存）** |
| **解析 (Parse)** | 是 | 低 | 是 | 是 |
| **渲染 (Render)** | 是 | 低 | 部分 | 是 |

核心结论：只有“处理”阶段涉及非确定性的 LLM 调用。其他阶段应构建为确定性的转换，以便独立测试和迭代。

## 文件系统状态管理模式

### 目录结构模式
```text
project/
├── data/{batch_id}/{item_id}/
│   ├── raw.json         # 获取阶段输出
│   ├── prompt.md        # 准备阶段输出
│   ├── response.md      # 处理阶段（LLM 原始响应）输出
│   └── parsed.json      # 解析阶段输出
└── output/{batch_id}/index.html
```

### 状态检查与清理
- **需要处理 (Needs processing)**：检查给定的 output 文件是否存在。
- **清理/重试 (Clean/Retry)**：删除指定阶段及其下游阶段的所有输出文件。

## 并行执行模式
- **ThreadPoolExecutor**：并行处理多个 LLM 调用，控制线程数（通常 5-15 个）以平衡速度与 API 频率限制。
- **频率限制 (Rate Limiting)**：使用装饰器来平滑请求间隔。

## 解析模式 (Parsing Patterns)
- **正则提取 (Regex Extraction)**：针对标题、分值、列表项等内容构建鲁棒的正则表达式，处理 Markdown 语法干扰和空白符波动。
- **优雅降级 (Graceful Degradation)**：即便部分维度解析失败，也要记录错误并继续处理其余字段，而不是让整个流水线崩溃。

## 错误处理与重试
- **指数退避重试 (Retry with Exponential Backoff)**：遇到 TransientError 时，等待 1, 2, 4... 秒后重试。
- **错误日志记录 (Error Logging)**：将错误和当时的任务上下文序列化保存，方便事后分析。

## 成本估算模式 (Cost Estimation)
- **Token 计数 (Tiktoken)**：在实际调用前估算输入和输出的 Token 数。
- **成本公式**：`总成本 = (总 Token / 1M) * 每百万 Token 单价`。

## Checkpoint 与 Resume 模式
针对长时间运行的流水线，利用 Checkpoint 文件记录已完成、失败和当前处理的项目 ID，支持断点续传。

## 测试模式
- **阶段单元测试**：独立测试提示词生成逻辑和解析逻辑（使用模拟的 LLM 输出）。
- **端到端集成测试**：在测试环境下运行单一项目的完整流程。
