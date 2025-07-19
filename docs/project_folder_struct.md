```text
RAG-Anything/
├── **raganything/**                # 核心功能包
│   ├── __init__.py                # 包初始化
│   ├── batch.py                   # 批量处理工具
│   ├── config.py                  # 全局配置管理
│   ├── mineru_parser.py           # 多格式文档解析引擎 (核心!)
│   ├── modalprocessors.py         # 多模态处理器 (图像/表格/公式)
│   ├── processor.py               # 处理流水线控制器
│   ├── prompt.py                  # 提示词模板管理
│   ├── query.py                   # 查询处理模块
│   ├── raganything.py             # 主入口类
│   └── utils.py                   # 通用工具函数
│
├── **examples/**                  # 应用案例
│   ├── image_format_test.py       # 图像格式处理示例
│   ├── modalprocessors_example.py # 多模态处理器使用示例
│   ├── office_document_test.py    # Office文档处理示例
│   ├── raganything_example.py     # 端到端流程示例
│   └── text_format_test.py        # 文本格式处理示例
│
├── **docs/**                      # 文档
│   └── context_aware_processing.md # 上下文感知处理技术说明
│
├── **assets/**                    # 资源文件
│   ├── logo.png                   # 项目Logo
│   └── rag_anything_framework.png # 架构示意图
│
├── raganything.egg-info/          # Python包元数据
├── env.example                    # 环境变量示例
├── MANIFEST.in                    # 打包配置文件
├── requirements.txt               # 依赖库
├── setup.py                       # 安装脚本
├── README.md                      # 英文说明
├── README_zh.md                   # 中文说明
└── LICENSE                        # 许可证
```