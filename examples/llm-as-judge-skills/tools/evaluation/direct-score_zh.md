# 直接评分工具 (Direct Score Tool)

## 用途

使用评分标准 (Rubric) 对单个 LLM 响应进行对照评估并打分。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const directScore = tool({
  description: `通过对照特定标准对响应进行打分。
适用于需要评估质量维度的客观评估，
例如准确性、完整性、清晰度或任务遵循程度。
返回带有理由说明的结构化评分。`,

  parameters: z.object({
    response: z.string()
      .describe("待评估的 LLM 响应"),
    
    prompt: z.string()
      .describe("生成该响应的原始提示词/指令"),
    
    context: z.string().optional()
      .describe("额外背景信息，如检索到的文档或对话历史"),
    
    criteria: z.array(z.object({
      name: z.string().describe("标准名称 (例如 '准确性')"),
      description: z.string().describe("该标准衡量什么"),
      weight: z.number().min(0).max(1).default(1)
        .describe("相对重要性，所有权重的总和应为 1")
    })).min(1).describe("评估打分所对照的一系列标准"),
    
    rubric: z.object({
      scale: z.enum(["1-3", "1-5", "1-10"]).default("1-5"),
      levelDescriptions: z.record(z.string(), z.string()).optional()
        .describe("可选：各分值等级的描述")
    }).optional().describe("评分标准配置")
  }),

  execute: async (input) => {
    // 实现上会委托给作为评估者的 LLM
    return evaluateWithLLM(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| response | string | 是 | 待评估的响应 |
| prompt | string | 是 | 生成该响应的原始提示词 |
| context | string | 否 | 额外背景信息 (RAG 文档、历史记录) |
| criteria | Criterion[] | 是 | 评估标准列表 |
| rubric | Rubric | 否 | 评分分制及等级描述 |

### Criterion 对象
```typescript
{
  name: string;        // 例如 "事实准确性"
  description: string; // 例如 "响应不包含事实错误"
  weight: number;      // 0-1, 相对重要性
}
```

### Rubric 对象
```typescript
{
  scale: "1-3" | "1-5" | "1-10";
  levelDescriptions?: {
    "1": "极差 - 存在重大问题",
    "2": "低于平均水平 - 存在若干问题",
    "3": "平均水平 - 存在部分问题",
    "4": "良好 - 仅有轻微问题",
    "5": "优秀 - 无问题"
  }
}
```

## 输出 Schema

```typescript
interface DirectScoreResult {
  success: boolean;
  
  scores: {
    criterion: string;
    score: number;
    maxScore: number;
    justification: string; // 理由
    examples: string[];    // 来自响应的具体示例
  }[];
  
  overallScore: number;    // 总分
  weightedScore: number;   // 加权得分
  
  summary: {
    strengths: string[];   // 优点
    weaknesses: string[];  // 缺点
    suggestions: string[]; // 建议
  };
  
  metadata: {
    evaluationTimeMs: number;
    criteriaCount: number;
    rubricScale: string;
  };
}
```

## 使用示例

```typescript
const result = await directScore.execute({
  response: "机器学习是人工智能的一个子集，它使系统能够从数据中学习……",
  
  prompt: "向初学者解释机器学习",
  
  criteria: [
    {
      name: "准确性",
      description: "解释的技术正确性",
      weight: 0.4
    },
    {
      name: "清晰度",
      description: "初学者是否易于理解",
      weight: 0.3
    },
    {
      name: "完整性",
      description: "是否充分涵盖了关键概念",
      weight: 0.3
    }
  ],
  
  rubric: {
    scale: "1-5",
    levelDescriptions: {
      "1": "完全未达到标准",
      "2": "存在重大缺陷",
      "3": "可接受但有待改进",
      "4": "良好，仅有轻微问题",
      "5": "优秀，无问题"
    }
  }
});
```

## 实现说明

1. **思维链 (Chain-of-Thought)**：实现时应使用 CoT 提示词，以获得更可靠的评分。
2. **校准 (Calibration)**：在每个等级中包含 few-shot 评分示例。
3. **理由先行**：在给出分数前要求先说明理由，以减少偏差。
4. **长度归一化**：在适当时考虑响应长度的影响。
