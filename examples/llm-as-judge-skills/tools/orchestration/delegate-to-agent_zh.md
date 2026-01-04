# 委派给智能体工具 (Delegate to Agent Tool)

## 用途

将任务路由给专门的智能体执行。处理上下文传递、结果收集以及错误管理。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const delegateToAgent = tool({
  description: `将任务委派给专门的智能体。
当子任务需要特定能力时使用。
传递智能体成功执行所需的完整上下文。`,

  parameters: z.object({
    agentName: z.enum(["evaluator (评估者)", "researcher (研究员)", "writer (作家)", "analyst (分析师)"])
      .describe("要委派给的智能体名称"),
    
    task: z.string()
      .describe("对智能体应当执行的任务的清晰描述"),
    
    context: z.object({
      previousOutputs: z.array(z.string()).optional()
        .describe("该智能体所需的先前步骤的输出"),
      
      documents: z.array(z.string()).optional()
        .describe("相关的文档或数据"),
      
      constraints: z.array(z.string()).optional()
        .describe("需要遵守的要求或限制条件")
    }).optional(),
    
    expectedOutput: z.object({
      format: z.enum(["text (文本)", "json", "markdown", "structured (结构化)"])
        .describe("预期的输出格式"),
      
      schema: z.string().optional()
        .describe("若格式为结构化，提供 JSON Schema"),
      
      maxLength: z.number().optional()
        .describe("最大长度限制")
    }).optional(),
    
    timeout: z.number().default(60000)
      .describe("超时时间（毫秒）")
  }),

  execute: async (input) => {
    return executeAgentDelegation(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| agentName | enum | 是 | 目标智能体 |
| task | string | 是 | 任务描述 |
| context | object | 否 | 上下文及依赖项 |
| expectedOutput | object | 否 | 输出要求 |
| timeout | number | 否 | 超时时间（毫秒，默认: 60000） |

## 输出 Schema

```typescript
interface DelegationResult {
  success: boolean;
  
  agentName: string;
  task: string;
  
  output: {
    content: string | object;
    format: string;
  };
  
  execution: {
    startTime: string;
    endTime: string;
    durationMs: number;
    tokenUsage: {
      prompt: number;
      completion: number;
    };
  };
  
  error?: {
    code: string;
    message: string;
    retryable: boolean;
  };
}
```

## 可用智能体示例

### 评估者智能体 (Evaluator Agent)
```typescript
await delegateToAgent.execute({
  agentName: "evaluator",
  task: "对照准确性和清晰度标准评估该响应的质量",
  context: {
    documents: [待评估响应],
    constraints: ["使用 1-5 分制", "包含理由说明"]
  },
  expectedOutput: { format: "structured" }
});
```

### 研究员智能体 (Researcher Agent)
```typescript
await delegateToAgent.execute({
  agentName: "researcher",
  task: "研究当前 LLM 评估的最佳实践",
  context: {
    constraints: ["重点关注 2024 年发表的文献", "包含引用书目"]
  },
  expectedOutput: { format: "markdown" }
});
```

### 作家智能体 (Writer Agent)
```typescript
await delegateToAgent.execute({
  agentName: "writer",
  task: "为这些研究结果撰写一份执行摘要",
  context: {
    previousOutputs: [研究结果],
    constraints: ["最多 500 字", "面向非技术受众"]
  },
  expectedOutput: { format: "text", maxLength: 2500 }
});
```

### 分析师智能体 (Analyst Agent)
```typescript
await delegateToAgent.execute({
  agentName: "analyst",
  task: "分析直接评分与成对比较之间的权衡",
  context: {
    documents: [评估数据],
    constraints: ["尽可能进行定量分析"]
  },
  expectedOutput: { format: "structured" }
});
```

## 错误处理

```typescript
const errorCodes = {
  "AGENT_NOT_FOUND": "指定的智能体不存在",
  "TIMEOUT": "智能体未在超时时间内完成任务",
  "CONTEXT_TOO_LARGE": "上下文超过了智能体的处理能力",
  "INVALID_OUTPUT": "智能体输出不符合预期格式",
  "AGENT_ERROR": "智能体在执行过程中遇到错误"
};
```

## 实现说明

1. **上下文优化**：在传递之前，根据需要对上下文进行压缩。
2. **超时处理**：根据智能体类型设置现实的超时时间。
3. **重试逻辑**：针对瞬态错误实施重试机制。
4. **审计追踪**：记录所有委派操作以实现可追溯性。
5. **资源管理**：跟踪跨委派操作的 token 使用情况。
