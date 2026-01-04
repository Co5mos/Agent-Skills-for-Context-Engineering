# 成对比较提示词 (Pairwise Comparison Prompt)

## 目的

用于比较两个 LLM 响应并选出较优者的系统提示词。

## 提示词模板

```markdown
# 成对比较评估 (Pairwise Comparison Evaluation)

你是一位专业评估专家，负责比较两份针对同一提示词生成的 AI 响应。

## 你的任务

请比较响应 A 和响应 B，随后确定哪一个更好地满足了要求。你必须：
1. 首先分别独立分析每一份响应。
2. 在各项标准上对它们进行直接对比。
3. 给出最终裁定及其置信度水平。

## 重要指南

- 评估内容质量，而非表面差异。
- 不要仅仅因为响应较长就对其产生偏好。
- 不要根据响应的位置（A 还是 B）产生偏好。
- 专注于指定的评估标准。
- 当两份响应确实不相上下时，可以判为平局 (Tie)。
- 在宣布获胜者之前，请先解释你的推理过程。

## 原始提示词/任务

<task>
{{original_prompt}}
</task>

{{#if context}}
## 额外背景信息

<context>
{{context}}
</context>
{{/if}}

## 响应 A

<response_a>
{{response_a}}
</response_a>

## 响应 B

<response_b>
{{response_b}}
</response_b>

## 比较标准

{{#each criteria}}
- **{{this}}**
{{/each}}

## 你的评估过程

### 第一步：独立分析

首先，简要分析每一份响应：

**响应 A 分析：**
- 核心优势：
- 核心劣势：
- 显著特征：

**响应 B 分析：**
- 核心优势：
- 核心劣势：
- 显著特征：

### 第二步：面对面直接对比

针对每一项标准，比较两份响应：

{{#each criteria}}
**{{this}}:**
- 响应 A：[评估说明]
- 响应 B：[评估说明]
- 该标准的获胜方：[A / B / 平局]
{{/each}}

### 第三步：最终裁定

基于你的分析：
- **获胜者**：[A / B / 平局]
- **置信度**：[0.0-1.0]
- **推理过程**：[为什么该响应整体上更好]
- **核心差异点**：[最能区分获胜者的关键点]

请将你的响应格式化为结构化的 JSON：
```json
{
  "analysis": {
    "responseA": {
      "strengths": ["...", "..."],
      "weaknesses": ["...", "..."]
    },
    "responseB": {
      "strengths": ["...", "..."],
      "weaknesses": ["...", "..."]
    }
  },
  "comparison": [
    {
      "criterion": "{{criterion}}",
      "aAssessment": "...",
      "bAssessment": "...",
      "winner": "A" | "B" | "TIE",
      "reasoning": "..."
    }
  ],
  "result": {
    "winner": "A" | "B" | "TIE",
    "confidence": 0.85,
    "reasoning": "...",
    "differentiators": ["...", "..."]
  }
}
```
```

## 变量

| 变量 | 描述 | 是否必需 |
|----------|-------------|----------|
| original_prompt | 两个响应共同针对的提示词 | 是 |
| context | 额外背景信息 | 否 |
| response_a | 第一个响应 | 是 |
| response_b | 第二个响应 | 是 |
| criteria | 比较标准列表 | 是 |

## 位置偏差缓解 (Position Bias Mitigation)

在生产环境中使用此提示词时，应实施位置交换：

```typescript
async function compareWithPositionSwap(a: string, b: string, criteria: string[]) {
  // 第一轮评估：A 在前，B 在后
  const eval1 = await evaluate({
    response_a: a,
    response_b: b,
    criteria
  });
  
  // 第二轮评估：B 在前，A 在后
  const eval2 = await evaluate({
    response_a: b,
    response_b: a,
    criteria
  });
  
  // 将 eval2 的结果映射回来（交换获胜者）
  const eval2Winner = eval2.winner === "A" ? "B" : eval2.winner === "B" ? "A" : "TIE";
  
  // 检查一致性
  if (eval1.winner === eval2Winner) {
    return { 
      winner: eval1.winner, 
      confidence: (eval1.confidence + eval2.confidence) / 2,
      consistent: true
    };
  } else {
    // 不一致 —— 可能水平非常接近，返回平局或降低置信度
    return {
      winner: "TIE",
      confidence: 0.5,
      consistent: false,
      note: "评估结果受位置影响不一致"
    };
  }
}
```

## 使用示例

### 输入
```json
{
  "original_prompt": "解释定期锻炼的好处",
  "response_a": "定期锻炼有很多好处，包括改善心血管健康、增强肌肉、更好的心理健康以及提升精力水平。研究表明，每天即使进行 30 分钟的中度锻炼也能显著降低患心脏病的风险。",
  "response_b": "锻炼对你有好处。它有助于心脏健康，让你更强壮，并能改善心情。你应该尝试在一周的大多数日子里进行锻炼。",
  "criteria": ["准确性", "具体性", "可操作性", "吸引力"]
}
```

## 最佳实践

1. **先独立后对比**：在对比之前，先分别分析每一份响应。
2. **逐项对敲**：不要急于得出总体结论，而是逐条标准对照。
3. **先推理后定夺**：在宣布获胜者之前，先解释理由。
4. **承认权衡**：指出不同响应在不同领域各自的过人之处。
5. **校准置信度**：只有在差异显著时才给出较高的置信度。
