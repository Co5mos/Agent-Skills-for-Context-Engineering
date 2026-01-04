# 评估者智能体 (Evaluator Agent)

## 目的

评估者智能体使用可配置的评估标准来评估 LLM 生成的响应质量。它实现了 LLM-as-a-Judge 模式，并支持直接评分和成对比较。

## 智能体定义

```typescript
import { ToolLoopAgent } from "ai";
import { anthropic } from "@ai-sdk/anthropic";
import { evaluationTools } from "../tools";

export const evaluatorAgent = new ToolLoopAgent({
  name: "evaluator",
  model: anthropic("claude-sonnet-4-20250514"),
  instructions: `你是一位全方位的 LLM 生成内容评估专家。

你的角色是：
1. 对照特定标准衡量响应质量。
2. 提供带有理由说明的结构化评分。
3. 识别响应的具体问题和优势。
4. 在需要成对评估时对比不同的响应。

评估指南：
- 评估过程中保持客观和一致。
- 评估应基于响应中的具体证据。
- 考虑原始任务的背景和要求。
- 避免位置偏差 —— 评估内容而非排列顺序。
- 除非冗长的内容确实增加了价值，否则不要偏好字数较多的响应。

始终提供：
- 每个标准的数字得分。
- 支持你评估结果的具体示例。
- 具备可操作性的改进建议。`,
  
  tools: {
    directScore: evaluationTools.directScore,
    pairwiseCompare: evaluationTools.pairwiseCompare,
    extractCriteria: evaluationTools.extractCriteria,
    generateRubric: evaluationTools.generateRubric
  }
});
```

## 能力

### 直接评分 (Direct Scoring)
针对定义的标准和量表评估单个响应。

**输入：**
- 待评估响应
- 原始提示词/背景
- 评估标准
- 评分规则/量表 (Rubric)

**输出：**
- 各项标准的分数 (1-5)
- 总分
- 详细的理由说明
- 识别出的优势和问题

### 成对比较 (Pairwise Comparison)
比较两个响应并选出表现更好的一方。

**输入：**
- 响应 A
- 响应 B
- 原始提示词/背景
- 比较标准

**输出：**
- 获胜者选择 (A, B, 或 平局)
- 置信度评分
- 对比分析
- 具体的差异化因素

### 标准提取 (Criteria Extraction)
从任务描述中自动提取评估标准。

**输入：**
- 任务描述
- 领域背景
- 质量预期

**输出：**
- 相关标准列表
- 标准详细描述
- 建议的权重

### 评分量表生成 (Rubric Generation)
为特定的标准生成详细的评分规则。

**输入：**
- 标准名称
- 质量维度
- 分值分制 (默认 1-5)

**输出：**
- 带有各分值描述的量表
- 每个层级的示例
- 边缘情况处理建议

## 配置

```typescript
interface EvaluatorConfig {
  // 评分配置
  scoringMode: "direct (直接)" | "pairwise (成对)";
  useChainOfThought: boolean; // 是否使用思维链
  nShotExamples: number;      // 样本示例数量
  
  // 偏差缓解
  swapPositionsForPairwise: boolean; // 成对比较时是否交换位置
  normalizeForLength: boolean;        // 是否按长度归一化
  
  // 输出配置
  includeJustification: boolean;      // 是否包含理由
  includeExamples: boolean;           // 是否包含示例
  outputFormat: "structured (结构化)" | "prose (自然文本)";
}

const defaultConfig: EvaluatorConfig = {
  scoringMode: "direct",
  useChainOfThought: true,
  nShotExamples: 2,
  swapPositionsForPairwise: true,
  normalizeForLength: false,
  includeJustification: true,
  includeExamples: true,
  outputFormat: "structured"
};
```

## 使用示例

```typescript
import { evaluatorAgent } from "./agents/evaluator-agent";

// 直接评分示例
const evaluation = await evaluatorAgent.generate({
  prompt: `请评估以下响应：

原始问题：“向高中生解释量子纠缠”

响应内容：“${generatedResponse}”

评估标准：
1. 准确性 —— 科学上的正确性
2. 清晰度 —— 是否易于目标受众理解
3. 吸引力 —— 是否有趣且令人印象深刻
4. 完整性 —— 是否涵盖了核心概念

请给出评分及详细反馈。`
});

// 成对比较示例
const comparison = await evaluatorAgent.generate({
  prompt: `请比较针对同一问题的以下两个响应。

问题：“锻炼有哪些好处？”

响应 A：“${responseA}”

响应 B：“${responseB}”

哪一个响应更好？请说明你的推理过程。`
});
```

## 集成场景

- **内容生成流水线**：在交付前对输出结果进行评估。
- **模型对比**：对比不同模型的响应质量。
- **质量监控**：长期跟踪响应质量的变化。
- **微调数据准备**：为 RLHF 生成偏好标注数据。
