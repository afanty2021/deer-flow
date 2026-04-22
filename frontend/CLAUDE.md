# 前端模块 - DeerFlow

[根目录](../CLAUDE.md) > **frontend**

> **最后更新**: 2026-04-13
> **框架**: Next.js 16 + React 19 + TypeScript 5.8
> **包管理器**: pnpm 10.26.2+
> **Node.js**: 22+

## 模块概述

DeerFlow 前端是一个 Next.js 16 网页界面，提供基于线程的 AI 对话、流式响应、工件预览和任务跟踪功能。它通过 LangGraph SDK 与后端 LangGraph 服务通信。

## 命令

| 命令 | 用途 |
| --- | --- |
| `pnpm dev` | 开发服务器，使用 Turbopack (http://localhost:3000) |
| `pnpm build` | 生产构建 |
| `pnpm check` | Lint + 类型检查（提交前运行） |
| `pnpm lint` | 仅 ESLint |
| `pnpm lint:fix` | ESLint 自动修复 |
| `pnpm test` | 使用 Vitest 运行单元测试 |
| `pnpm test:e2e` | 使用 Playwright (Chromium) 运行 E2E 测试 |
| `pnpm typecheck` | TypeScript 类型检查 (`tsc --noEmit`) |
| `pnpm start` | 启动生产服务器 |

单元测试位于 `tests/unit/` 下，镜像 `src/` 布局（例如 `tests/unit/core/api/stream-mode.test.ts` 测试 `src/core/api/stream-mode.ts`）。由 Vitest 驱动；通过 `@/` 路径别名导入源模块。

E2E 测试位于 `tests/e2e/` 下，使用 Playwright 配合 Chromium。它们通过 `page.route()` 网络拦截模拟所有后端 API，测试真实的页面交互（导航、聊天输入、流式响应）。配置文件：`playwright.config.ts`。

## 架构

```
Frontend (Next.js) ──▶ LangGraph SDK ──▶ LangGraph Backend (lead_agent)
                                              ├── Sub-Agents
                                              └── Tools & Skills
```

### 数据流

1. 用户输入 → 线程钩子 (`core/threads/hooks.ts`) → LangGraph SDK 流式传输
2. 流式事件更新线程状态（消息、工件、待办）
3. TanStack Query 管理服务端状态；localStorage 存储用户设置
4. 组件订阅线程状态并渲染更新

### 关键模式

- **默认使用服务端组件**，仅在交互式组件上使用 `"use client"`
- **线程钩子**（`useThreadStream`、`useSubmitThread`、`useThreads`）是主要的 API 接口
- **LangGraph 客户端**是通过 `core/api/` 中的 `getAPIClient()` 获取的单例
- **环境变量验证**使用 `@t3-oss/env-nextjs` 和 Zod 模式（`src/env.js`）。使用 `SKIP_ENV_VALIDATION=1` 跳过

## 模块结构

```
frontend/
├── src/
│   ├── app/                    # Next.js App Router
│   │   ├── layout.tsx          # 根布局（包含 providers）
│   │   ├── page.tsx            # 落地页
│   │   └── workspace/          # 聊天工作区
│   │       └── chats/
│   │           └── [thread_id]/  # 线程页面
│   ├── components/             # React 组件
│   │   ├── ui/                 # Shadcn UI 原语（自动生成）
│   │   ├── ai-elements/        # Vercel AI SDK 元素（自动生成）
│   │   ├── workspace/          # 聊天页面组件
│   │   │   ├── messages/       # 消息显示
│   │   │   ├── artifacts/      # 工件预览
│   │   │   ├── chats/          # 聊天界面
│   │   │   └── settings/       # 设置面板
│   │   └── landing/            # 落地页区块
│   ├── core/                   # 业务逻辑
│   │   ├── threads/            # 线程管理
│   │   │   ├── hooks.ts        # React 线程钩子
│   │   │   ├── types.ts        # TypeScript 类型
│   │   │   └── utils.ts        # 线程工具
│   │   ├── api/                # LangGraph 客户端
│   │   │   ├── api-client.ts   # 单例 API 客户端
│   │   │   └── stream-mode.ts  # 流式传输工具
│   │   ├── artifacts/          # 工件处理
│   │   │   ├── hooks.ts        # 工件钩子
│   │   │   ├── loader.ts       # 工件加载
│   │   │   └── utils.ts        # 工件工具
│   │   ├── i18n/               # 国际化
│   │   │   ├── locales/        # en-US、zh-CN
│   │   │   ├── hooks.ts        # i18n 钩子
│   │   │   └── server.ts       # 服务端 i18n
│   │   ├── settings/           # 用户设置
│   │   │   ├── hooks.ts        # 设置钩子
│   │   │   └── local.ts        # localStorage
│   │   ├── memory/             # 记忆系统
│   │   │   ├── hooks.ts        # 记忆钩子
│   │   │   └── types.ts        # 记忆类型
│   │   ├── skills/             # 技能管理
│   │   │   ├── hooks.ts        # 技能钩子
│   │   │   └── type.ts         # 技能类型
│   │   ├── mcp/                # MCP 集成
│   │   │   ├── hooks.ts        # MCP 钩子
│   │   │   └── types.ts        # MCP 类型
│   │   ├── models/             # 模型管理
│   │   │   ├── hooks.ts        # 模型钩子
│   │   │   └── types.ts        # 模型类型
│   │   ├── agents/             # 代理系统
│   │   │   ├── hooks.ts        # 代理钩子
│   │   │   └── types.ts        # 代理类型
│   │   ├── messages/           # 消息处理
│   │   │   └── utils.ts        # 消息工具
│   │   ├── uploads/            # 文件上传
│   │   │   ├── hooks.ts        # 上传钩子
│   │   │   └── api.ts          # 上传 API
│   │   ├── tasks/              # 任务/待办处理
│   │   │   └── types.ts        # 任务类型
│   │   ├── config/             # 配置
│   │   │   └── index.ts        # 应用配置
│   │   ├── utils/              # 工具函数
│   │   │   └── ...             # 辅助函数
│   │   └── rehype/             # Markdown 处理
│   ├── hooks/                  # 共享 React 钩子
│   │   └── use-mobile.ts       # 移动端检测
│   ├── lib/                    # 工具库
│   │   └── utils.ts            # Tailwind cn() 函数
│   ├── server/                 # 服务端代码
│   │   └── better-auth/        # 认证（尚未激活）
│   ├── styles/                 # 全局样式
│   │   └── globals.css         # Tailwind + CSS 变量
│   ├── env.js                  # 环境变量验证
│   └── typings/                # TypeScript 定义
│       └── md.d.ts             # Markdown 类型
├── tests/                      # 测试目录
│   └── unit/                   # 单元测试
│       └── core/               # 核心模块测试
│           ├── api/            # API 测试
│           ├── threads/        # 线程测试
│           └── uploads/        # 上传测试
├── public/                     # 静态资源
├── next.config.js              # Next.js 配置
├── tailwind.config.ts          # Tailwind 配置
├── tsconfig.json               # TypeScript 配置
├── vitest.config.ts            # Vitest 配置
├── package.json                # 依赖
└── .env.example                # 环境变量模板
```

## 入口点 & 启动

### 开发服务器

```bash
# 从前端目录
pnpm dev

# 或从项目根目录
make dev
```

**入口点**: `src/app/layout.tsx` → `src/app/page.tsx`

### 生产构建

```bash
pnpm build
pnpm start
```

## 代码风格

- **导入顺序**: 强制排序（内置 → 外部 → 内部 → 父级 → 同级），按字母排序，组之间用空行分隔。使用内联类型导入：`import { type Foo }`
- **未使用的变量**: 前缀为 `_`
- **类名**: 使用 `@/lib/utils` 中的 `cn()` 处理条件 Tailwind 类
- **路径别名**: `@/*` 映射到 `src/*`
- **组件**: `ui/` 和 `ai-elements/` 从注册表生成（Shadcn、MagicUI、React Bits、Vercel AI SDK）—— 不要手动编辑

## 环境变量

后端 API URL 是可选的；默认使用 nginx 代理：

```
NEXT_PUBLIC_BACKEND_BASE_URL=http://localhost:8001
NEXT_PUBLIC_LANGGRAPH_BASE_URL=http://localhost:2024
```

在 `src/env.js` 中定义，使用 Zod 验证：

```typescript
// 后端 URL（可选，默认通过 nginx 代理）
NEXT_PUBLIC_BACKEND_BASE_URL: string | undefined;  // 默认: "" (通过 nginx)
NEXT_PUBLIC_LANGGRAPH_BASE_URL: string;             // 默认: "/api/langgraph"

// 跳过 Docker 构建的验证
SKIP_ENV_VALIDATION: "true" | "false";
```

### Next.js 配置 (`next.config.js`)

```javascript
const config = {
  devIndicators: false,  // 隐藏开发指示器
};
```

### Tailwind 配置 (`tailwind.config.ts`)

使用 Tailwind v4，在 `src/styles/globals.css` 中使用 `@import` 语法。

主题的 CSS 变量在 globals.css 中定义。

## 核心接口

### 线程钩子 (`src/core/threads/hooks.ts`)

```typescript
// 线程管理
const useThreads = () => ({
  threads: Thread[],
  createThread: () => Promise<Thread>,
  deleteThread: (id: string) => Promise<void>,
  renameThread: (id: string, title: string) => Promise<void>,
});

// 线程流式传输
const useThreadStream = (threadId: string) => ({
  stream: (message: string) => AsyncGenerator<StreamEvent>,
  stop: () => void,
  isStreaming: boolean,
});

// 线程提交
const useSubmitThread = (threadId: string) => ({
  submit: (message: string, files?: File[]) => Promise<void>,
  isSubmitting: boolean,
});
```

### API 客户端 (`src/core/api/api-client.ts`)

```typescript
class APIClient {
  // 线程
  createThread(): Promise<Thread>;
  getThread(threadId: string): Promise<Thread>;
  deleteThread(threadId: string): Promise<void>;

  // 消息
  createMessage(threadId: string, content: string): AsyncGenerator<StreamEvent>;

  // 运行
  createRun(threadId: string, messageId: string): AsyncGenerator<StreamEvent>;
}
```

### 流式事件 (`src/core/api/stream-mode.ts`)

```typescript
type StreamEvent =
  | { type: "values"; data: ThreadState }
  | { type: "messages-tuple"; data: Message }
  | { type: "end" };
```

## 依赖项

### 核心框架

**Next.js 16**:
- `next@^16.1.4` - React 框架
- `react@^19.0.0` - UI 库
- `react-dom@^19.0.0` - DOM 渲染

**TypeScript**:
- `typescript@^5.8.2` - 类型系统
- `typescript-eslint@^8.27.0` - TypeScript ESLint

### UI 库

**Shadcn UI** (Radix UI 原语):
- `@radix-ui/react-*` - 无头 UI 组件
- `class-variance-authority` - 组件变体
- `tailwind-merge` - Tailwind 类合并

**Vercel AI SDK**:
- `ai@^6.0.33` - AI 流式传输工具
- `@langchain/langgraph-sdk@^1.5.3` - LangGraph 客户端
- `streamdown@1.4.0` - Markdown 流式渲染器

**其他 UI**:
- `@xyflow/react@^12.10.0` - 流程图
- `codemirror@^6.0.2` - 代码编辑器
- `sonner@^2.0.7` - Toast 通知
- `cmdk@^1.1.1` - 命令面板

### 工具库

**状态管理**:
- `@tanstack/react-query@^5.90.17` - 服务端状态
- `zustand` (如果使用) - 客户端状态

**样式**:
- `tailwindcss@^4.0.15` - 原子化 CSS
- `postcss@^8.5.3` - CSS 处理
- `tw-animate-css@^1.4.0` - 动画

**其他**:
- `nanoid@^5.1.6` - ID 生成
- `date-fns@^4.1.0` - 日期工具
- `zod@^3.24.2` - 模式验证

### 开发依赖

- `eslint@^9.23.0` - 代码检查
- `eslint-config-next@^15.2.3` - Next.js ESLint 配置
- `prettier@^3.5.3` - 代码格式化
- `prettier-plugin-tailwindcss@^0.6.11` - Tailwind 格式化
- `vitest@^3.0.0` - 单元测试框架

## 数据模型

### 线程 (`src/core/threads/types.ts`)

```typescript
interface Thread {
  thread_id: string;
  title: string | null;
  created_at: string;
  updated_at: string;
  values?: {
    artifacts?: Artifact[];
    todos?: TodoItem[];
  };
}
```

### 消息 (`src/core/messages/types.ts`)

```typescript
interface Message {
  id: string;
  role: "user" | "assistant" | "system";
  content: string;
  created_at?: string;
  tool_calls?: ToolCall[];
}
```

### 工件 (`src/core/artifacts/types.ts`)

```typescript
interface Artifact {
  path: string;
  type: "image" | "file" | "directory";
  content?: string;
}
```

### 待办项 (`src/core/tasks/types.ts`)

```typescript
interface TodoItem {
  id: string;
  content: string;
  status: "pending" | "in_progress" | "completed";
  created_at: string;
}
```

## 代码质量

### 类型检查

```bash
pnpm typecheck
# 或
tsc --noEmit
```

### 代码检查

```bash
pnpm lint
# 或
eslint . --ext .ts,.tsx

# 自动修复
pnpm lint:fix
# 或
eslint . --ext .ts,.tsx --fix
```

### 检查全部

```bash
pnpm check
# 运行: lint + typecheck
```

### 测试

```bash
pnpm test
# 运行 Vitest 单元测试
```

## 核心功能

### 基于线程的聊天

- **线程创建**: 首次消息时自动创建
- **线程列表**: 在侧边栏显示
- **线程管理**: 重命名、删除、切换
- **实时更新**: 通过 SSE 流式响应

### 流式响应

使用 LangGraph SDK 进行服务器发送事件：

```typescript
for await (const event of client.stream(threadId, messageId)) {
  if (event.event === "messages/tuples") {
    // 用新消息内容更新 UI
  }
}
```

### 工件预览

- **自动检测**: 从代理输出检测工件
- **特定类型渲染**: 图片、文件、代码
- **懒加载**: 按需加载
- **下载支持**: 文件下载

### 待办跟踪

- **实时更新**: 来自代理的 TodoList
- **状态跟踪**: pending → in_progress → completed
- **视觉反馈**: 进度指示器

### 文件上传

- **多文件支持**: 上传多个文件
- **文档转换**: PDF、PPT、Excel、Word
- **预览**: 在聊天中显示上传的文件

### 国际化

- **支持语言**: en-US、zh-CN
- **服务端**: 基于 Cookie 的语言环境检测
- **客户端**: 基于钩子的翻译

## 路由

### App Router 结构

```
/ (落地页)
/workspace/chats/[thread_id] (聊天页面)
```

### 动态路由

- `/workspace/chats/[thread_id]` - 线程特定聊天
  - 使用 `useThreadStream` 进行流式传输
  - 显示消息、工件、待办
  - 处理文件上传

## 状态管理

### 服务端状态 (TanStack Query)

用于：
- 线程列表
- 模型配置
- 技能状态
- 记忆数据
- MCP 配置

### 客户端状态 (React 钩子 + useState)

用于：
- 当前线程
- 流式传输状态
- UI 状态（模态框、面板）

### localStorage

用于：
- 用户偏好
- 设置
- 主题

## 测试

单元测试使用 **Vitest** 框架：

- 测试文件位于 `tests/unit/` 下
- 测试文件镜像 `src/` 布局
- 使用 `@/` 路径别名导入源模块
- 配置文件: `vitest.config.ts`

### 测试示例

```typescript
// tests/unit/core/api/stream-mode.test.ts
import { describe, it, expect } from 'vitest';
import { someFunction } from '@/core/api/stream-mode';

describe('stream-mode', () => {
  it('should work correctly', () => {
    expect(someFunction()).toBe(true);
  });
});
```

### 运行测试

```bash
# 运行所有测试
pnpm test

# 监听模式
pnpm test --watch

# 覆盖率
pnpm test --coverage
```

## 相关文件

### 配置
- `next.config.js` - Next.js 配置
- `tailwind.config.ts` - Tailwind 配置
- `tsconfig.json` - TypeScript 配置
- `vitest.config.ts` - Vitest 配置
- `package.json` - 依赖
- `.env.example` - 环境变量模板

### 关键组件
- `src/app/layout.tsx` - 根布局
- `src/app/page.tsx` - 落地页
- `src/app/workspace/chats/[thread_id]/page.tsx` - 聊天页面

### 核心逻辑
- `src/core/threads/hooks.ts` - 线程管理
- `src/core/api/api-client.ts` - API 客户端
- `src/core/artifacts/hooks.ts` - 工件处理

## 变更记录

### 2026-04-13 - 合并上游更新
- ✅ 添加 Vitest 测试框架支持
- ✅ 更新命令表格，添加 `pnpm test` 命令
- ✅ 更新模块结构图，添加 `tests/` 目录
- ✅ 更新依赖列表，添加 vitest
- ✅ 添加测试章节说明
- ✅ 更新日期为 2026-04-13

### 2025-03-13 - 模块文档更新
- 添加面包屑导航
- 重组结构，章节更清晰
- 添加模块结构图
- 记录所有核心接口
- 增强依赖和代码风格章节
- **文档改为简体中文**
