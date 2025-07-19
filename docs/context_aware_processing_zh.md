# RAGAnything 中的上下文感知多模态处理

本文档描述了 RAGAnything 中的上下文感知多模态处理功能，该功能在分析图像、表格、方程和其他多模态内容时，为大型语言模型（LLM）提供周围内容信息，从而提升准确性和相关性。

## 概述

上下文感知功能使 RAGAnything 在处理多模态内容时，能够自动提取并提供周围文本内容作为上下文。这为 AI 模型提供了关于内容在文档结构中位置和关系的额外信息，从而实现更准确且与上下文相关的分析。

### 主要优势

- **准确性提升**：上下文帮助 AI 理解多模态内容的目的和含义
- **语义连贯性**：生成的描述与文档上下文和术语保持一致
- **自动化集成**：文档处理期间自动启用上下文提取
- **灵活配置**：支持多种提取模式和过滤选项

## 核心功能

### 1. 配置支持
- **集成化配置**：完整的上下文选项集成在 `RAGAnythingConfig` 中
- **环境变量支持**：通过环境变量配置所有上下文参数
- **动态更新**：支持运行时配置更新
- **内容格式控制**：可配置的内容源格式检测

### 2. 自动化集成
- **自动初始化**：模态处理器自动接收分词器和上下文配置
- **内容源设置**：文档处理自动设置上下文提取的内容源
- **位置信息传递**：自动向处理器传递位置信息（page_idx, index）
- **批处理支持**：支持上下文感知的批量文档处理

### 3. 高级分词管理
- **精准分词计数**：使用 LightRAG 的分词器进行精确分词计算
- **智能边界保留**：在句子/段落边界处截断
- **向后兼容**：分词器不可用时回退到字符截断

### 4. 通用上下文提取
- **多格式支持**：支持 MinerU、纯文本、自定义格式
- **灵活模式**：基于页面和基于分块的上下文提取
- **内容过滤**：可配置的内容类型过滤
- **标题支持**：可选包含文档标题和结构

## 配置

### RAGAnythingConfig 参数

```python
# 上下文提取配置
context_window: int = 1                    # 上下文窗口大小（页面/分块）
context_mode: str = "page"                 # 上下文模式（"page" 或 "chunk"）
max_context_tokens: int = 2000             # 最大上下文 token 数
include_headers: bool = True               # 包含文档标题
include_captions: bool = True              # 包含图像/表格标题
context_filter_content_types: List[str] = ["text"]  # 包含的内容类型
content_format: str = "minerU"             # 上下文提取的默认内容格式
```

### 环境变量

```bash
# 上下文提取设置
CONTEXT_WINDOW=2
CONTEXT_MODE=page
MAX_CONTEXT_TOKENS=3000
INCLUDE_HEADERS=true
INCLUDE_CAPTIONS=true
CONTEXT_FILTER_CONTENT_TYPES=text,image
CONTENT_FORMAT=minerU
```

## 使用指南

### 1. 基础配置

```python
from raganything import RAGAnything, RAGAnythingConfig

# 创建带上下文设置的配置
config = RAGAnythingConfig(
    context_window=2,
    context_mode="page",
    max_context_tokens=3000,
    include_headers=True,
    include_captions=True,
    context_filter_content_types=["text", "image"],
    content_format="minerU"
)

# 创建 RAGAnything 实例
rag_anything = RAGAnything(
    config=config,
    llm_model_func=your_llm_function,
    embedding_func=your_embedding_function
)
```

### 2. 自动文档处理

```python
# 文档处理期间自动启用上下文
await rag_anything.process_document_complete("document.pdf")
```

### 3. 手动内容源配置

```python
# 为特定内容列表设置上下文源
rag_anything.set_content_source_for_context(content_list, "minerU")

# 运行时更新上下文配置
rag_anything.update_context_config(
    context_window=1,
    max_context_tokens=1500,
    include_captions=False
)
```

### 4. 直接使用模态处理器

```python
from raganything.modalprocessors import (
    ContextExtractor,
    ContextConfig,
    ImageModalProcessor
)

# 配置上下文提取
config = ContextConfig(
    context_window=1,
    context_mode="page",
    max_context_tokens=2000,
    include_headers=True,
    include_captions=True,
    filter_content_types=["text"]
)

# 初始化上下文提取器
context_extractor = ContextExtractor(config)

# 初始化带上下文支持的模态处理器
processor = ImageModalProcessor(lightrag, caption_func, context_extractor)

# 设置内容源
processor.set_content_source(content_list, "minerU")

# 带上下文处理
item_info = {
    "page_idx": 2,
    "index": 5,
    "type": "image"
}

result = await processor.process_multimodal_content(
    modal_content=image_data,
    content_type="image",
    file_path="document.pdf",
    entity_name="架构图",
    item_info=item_info
)
```

## 上下文模式

### 基于页面的上下文 (`context_mode="page"`)
- 基于页面边界提取上下文
- 使用内容项的 `page_idx` 字段
- 适用于结构化文档内容
- 示例：包含当前图像前后 2 页的文本

### 基于分块的上下文 (`context_mode="chunk"`)
- 基于内容项位置提取上下文
- 使用内容列表中的顺序位置
- 适用于细粒度控制
- 示例：包含当前表格前后 5 个内容项

## 处理流程

### 1. 文档解析
```
文档输入 → MinerU 解析 → 内容列表生成
```

### 2. 上下文设置
```
内容列表 → 设为上下文源 → 所有模态处理器获得上下文能力
```

### 3. 多模态处理
```
多模态内容 → 提取周围上下文 → 增强的 LLM 分析 → 更准确的结果
```

## 内容源格式

### MinerU 格式
```json
[
    {
        "type": "text",
        "text": "文档内容...",
        "text_level": 1,
        "page_idx": 0
    },
    {
        "type": "image",
        "img_path": "images/figure1.jpg",
        "img_caption": ["图 1: 架构"],
        "page_idx": 1
    }
]
```

### 自定义文本块
```python
text_chunks = [
    "第一个文本块...",
    "第二个文本块...",
    "第三个文本块..."
]
```

### 纯文本
```python
full_document = "包含所有内容的完整文档文本..."
```

## 配置示例

### 高精度上下文
用于最小上下文的聚焦分析:
```python
config = RAGAnythingConfig(
    context_window=1,
    context_mode="page",
    max_context_tokens=1000,
    include_headers=True,
    include_captions=False,
    context_filter_content_types=["text"]
)
```

### 全面上下文
用于丰富上下文的广泛分析:
```python
config = RAGAnythingConfig(
    context_window=2,
    context_mode="page",
    max_context_tokens=3000,
    include_headers=True,
    include_captions=True,
    context_filter_content_types=["text", "image", "table"]
)
```

### 基于分块的分析
用于细粒度顺序上下文:
```python
config = RAGAnythingConfig(
    context_window=5,
    context_mode="chunk",
    max_context_tokens=2000,
    include_headers=False,
    include_captions=False,
    context_filter_content_types=["text"]
)
```

## 性能优化

### 1. 精准 token 控制
- 使用真实分词器进行精确 token 计数
- 避免超过 LLM token 限制
- 提供一致的性能表现

### 2. 智能截断
- 在句子边界处截断
- 保持语义完整性
- 添加截断指示符

### 3. 缓存优化
- 可重用上下文提取结果
- 减少冗余计算开销

## 高级功能

### 上下文截断
系统自动截断上下文以适应 token 限制：
- 使用实际分词器进行准确 token 计数
- 尝试在句子边界（句号）结束
- 必要时回退到行边界
- 为截断内容添加 "..." 指示符

### 标题格式化
当 `include_headers=True` 时，标题使用 Markdown 样式前缀：
```
# 一级标题
## 二级标题
### 三级标题
```

### 标题集成
当 `include_captions=True` 时，图像和表格标题按以下格式包含：
```
[图像: 图 1 标题文本]
[表格: 表 1 标题文本]
```

## 与 RAGAnything 的集成

上下文感知功能无缝集成到 RAGAnything 的工作流中：

1. **自动设置**: 自动创建和配置上下文提取器
2. **内容源管理**: 文档处理自动设置内容源
3. **处理器集成**: 所有模态处理器获得上下文能力
4. **配置一致性**: 单一配置系统管理所有上下文设置

## 错误处理

系统包含健壮的错误处理机制：
- 优雅处理缺失或无效的内容源
- 对不支持的格式返回空上下文
- 记录配置问题的警告日志
- 即使上下文提取失败也继续处理

## 兼容性

- **向后兼容**: 现有代码无需修改即可工作
- **可选功能**: 可选择启用/禁用上下文
- **灵活配置**: 支持多种配置组合

## 最佳实践

1. **Token 限制**: 确保 `max_context_tokens` 不超过 LLM 上下文限制
2. **性能影响**: 更大的上下文窗口会增加处理时间
3. **内容质量**: 上下文质量直接影响分析准确性
4. **窗口大小**: 根据内容结构匹配窗口大小（文档 vs 文章）
5. **内容过滤**: 使用 `context_filter_content_types` 减少噪音

## 故障排除

### 常见问题

**上下文未提取**
- 检查是否调用了 `set_content_source_for_context()`
- 确认 `item_info` 包含必要字段 (`page_idx`, `index`)
- 验证内容源格式是否正确

**上下文过长/过短**
- 调整 `max_context_tokens` 设置
- 修改 `context_window` 大小
- 检查 `context_filter_content_types` 配置

**上下文不相关**
- 优化 `context_filter_content_types` 排除噪音
- 减小 `context_window` 大小
- 若标题无帮助则设置 `include_captions=False`

**配置问题**
- 确认环境变量设置正确
- 检查 RAGAnythingConfig 参数名称
- 确保 content_format 与数据源匹配

## 示例

查看以下示例文件获取完整用法演示：

- **配置示例**: See how to set up different context configurations
- **集成示例**: Learn how to integrate context-aware processing into your workflow
- **自定义处理器**: Examples of creating custom modal processors with context support

## API 参考

详细 API 文档请参阅以下文件的文档字符串:
- `raganything/modalprocessors.py` - 上下文提取和模态处理器
- `raganything/config.py` - 配置选项
- `raganything/raganything.py` - 主 RAGAnything 类集成
