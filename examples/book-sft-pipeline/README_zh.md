# 书籍 SFT 流水线 (Book SFT Pipeline)

这是一个用于训练语言模型模仿任何作者文风的独立技能。它是主上下文工程（Context Engineering）技能集合的一个**独立插件**。

## 安装

### Claude Code

```bash
# 首先添加 marketplace
/plugin marketplace add muratcankoylan/Agent-Skills-for-Context-Engineering

# 安装 book-sft-pipeline 插件
/plugin install book-sft-pipeline@context-engineering-marketplace
```

### Cursor / Codex / IDE

将 `SKILL.md` 复制到你的 `.rules` 或项目技能文件夹中。

### 手动

在智能体的上下文中直接引用 `SKILL.md` 文件。

## 包含内容

```
book-sft-pipeline/
├── README.md                 # 本文件
├── SKILL.md                  # 完整的技能文档（独立）
├── examples/
│   └── gertrude-stein/       # 包含真实输出的完整案例研究
│       ├── README.md         # 结果与分析
│       ├── sample_outputs.md # 原始模型输出
│       ├── training_config.json
│       ├── dataset_sample.jsonl
│       └── pangram/          # AI 检测器截图
├── scripts/
│   └── pipeline_example.py   # 概念性实现
└── references/
    ├── segmentation-strategies.md
    ├── tinker-format.md
    └── tinker.txt
```

## 关键结果

基于 Gertrude Stein 的 "Three Lives" (1909) 训练 Qwen3-8B-Base 的结果：

| 指标 | 数值 |
|--------|-------|
| 训练样本数 | 592 |
| 损失 (Loss) 降幅 | 97% |
| Pangram AI 检测器 | 70% 人类撰写 |
| 训练时间 | 15 分钟 |
| 总成本 | $2 |

## 相关的上下文工程技能

本技能应用了 [上下文工程智能体技能集合](../../README_zh.md) 中的模式：

| 技能 | 应用 |
|-------|-------------|
| [项目开发](../../skills/project-development/SKILL_zh.md) | 分阶段流水线架构 |
| [上下文压缩](../../skills/context-compression/SKILL_zh.md) | 分段策略 |
| [多智能体模式](../../skills/multi-agent-patterns/SKILL_zh.md) | 编排者模式 |
| [评估](../../skills/evaluation/SKILL_zh.md) | 现代场景测试 |
| [上下文基础](../../skills/context-fundamentals/SKILL_zh.md) | 提示词多样性 |

## 资源

- [Hugging Face 上的数据集](https://huggingface.co/datasets/MuratcanKoylan/gertrude-stein-style-sft)
- [研究论文](https://arxiv.org/pdf/2510.13939) (Chakrabarty et al. 2025)

## 许可证

MIT
