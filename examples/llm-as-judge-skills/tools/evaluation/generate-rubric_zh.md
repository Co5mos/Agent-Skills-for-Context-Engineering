# 评分标准生成工具 (Generate Rubric Tool)

## 用途

为给定的评估标准自动生成评分量表 (Rubric)。该工具会为每个分值等级创建详细描述，确保评估的一致性。

## 工具定义

```typescript
import { tool } from "ai";
import { z } from "zod";

export const generateRubric = tool({
  description: `为评估标准生成评分量表。
为每个分值等级创建详细的描述。
当您需要建立一致的评估标准时，请使用此工具。`,

  parameters: z.object({
    criterionName: z.string()
      .describe("标准名称 (例如 '事实准确性')"),
    
    criterionDescription: z.string()
      .describe("该标准衡量什么"),
    
    scale: z.enum(["1-3", "1-5", "1-10"]).default("1-5")
      .describe("使用的评分分制"),
    
    domain: z.string().optional()
      .describe("领域背景 (例如 '医学写作', '代码审查')"),
    
    includeExamples: z.boolean().default(true)
      .describe("是否为每个分值等级包含示例文本"),
    
    strictness: z.enum(["lenient (宽松)", "balanced (平衡)", "strict (严格)"]).default("balanced")
      .describe("评分界限的定义严苛程度")
  }),

  execute: async (input) => {
    return generateRubricWithLLM(input);
  }
});
```

## 输入 Schema

| 字段 | 类型 | 必需 | 描述 |
|-------|------|----------|-------------|
| criterionName | string | 是 | 标准名称 |
| criterionDescription | string | 是 | 该标准衡量的内容 |
| scale | enum | 否 | 评分分制 (默认: 1-5) |
| domain | string | 否 | 领域背景 |
| includeExamples | boolean | 否 | 是否包含示例 (默认: true) |
| strictness | enum | 否 | 分值界限的严苛程度 |

## 输出 Schema

```typescript
interface GeneratedRubric {
  success: boolean;
  
  criterion: {
    name: string;
    description: string;
  };
  
  scale: {
    min: number;
    max: number;
    type: string;
  };
  
  levels: {
    score: number;
    label: string;        // 例如 "优秀", "差"
    description: string;  // 详细描述
    characteristics: string[];  // 关键特征
    example?: string;     // 该分值的示例文本
  }[];
  
  scoringGuidelines: string[]; // 评分指南
  
  edgeCases: {            // 边缘情况处理
    situation: string;    // 情境
    guidance: string;     // 指导建议
  }[];
  
  metadata: {
    domain: string | null;
    strictness: string;
    generationTimeMs: number;
  };
}
```

## 使用示例

```typescript
const rubric = await generateRubric.execute({
  criterionName: "代码可读性",
  criterionDescription: "代码易于阅读和理解的程度",
  scale: "1-5",
  domain: "代码审查",
  includeExamples: true,
  strictness: "balanced"
});

// 结果示例:
// {
//   criterion: {
//     name: "代码可读性",
//     description: "代码易于阅读和理解的程度"
//   },
//   scale: { min: 1, max: 5, type: "1-5" },
//   levels: [
//     {
//       score: 1,
//       label: "差",
//       description: "代码极其难以理解……",
//       characteristics: [
//         "变量命名无意义",
//         "逻辑深层嵌套且无解释",
//         "复杂部分没有任何注释"
//       ],
//       example: "function x(a,b,c){return a?b+c:c-b;}"
//     },
//     {
//       score: 5,
//       label: "优秀",
//       description: "代码一目了然，极易理解……",
//       characteristics: [
//         "变量和函数命名具有自解释性",
//         "有适当的注释解释‘为什么’这么写",
//         "逻辑结构清晰"
//       ],
//       example: "function calculateShippingCost(weight, distance, expedited) {\n  // 基本费率加上每英里费用\n  const baseCost = weight * BASE_RATE_PER_KG;\n  ..."
//     },
//     ...
//   ],
//   scoringGuidelines: [
//     "关注不熟悉该代码库的人阅读时的清晰度",
//     "同时考虑命名和结构",
//     "注释应解释‘为什么’，而非‘做了什么’"
//   ],
//   edgeCases: [
//     {
//       situation: "代码使用了特定领域的缩写",
//       guidance: "如果缩写在该领域是标准的，则可以接受"
//     }
//   ]
// }
```

## 评分量表模板示例

### 事实准确性 (1-5)
```
5: 所有断言均事实正确，来源可靠
4: 存在细微的事实问题，非关键性
3: 存在部分事实错误，但核心观点正确
2: 多处事实错误，影响了可靠性
1: 根本性错误或具有误导性
```

### 清晰度 (1-5)
```
5: 极易理解，结构良好
4: 清晰，仅有细微歧义
3: 总体清晰，部分章节存在困惑
2: 难以跟进，组织结构不清晰
1: 无法理解或语无伦次
```

### 完整性 (1-5)
```
5: 全面涵盖了所有方面
4: 涵盖了主要观点，存在细微缺漏
3: 涵盖了核心要求，但有显著缺漏
2: 缺失重要的必需元素
1: 未能回答问题
```

## 实现说明

1. **领域自适应**：评分标准应反映特定领域的预期要求。
2. **界限清晰度**：相邻分值之间应有明确的区别。
3. **示例质量**：示例应贴近现实，而非极端的“稻草人”案例。
4. **边缘情况覆盖**：预见到常见的模糊情境。
5. **校准**：在使用前，针对已知样本测试评分标准的效果。
