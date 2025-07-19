# RAGAnything 上下文感知功能超详细小白指南

## 什么是上下文感知功能？

想象一下，你看到一张图片但不知道它是什么意思。如果有人告诉你："这张图在第3页，前面讲的是如何做蛋糕，后面讲的是如何装饰蛋糕"，你就能猜到这张图可能是蛋糕的图片。这就是**上下文感知功能**——它让电脑在分析图片、表格等内容时，也能知道这些内容在文档中的位置和周围文字，从而更准确地理解它们。

### 🧠 简单比喻
- 普通模式：只看图片本身 → 只能知道这是"一个圆形物体"
- 上下文感知模式：看图片+前后文字 → 知道这是"第5页的蛋糕制作流程图"

## 为什么需要这个功能？

| 问题 | 没有上下文 | 有上下文 |
|------|------------|----------|
| 图片理解 | 可能误解为普通食物图 | 知道是实验数据图表 |
| 表格分析 | 只看数字不知道含义 | 结合前后文字知道是销售报表 |
| 公式解读 | 看不懂复杂数学符号 | 知道是某个理论推导过程 |
| 结果准确性 | 经常出错 | 更准确相关 |

## 🛠 功能设置超简单教程

### 第一步：基础设置（就像调手机亮度）

```python
from raganything import RAGAnything, RAGAnythingConfig

# 创建简单配置（就像设置手机）
config = RAGAnythingConfig(
    context_window=2,           # 看前后2页内容
    context_mode="page",        # 按页查看（也可以按块）
    max_context_tokens=2000,    # 最多看2000个字
    include_headers=True,       # 包含标题
    include_captions=True,      # 包含图片说明
    context_filter_content_types=["text", "image"]  # 只看文字和图片
)

# 启动RAGAnything（就像打开手机APP）
rag_anything = RAGAnything(config=config)
```

### 第二步：处理文档（就像拍照识别）

```python
# 自动处理PDF文档（会自动添加上下文）
await rag_anything.process_document_complete("我的文档.pdf")
```

### 第三步：提问（就像问问题）

```python
# 问关于文档的问题
answer = rag_anything.query("第5页的图表说明什么？")
print(answer)
# 输出：该图表展示了过去6个月的销售增长趋势...
```

## 🔍 核心功能详解

### 1. 上下文模式选择（两种看文档的方式）

| 模式 | 特点 | 适合场景 | 示例 |
|------|------|----------|------|
| **页面模式**<br>(context_mode="page") | 按文档页码查看 | 标准文档、PDF、报告 | 看图片时，同时看前1页和后1页的文字 |
| **块模式**<br>(context_mode="chunk") | 按内容块查看 | 网页内容、长文章 | 看表格时，同时看前3段和后3段文字 |

### 2. 上下文窗口大小（看多少周围内容）

```python
context_window=2  # 数字越大看的越多
```
- 设1：只看当前页/块
- 设2：看当前+前后各1页/块
- 设3：看当前+前后各2页/块

> 📌 建议：刚开始用2最合适，太大可能包含无关信息

### 3. 包含哪些内容（过滤不需要的）

```python
context_filter_content_types=["text", "image"]
```
- "text"：文字内容
- "image"：图片
- "table"：表格
- "formula"：数学公式

> 📌 技巧：如果只关心文字，就只选["text"]，避免干扰

### 4. 标题和说明处理

| 设置 | 效果 | 示例 |
|------|------|------|
| `include_headers=True` | 包含标题 | 自动添加"# 章节标题" |
| `include_captions=True` | 包含图片说明 | 自动添加"[图片: 图1说明]" |

## 🚀 不同场景配置示例

### 场景1：快速查看图片含义（新手推荐）

```python
config = RAGAnythingConfig(
    context_window=1,           # 只看当前页
    context_mode="page",        # 按页查看
    max_context_tokens=1000,    # 限1000字
    include_headers=True,       # 包含标题
    include_captions=True,      # 包含说明
    context_filter_content_types=["text"]  # 只看文字
)
```
> ✅ 适合：快速理解文档中的图片和图表

### 场景2：深度分析技术文档

```python
config = RAGAnythingConfig(
    context_window=2,           # 看前后2页
    context_mode="page",        # 按页查看
    max_context_tokens=3000,    # 限3000字
    include_headers=True,       # 包含标题
    include_captions=True,      # 包含说明
    context_filter_content_types=["text", "table", "formula"]  # 看文字、表格和公式
)
```
> ✅ 适合：分析论文、技术手册等复杂文档

### 场景3：分析长文章/网页内容

```python
config = RAGAnythingConfig(
    context_window=3,           # 看前后3块
    context_mode="chunk",       # 按内容块查看
    max_context_tokens=2000,    # 限2000字
    include_headers=False,      # 不包含标题（网页可能没标题）
    include_captions=False,     # 不包含说明
    context_filter_content_types=["text"]  # 只看文字
)
```
> ✅ 适合：博客文章、新闻网页等内容

## 🧩 实际使用案例

### 案例1：理解年度报告中的图表

```python
# 处理PDF报告
report = rag_anything.process_document("年度财务报告.pdf")

# 提问
answer = rag_anything.query("第15页的柱状图说明了什么？")
print(answer)
```
输出结果：
```
该柱状图位于"第四季度业绩"章节，展示了Q4各产品线的收入对比。
其中产品A收入最高(¥120万)，产品C最低(¥40万)，与正文中提到的市场策略一致。
```

### 案例2：解读论文中的公式

```python
# 处理学术论文
paper = rag_anything.process_document("AI研究论文.pdf")

# 提问
answer = rag_anything.query("公式3在文章中起什么作用？")
print(answer)
```
输出结果：
```
公式3位于"模型优化"章节，是核心算法的一部分。
它定义了损失函数的计算方法，前文介绍了它的理论基础，
后文展示了它在实验中的应用效果。
```

### 案例3：分析产品手册

```python
# 处理产品手册
manual = rag_anything.process_document("相机使用手册.docx")

# 提问
answer = rag_anything.query("第8页的示意图中红色按钮的功能是什么？")
print(answer)
```
输出结果：
```
红色按钮是"紧急停止"按钮，位于安全操作章节。
正文说明："当设备异常时，立即按下红色紧急停止按钮切断电源"，
后文还提供了复位该按钮的操作步骤。
```

## ❓ 常见问题解答

### Q1：开启上下文功能会让处理变慢吗？
> 会稍微慢一点，就像拍照时开启HDR需要多花一点时间。但你可以：
> - 减小`context_window`值
> - 降低`max_context_tokens`值
> - 减少`context_filter_content_types`中的类型

### Q2：为什么有时返回的上下文不相关？
> 可能原因：
> 1. `context_window`太大 → 包含太多无关内容（减小数值）
> 2. 过滤设置不对 → 包含图片等非文字内容（设置`context_filter_content_types=["text"]`）
> 3. 文档结构乱 → 尝试切换`context_mode`为"chunk"

### Q3：如何知道当前配置是否合适？
> 使用测试文档检查：
```python
# 测试不同配置
test_results = []
for window_size in [1, 2, 3]:
    config.context_window = window_size
    answer = rag_anything.query("测试问题")
    test_results.append(f"窗口大小{window_size}: {answer}")

# 比较结果选择最佳
print(test_results)
```

### Q4：处理时出现错误怎么办？
> 检查清单：
> - 确认文档路径正确
> - 检查`config`参数名拼写正确
> - 确保`content_format`设置匹配文档类型
> - 查看错误信息中的具体提示

## 💡 使用技巧与最佳实践

1. **起步设置**：第一次使用时，用这个配置最安全：
   ```python
   context_window=2
   context_mode="page"
   max_context_tokens=2000
   include_headers=True
   include_captions=True
   context_filter_content_types=["text"]
   ```

2. **文档类型建议**：
   - PDF报告 → 用"page"模式
   - 网页文章 → 用"chunk"模式
   - 含多图表文档 → 开启`include_captions`

3. **性能优化**：
   ```python
   # 在config.py中可以调整
   IMAGE_PROCESSING_RESOLUTION = 768  # 降低图片分辨率提升速度
   TABLE_ANALYSIS_DEPTH = "basic"     # 简化表格分析
   ```

4. **进阶技巧**：
   ```python
   # 运行时调整配置
   rag_anything.update_context_config(
       context_window=1,           # 临时缩小范围
       max_context_tokens=1000,    # 临时减少字数
       include_captions=False      # 临时关闭说明
   )
   ```

## 📚 学习资源

### 新手入门路径：
1. 运行`examples/raganything_example.py`体验基础功能
2. 修改`context_window`值感受不同效果
3. 尝试处理自己的文档

### 示例文件位置：
```
RAG-Anything/
└── examples/
    ├── raganything_example.py       # 基础使用示例
    ├── office_document_test.py      # Office文档处理
    └── image_format_test.py         # 图片处理专项
```

### 需要帮助时：
1. 查看`docs/context_aware_processing.md`文档
2. 阅读代码中的说明（特别是`modalprocessors.py`）
3. 在GitHub提交问题：https://github.com/HKUDS/RAG-Anything/issues

