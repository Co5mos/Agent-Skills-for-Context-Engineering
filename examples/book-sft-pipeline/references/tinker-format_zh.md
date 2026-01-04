# Tinker 格式规范 (Tinker Format Specification)

本参考文档详细记录了进行 Tinker 监督式微调 (SFT) 所需的精确数据结构。

## 核心数据类型

### Datum (数据单元)

Tinker 中的基本训练单元：

```python
from tinker import types

datum = types.Datum(
    model_input=types.ModelInput.from_ints(tokens=input_tokens),
    loss_fn_inputs={
        "target_tokens": target_tokens,  # 列表[int] - 为了预测下一个 token 需向后偏移 1 位
        "weights": weights               # 列表[float] - 提示词为 0.0，补全内容为 1.0
    }
)
```

### ModelInput (模型输入)

分词后输入的容器：

```python
# 仅文本输入
model_input = types.ModelInput.from_ints(tokens=[...])

# 多模态输入 (适用于 VLM)
model_input = types.ModelInput(chunks=[
    types.EncodedTextChunk(tokens=[...]),
    types.ImageChunk(data=image_bytes, format="png"),
    types.EncodedTextChunk(tokens=[...])
])
```

### Token 权重分配

权重数组决定了哪些 token 会参与损失函数的计算：

| Token 类型 | 权重 | 描述 |
|------------|--------|-------------|
| 系统提示词 (System prompt) | 0.0 | 背景信息，不参与学习 |
| 用户消息 (User message) | 0.0 | 输入提示词 |
| 助手回复 (Assistant message) | 1.0 | 目标补全内容 |
| 特殊 token | 0.0 | EOS, BOS, 分隔符等 |

## 渲染器系统 (Renderer System)

Tinker 使用渲染器将消息列表转换为带有正确权重的 token。

### 使用内置渲染器

```python
from tinker_cookbook import renderers, tokenizer_utils

# 获取对应模型的分词器 (Tokenizer)
tokenizer = tokenizer_utils.get_tokenizer("meta-llama/Llama-3.1-8B-Instruct")

# 获取相应的渲染器
renderer = renderers.get_renderer("llama3", tokenizer)

# 将消息转换为训练格式
messages = [
    {"role": "system", "content": "你是一位创意作家……"},
    {"role": "user", "content": "写一段 500 字的摘录……"},
    {"role": "assistant", "content": "实际的书籍内容……"}
]

model_input, weights = renderer.build_supervised_example(messages)
```

### 渲染器输出可视化

渲染器为每个 token 分配权重：

```
Token          Weight (权重)
<|im_start|>   0.0
system         0.0
\n             0.0
You are...     0.0
<|im_end|>     0.0
...            ...
<|im_start|>   0.0
assistant      0.0
\n             0.0
The actual     1.0    <- 补全内容开始
book text      1.0
...            1.0
<|im_end|>     1.0    <- 最后一个 token 被加权
```

## JSONL 格式

对于批量处理，使用标准的对话 JSONL 格式：

```json
{"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
{"messages": [{"role": "system", "content": "..."}, {"role": "user", "content": "..."}, {"role": "assistant", "content": "..."}]}
```

### 将 JSONL 转换为 Datum

```python
import json
from tinker import types
from tinker_cookbook import renderers, tokenizer_utils

def load_dataset(jsonl_path: str, model_name: str) -> list[types.Datum]:
    """加载 JSONL 并转换为 Tinker Datum 对象。"""
    
    tokenizer = tokenizer_utils.get_tokenizer(model_name)
    renderer = renderers.get_renderer("llama3", tokenizer)
    
    data = []
    with open(jsonl_path) as f:
        for line in f:
            example = json.loads(line)
            messages = example["messages"]
            
            model_input, weights = renderer.build_supervised_example(messages)
            
            # 获取 token 序列
            input_tokens = model_input.to_ints()
            target_tokens = input_tokens[1:]  # 偏移以进行下一个 token 预测
            input_tokens = input_tokens[:-1]
            weights = weights[1:]  # 使权重与目标对齐
            
            datum = types.Datum(
                model_input=types.ModelInput.from_ints(tokens=input_tokens),
                loss_fn_inputs={
                    "target_tokens": target_tokens,
                    "weights": weights
                }
            )
            data.append(datum)
    
    return data
```

## 训练循环集成

```python
import tinker
from tinker import types

async def train_on_book_dataset(
    dataset: list[types.Datum],
    model_name: str,
    learning_rate: float = 1e-4,
    epochs: int = 1
):
    """在书籍 SFT 数据集上进行训练。"""
    
    service_client = tinker.ServiceClient()
    training_client = await service_client.create_lora_training_client_async(
        base_model=model_name,
        rank=32
    )
    
    for epoch in range(epochs):
        for batch_start in range(0, len(dataset), 1):  # 批次大小为 1
            batch = dataset[batch_start:batch_start + 1]
            
            # 使用交叉熵损失进行前向-后向传播
            fwd_bwd_future = await training_client.forward_backward_async(
                batch, 
                loss_fn="cross_entropy"
            )
            
            # 使用更激进的学习率进行优化器步进
            optim_future = await training_client.optim_step_async(
                types.AdamParams(learning_rate=learning_rate * 2.0)
            )
            
            # 等待完成
            fwd_bwd_result = await fwd_bwd_future
            optim_result = await optim_future
```

## 核心限制与约束

1. **批次大小 (Batch Size)**：进行风格迁移时请使用 1。较大的批次会平均掉风格梯度的特征。

2. **序列长度**：保持数据块在 1000 个 token 以内。过长的序列会稀释局部的风格模式。

3. **学习率**：使用 2 倍乘数（例如使用 2e-4 而非 1e-4）以实现更快的风格收敛。

4. **Token 对齐**：目标 token 必须相对于输入 token 向后偏移 1 个位置。

5. **权重精度**：权重应为 float32 类型，通常取值 0.0 或 1.0。

## 模型选择

对于书籍 SFT 任务，请考虑：

| 模型 | 使用场景 |
|-------|----------|
| meta-llama/Llama-3.1-8B-Instruct | 通用的风格迁移 |
| Qwen/Qwen3-30B-A3B | 更高质量，MoE 效率 |
| GPT-4o (通过 OpenAI) | 仅用于生成数据，而非 Tinker 微调 |

## 参考文献

- Tinker Cookbook: `tinker_cookbook/supervised/train.py`
- 渲染器实现: `tinker_cookbook/renderers.py`
- 类型定义: `tinker/types.py`
