---
name: book-sft-pipeline
description: 当用户要求“对书籍进行微调”、“创建 SFT 数据集”、“训练风格模型”、“提取 ePub 文本”，或提到风格迁移、LoRA 训练、书籍分段或作者语气复制时使用此技能。
version: 2.0.0
---

# 书籍 SFT 流水线 (Book SFT Pipeline)

一个用于将书籍转换为 SFT 数据集并训练风格迁移模型的完整系统。该技能教授从原始 ePub 到能够以任何作者语气写作的模型的完整流水线。

## 何时激活

在以下情况激活此技能：
- 从文学作品构建微调数据集
- 创建作者语气或风格迁移模型
- 为 Tinker 或类似的 SFT 平台准备训练数据
- 为长篇内容设计文本分段流水线
- 在有限数据上训练小模型（8B 或更小）

## 核心概念

### 书籍 SFT 的三大支柱

**1. 智能分段 (Intelligent Segmentation)**
文本块必须在语义上连贯。在句中截断会教导模型产生碎片化的输出。目标：每个块 150-400 个单词，且始终在自然边界处截断。

**2. 多样化的指令生成 (Diverse Instruction Generation)**
使用多种提示词模板和系统提示词来防止过拟合。单一的提示词风格会导致模型死记硬背。建议使用 15 个以上的提示词模板和 5 个以上的系统提示词。

**3. 风格重于内容 (Style Over Content)**
目标是学习作者的节奏和词汇模式，而不是记住情节。合成指令应描述发生了什么，而不是直接引用文本。

## 流水线架构

```
┌─────────────────────────────────────────────────────────────────┐
│                      协调者智能体 (ORCHESTRATOR)                  │
│  协调流水线各阶段，管理状态，处理故障                             │
└──────────────────────┬──────────────────────────────────────────┘
                       │
        ┌───────────────┼───────────────┬───────────────┐
        ▼               ▼               ▼               ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  提取智能体  │ │  分段智能体  │ │  指令智能体  │ │ 数据集构建器 │
│ ePub → 文本  │ │ 文本 → 块    │ │ 块 → 提示词  │ │ 键值对 →     │
│              │ │ 150-400 词   │ │              │ │ JSONL        │
└──────────────┘ └──────────────┘ └──────────────┘ └──────────────┘
                       │
        ┌───────────────┴───────────────┐
        ▼                               ▼
┌──────────────┐               ┌──────────────┐
│  训练智能体  │               │  验证智能体  │
│ 在 Tinker 上 │               │  AI 检测器   │
│ 进行 LoRA    │               │  原创性验证  │
└──────────────┘               └──────────────┘
```

## 第一阶段：文本提取 (Text Extraction)

### 关键规则
1. **始终选择 ePub 而非 PDF** - OCR 错误会变成模型学到的模式。
2. **使用段落级提取** - 从 `<p>` 标签提取以保留换行。
3. **移除前后文** - 版权页和目录会污染数据集。

```python
# 从 ePub 段落中提取文本
from epub2 import EPub
from bs4 import BeautifulSoup

def extract_epub(path):
    book = EPub(path)
    chapters = []
    for item in book.flow:
        html = book.get_chapter(item.id)
        soup = BeautifulSoup(html, 'html.parser')
        paragraphs = [p.get_text().strip() for p in soup.find_all('p')]
        chapters.append('\n\n'.join(p for p in paragraphs if p))
    return '\n\n'.join(chapters)
```

## 第二阶段：智能分段 (Intelligent Segmentation)

### 更小的块 + 重叠
相比于较大的块（250-650 词），更小的块（150-400 词）能产生更多的训练样本和更好的风格迁移效果。

```python
def segment(text, min_words=150, max_words=400):
    paragraphs = text.split('\n\n')
    chunks, buffer, buffer_words = [], [], 0
    
    for para in paragraphs:
        words = len(para.split())
        if buffer_words + words > max_words and buffer_words >= min_words:
            chunks.append('\n\n'.join(buffer))
            # 保留最后一个段落以进行重叠
            buffer = [buffer[-1], para] if buffer else [para]
            buffer_words = sum(len(p.split()) for p in buffer)
        else:
            buffer.append(para)
            buffer_words += words
    
    if buffer:
        chunks.append('\n\n'.join(buffer))
    return chunks
```

### 预期结果
对于一本 86,000 词的书：
- 旧方法（250-650 词）：约 150 个块
- 新方法（150-400 词 + 重叠）：约 300 个块
- 每个块 2 个变体：600+ 个训练样本

## 第三阶段：多样化指令生成 (Diverse Instruction Generation)

### 关键见解
使用单一的提示词模板会导致模型死记硬背。多样化的模板能教导模型掌握底层的风。

```python
SYSTEM_PROMPTS = [
    "你是一位能够模仿特定文学风格的创意写作专家。",
    "你是一位对经典散文风格有深厚了解的文学作家。",
    "你是一位擅长模仿独特作者语气的创意作家。",
    "你创作的散文能捕捉到现代主义文学的精髓。",
    "你是一位能传达经典美国作者神韵的天才作家。",
]

PROMPT_TEMPLATES = [
    "以 {author} 的风格写一段文字：{desc}",
    "模仿 {author} 的语气来描述：{desc}",
    "用 {author} 独特的散文风格描写：{desc}",
    "像 {author} 那样写出这个场景：{desc}",
    "运用 {author} 的重复性技巧来描述：{desc}",
    "在这段文字中捕捉 {author} 的节奏：{desc}",
    "像 {author} 一样写作：{desc}",
    "以 {author} 的口吻写下：{desc}",
    "这是一项文学练习。请模仿 {author} 进行写作：{desc}",
    "你能用 {author} 的风格写作吗？{desc}",
]
```

### 指令生成

```python
INSTRUCTION_PROMPT = """用 2-3 句话描述这段摘录中发生了什么。
重点关注：出现的角色、动作、情感、环境。
不要直接引用文本。

摘录：
{text}
"""

# 使用快速、廉价的 LLM（例如 Gemini Flash）
instruction = llm_call(INSTRUCTION_PROMPT.format(text=chunk))
```

## 第四阶段：数据集构建 (Dataset Construction)

### 消息格式

```json
{
    "messages": [
        {"role": "system", "content": "你是一位创意写作专家..."},
        {"role": "user", "content": "以作者的风格写作：场景描述..."},
        {"role": "assistant", "content": "来自块的实际书籍文本..."}
    ]
}
```

### 每个块对应多个变体

```python
def build_examples(chunk, instruction, author, variants=2):
    examples = []
    for i in range(variants):
        system = SYSTEM_PROMPTS[i % len(SYSTEM_PROMPTS)]
        template = PROMPT_TEMPLATES[(chunk.id + i) % len(PROMPT_TEMPLATES)]
        user = template.format(author=author, desc=instruction)
        examples.append({"messages": [
            {"role": "system", "content": system},
            {"role": "user", "content": user},
            {"role": "assistant", "content": chunk.text}
        ]})
    return examples
```

## 第五阶段：在 Tinker 上进行 LoRA 训练

### 配置

```python
CONFIG = {
    "model_name": "Qwen/Qwen3-8B-Base",  # 使用 Base 模型，而非 Instruct 版
    "lora_rank": 32,                      # 352MB 适配器
    "learning_rate": 5e-4,                # LoRA 通常使用较高的学习率
    "batch_size": 4,
    "epochs": 3,
}
```

### 为什么使用 Base 模型？
使用**基础**（预训练）模型，而非指令微调（Instruct）版本：
- 基础模型在塑造新风格方面更具延展性。
- 指令模型具有难以覆盖的既定模式。
- 风格是一种底层模式，基础模型能更好地捕捉它。

### 训练循环

```python
import tinker
from tinker import types

training_client = await service_client.create_lora_training_client_async(
    base_model="Qwen/Qwen3-8B-Base",
    rank=32
)

for epoch in range(3):
    for batch in batches:
        await training_client.forward_backward_async(batch, loss_fn="cross_entropy")
        await training_client.optim_step_async(types.AdamParams(learning_rate=5e-4))

result = await training_client.save_weights_for_sampler_async(name="final")
```

## 第六阶段：验证 (Validation)

### 现代场景测试 (Modern Scenario Test)
使用原书中不可能出现的场景进行测试：

```python
TEST_PROMPTS = [
    "描写一个正在做拿铁的咖啡师",
    "描述恋人通过短信进行交流",
    "描写一个对气候变化感到焦虑的人",
]
```
如果模型能将风格特征应用到现代场景中，说明它学到的是**风格**而非**内容**。

### 原创性验证

```bash
# 在训练数据中搜索输出的短语
grep "输出中的特定短语" dataset.jsonl
# 结果应该是：无匹配
```

### AI 检测器测试
使用 GPTZero, Pangram 或 ZeroGPT 测试输出。

## 已知问题及解决方案

### 角色名称泄漏
**症状**：模型在处理新场景时使用了原书中的角色名。
**原因**：单本书籍的角色名称多样性有限。
**解决方案**：使用多本书籍进行训练，或添加合成示例。

### 模型复读原文
**症状**：输出包含训练数据中的原句。
**原因**：提示词变体太少或训练轮数（epochs）过多。
**解决方案**：使用 15 个以上的模板，将训练轮数限制在 3 轮以内。

### 输出碎片化
**症状**：句子意群不完整。
**原因**：分段不当导致在思路上截断。
**解决方案**：始终在段落边界处截断。

## 指南板

1. **始终选择 ePub 而非 PDF** - OCR 错误会变成模型学到的模式。
2. **切勿在句中截断** - 边界必须在语法上是完整的。
3. **使用多样化的提示词** - 15+ 个模板，5+ 个系统提示词。
4. **使用基础（Base）模型** - 而非指令（Instruct）版本。
5. **使用更小的块** - 150-400 词以获得更多样本。
6. **保留测试集** - 至少 50 个样本。
7. **在现代场景上测试** - 证明是风格迁移而非死记硬背。
8. **验证原创性** - 在训练数据中 grep 输出短语。

## 预期结果

| 指标 | 数值 |
|--------|-------|
| 训练样本 | 每本书 500-1000 个 |
| 模型 | Qwen/Qwen3-8B-Base |
| LoRA rank | 32 |
| 适配器大小 | 约 350 MB |
| 训练时间 | 约 15 分钟 |
| Loss 下降 | 90% 以上 |
| 风格迁移成功率 | 约 50% 完美 |

## 成本预估

| 组件 | 成本 |
|-----------|------|
| LLM (指令生成) | 约 $0.50 |
| Tinker 训练 (15 分钟) | 约 $1.50 |
| **总计** | **约 $2.00** |

## 与上下文工程技能的集成

本示例应用了上下文工程助手技能集合中的多项技能：

### 项目开发 (project-development)
流水线遵循分阶段、幂等的架构模式：
- **获取 (Acquire)**：从 ePub 提取文本
- **准备 (Prepare)**：划分为训练块
- **处理 (Process)**：生成合成指令
- **解析 (Parse)**：构建消息格式
- **渲染 (Render)**：输出 Tinker 兼容的 JSONL
- **训练 (Train)**：LoRA 微调
- **验证 (Validate)**：现代场景测试

每个阶段都是可恢复的，并产生中间产物以便调试。

### 上下文压缩 (context-compression)
分段是训练时的一种上下文压缩形式。应用了来自“上下文压缩”的核心见解：信息密度比信息数量更重要。更小、连贯的块（150-400 词）比更大、更稀释的块能产生更好的风格迁移效果。

该双层策略镜像了上下文压缩评估：
- 第 1 层：快速、确定性的压缩
- 第 2 层：针对边缘情况的 LLM 辅助压缩

### 多智能体模式 (multi-agent-patterns)
流水线使用了**主管/协调者**模式：
- 协调者负责各阶段协调并管理状态。
- 专业智能体（提取、分段、指令、构建器）拥有隔离的上下文。
- 每个智能体仅接收完成其任务所需的信息。

这符合“子智能体存在的主要目的是隔离上下文而非模拟角色”的原则。

### 评估 (evaluation)
验证遵循**终态评估**模式：
- 功能测试：输出是否符合预期的风格特征？
- 原创性验证：内容是否是真实生成的？
- 外部验证：AI 检测器评分

“现代场景”测试是一种出域（out-of-distribution）评估形式，证明了模型的泛化能力。

### 上下文基础 (context-fundamentals)
提示词多样性防止了注意力在单一模式上坍缩。当使用完全相同的提示词结构进行训练时，模型会记住指令-响应的映射关系。多样化的模板强制注意力分散到风格模式本身上。

## 参考资料

内部参考：
- [分段策略](./references/segmentation-strategies_zh.md) - 文本块划分模式
- [Tinker 格式规范](./references/tinker-format_zh.md) - 数据项结构
- [Tinker API 文档](./references/tinker_zh.txt) - 完整 API 参考

上下文工程助手技能中的相关技能：
- project-development - 流水线架构模式
- context-compression - 压缩策略
- multi-agent-patterns - 智能体协调
- evaluation - 评估框架
- context-fundamentals - 注意力与信息密度

外部资源：
- [研究论文](https://arxiv.org/pdf/2510.13939) - Chakrabarty et al. 2025
- [Hugging Face 上的数据集](https://huggingface.co/datasets/MuratcanKoylan/gertrude-stein-style-sft)
- [Gertrude Stein 案例研究](./examples/gertrude-stein/README_zh.md) - 完整的运行示例

---

## 技能元数据

**创建日期**: 2025-12-26
**最近更新**: 2025-12-28
**作者**: Muratcan Koylan
**版本**: 2.0.0
**是否独立**: 是（独立于主上下文工程集合）
