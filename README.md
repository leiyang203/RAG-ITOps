# RAG-企业 IT 运维知识库

> 面向企业 IT 运维场景的 RAG（Retrieval-Augmented Generation）知识库系统。后端基于 FastAPI + LangChain + Chroma，前端基于 Vue 3 + Vite + Element Plus，支持文档入库、混合检索和智能问答。

---

## ✨ 核心特性

- 📥 **多渠道入库** — 页面上传 / 命令行上传 / 网页爬取
- ✂️ **智能切分** — 基于语义完整性的 chunk_size + chunk_overlap 调优
- 🔍 **混合检索** — 向量粗排（Top 50）+ 精排（Top 5）两级召回
- 🧹 **清洗降噪** — 入库前 HTML → Markdown 清洗，元数据结构化
- 🤖 **智能问答** — 检索增强，基于上下文生成精准回答
- 🎨 **Web 管理台** — 知识库管理界面 + 智能问答界面

## 📐 系统架构

```
┌────────────────────────────────────────────────────────────┐
│                  Vue 3 前端 (knowlege_platform_ui/)         │
│       Element Plus + Vite + Axios                          │
│  ┌──────────────────┐    ┌──────────────────────────────┐ │
│  │  知识库管理页面    │    │  智能问答页面                 │ │
│  │  /views/Knowledge  │    │  /views/Chat                 │ │
│  └────────┬─────────┘    └──────────────┬───────────────┘ │
└───────────┼──────────────────────────────┼────────────────┘
            │ HTTP (multipart / JSON)         │ HTTP JSON
┌───────────▼──────────────────────────────▼────────────────┐
│              FastAPI 后端 (knowledge/)                      │
│  ┌─────────────────┐  ┌─────────────────────────────────┐ │
│  │  POST /upload    │  │  POST /query                    │ │
│  │  文档上传入库     │  │  知识库问答                     │ │
│  └────────┬────────┘  └──────────────┬──────────────────┘ │
└───────────┼──────────────────────────┼─────────────────────┘
            │                          │
┌───────────▼──────────────────────────▼─────────────────────┐
│                 RAG 核心流程                                │
│                                                             │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────────┐ │
│  │ 清洗降噪  │→ │ 切分入库  │→ │ 混合检索  │→ │ LLM 生成回答 │ │
│  │ HTML→MD  │  │Chroma向量 │  │粗排+精排  │  │ LangChain   │ │
│  └──────────┘  └──────────┘  └──────────┘  └─────────────┘ │
│                                                             │
│  CLI 工具:                                                  │
│  ┌──────────────────┐  ┌────────────────────────────────┐  │
│  │ upload_cli.py     │  │ crawl_cli.py                    │  │
│  │ 本地文件入库       │  │ 网页抓取 → 清洗 → 自动入库        │  │
│  └──────────────────┘  └────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

## 🧠 RAG 核心流程

```
               用户提问
                  │
                  ▼
┌──────────────────────────────┐
│  1. 混合检索 (retrieval_service)│
│                                │
│   ┌─────────────┐             │
│   │ 向量粗排 Top 50 │  ← Chroma (Embedding)
│   └──────┬──────┘             │
│          │                    │
│          ▼                    │
│   ┌─────────────┐             │
│   │ 精排 Top 5   │  ← 相似度 + 阈值过滤
│   └──────┬──────┘             │
└──────────┼─────────────────────┘
           │ 检索上下文
           ▼
┌──────────────────────────────┐
│  2. Query 处理 (query_service) │
│                                │
│   构造 Prompt (上下文 + 问题)   │
│          │                     │
│          ▼                     │
│   LangChain → LLM 生成答案     │
└──────────┼─────────────────────┘
           │
           ▼
       返回回答
```

### 检索优化方案（已实现）

| 优化点 | 实现方式 | 配置 |
| --- | --- | --- |
| **粗排 + 精排** | 向量检索先召回 Top 50，再精排取 Top 5 | `TOP_ROUGH=50`, `TOP_FINAL=5` |
| **Chunk 切分** | `chunk_size=3000`, `chunk_overlap=200` 平衡语义完整性与粒度 | `CHUNK_SIZE`, `CHUNK_OVERLAP` |
| **中文分词** | jieba 处理中文关键词 | ✅ 已集成 |
| **HTML 清洗** | BeautifulSoup + markdownify | ✅ 已集成 |
| **元数据对齐** | 入库时统一 metadata 结构，辅助检索 | ✅ 已集成 |

### 🚧 待优化方向（Roadmap）

- [ ] 集成多路召回（BM25 + 向量 + 知识图谱）
- [ ] 引入 Cross-Encoder 精排模型
- [ ] 处理 Markdown 图片（多模态）
- [ ] 实现文档去重函数
- [ ] 动态阈值返回 Top_N（混合模式）
- [ ] Rerank 模型集成

---

## 功能

- ✅ 知识库文档上传与切分入库
- ✅ 混合检索（粗排 + 精排）后调用大模型生成答案
- ✅ 网页知识抓取并清洗为 Markdown
- ✅ 知识库管理与智能问答界面

## 项目结构

```
RAG-ITOps/
├── knowledge/                  # FastAPI 后端
│   ├── api/                    # 路由与服务入口
│   │   ├── main.py             # FastAPI 应用入口 (port 8001)
│   │   └── routers.py          # /upload, /query 路由
│   ├── cli/                    # 命令行工具
│   │   ├── crawl_cli.py        # 网页抓取入库
│   │   └── upload_cli.py      # 本地文件上传入库
│   ├── config/                 # 环境配置
│   │   └── settings.py         # Pydantic Settings（切分/检索/路径）
│   ├── repositories/           # 数据访问层
│   │   ├── file_repository.py
│   │   └── vector_store_repository.py
│   ├── services/               # 业务服务层
│   │   ├── crawler/            # 网页爬取 + HTML 清洗
│   │   │   ├── client.py       # 爬取客户端
│   │   │   └── parser.py       # HTML → Markdown 解析
│   │   ├── ingestion/          # 文档入库处理
│   │   │   └── ingestion_processor.py
│   │   ├── retrieval_service.py # 混合检索核心
│   │   └── query_service.py    # 问答生成
│   ├── schemas/                # 请求 / 响应模型
│   │   └── schema.py           # UploadResponse, QueryRequest, QueryResponse
│   ├── utils/                  # 工具函数
│   │   ├── markdown_utils.py
│   │   └── text_utils.py
│   ├── data/                   # 运行时数据（.gitkeep）
│   │   ├── crawl/              # 爬取的 Markdown 文件
│   │   └── tmp/                # 临时文件
│   ├── chroma_kb1/             # Chroma 向量库（运行时生成）
│   ├── .env.example
│   └── requirements.txt
│
└── knowlege_platform_ui/       # Vue 3 前端
    ├── src/
    │   ├── views/
    │   │   ├── Knowledge.vue   # 知识库管理
    │   │   └── Chat.vue        # 智能问答
    │   ├── api/
    │   │   ├── knowledge.js    # 知识库相关接口
    │   │   └── request.js      # Axios 封装
    │   ├── layout/
    │   ├── App.vue
    │   └── main.js
    ├── index.html
    ├── vite.config.js          # /api → 127.0.0.1:8001 代理
    └── package.json
```

## 快速开始

### 1. 后端

```powershell
cd knowledge
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
copy .env.example .env
# 在 .env 中填写 API_KEY 等配置
python -m api.main
```

后端默认监听 `http://127.0.0.1:8001`。

### 2. 前端

```powershell
cd knowlege_platform_ui
npm install
npm run dev
```

前端默认运行在 `http://127.0.0.1:3000`，并将 `/api` 代理到后端 `8001` 端口。

### 3. 入库知识

- 🖥️ **页面上传** — 前端知识库管理页面上传 Markdown 文档
- 📟 **命令行上传** — `python knowledge/cli/upload_cli.py` 指定本地文件
- 🌐 **网页爬取** — `python knowledge/cli/crawl_cli.py` 抓取知识源后自动清洗入库

## 环境变量

复制 `knowledge/.env.example` 为 `knowledge/.env` 后填写：

| 变量 | 说明 | 示例 |
| --- | --- | --- |
| `API_KEY` | LLM / Embedding 服务密钥 | `sk-xxxxxxxx` |
| `BASE_URL` | OpenAI 兼容接口地址 | `https://api.deepseek.com/v1` |
| `MODEL` | 问答模型 | `deepseek-chat` |
| `EMBEDDING_MODEL` | 向量化模型 | `text-embedding-v2` |
| `KNOWLEDGE_BASE_URL` | 知识源站点（爬取用） | `https://its.example.com/wiki` |

### 内置配置（config/settings.py）

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| `CHUNK_SIZE` | 3000 | 文档切分块大小（字符） |
| `CHUNK_OVERLAP` | 200 | 相邻块重叠字符数 |
| `TOP_ROUGH` | 50 | 粗排召回数量 |
| `TOP_FINAL` | 5 | 精排最终返回数量 |
| `VECTOR_STORE_PATH` | `knowledge/chroma_kb1` | Chroma 向量库本地路径 |

> ⚠️ 请勿将 `.env`、向量库数据和爬取原文提交到 Git。

## 📡 API 接口

### 文档上传入库

```
POST /upload
Content-Type: multipart/form-data
```

返回：

```json
{
  "status": "success",
  "message": "文档上传知识库成功",
  "file_name": "运维手册.md",
  "chunks_added": 42
}
```

### 智能问答

```
POST /query
Content-Type: application/json

{
  "question": "服务器 CPU 占用过高怎么排查？"
}
```

返回：

```json
{
  "question": "服务器 CPU 占用过高怎么排查？",
  "answer": "根据运维手册，CPU 占用过高可从以下几个方面排查..."
}
```

## 技术栈

| 层级 | 技术 | 说明 |
| --- | --- | --- |
| **Web 框架** | FastAPI + Uvicorn | 高性能异步后端 |
| **RAG 框架** | LangChain (≥1.0.7) | 链式构建检索流程 |
| **向量数据库** | Chroma | 本地轻量向量存储 |
| **中文分词** | jieba | 中文关键词 / 切词 |
| **文档解析** | unstructured + markdownify | 多格式解析 + HTML 清洗 |
| **机器学习** | scikit-learn | 本地相似度计算 |
| **前端** | Vue 3 + Vite + Element Plus | 现代化管理台 |
| **HTTP 客户端** | Axios + requests | 前端 / 后端网络请求 |

## 📝 License

MIT License
