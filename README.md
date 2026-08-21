# CV Agent Pro · AI 求职 Copilot

> 基于**多 Agent 协作 + RAG 知识库**的 AI 求职助手。从单文件前端 Demo 升级为**前后端分离、多用户、可部署**的真实 AI 产品架构。

<p align="center">
  <b>简历分析</b> · <b>JD 匹配</b> · <b>话术生成</b> · <b>定制简历</b> · <b>模拟面试</b> · <b>面试复盘</b> · <b>求职知识库 RAG</b>
</p>

---

## ✨ 项目介绍

CV Agent Pro 是面向求职者的 AI 求职 Copilot，通过 **9 个专业 Agent**（简历分析、JD 分析、匹配、话术、改写、出题、评估、复盘、对话）+ **Orchestrator 编排器**协作，覆盖求职全流程：

| 能力 | 说明 |
| --- | --- |
| 📄 简历解析 | 上传 PDF / 粘贴文本，Agent 自动提取结构化信息 |
| 🎯 JD 匹配 | 粘贴目标岗位 JD，多 Agent 计算匹配度（0-100）并给出硬性门槛与短板 |
| 💬 投递话术 | 一键生成 Boss 直聘 / 邮件 / 英文 Cover Letter（3 种风格） |
| ✍️ 定制简历 | 针对目标岗位 STAR 法则改写，不覆盖原版，标注「建议补充」 |
| 🎤 模拟面试 | AI 逐题提问 → 作答 → 逐题评分 + STAR 改写 + 整体总结 |
| 📋 面试复盘 | 上传面试文字稿，输出多维度复盘报告与优化答案 |
| 📚 求职知识库 | 个人面经/公司资料 RAG 检索，喂给面试与对话 Agent |

## 🏗️ 产品架构

```
用户 ──▶ Frontend (Next.js + React + TS + Tailwind)
                │  REST API (JWT)
                ▼
        Backend (FastAPI + SQLAlchemy)
          ├──────────┬──────────────┬───────────────┐
          ▼          ▼              ▼               ▼
    PostgreSQL   文件存储(本地)   DeepSeek API    Agent Orchestrator
   (生产用Supabase)                             ┌─────────────────┐
                                                │ 9 个 AI Agent   │
                                                │ 并行 + 串联编排   │
                                                └─────────────────┘
```

**关键设计：所有 AI 请求都经过后端**，DeepSeek API Key 只存在于服务端环境变量，前端零接触，彻底消除原版 `index.html` 的 Key 泄露风险。

## 🤖 Agent 流程图

```
          ┌─────────────────────────────────────────┐
          │          Orchestrator（规则层，不调 LLM） │
          └─────────────────────────────────────────┘
             并行 ▼                         ▼
      ┌──────────────┐             ┌──────────────┐
      │ 简历分析 Agent │             │  JD 分析 Agent│
      └──────┬───────┘             └──────┬───────┘
             └──────────┬─────────────────┘
                        ▼
              ┌─────────────────┐
              │  匹配分析 Agent   │  → 匹配度 / 短板 / 建议
              └────────┬────────┘
               并行    ▼
      ┌──────────────────┐   ┌──────────────────┐
      │  话术生成 Agent    │   │  简历改写 Agent    │
      └──────────────────┘   └──────────────────┘
   （失败不阻断，独立重试）         （失败不阻断，独立重试）
```

## 🛠️ 技术栈

| 层 | 技术 |
| --- | --- |
| 前端 | Next.js 14 · React 18 · TypeScript · Tailwind CSS |
| 后端 | FastAPI · SQLAlchemy 2.0 · Pydantic v2 |
| 数据库 | PostgreSQL（生产，Supabase）· SQLite（本地，零安装） |
| AI | DeepSeek API（deepseek-chat）· 9 Agent 编排 · TF-IDF RAG |
| 认证 | JWT + PBKDF2 密码哈希（零编译依赖） |
| 部署 | Vercel（前端）· Render / Railway（后端）· Supabase（数据库） |

## 🚀 本地快速开始

### 0. 前置要求
- Python 3.9+、Node.js 18+、npm
- 一个 [DeepSeek](https://platform.deepseek.com) API Key

### 1. 启动后端（端口 8000）

```bash
cd backend
python -m venv .venv
# Windows: .\.venv\Scripts\activate
source .venv/bin/activate
pip install -r requirements.txt

cp .env.example .env        # 编辑 .env，填入 DEEPSEEK_API_KEY
uvicorn main:app --reload --port 8000
```

### 2. 启动前端（端口 3000）

```bash
cd frontend
npm install
npm run dev
```

访问 **http://localhost:3000** —— 点「游客体验」即可用预置 Demo 账号（`demo@cvagent.app`）体验，无需注册。

### 3. 环境变量说明

**backend/.env**

| 变量 | 说明 | 示例 |
| --- | --- | --- |
| `DATABASE_URL` | 数据库连接 | `sqlite:///./cv_agent.db` / `postgresql+psycopg2://user:pass@host:5432/cvagent` |
| `DEEPSEEK_API_KEY` | DeepSeek Key（**只存在后端**） | `sk-xxxx` |
| `JWT_SECRET` | JWT 签名密钥 | 随机长字符串 |
| `FRONTEND_ORIGIN` | 允许跨域的前端地址 | `http://localhost:3000` |
| `SEED_DEMO` | 是否写入 Demo 账号 | `true` |

**frontend/.env.local**

| 变量 | 说明 |
| --- | --- |
| `NEXT_PUBLIC_API_BASE` | 后端地址 | `http://127.0.0.1:8000` |

## 🌐 在线部署

| 组件 | 推荐平台 | 说明 |
| --- | --- | --- |
| Frontend | Vercel | 导入 `frontend/`，设置 `NEXT_PUBLIC_API_BASE` 为后端域名 |
| Backend | Render / Railway | 启动命令 `uvicorn main:app --host 0.0.0.0 --port 8000`，设置全部环境变量 |
| Database | Supabase PostgreSQL | 把 `DATABASE_URL` 指向 Supabase 连接串 |

> 或使用仓库根目录的 [docker-compose.yml](docker-compose.yml) 一键起 PostgreSQL + 后端。

## 📁 目录结构

```
cv-agent-pro
├── frontend/                # Next.js 前端
│   ├── pages/               # 工作台 / 简历 / 投递 / 面试 / 题库 / 登录注册
│   ├── components/          # AgentFlow / ScoreRing / 上传弹窗 等
│   ├── hooks/               # 自定义 Hook（预留）
│   ├── services/            # API 客户端（JWT 封装）+ 类型定义
│   ├── contexts/            # AuthContext
│   └── styles/              # 全局样式
├── backend/                 # FastAPI 后端
│   ├── api/                 # 路由（auth/resumes/jobs/applications/interviews/rag/chat）
│   ├── agents/              # 9 个 Agent 提示词 + LLM 客户端 + Agent 基类
│   ├── services/            # Orchestrator 编排 / RAG 检索 / PDF / 安全 / 种子数据
│   ├── models/              # SQLAlchemy 数据模型
│   ├── database/            # 数据库会话
│   └── main.py              # 应用入口
├── docker-compose.yml
└── README.md
```

## 🗺️ 后续规划（Roadmap）

- [x] 第一阶段：前后端分离 · API 后移 · 数据库存储 · 用户系统
- [x] 第二阶段（进行中）：Agent 流程实时可视化（SSE）· 简历版本管理（V1→V2→V3）· Demo 账号
- [ ] 第二阶段：在线 Demo 部署（Vercel + Render + Supabase）
- [ ] 第三阶段：企业知识库 RAG · 公司分析 · AI 职业规划 · 语音模拟面试

## 📄 License

MIT
