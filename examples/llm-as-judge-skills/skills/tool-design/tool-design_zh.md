# 助手工具设计技能 (Agent Tool Design Skill)

## 概览

工具是助手能力的基础。助手采取有意义行动的能力完全取决于：它能否可靠地生成有效的工具输入、这些输入与用户意图的匹配程度，以及工具输出指导后续步骤的有效性。

## 设计原则

### 1. 单一职责

每个工具应该只做好一件事。复杂的原子操作应由多个工具组合而成。

```typescript
// 错误做法：工具的功能过于繁杂
const analyzeAndSummarizeAndSend = { ... }

// 正确做法：分离关注点
const analyzeDocument = { ... }
const summarizeContent = { ... }
const sendEmail = { ... }
```

### 2. 清晰的输入 Schema

使用显式的、经过验证的 Schema，并带有描述性的字段名称和约束条件。

```typescript
const searchTool = tool({
  description: "根据语义相似度搜索文档",
  parameters: z.object({
    query: z.string().describe("自然语言搜索查询词"),
    limit: z.number().min(1).max(100).default(10)
      .describe("返回结果的最大数量"),
    filters: z.object({
      dateAfter: z.string().optional()
        .describe("ISO 日期字符串，仅返回此日期之后的文档"),
      source: z.enum(["internal", "external", "all"]).default("all")
    }).optional()
  }),
  execute: async (input) => { ... }
});
```

### 3. 可预测的输出结构

返回一致的、带有类型的输出，以便模型能够可靠地进行解析。

```typescript
interface ToolResult<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    retryable: boolean;
  };
  metadata: {
    executionTimeMs: number;
    source?: string;
  };
}
```

### 4. 优雅的错误处理

工具绝不应抛出未处理的异常。始终返回结构化的错误信息。

```typescript
execute: async (input) => {
  try {
    const result = await performAction(input);
    return { success: true, data: result };
  } catch (error) {
    return {
      success: false,
      error: {
        code: error.code ?? "UNKNOWN_ERROR",
        message: error.message,
        retryable: isRetryable(error)
      }
    };
  }
}
```

## 工具分类

### 只读工具 (Read-Only Tools)
- 数据库查询
- API 获取
- 文件读取
- 搜索操作

无需批准即可执行。返回数据但不修改状态。

### 状态修改工具 (State-Modifying Tools)
- 数据库写入
- 文件修改
- API POST/PUT/DELETE
- 系统配置更改

可能需要人工批准。考虑使用 `needsApproval` 标志。

### 危险工具 (Dangerous Tools)
- 文件删除
- 支付处理
- 生产环境部署
- 发送外部通信

应始终要求批准并进行审计日志记录。

## AI SDK 6 工具特性

### 工具执行批准
```typescript
const deleteTool = tool({
  description: "从系统中删除文件",
  parameters: z.object({
    path: z.string()
  }),
  needsApproval: true, // 需要人工批准
  execute: async ({ path }) => { ... }
});

// 根据输入进行动态批准
const commandTool = tool({
  description: "执行 shell 命令",
  parameters: z.object({
    command: z.string()
  }),
  needsApproval: ({ command }) => {
    return command.includes("rm") || command.includes("delete");
  },
  execute: async ({ command }) => { ... }
});
```

### 严格模式 (Strict Mode)
启用原生严格模式以确保符合 Schema：
```typescript
const strictTool = tool({
  description: "...",
  parameters: schema,
  strict: true, // 启用严格模式
  execute: async (input) => { ... }
});
```

### 输入示例 (Input Examples)
帮助模型理解预期的输入格式：
```typescript
const complexTool = tool({
  description: "创建日历事件",
  parameters: eventSchema,
  inputExamples: [
    {
      title: "团队站会",
      date: "2024-01-15",
      time: "09:00",
      duration: 30,
      attendees: ["alice@example.com", "bob@example.com"]
    }
  ],
  execute: async (input) => { ... }
});
```

### toModelOutput
控制返回给模型的内容：
```typescript
const readFileTool = tool({
  description: "读取文件内容",
  parameters: z.object({ path: z.string() }),
  execute: async ({ path }) => {
    const content = await fs.readFile(path, 'utf-8');
    return { path, content, size: content.length };
  },
  toModelOutput: (result) => {
    // 仅向模型发送截断后的内容
    return {
      path: result.path,
      content: result.content.slice(0, 5000),
      truncated: result.content.length > 5000
    };
  }
});
```

## 最佳实践

1. **描述性名称**：工具名称应清晰地表明其功能。
2. **全面的描述**：在工具描述中包含使用示例。
3. **合理的默认值**：为可选参数提供合理的默认值。
4. **幂等性**：尽可能将工具设计为可安全地重新执行。
5. **超时处理**：为外部操作实施超时机制。
6. **频率限制**：防止工具失控执行。
7. **日志记录**：记录所有工具调用，以便调试和审计。
