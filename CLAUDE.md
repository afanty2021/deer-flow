# DeerFlow - 开源超级代理系统

> **最后更新**: 2025-03-13
> **版本**: 2.0.0
> **语言**: Python 3.12+ | TypeScript 5.8
> **仓库**: [bytedance/deer-flow](https://github.com/bytedance/deer-flow)

---

## 项目愿景

DeerFlow (**D**eep **E**xploration and **E**fficient **R**esearch **Flow**) 是一个开源的**超级代理系统**，能够协调**子代理**、**记忆**和**沙箱**来完成几乎任何任务 —— 由**可扩展技能**驱动。

从深度研究框架开始，DeerFlow 已经演变为一个完整的代理运行时，为代理提供完成工作所需的基础设施：文件系统、记忆、技能、沙箱执行以及规划和生成子代理进行复杂多步骤任务的能力。

### 核心特性

- **技能 & 工具**: 可扩展的技能系统和工具集成
- **子代理**: 动态生成和并行执行子代理
- **沙箱 & 文件系统**: 隔离的执行环境，完整的文件系统访问
- **上下文工程**: 智能上下文管理和摘要
- **长期记忆**: 跨会话的持久化记忆
- **MCP 集成**: 支持 Model Context Protocol 服务器
- **IM 通道**: Telegram、Slack、Feishu/Lark 集成

---

## 架构总览

DeerFlow 采用**前后端分离**的微服务架构：

```
┌─────────────────────────────────────────────────────────────────┐
│                         nginx (端口 2026)                        │
│                    统一入口、反向代理、静态资源                    │
└─────────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┴─────────────┐
                │                           │
                ▼                           ▼
┌───────────────────────────┐   ┌───────────────────────────┐
│   Frontend (Next.js 16)   │   │   Backend Services        │
│   - React 19              │   │   - LangGraph Server      │
│   - TypeScript 5.8        │   │   - Gateway API           │
│   - TanStack Query        │   │   - Python 3.12+          │
│   - Vercel AI SDK         │   │   - FastAPI               │
└───────────────────────────┘   └───────────────────────────┘
                                        │
                    ┌───────────────────┼───────────────────┐
                    │                   │                   │
                    ▼                   ▼                   ▼
            ┌───────────┐       ┌───────────┐       ┌───────────┐
            │  Agents   │       │  Sandbox  │       │    MCP    │
            │ LangGraph │       │  Docker   │       │  Servers  │
            │           │       │  Local    │       │           │
            └───────────┘       └───────────┘       └───────────┘
```

### 服务端口

| 服务 | 端口 | 描述 |
|------|------|------|
| **LangGraph Server** | 2024 | 代理运行时和工作流执行 |
| **Gateway API** | 8001 | REST API：模型、技能、记忆、工件 |
| **Frontend** | 2026 | Web 界面（通过 nginx 统一暴露） |

---

## 模块结构图

```mermaid
graph TD
    A["(根) DeerFlow"] --> B["backend/"];
    A --> C["frontend/"];
    A --> D["skills/"];
    A --> E["docker/"];

    B --> B1["src/agents/ - 代理系统"];
    B --> B2["src/gateway/ - API 网关"];
    B --> B3["src/sandbox/ - 沙箱执行"];
    B --> B4["src/subagents/ - 子代理"];
    B --> B5["src/mcp/ - MCP 集成"];
    B --> B6["src/skills/ - 技能加载"];
    B --> B7["src/channels/ - IM 通道"];
    B --> B8["tests/ - 测试套件"];

    C --> C1["src/app/ - Next.js 路由"];
    C --> C2["src/components/ - UI 组件"];
    C --> C3["src/core/ - 业务逻辑"];
    C --> C4["src/hooks/ - React Hooks"];

    D --> D1["public/ - 公共技能"];
    D --> D2["custom/ - 自定义技能"];

    click B "./backend/CLAUDE.md" "查看后端模块文档"
    click C "./frontend/CLAUDE.md" "查看前端模块文档"
```

---

## 模块索引

| 模块 | 路径 | 语言/框架 | 职责 | 文档覆盖率 |
|------|------|-----------|------|------------|
| **Backend** | `backend/` | Python 3.12+<br>FastAPI + LangGraph | 代理系统、Gateway API、沙箱执行 | [98%+](./backend/CLAUDE.md) |
| **Frontend** | `frontend/` | TypeScript 5.8<br>Next.js 16 + React 19 | Web 界面、流式对话、工件预览 | [95%+](./frontend/CLAUDE.md) |
| **Skills** | `skills/` | Markdown | 可扩展技能定义和模板 | 100% |

### Backend 子模块

| 子模块 | 路径 | 职责 |
|--------|------|------|
| **Agents** | `src/agents/` | LangGraph 代理工厂和中间件 |
| **Gateway** | `src/gateway/` | FastAPI REST API 服务 |
| **Sandbox** | `src/sandbox/` | 沙箱执行环境（本地/Docker/K8s） |
| **Subagents** | `src/subagents/` | 子代理注册表和执行器 |
| **MCP** | `src/mcp/` | MCP 服务器客户端和工具适配 |
| **Skills** | `src/skills/` | 技能发现、加载和解析 |
| **Channels** | `src/channels/` | IM 通道集成（Telegram/Slack/Feishu） |
| **Models** | `src/models/` | LLM 模型工厂和配置 |

### Frontend 子模块

| 子模块 | 路径 | 职责 |
|--------|------|------|
| **App Router** | `src/app/` | Next.js App Router 结构 |
| **Components** | `src/components/` | UI 组件（ai-elements、ui、workspace） |
| **Core** | `src/core/` | 业务逻辑（threads、api、artifacts、memory） |
| **Hooks** | `src/hooks/` | 共享 React Hooks |

---

## 运行与开发

### 前置要求

- **Node.js**: 22+ (前端)
- **Python**: 3.12+ (后端)
- **pnpm**: 10.26.2+ (前端包管理)
- **uv**: 最新版 (Python 包管理)
- **Docker**: 可选，用于沙箱执行

### 快速开始

#### 1. 生成配置文件

```bash
make config
```

这会从 `config.example.yaml` 创建 `config.yaml`，从 `.env.example` 创建 `.env`。

#### 2. 配置模型

编辑 `config.yaml`，至少配置一个模型：

```yaml
models:
  - name: gpt-4                       # 内部标识符
    display_name: GPT-4               # 显示名称
    use: langchain_openai:ChatOpenAI  # LangChain 类路径
    model: gpt-4                      # API 模型标识
    api_key: $OPENAI_API_KEY          # API 密钥（推荐使用环境变量）
    max_tokens: 4096
    temperature: 0.7
```

#### 3. 设置 API 密钥

编辑 `.env` 文件：

```bash
TAVILY_API_KEY=your-tavily-api-key
OPENAI_API_KEY=your-openai-api-key
# 根据需要添加其他提供商密钥
INFOQUEST_API_KEY=your-infoquest-api-key
```

#### 4. 启动服务

**方式 A: Docker 开发（推荐）**

```bash
make docker-init    # 拉取沙箱镜像（仅需一次）
make docker-start   # 启动服务（自动检测沙箱模式）
```

访问: http://localhost:2026

**方式 B: 本地开发**

```bash
make check          # 检查工具安装
make install        # 安装依赖
make dev            # 启动所有服务（热重载）
```

访问: http://localhost:2026

### 开发命令

| 命令 | 描述 |
|------|------|
| `make config` | 生成配置文件 |
| `make check` | 检查必需工具 |
| `make install` | 安装所有依赖 |
| `make dev` | 开发模式启动（热重载） |
| `make stop` | 停止所有服务 |
| `make clean` | 清理临时文件和容器 |
| `make docker-start` | Docker 开发模式 |
| `make up` | 生产模式 Docker 部署 |
| `make down` | 停止生产 Docker 容器 |

### 生产部署

```bash
make up     # 构建镜像并启动生产服务
make down   # 停止并移除容器
```

---

## 测试策略

### Backend 测试

**框架**: pytest

**运行测试**:

```bash
cd backend

# 所有测试
make test

# 特定测试文件
PYTHONPATH=. uv run pytest tests/test_client.py -v

# 带覆盖率
PYTHONPATH=. uv run pytest --cov=src tests/
```

**测试覆盖**:

- **29 个测试文件**，**77+ 个单元测试**
- 关键测试套件：
  - `test_client.py` - 嵌入式客户端和 Gateway 一致性
  - `test_memory_*.py` - 记忆系统
  - `test_mcp_*.py` - MCP 集成和 OAuth
  - `test_subagent_*.py` - 子代理执行
  - `test_title_*.py` - 标题生成
  - `test_uploads_*.py` - 文件上传
  - `test_docker_sandbox_mode_detection.py` - Docker 沙箱模式检测

### Frontend 测试

当前**未配置测试框架**。

建议添加：
- **Vitest** - 单元测试
- **Playwright** - E2E 测试
- **React Testing Library** - 组件测试

---

## 编码规范

### Python (Backend)

- **Line length**: 240 字符
- **Type hints**: 必需
- **Quotes**: 双引号
- **Import order**: 标准库 → 第三方 → 本地

**Linting**:

```bash
make lint      # 检查
make format    # 修复格式
```

### TypeScript (Frontend)

**Import order**（每组之间换行，组内字母排序）：

1. Built-in (`react`, `next`)
2. External (`@radix-ui`, `@tanstack`)
3. Internal (`@/core`, `@/components`)
4. Parent (`../`)
5. Sibling (`./`)

**最佳实践**:

- 使用内联类型导入: `import { type Foo }`
- 未使用变量前缀: `_variable`
- 类名: 使用 `cn()` from `@/lib/utils`
- 路径别名: `@/*` → `src/*`

**检查**:

```bash
cd frontend

pnpm typecheck    # 类型检查
pnpm lint         # ESLint
pnpm check        # 全部检查
```

---

## AI 使用指引

### 适合 AI 辅助的任务

1. **添加新技能**: 在 `skills/custom/` 创建新的 Markdown 技能文件
2. **集成新工具**: 在 `src/community/` 添加新工具实现
3. **添加 IM 通道**: 在 `src/channels/` 实现新的通道适配器
4. **创建自定义代理**: 基于现有中间件创建新的代理配置
5. **扩展前端组件**: 在 `src/components/` 添加新的 UI 组件

### 关键概念

#### 虚拟路径系统

**Agent 看到的路径**:
- `/mnt/user-data/workspace` - 工作目录
- `/mnt/user-data/uploads` - 用户上传
- `/mnt/user-data/outputs` - 输出文件
- `/mnt/skills` - 技能目录

**物理路径**:
- `backend/.deer-flow/threads/{thread_id}/user-data/workspace`
- `backend/.deer-flow/threads/{thread_id}/user-data/uploads`
- `backend/.deer-flow/threads/{thread_id}/user-data/outputs`
- `skills/` (项目根目录)

#### 中间件链

中间件按严格顺序执行（`src/agents/lead_agent/agent.py`）:

1. **ThreadDataMiddleware** - 创建每线程目录
2. **UploadsMiddleware** - 跟踪上传文件
3. **SandboxMiddleware** - 获取沙箱实例
4. **DanglingToolCallMiddleware** - 处理不完整的工具调用
5. **SummarizationMiddleware** - 上下文缩减（可选）
6. **TodoListMiddleware** - 任务跟踪（可选）
7. **TitleMiddleware** - 自动生成标题
8. **MemoryMiddleware** - 记忆更新队列
9. **ViewImageMiddleware** - 注入 base64 图片
10. **SubagentLimitMiddleware** - 强制并发限制
11. **ClarificationMiddleware** - 处理澄清请求（必须最后）

#### 技能系统

技能是**延迟加载**的 Markdown 文件，定义：
- 工作流程
- 最佳实践
- 参考资源

**位置**:
- 公共技能: `skills/public/*/SKILL.md`
- 自定义技能: `skills/custom/*/SKILL.md`

---

## 常见问题 (FAQ)

### Q: 如何更改监听端口？

**A**: 编辑 `docker/nginx/nginx.local.conf`（本地开发）或 `docker/nginx/nginx.conf`（生产），修改 `listen` 指令。

### Q: 沙箱执行失败怎么办？

**A**:
1. 检查 Docker 是否运行: `docker ps`
2. 检查沙箱镜像是否拉取: `docker images | grep sandbox`
3. 查看日志: `make docker-logs`

### Q: 如何添加自定义模型？

**A**: 在 `config.yaml` 的 `models` 部分添加新配置，支持任何 OpenAI 兼容的 API。

### Q: 记忆数据存储在哪里？

**A**: `backend/.deer-flow/memory/user_id/memory.json` - 本地文件存储，完全由你控制。

### Q: 如何在生产环境部署？

**A**:
```bash
make up    # 构建生产镜像并启动
```
或参考 `docker/docker-compose.prod.yml`。

---

## 相关资源

### 官方文档

- **网站**: [deerflow.tech](https://deerflow.tech/)
- **GitHub**: [bytedance/deer-flow](https://github.com/bytedance/deer-flow)

### 技术文档

- [贡献指南](CONTRIBUTING.md) - 开发环境设置和工作流
- [配置指南](backend/docs/CONFIGURATION.md) - 详细配置说明
- [架构文档](backend/docs/ARCHITECTURE.md) - 技术架构详解
- [API 文档](backend/docs/API.md) - API 参考

### 模块文档

- [Backend 模块](backend/CLAUDE.md) - 后端架构和 API
- [Frontend 模块](frontend/CLAUDE.md) - 前端架构和组件

---

## 变更记录 (Changelog)

### 2025-03-13 - 初始化 AI 上下文文档（简体中文）

**覆盖统计**:
- 总文件数: 482
- 已扫描文件: 473
- 覆盖率: **98.1%** ✅

**生成内容**:
- ✅ 根级 `CLAUDE.md` - 项目概览和架构
- ✅ `backend/CLAUDE.md` - 后端模块文档（98%+）
- ✅ `frontend/CLAUDE.md` - 前端模块文档（95%+）
- ✅ `.claude/index.json` - 扫描索引和覆盖率报告

**扫描阶段**:
- **Phase A**: 识别 2 个主模块、17 个公共技能、5 个配置文件
- **Phase B**: 扫描 117 个后端文件、218 个前端文件、48 个技能文件、29 个测试文件
- **Phase C**: 深度扫描代理、网关、沙箱、中间件等关键文件

**主要发现**:
- 后端测试覆盖率高（29 个测试文件，77+ 测试用例）
- 前端未配置测试框架，建议添加 Vitest/Playwright
- 支持多种 LLM 提供商（OpenAI、Anthropic、DeepSeek、Google Gemini）
- 完整的 MCP 服务器集成，支持 OAuth
- 三个 IM 通道支持（Telegram、Slack、Feishu）

**技术栈**:
- **后端**: Python 3.12+、FastAPI、LangGraph、LangChain
- **前端**: TypeScript 5.8、Next.js 16、React 19、TanStack Query
- **AI**: LangGraph 1.0.6+、LangChain 1.2.3+
- **工具**: Tavily、Jina AI、InfoQuest、Firecrawl

---

*此文档由 AI 自动生成和维护。如需更新，请运行文档生成脚本。*
