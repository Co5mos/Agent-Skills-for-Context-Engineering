# LLM 评估器技能 (LLM-Evaluator Skill)

## 概览

LLM 评估器（LLM-as-a-Judge）是设计用于评估另一个大语言模型对指令或查询的响应质量的大语言模型。此技能为构建评估系统提供了基础知识。

## 关键考虑因素

### 基准选择 (Baseline Selection)
- **人工标注员**：力求使 LLM 与人类的相关性达到人类与人类之间的相关性水平。LLM 评估器比人工标注快几个数量级，且成本更低。
- **微调分类器**：目标是获得与微调分类器类似的召回率和精确率。这是一项更具挑战性的基准，因为微调模型是针对特定任务进行优化的。

### 评分方法

| 方法 | 使用场景 | 可靠性 |
|----------|----------|-------------|
| **直接评分 (Direct Scoring)** | 客观任务（事实性、毒性、指令遵循） | 更适合二元分类 |
| **成对比较 (Pairwise Comparison)** | 主观评估（语气、说服力、连贯性） | 对于偏好任务更为可靠 |
| **基于参考 (Reference-Based)** | 与标准答案（Gold Standard）对比 | 需要真值（Ground Truth）参考 |

### 评估指标

**分类指标**（二元任务的首选）：
- 召回率 (Recall) 和精确率 (Precision)
- F1 分数
- Cohen's κ (Kappa)

**相关性指标**（适用于李克特量表任务）：
- Spearman's ρ (rho)
- Kendall's τ (tau)

## 已知偏差

1. **位置偏差 (Position Bias)**：在成对比较中，LLM 评估器往往偏好处于特定位置（通常是第一个位置）的响应。
2. **冗长偏差 (Verbosity Bias)**：即使质量不高，也倾向于选择更长、更冗长的回答。
3. **自我增强偏差 (Self-Enhancement Bias)**：LLM 评估器更偏好由其自身生成的答案。

## 缓解策略

- 交换响应位置并取平均结果。
- 评估时针对长度进行归一化。
- 使用多模型评审团 (Panel of LLMs, PoLL) 而非单一裁判。
- 加入“不要过度思考”之类的指令。
- 使用思维链 (CoT) + N-shot 提示词以提高可靠性。

## 实现模式

```typescript
interface EvaluatorConfig {
  scoringApproach: 'direct' | 'pairwise' | 'reference-based';
  criteria: EvaluationCriteria[];
  metrics: MetricType[];
  useCoT: boolean;
  nShot: number;
}

interface EvaluationCriteria {
  name: string;
  description: string;
  rubric: RubricLevel[];
}

interface RubricLevel {
  score: number;
  description: string;
}
```

## 参考资料

查阅的关键论文：
- Constitutional AI (Anthropic)
- G-Eval: NLG Evaluation using GPT-4
- SelfCheckGPT: Zero-Resource Hallucination Detection
- Prometheus: Fine-grained Evaluation Capability
- MT-Bench and Chatbot Arena
