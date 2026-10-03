# RAG-企业IT运维知识库

面向企业 IT 运维场景的 RAG 知识库系统。后端基于 FastAPI + LangChain + Chroma，前端基于 Vue 3 + Vite + Element Plus，支持文档入库、混合检索和智能问答。

## 功能

- 知识库文档上传与切分入库
- 混合检索（粗排 + 精排）后调用大模型生成答案
- 网页知识抓取并清洗为 Markdown
- 知识库管理与智能问答界面

## 项目结构

```
RAG/
├── knowledge/                 # FastAPI 后端
│   ├── api/                   # 路由与服务入口
│   ├── cli/                   # 爬取 / 上传命令行
│   ├── config/                # 环境配置
│   ├── repositories/          # 文件与向量库访问
│   ├── services/              # 爬取、入库、检索、问答
│   ├── schemas/               # 请求响应模型
│   └── requirements.txt
└── knowlege_platform_ui/      # Vue 3 前端
    └── src/
        ├── views/             # 知识库管理、智能问答
        ├── api/               # 后端接口封装
        └── layout/
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

- 页面上传 Markdown 文档
- 或使用 `knowledge/cli/upload_cli.py`
- 或使用 `knowledge/cli/crawl_cli.py` 抓取知识源后入库

## 环境变量

复制 `knowledge/.env.example` 为 `knowledge/.env` 后填写：

| 变量 | 说明 |
| --- | --- |
| `API_KEY` | LLM / Embedding 服务密钥 |
| `BASE_URL` | OpenAI 兼容接口地址 |
| `MODEL` | 问答模型 |
| `EMBEDDING_MODEL` | 向量化模型 |
| `KNOWLEDGE_BASE_URL` | 知识源站点 |

请勿将 `.env`、向量库数据和爬取原文提交到 Git。

## 技术栈

- 后端：Python、FastAPI、LangChain、Chroma、jieba
- 前端：Vue 3、Vite、Element Plus、Axios