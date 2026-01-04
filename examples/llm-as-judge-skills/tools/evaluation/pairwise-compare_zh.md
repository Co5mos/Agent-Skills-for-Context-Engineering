# 成对比较工具 (Pairwise Compare Tool)

## 用途

比较两个 LLM 响应，并确定哪一个更好地满足了给定的标准。在处理主观评估时，这通常比直接评分更可靠。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const pairwiseCompare = tool({
  description: `比较两个响应并选出较优者。
适用于语气、说服力或写作风格等主观评估。
在评估偏好时，这比直接评分更可靠。
返回获胜的选择以及详细的对比分析。`,

  parameters: z.object({
    responseA: z.string()
      .describe("待比较的第一个响应"),
    
    responseB: z.string()
      .describe("待比较的第二个响应"),
    
    prompt: z.string()
      .describe("两个响应均针对的原始提示词"),
    
    context: z.string().optional()
      .describe("如果相关，提供额外的背景信息"),
    
    criteria: z.array(z.string())
      .describe("比较的维度，例如 ['清晰度', '吸引力', '准确性']"),
    
    allowTie: z.boolean().default(true)
      .describe("是否允许判为平局"),
    
    swapPositions: z.boolean().default(true)
      .describe("进行两次评估，交换位置以减少位置偏差")
  }),

  execute: async (input) => {
    if (input.swapPositions) {
      return evaluateWithPositionSwap(input);
    }
    return evaluatePairwise(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| responseA | string | 是 | 第一个响应 |
| responseB | string | 是 | 第二个响应 |
| prompt | string | 是 | 原始提示词 |
| context | string | 否 | 额外背景信息 |
| criteria | string[] | 是 | 比较维度列表 |
| allowTie | boolean | 否 | 是否允许平局 (默认: true) |
| swapPositions | boolean | 否 | 是否交换位置以减少偏差 (默认: true) |

## 输出 Schema

```typescript
interface PairwiseCompareResult {
  success: boolean;
  
  winner: "A" | "B" | "TIE";
  confidence: number; // 置信度 0-1
  
  comparison: {
    criterion: string;
    winner: "A" | "B" | "TIE";
    reasoning: string;  // 推理
    aStrength: string;  // A 的优点
    bStrength: string;  // B 的优点
  }[];
  
  overallReasoning: string; // 总体推理
  
  differentiators: {
    aAdvantages: string[]; // A 的核心优势
    bAdvantages: string[]; // B 的核心优势
  };
  
  // 如果 swapPositions 为 true
  positionConsistency?: {
    firstPassWinner: "A" | "B" | "TIE";
    secondPassWinner: "A" | "B" | "TIE";
    consistent: boolean; // 是否具有一致性
  };
  
  metadata: {
    evaluationTimeMs: number;
    positionsSwapped: boolean; // 是否进行了位置交换
  };
}
```

## 使用示例

```typescript
const result = await pairwiseCompare.execute({
  responseA: "锻炼可以改善心血管健康，增强肌肉，并提升心理幸福感……",
  
  responseB: "定期健身有很多好处。你会感觉到更好，看起来也更好……",
  
  prompt: "解释定期锻炼的好处",
  
  criteria: ["准确性", "具体性", "吸引力", "完整性"],
  
  allowTie: true,
  swapPositions: true
});

// 结果示例:
// {
//   winner: "A",
//   confidence: 0.85,
//   comparison: [
//     {
//       criterion: "准确性",
//       winner: "A",
//       reasoning: "响应 A 使用了特定的医学术语……",
//       aStrength: "提到了心血管、肌肉和心理健康",
//       bStrength: "较为笼统但并无错误"
//     },
//     ...
//   ],
//   ...
// }
```

## 位置交换算法 (Position Swap Algorithm)

为了缓解位置偏差：

```typescript
async function evaluateWithPositionSwap(input) {
  // 第一轮：原始顺序
  const pass1 = await evaluate({
    first: input.responseA,
    second: input.responseB,
    ...input
  });
  
  // 第二轮：交换顺序
  const pass2 = await evaluate({
    first: input.responseB,
    second: input.responseA,
    ...input
  });
  
  // 调整并比对结果
  const pass2Adjusted = pass2.winner === "A" ? "B" : pass2.winner === "B" ? "A" : "TIE";
  
  if (pass1.winner === pass2Adjusted) {
    return {
      ...pass1,
      positionConsistency: { consistent: true, ... }
    };
  } else {
    // 不一致 —— 返回平局或降低置信度
    return {
      winner: "TIE",
      confidence: 0.5,
      positionConsistency: { consistent: false, ... },
      ...
    };
  }
}
```

## 实现说明

1. **缓解位置偏差**：在生产环境中，请务必使用 `swapPositions: true`。
2. **标准顺序**：按重要性排列评估维度，以便更好地聚焦。
3. **平局处理**：考虑领域特性 —— 某些任务应该极少出现平局。
4. **置信度校准**：当两份响应水平接近时，降低置信度数值。
5. **长度考量**：注意其中一个响应是否明显长于另一个。
6, **思维链理由说明**：所有评估都需要带证据的理由说明，提高可靠性。
