# 工具索引 (Tools Index)

工具提供了智能体可以用来完成任务的具体能力。

## 工具类别

### 评估工具 (Evaluation Tools)
**路径**：`tools/evaluation/`

用于评估 LLM 输出质量的工具。

| 工具 | 用途 | 是否需批准 |
|------|---------|----------|
| `directScore` | 对照标准对响应进行打分 | 否 |
| `pairwiseCompare` | 比较两个响应 | 否 |
| `generateRubric` | 生成评分标准 (Rubric) | 否 |
| `extractCriteria` | 从任务中提取标准 | 否 |

---

### 研究工具 (Research Tools)
**路径**：`tools/research/`

用于收集和处理信息的工具。

| 工具 | 用途 | 是否需批准 |
|------|---------|----------|
| `webSearch` | 搜索网页 | 否 |
| `readUrl` | 从 URL 中提取内容 | 否 |
| `extractClaims` | 识别文本中的断言 (Claims) | 否 |
| `verifyClaim` | 交叉核实某个断言 | 否 |
| `synthesize` | 综合研究发现 | 否 |

---

### 编排工具 (Orchestration Tools)
**路径**：`tools/orchestration/`

用于管理多智能体工作流的工具。

| 工具 | 用途 | 是否需批准 |
|------|---------|----------|
| `delegateToAgent` | 将任务路由给特定智能体 | 否 |
| `parallelExecution` | 并发运行任务 | 否 |
| `waitForCompletion` | 等待异步任务完成 | 否 |
| `synthesizeResults` | 汇总多个智能体的输出 | 否 |
| `handleError` | 管理失败情况 | 否 |

## 工具设计模式

### 标准工具结构

```typescript
export const toolName = tool({
  description: "清晰描述该工具的作用",
  
  parameters: z.object({
    // 必需参数置于最前
    requiredParam: z.string().describe("该参数的用途"),
    
    // 带有默认值的可选参数
    optionalParam: z.number().default(10)
      .describe("该参数控制什么")
  }),
  
  // 对于危险操作进行批准
  needsApproval: false, // 或为 true，或为函数
  
  // 严格模式以确保符合 Schema
  strict: true,
  
  execute: async (input) => {
    try {
      const result = await performOperation(input);
      return { success: true, data: result };
    } catch (error) {
      return {
        success: false,
        error: {
          code: error.code ?? "UNKNOWN",
          message: error.message,
          retryable: isRetryable(error)
        }
      };
    }
  },
  
  // 可选：控制模型看到的内容
  toModelOutput: (result) => ({
    summary: result.data.summary,
    truncated: result.data.full.length > 5000
  })
});
```

### 错误响应模式

```typescript
interface ToolError {
  code: string;       // 机器可读的错误代码
  message: string;    // 人类可读的消息
  retryable: boolean; // 是否通过重试可能解决
  details?: object;   // 额外的背景信息
}

interface ToolResult<T> {
  success: boolean;
  data?: T;
  error?: ToolError;
  metadata: {
    executionTimeMs: number;
    [key: string]: any;
  };
}
```

## 添加新工具

1. 确定类别或创建新类别：`tools/<类别名称>/`
2. 创建工具文件：`tools/<类别名称>/<工具名称>.md`
3. 定义以下内容：
   - 用途与描述
   - 带有 Zod Schema 的输入参数
   - 输出 Schema
   - 错误代码
   - 使用示例
4. 更新此索引文件
5. 分配给相关的智能体

## 工具选择指南

### 当智能体需要……时：

| 行动 | 工具类别 | 建议使用的工具 |
|--------|---------------|-----------------|
| 评估质量 | 评估 (Evaluation) | directScore, pairwiseCompare |
| 查找信息 | 研究 (Research) | webSearch, readUrl |
| 核实事实 | 研究 (Research) | verifyClaim, extractClaims |
| 协调工作 | 编排 (Orchestration) | delegateToAgent |
| 等待结果 | 编排 (Orchestration) | waitForCompletion |
