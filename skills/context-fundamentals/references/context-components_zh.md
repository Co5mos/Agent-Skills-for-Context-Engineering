# 上下文组件：技术参考 (Context Components: Technical Reference)

本文档为智能体系统中的每个上下文组件提供详细的技术参考。

## 系统提示词工程 (System Prompt Engineering)

### 结构设计
将系统提示词组织成界限清晰的不同部分。推荐结构如下：

```xml
<BACKGROUND_INFORMATION>
关于领域、用户偏好或项目特定细节的上下文
</BACKGROUND_INFORMATION>

<INSTRUCTIONS>
核心行为准则和任务指令
</INSTRUCTIONS>

<TOOL_GUIDANCE>
何时以及如何使用可用工具
</TOOL_GUIDANCE>

<OUTPUT_DESCRIPTION>
预期的输出格式和质量标准
</OUTPUT_DESCRIPTION>
```

这种结构允许助手快速定位相关信息，并在高级实现中实现选择性的上下文加载。

### 高度校准 (Altitude Calibration)
指令的“高度”指的是抽象级别。参考以下示例：

**过低（脆性）：**
```
如果用户询问价格，请查看 docs/pricing.md 中的价格表。
如果表显示为美元，请使用 config/exchange_rates.json 中的汇率转换为欧元。
如果用户在欧盟，请按 config/vat_rates.json 中的适用税率添加增值税。
格式化回复时包含货币符号、两位小数以及关于增值税的说明。
```

**过高（模糊）：**
```
帮助用户解决价格问题。要提供有帮助且准确的信息。
```

**最佳（启发式驱动）：**
```
对于价格查询：
1. 从 docs/pricing.md 获取当前汇率。
2. 应用位置调整（见 config/location_defaults.json）。
3. 使用适当的货币和税务考量进行格式化。

优先提供确切数字而非估算。如果无法获取汇率，请明确说明而非随意推测。
```

最佳高度提供了清晰的步骤，同时在执行过程中保持了灵活性。

## 工具定义规范 (Tool Definition Specification)

### Schema 结构
每个工具应定义如下：

```python
{
    "name": "tool_function_name",
    "description": "清晰描述工具的功能以及何时使用它",
    "parameters": {
        "type": "object",
        "properties": {
            "param_name": {
                "type": "string",
                "description": "该参数控制什么",
                "default": "合理的默认值"
            }
        },
        "required": ["param_name"]
    },
    "returns": {
        "type": "object",
        "description": "工具返回什么及其结构"
    }
}
```

### 描述工程
工具描述应回答：工具做什么、何时使用以及产生什么。包括使用背景、示例和边缘情况。

**弱描述：**
```
搜索数据库获取客户信息。
```

**强描述：**
```
通过 ID 或电子邮件检索客户信息。

适用场景：
- 用户询问特定客户的详情、历史或状态。
- 用户提供了客户标识符并需要相关信息。

返回包含以下内容的客户对象：
- 基本信息（姓名、邮箱、账户状态）
- 订单历史摘要
- 服务单数量

如果未找到客户则返回 null。如果数据库不可达则返回错误。
```

## 检索文档管理 (Retrieved Document Management)

### 标识符设计
设计具有语义且支持高效检索的标识符：

**差的标识符：**
- `data/file1.json`
- `ref/ref.md`

**强的标识符：**
- `customer_pricing_rates.json`
- `engineering_onboarding_checklist.md`

强的标识符允许助手即使不使用搜索工具也能定位到相关文件。

### 文档切片策略 (Document Chunking Strategy)
对于大型文档，应有策略地切片以保持语义连贯性：

```python
# 语义切片的伪代码
def chunk_document(content):
    """在自然语义边界处拆分文档。"""
    boundaries = find_section_headers(content)
    boundaries += find_paragraph_breaks(content)
    boundaries += find_logical_breaks(content)
    
    chunks = []
    # 根据 MIN_CHUNK_SIZE 和 MAX_CHUNK_SIZE 进行过滤和拆分
    # ...
    return chunks
```

避免使用强制的字符限制，以免在中途拆分句子或概念。

## 消息历史管理 (Message History Management)

### 轮次表示 (Turn Representation)
结构化消息历史以保留关键信息：

```python
{
    "role": "user" | "assistant" | "tool",
    "content": "消息文本",
    "reasoning": "可选的思维链",
    "tool_calls": [...],
    "tool_output": "...",
    "summary": "如果对话很长，提供紧凑摘要"
}
```

### 摘要注入模式 (Summary Injection Pattern)
对于长对话，每隔一段时间注入一次摘要：

```python
def inject_summaries(messages, summary_interval=20):
    """定期注入摘要以保留上下文。"""
    # 逻辑：每过 summary_interval 轮，生成前一段的摘要并注入一条 system 消息
    # ...
```

## 工具输出优化 (Tool Output Optimization)

### 响应格式
提供响应格式选项以控制 token 使用：

```python
def get_customer_response_format():
    return {
        "format": "concise | detailed", # 简洁 | 详细
        "fields": ["id", "name", "email", "status", "history_summary"]
    }
```

### 观察掩码 (Observation Masking)
对于冗长的工具输出，考虑使用遮蔽模式：

```python
def mask_observation(output, max_length=500):
    """用紧凑的引用替换长的观察结果。"""
    if len(output) <= max_length:
        return output
    
    reference_id = store_observation(output)
    return f"[之前的观察结果已省略。完整内容存储在引用 {reference_id} 中]"
```

## 上下文预算估算 (Context Budget Estimation)

### 上下文预算分配

| 组件 | 典型范围 | 备注 |
| :--- | :--- | :--- |
| 系统提示词 | 500-2000 tokens | 会话期间保持稳定 |
| 工具定义 | 每个工具 100-500 | 随工具数量增长 |
| 检索文档 | 变动 | 通常是最大的消耗者 |
| 消息历史 | 变动 | 随对话增长 |
| 工具输出 | 变动 | 可能主导上下文 |

## 渐进式披露实现 (Progressive Disclosure Implementation)

### 技能激活模式
```python
def activate_skill_context(skill_name, task_description):
    """当任务匹配技能描述时加载技能上下文。"""
    # 仅为最相关的技能加载完整内容
```

### 引用加载模式
```python
def get_reference(file_reference):
    """仅在明确需要时加载参考文件。"""
    if not file_reference.is_loaded:
        file_reference.content = read_file(file_reference.path)
        file_reference.is_loaded = True
    return file_reference.content
```

这种模式确保文件在会话中只加载一次并进行缓存。
