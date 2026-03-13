# 后端模块 - DeerFlow

[根目录](../CLAUDE.md) > **backend**

> **最后更新**: 2025-03-13
> **Python 版本**: 3.12+
> **包管理器**: uv

## 模块概述

DeerFlow 后端是一个基于 LangGraph 的 AI 代理系统，提供具有沙箱执行、持久化记忆、子代理委托和可扩展工具集成的"超级代理"。它由两个主要服务组成：

1. **LangGraph 服务器** (端口 2024)：代理运行时和工作流执行
2. **Gateway API** (端口 8001)：模型、MCP、技能、记忆、工件的 REST API

## 模块结构

```
backend/
├── src/
│   ├── agents/              # LangGraph 代理系统
│   │   ├── lead_agent/      # 主代理工厂 + 提示词
│   │   ├── middlewares/     # 11 个中间件组件
│   │   ├── memory/          # 记忆提取 & 队列
│   │   └── thread_state.py  # ThreadState 模式定义
│   ├── gateway/             # FastAPI Gateway API
│   │   ├── app.py           # FastAPI 应用
│   │   └── routers/         # 9 个路由模块
│   ├── sandbox/             # 沙箱执行系统
│   │   ├── local/           # 本地文件系统提供者
│   │   ├── sandbox.py       # 抽象沙箱接口
│   │   ├── tools.py         # bash、ls、read/write/str_replace
│   │   └── middleware.py    # 沙箱生命周期管理
│   ├── subagents/           # 子代理委托系统
│   │   ├── builtins/        # general-purpose、bash 代理
│   │   ├── executor.py      # 后台执行引擎
│   │   └── registry.py      # 代理注册表
│   ├── tools/builtins/      # 内置工具
│   ├── mcp/                 # MCP 集成
│   ├── models/              # 模型工厂
│   ├── skills/              # 技能发现 & 加载
│   ├── config/              # 配置系统
│   ├── community/           # 社区工具 (tavily、jina_ai 等)
│   ├── reflection/          # 动态模块加载
│   ├── utils/               # 工具函数
│   ├── channels/            # IM 通道 (Feishu、Slack、Telegram)
│   └── client.py            # 嵌入式 Python 客户端
├── tests/                   # 29 个测试文件
├── docs/                    # 15 个文档文件
├── langgraph.json           # LangGraph 服务器配置
├── pyproject.toml           # Python 依赖
└── Makefile                 # 后端命令
```

## 入口点 & 启动

### LangGraph 服务器

```bash
# 从后端目录
make dev

# 或直接运行
uv run langgraph dev --no-browser --allow-blocking
```

**入口点**: `src.agents:make_lead_agent` (在 `langgraph.json` 中注册)

### Gateway API

```bash
# 从后端目录
make gateway

# 或直接运行
uv run uvicorn src.gateway.app:app --host 0.0.0.0 --port 8001
```

**入口点**: `src.gateway.app:create_app()`

## 核心接口

### 代理系统 (`src/agents/`)

**主代理工厂** (`lead_agent/agent.py`):
```python
def make_lead_agent(config: RunnableConfig) -> CompiledGraph:
    """创建包含所有中间件和工具的主代理。"""
```

**ThreadState** (`thread_state.py`):
```python
class ThreadState(AgentState):
    sandbox: Sandbox | None
    thread_data: ThreadData
    title: str | None
    artifacts: dict[str, Artifact]
    todos: list[TodoItem]
    uploaded_files: list[str]
    viewed_images: dict[str, str]
```

### Gateway API 路由 (`src/gateway/routers/`)

| 路由 | 基础路径 | 关键端点 |
|------|----------|----------|
| **agents** | `/api/agents` | 线程管理、聊天流式传输 |
| **models** | `/api/models` | `GET /`、`GET /{name}` |
| **mcp** | `/api/mcp` | `GET /config`、`PUT /config` |
| **skills** | `/api/skills` | `GET /`、`GET /{name}`、`PUT /{name}`、`POST /install` |
| **memory** | `/api/memory` | `GET /`、`POST /reload`、`GET /config`、`GET /status` |
| **uploads** | `/api/threads/{id}/uploads` | `POST /`、`GET /list`、`DELETE /{filename}` |
| **artifacts** | `/api/threads/{id}/artifacts` | `GET /{path}` |
| **channels** | `/api/channels` | 通道状态、命令 |
| **suggestions** | `/api/suggestions` | 聊天建议 |

### 沙箱接口 (`src/sandbox/sandbox.py`)

```python
class Sandbox(ABC):
    @abstractmethod
    def execute_command(self, command: str) -> str: ...

    @abstractmethod
    def read_file(self, path: str) -> str: ...

    @abstractmethod
    def write_file(self, path: str, content: str, append: bool = False) -> None: ...

    @abstractmethod
    def list_dir(self, path: str, max_depth: int = 2) -> list[str]: ...
```

## 依赖项

### 核心依赖 (`pyproject.toml`)

**代理框架**:
- `langchain>=1.2.3` - LLM 框架
- `langgraph>=1.0.6` - 多代理编排
- `langgraph-api>=0.7.0,<0.8.0` - LangGraph 服务器
- `langgraph-runtime-inmem>=0.22.1` - 内存运行时

**LLM 提供商**:
- `langchain-openai>=1.1.7` - OpenAI 集成
- `langchain-anthropic>=1.3.4` - Anthropic Claude
- `langchain-deepseek>=1.0.1` - DeepSeek 模型
- `langchain-google-genai>=4.2.1` - Google Gemini

**API & 工具**:
- `fastapi>=0.115.0` - REST API 框架
- `uvicorn[standard]>=0.34.0` - ASGI 服务器
- `langchain-mcp-adapters>=0.1.0` - MCP 集成
- `tavily-python>=0.7.17` - 网络搜索
- `firecrawl-py>=1.15.0` - 网页抓取

**IM 通道**:
- `python-telegram-bot>=21.0` - Telegram
- `slack-sdk>=3.33.0` - Slack
- `lark-oapi>=1.4.0` - Feishu/Lark

**工具**:
- `pydantic>=2.12.5` - 数据验证
- `pyyaml>=6.0.3` - YAML 解析
- `tiktoken>=0.8.0` - Token 计数
- `markitdown[all,xlsx]>=0.0.1a2` - 文档转换

**开发依赖**:
- `pytest>=8.0.0` - 测试框架
- `ruff>=0.14.11` - 代码检查和格式化

## 数据模型

### 线程状态 (`src/agents/thread_state.py`)

```python
@dataclass
class Artifact:
    path: str
    type: str  # "image" | "file" | "directory"
    content: str | None

@dataclass
class TodoItem:
    id: str
    content: str
    status: str  # "pending" | "in_progress" | "completed"
    created_at: str

@dataclass
class ThreadData:
    thread_id: str
    assistant_id: str
    base_dir: str
    workspace_dir: str
    uploads_dir: str
    outputs_dir: str
```

### 记忆结构 (`src/agents/memory/`)

```python
@dataclass
class Fact:
    id: str
    content: str
    category: str  # "preference" | "knowledge" | "context" | "behavior" | "goal"
    confidence: float  # 0-1
    created_at: str
    source: str

@dataclass
class MemoryData:
    workContext: str | None
    personalContext: str | None
    topOfMind: str | None
    recentMonths: str | None
    earlierContext: str | None
    longTermBackground: str | None
    facts: list[Fact]
```

## 配置

### 主配置 (`config.yaml` - 项目根目录)

优先级：
1. `DEER_FLOW_CONFIG_PATH` 环境变量
2. `backend/` 中的 `config.yaml`
3. 项目根目录中的 `config.yaml`（推荐）

关键部分：
- `models[]` - LLM 配置
- `tools[]` - 工具定义
- `sandbox` - 沙箱提供者
- `skills` - 技能路径
- `memory` - 记忆系统
- `channels` - IM 集成
- `subagents` - 子代理设置
- `title` - 自动标题生成
- `summarization` - 上下文摘要

### 扩展配置 (`extensions_config.json` - 项目根目录)

```json
{
  "mcpServers": {
    "server-name": {
      "enabled": true,
      "type": "stdio|sse|http",
      "command": "...",
      "args": [...],
      "oauth": {...}
    }
  },
  "skills": {
    "skill-name": {
      "enabled": true
    }
  }
}
```

## 测试

### 测试覆盖

- **29 个测试文件** 在 `backend/tests/` 中
- **77 个单元测试** 在 `test_client.py` 中（包括 Gateway 一致性）
- **CI/CD**: GitHub Actions 在每次 PR 时运行

### 关键测试套件

| 测试文件 | 覆盖内容 |
|-----------|----------|
| `test_client.py` | 嵌入式客户端、Gateway 一致性 |
| `test_memory_*.py` | 记忆系统（注入、更新） |
| `test_mcp_*.py` | MCP 集成、OAuth |
| `test_subagent_*.py` | 子代理执行、超时 |
| `test_title_*.py` | 标题生成 |
| `test_uploads_*.py` | 文件上传处理 |
| `test_docker_sandbox_mode_detection.py` | Docker 沙箱模式 |
| `test_provisioner_kubeconfig.py` | Kubeconfig 处理 |

### 运行测试

```bash
# 所有测试
make test

# 特定测试文件
PYTHONPATH=. uv run pytest tests/test_client.py -v

# 带覆盖率
PYTHONPATH=. uv run pytest --cov=src tests/
```

## 代码质量

### 代码检查 & 格式化

```bash
# 检查
make lint
# 或
uvx ruff check .

# 格式化
make format
# 或
uvx ruff check . --fix && uvx ruff format .
```

**配置**:
- 行长度：240 字符
- Python 3.12+ 带类型提示
- 双引号、空格缩进

## 核心功能实现

### 中间件链

中间件在 `src/agents/lead_agent/agent.py` 中按严格顺序执行：

1. **ThreadDataMiddleware** - 创建每线程目录
2. **UploadsMiddleware** - 跟踪上传文件
3. **SandboxMiddleware** - 获取沙箱实例
4. **DanglingToolCallMiddleware** - 处理不完整的工具调用
5. **SummarizationMiddleware** - 上下文缩减（可选）
6. **TodoListMiddleware** - 任务跟踪（可选）
7. **TitleMiddleware** - 自动生成标题
8. **MemoryMiddleware** - 排队等待记忆更新
9. **ViewImageMiddleware** - 注入 base64 图片
10. **SubagentLimitMiddleware** - 强制并发限制
11. **ClarificationMiddleware** - 处理澄清请求（必须是最后一个）

### 虚拟路径系统

**代理看到**:
- `/mnt/user-data/workspace`
- `/mnt/user-data/uploads`
- `/mnt/user-data/outputs`
- `/mnt/skills`

**物理路径**:
- `backend/.deer-flow/threads/{thread_id}/user-data/workspace`
- `backend/.deer-flow/threads/{thread_id}/user-data/uploads`
- `backend/.deer-flow/threads/{thread_id}/user-data/outputs`
- `skills/` (来自项目根目录)

### 工具系统

`get_available_tools()` 从以下来源组装工具：
1. **配置定义的工具** - 从 `config.yaml`
2. **MCP 工具** - 从启用的 MCP 服务器
3. **内置工具**:
   - `present_files` - 展示输出文件
   - `ask_clarification` - 请求用户输入
   - `view_image` - 读取图片为 base64
4. **子代理工具**（如果启用）:
   - `task` - 委托给子代理

## 嵌入式客户端

`DeerFlowClient` (`src/client.py`) 提供进程内访问：

```python
from src.client import DeerFlowClient

client = DeerFlowClient()

# 聊天
response = client.chat("分析这篇论文", thread_id="my-thread")

# 流式传输
for event in client.stream("你好"):
    if event.type == "messages-tuple":
        print(event.data["content"])

# 管理
models = client.list_models()
skills = client.list_skills()
memory = client.get_memory()
```

**Gateway 一致性**: 所有返回字典的方法都符合 Gateway Pydantic 模式（通过 CI 中的 `TestGatewayConformance` 验证）。

## IM 通道

**支持的平台**:
- **Telegram** - Bot API (长轮询)
- **Slack** - Socket Mode
- **Feishu/Lark** - WebSocket

**组件**:
- `message_bus.py` - 异步发布/订阅中心
- `store.py` - 线程 ID 持久化
- `manager.py` - 核心分发器
- `base.py` - 抽象基类
- `service.py` - 生命周期管理

## 相关文件

### 配置
- `langgraph.json` - LangGraph 服务器配置
- `pyproject.toml` - Python 依赖
- `config.yaml` - 主应用配置（项目根目录）
- `extensions_config.json` - MCP/技能配置（项目根目录）

### 文档
- `docs/ARCHITECTURE.md` - 技术架构
- `docs/CONFIGURATION.md` - 配置指南
- `docs/API.md` - API 参考
- `docs/FILE_UPLOAD.md` - 文件上传功能
- `docs/MCP_SERVER.md` - MCP 集成
- `docs/plan_mode_usage.md` - 计划模式与 TodoList
- `docs/summarization.md` - 上下文摘要

## 变更记录

### 2025-03-13 - 模块文档更新
- 添加面包屑导航
- 重组结构，章节更清晰
- 添加模块结构图
- 记录所有核心接口
- 增强测试和代码质量章节
- **文档改为简体中文**
