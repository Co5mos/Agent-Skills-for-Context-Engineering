# 示例：格特鲁德·斯泰因风格迁移 (Gertrude Stein Style Transfer)

这是一个完整的案例研究，展示了如何使用格特鲁德·斯泰因 (Gertrude Stein) 1909 年的小说《三个女人》(Three Lives) 训练 Qwen3-8B-Base 模型，使其学会斯泰因独特的写作风格。

## 项目概览

| 指标 | 数值 |
|--------|-------|
| 原始书籍 | 《三个女人》(1909) |
| 书籍字数 | 约 86,000 字 |
| 训练样本数 | 592 |
| 测试样本数 | 50 |
| 基础模型 | Qwen/Qwen3-8B-Base |
| LoRA Rank | 32 |
| Adapter 大小 | 352 MB |
| 训练耗时 | 约 15 分钟 |
| 总成本 | 约 2 美元 |

## 训练指标

### 损失函数轨迹 (Loss Trajectory)

| 步数 | 训练损失 | 测试损失 |
|------|-----------|-----------|
| 5 | 7579.67 | - |
| 40 | 4032.35 | 7584.85 |
| 100 | 2392.17 | 2127.44 |
| 140 | 2863.17 | 752.95 |
| 180 | 1327.41 | 229.88 |
| 200 | 1044.49 | 213.36 |
| 最终 | 约 800 | 约 200 |

损失降幅：97%（测试损失从 7584 降至 213）

## 习得的风格特征

模型成功掌握了格特鲁德·斯泰因的以下特征：

1. **重复的句子结构**："She was a good woman. She was always a good woman."（她是个好女人。她总是个好女人。）
2. **简单的词汇**：使用常用词，没有华丽修饰的辞藻。
3. **逗号分隔的形容词**："a dark, sweet, little, pretty girl"（一个黝黑、甜美、娇小、漂亮的女孩）。
4. **现在进行时态**：惯用 "She was always doing", "He was thinking"。
5. **角色聚焦**：比起情节，更注重深刻的心理观察。
6. **节奏感与催眠感**：近乎音乐律动的散文体。

## 输出样本

### 现代场景：房地产办公室

**提示词**：以格特鲁德·斯泰因的风格，描写一个在房地产办公室工作的办事员疲惫地回到家。

**输出内容**：
> It was a very busy day for the clerk in the real estate office. He came home to his small house in the working class part of the town, very tired. He went into the kitchen where his wife was cooking.
>
> "Hi, Sis, home for dinner?" he said to her.
>
> She looked at him and saw that he was very tired. She looked at him and then looked away into the fire. She said nothing in answer to him.
>
> He took off his hat and coat and then sat down on the steps just inside the door to take a rest. He was very tired today, he sat there and just sort of dozed a little.

**分析**：原创内容核实 —— "real estate office"（房地产办公室）, "working class"（工人阶级）, "Hi, Sis" 等词汇在训练数据中并未出现。

### 现代场景：短信

**提示词**：描写两个通过短信进行交流的恋人。

**输出内容**：
> "Hey, Herman, I just got a minute to be gone from my patient. Want to talk to you for a little while, Herman," Lena wrote one night, and then she sent the text to him.
>
> Herman read it and then he wrote her back, "I was just getting ready to go to bed, Lena. Want me to come and see you tomorrow, Lena. Want me to bring something to you, Lena. Want me to bring you some ice cream, Lena. Want me to bring you some of those little donuts..."

**分析**：将斯泰因经典的“首句重复”手法完美应用到了现代的短信格式中。

## AI 检测器结果

使用 Pangram AI 检测器进行测试：**100% 人类撰写 (100% Human Written)**。

测试了多个样本，所有样本均被评分为人类撰写的散文。

## 验证方法论

### 现代场景测试

我们在 1909 年不可能存在的场景中测试了该模型：
- 制作拿铁的咖啡师
- 刷社交媒体
- 视频通话
- 外卖配送员
- 气候变化焦虑

当风格特征出现在现代背景下时，证明模型学会的是**风格**而非单纯的**内容**。

### 原创性校验

在训练数据中搜索输出的短语，确保非记忆（Memorization）：

```bash
grep "real estate office" dataset.jsonl    # 无匹配
grep "working class" dataset.jsonl          # 无匹配
grep "Hi, Sis" dataset.jsonl                # 无匹配
grep "text messages" dataset.jsonl          # 无匹配
```

## 已知局限性

### 角色名称泄漏 (约 30% 的输出)

模型有时会在现代场景中沿用原著中的角色名称 (Melanctha, Mrs. Lehntman, Anna)。这是因为从一本书提取的 592 个样本意味着这些名字出现了数百次。

**缓解措施**：使用同一作者的多本书籍进行训练，或者加入使用不同名字的合成采样数据。

### 成功率分布

- 完美风格迁移：约 50%
- 带名称泄漏的风格迁移：约 30%
- 部分风格迁移：约 15%
- 失败：约 5%

对于仅在一本书上训练的 8B 模型来说，50% 的完美率是符合预期的。

## 采用的配置

### 数据集生成配置

```python
CONFIG = {
    "min_words": 150,
    "max_words": 400,
    "overlap": True,  # 最后一个段落会带入下一个数据块
    "variants_per_chunk": 2,
    "prompt_templates": 15,
    "system_prompts": 5,
    "instruction_model": "gemini-2.0-flash-lite",
}
```

### 训练配置

```python
CONFIG = {
    "model_name": "Qwen/Qwen3-8B-Base",
    "lora_rank": 32,
    "learning_rate": 5e-4,
    "batch_size": 4,
    "epochs": 3,
    "eval_every": 20,
    "save_every": 50,
}
```

## 关键收获

1. **更小的数据块效果更好**：150-400 字比 250-650 字能产生更多训练样本，且风格迁移效果更佳。
2. **提示词多样性至关重要**：15 个模板 × 5 个系统提示词 = 75 种变化，有效防止了模型过度记忆内容。
3. **基础模型优于指令微调模型**：Qwen3-8B-Base 比指令微调 (Instruct) 版本更具可塑性。
4. **现代场景测试证明了迁移能力**：如果风格能应用到现代背景，说明模型掌握的是模式而非内容。
5. **约 2 美元即可解决**：用于生成指令的 LLM 调用约 0.50 美元，加上 Tinker 训练费用约 1.50 美元。

## 相关文件

- `sample_outputs.md` —— 完整的模型输出与分析。
- `training_config.json` —— 采用的精确配置方案。
- `dataset_sample.jsonl` —— 训练样本示例。
 luxury
