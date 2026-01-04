# 文本分段策略 (Segmentation Strategies)

本指南介绍了如何将书籍拆分为训练数据块 (Training Chunks) 的进阶模式，同时确保叙事的连贯性。

## 分段中的核心挑战

书籍在创建训练数据时面临独特的挑战：

1. **段落长度多变**：有些作家的单个段落可能长达 1000 字以上。
2. **对话密集章节**：简短的对话交替，单个看往往太小而无法构成有效的训练块。
3. **场景界限**：由于叙事节奏，自然的断点可能与字数统计互不匹配。
4. **文体变化**：作家在叙述、对话和阐述之间可能会切换语调、声音。

糟糕的分段会导致模型学会产生：
- 不完整的思想表达。
- 突兀的结尾。
- 支离破碎的转场。
- 破碎的风格模式。

## 分层策略 (Two-Tier Strategy)

### 第一层：基于段落的累加

对于结构良好的文本，这是默认采用的方法：

```python
class Tier1Segmenter:
    def __init__(self, min_words: int = 250, max_words: int = 650):
        self.min_words = min_words
        self.max_words = max_words
    
    def segment(self, text: str) -> list[Chunk]:
        paragraphs = self._split_paragraphs(text)
        chunks = []
        current = ChunkBuilder()
        
        for para in paragraphs:
            word_count = len(para.split())
            
            # 检查单个段落是否超过最大限制
            if word_count > self.max_words:
                # 如果当前已有累积的内容，先将其封存为一个数据块
                if current.word_count > 0:
                    chunks.append(current.build())
                    current = ChunkBuilder()
                
                # 将此超长段落标记为需经由第二层策略处理
                chunks.append(Chunk(
                    text=para,
                    requires_tier2=True,
                    word_count=word_count
                ))
                continue
            
            # 判断加入此段落是否会导致当前块溢出
            if current.word_count + word_count > self.max_words:
                if current.word_count >= self.min_words:
                    chunks.append(current.build())
                    current = ChunkBuilder()
            
            current.add(para)
        
        # 处理最后一个残余块
        if current.word_count > 0:
            chunks.append(current.build())
        
        return chunks
    
    def _split_paragraphs(self, text: str) -> list[str]:
        # 按双换行符拆分，保留段落内部的单换行符
        paragraphs = text.split('\n\n')
        return [p.strip() for p in paragraphs if p.strip()]
```

### 第二层：LLM 辅助分段

针对无法在段落边界处简单拆分的超长段落：

```python
class Tier2Segmenter:
    def __init__(self, model: str = "gpt-4o"):
        self.model = model
        self.prompt_template = self._load_prompt()
    
    async def segment(self, oversized_chunk: Chunk) -> list[Chunk]:
        """使用 LLM 拆分超长段落。"""
        
        response = await self._call_llm(
            self.prompt_template.format(text=oversized_chunk.text)
        )
        
        segments = self._parse_segments(response)
        
        # 验证是否做到了“零删除”
        original_words = len(oversized_chunk.text.split())
        segmented_words = sum(len(s.split()) for s in segments)
        
        if abs(original_words - segmented_words) > 5:  # 允许极微小的字数误差
            raise SegmentationError(
                f"字数统计不匹配: {original_words} -> {segmented_words}"
            )
        
        return [
            Chunk(text=s, requires_tier2=False, word_count=len(s.split()))
            for s in segments
        ]
    
    def _load_prompt(self) -> str:
        return """将这段文本拆分为至少 300-350 字的摘录。

要求：
- 每段摘录从开头起必须在语法上是完整的。
- 每段摘录不能让人感到突兀的中断。
- 零删除 - 必须完全保留原始的总字数。
- 在语法自然的断点处拆分：
  * 在完整的对话往来之后。
  * 在场景转场处。
  * 在完整的想法或描写之后。
  * 在段落换行符自然发生的地方。
- 避免拆分为过多的细小摘录。
- 直接以摘录内容开始输出。
- 使用 ===SEGMENT=== 分隔各段摘录。

待拆分文本：
{text}
"""
    
    def _parse_segments(self, response: str) -> list[str]:
        segments = response.split("===SEGMENT===")
        return [s.strip() for s in segments if s.strip()]
```

## 场景感知分段 (Scene-Aware Segmentation)

为了获得更高质量的结果，可以检测场景边界：

```python
class SceneAwareSegmenter:
    """在字数限制内，优先在场景边界处断开。"""
    
    SCENE_MARKERS = [
        r'\n\n\* \* \*\n\n',      # 星号分隔符
        r'\n\n---\n\n',            # 横线分隔符
        r'\n\n###\n\n',            # 井号分隔符
        r'\n\nCHAPTER \d+',        # 章节标题
        r'\n\n[A-Z]{3,}\n\n',      # 全大写的场景分隔标识
    ]
    
    def find_scene_breaks(self, text: str) -> list[int]:
        """查找场景分隔符的字符位置。"""
        breaks = []
        for pattern in self.SCENE_MARKERS:
            for match in re.finditer(pattern, text):
                breaks.append(match.start())
        return sorted(set(breaks))
    
    def segment_with_scenes(self, text: str) -> list[Chunk]:
        scene_breaks = self.find_scene_breaks(text)
        
        # 如果存在场景分隔符，优先在此处断开，而非随意的段落处
        if scene_breaks:
            return self._segment_at_scenes(text, scene_breaks)
        else:
            return Tier1Segmenter().segment(text)
```

## 对话处理 (Dialogue Handling)

对话密集的章节需要特殊处理：

```python
class DialogueAwareSegmenter:
    """对一组对话往来进行分组，以保持对话的连贯性。"""
    
    def is_dialogue_paragraph(self, para: str) -> bool:
        """检查段落是否主要由对话组成。"""
        # 统计对话标识符
        quote_count = para.count('"') + para.count("'")
        word_count = len(para.split())
        
        # 如果超过 20% 的单词处于引号内，则认为是对话密集型段落
        return quote_count > word_count * 0.2
    
    def segment(self, text: str) -> list[Chunk]:
        paragraphs = text.split('\n\n')
        chunks = []
        current = ChunkBuilder()
        in_dialogue_block = False
        
        for para in paragraphs:
            is_dialogue = self.is_dialogue_paragraph(para)
            
            # 不要在对话往来的中间部分断开
            if is_dialogue:
                in_dialogue_block = True
                current.add(para)
            else:
                if in_dialogue_block:
                    # 对话块结束 - 这是一个极好的断点
                    in_dialogue_block = False
                    if current.word_count >= 250:
                        chunks.append(current.build())
                        current = ChunkBuilder()
                
                current.add(para)
                
                # 检查是否超过了最大字数
                if current.word_count > 650:
                    chunks.append(current.build())
                    current = ChunkBuilder()
        
        if current.word_count > 0:
            chunks.append(current.build())
        
        return chunks
```

## 验证流水线 (Validation Pipeline)

每一个分段结果都应通过验证：

```python
class SegmentationValidator:
    def validate(self, chunks: list[Chunk]) -> ValidationResult:
        errors = []
        warnings = []
        
        for i, chunk in enumerate(chunks):
            # 检查字数边界
            if chunk.word_count < 200:
                warnings.append(f"数据块 {i}: 仅有 {chunk.word_count} 字")
            if chunk.word_count > 700:
                errors.append(f"数据块 {i}: {chunk.word_count} 字超过了最大限制")
            
            # 检查句子完整性
            if not self._ends_with_terminal(chunk.text):
                errors.append(f"数据块 {i}: 在句中戛然而止")
            
            if not self._starts_grammatically(chunk.text):
                errors.append(f"数据块 {i}: 在句中开始")
            
            # 检查孤立的引号
            if chunk.text.count('"') % 2 != 0:
                warnings.append(f"数据块 {i}: 引号不匹配")
        
        return ValidationResult(
            valid=len(errors) == 0,
            errors=errors,
            warnings=warnings
        )
    
    def _ends_with_terminal(self, text: str) -> bool:
        text = text.strip()
        return text[-1] in '.!?"\'—'
    
    def _starts_grammatically(self, text: str) -> bool:
        text = text.strip()
        # 应以大写字母或引号开头
        return text[0].isupper() or text[0] in '"\'—'
```

## 性能考量

| 策略 | 速度 | 质量 | 适用场景 |
|----------|-------|---------|----------|
| 仅第一层 (Tier 1) | 极快 | 中等 | 结构良好的散文 |
| 第一层 + 第二层 | 中等 | 高 | 段落长度参差不齐的文本 |
| 场景感知型 | 快 | 高 | 有清晰场景分隔的推演类小说 |
| 对话感知型 | 中等 | 高 | 对话较多的小说 |

## 边缘情况处理

**1. 意识流写作**
- 单个段落跨越数页。
- 解决方案：强制使用配备了精确句子边界检测的第二层策略。

**2. 诗歌或韵文**
- 换行具有语义价值，而非排版格式。
- 解决方案：将每一段 (Stanza) 视为不可分割的原子单位。

**3. 带有列表/要点的非虚构类文本**
- 列表项会干扰段落检测。
- 解决方案：预处理，将列表项转化为连贯的叙述文本。

**4. 视角切换 (多位叙述者)**
- 章节点内可能发生语声切换。
- 解决方案：检测叙述者标识，并优先在此处断开。

## 流水线集成示例

```python
class SegmentationAgent:
    def __init__(self, config: SegmentationConfig):
        self.tier1 = Tier1Segmenter(
            min_words=config.min_words,
            max_words=config.max_words
        )
        self.tier2 = Tier2Segmenter(model=config.tier2_model)
        self.validator = SegmentationValidator()
    
    async def segment(self, text: str) -> list[Chunk]:
        # 第一阶段：执行第一层分段
        chunks = self.tier1.segment(text)
        
        # 第二阶段：使用第二层处理超长数据块
        final_chunks = []
        for chunk in chunks:
            if chunk.requires_tier2:
                sub_chunks = await self.tier2.segment(chunk)
                final_chunks.extend(sub_chunks)
            else:
                final_chunks.append(chunk)
        
        # 第三阶段：执行验证
        result = self.validator.validate(final_chunks)
        if not result.valid:
            raise SegmentationError(result.errors)
        
        if result.warnings:
            logger.warning(f"分段警告: {result.warnings}")
        
        return final_chunks
```
