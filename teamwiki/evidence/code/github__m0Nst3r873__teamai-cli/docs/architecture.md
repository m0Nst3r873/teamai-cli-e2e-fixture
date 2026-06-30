已收集完整的架构信息，现在生成文档。

# TeamAI CLI — 技术架构总览

## 项目概述

TeamAI CLI 是一个 Git-native 的团队 AI 知识共享框架，通过 CLI 工具在团队成员的 AI 编码工具（Claude Code、Cursor、Codex、CodeBuddy）之间同步 skills、rules、docs、env、agents、hooks、wiki 七类资源。核心设计哲学：以 Git 仓库作为同步传输层（无需服务端）、BM25 全文检索实现离线可解释的知识召回、Provider 抽象屏蔽 GitHub/TGit 双平台差异。

## 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLI Entry (Commander)                        │
│  init │ push │ pull │ recall │ import │ codebase │ hooks │ doctor   │
└────┬──────┬──────┬──────┬──────┬──────┬──────┬──────┬───────────────┘
     │      │      │      │      │      │      │      │
     ▼      ▼      ▼      ▼      ▼      ▼      ▼      ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        Command Layer                                  │
│  init.ts │ push.ts │ pull.ts │ recall.ts │ import.ts │ codebase-cmd │
│  status.ts │ hooks-cmd.ts │ doctor.ts │ contribute.ts │ ...          │
└────┬──────────────┬──────────────┬──────────────┬───────────────────┘
     │              │              │              │
     ▼              ▼              ▼              ▼
┌──────────┐ ┌──────────────┐ ┌──────────────┐ ┌──────────────────────┐
│ Resources│ │  Providers   │ │ Recall/Search│ │   Wiki Engine        │
│ Handler  │ │  (Git Host)  │ │              │ │                      │
│──────────│ │──────────────│ │──────────────│ │──────────────────────│
│ base.ts  │ │ types.ts     │ │ recall.ts    │ │ core/                │
│ skills   │ │ registry.ts  │ │ search-index │ │ adapters/            │
│ rules    │ │ github/      │ │ code-knowl-  │ │ code-knowledge/      │
│ docs     │ │ tgit/        │ │   edge-recall│ │ manifest-compiler    │
│ env      │ │              │ │ auto-recall  │ │ doc-graph-extractor  │
│ agents   │ └──────┬───────┘ └──────┬───────┘ │ interface-scanner    │
│ hooks    │        │                │          └──────────┬───────────┘
│ wiki     │        │                │                     │
└────┬─────┘        │                │                     │
     │              │                │                     │
     ▼              ▼                ▼                     ▼
┌─────────────────────────────────────────────────────────────────────┐
│                          Utils Layer                                  │
│  git.ts │ fs.ts │ ai-client.ts │ cache-index.ts │ search-index.ts   │
│  logger.ts │ path-safety.ts │ tool-names.ts │ repo-cache.ts │ ...   │
└────┬──────────────┬──────────────┬──────────────────────────────────┘
     │              │              │
     ▼              ▼              ▼
┌──────────┐ ┌──────────────┐ ┌──────────────┐
│ Git Repo │ │  Local FS    │ │ AI Tools     │
│ (Team)   │ │ (~/.teamai/) │ │ (.claude/    │
│          │ │              │ │  .cursor/    │
│          │ │              │ │  .codex/ ...) │
└──────────┘ └──────────────┘ └──────────────┘
```

## 组件关系矩阵

| 源组件 | 目标组件 | 关系类型 | 通信方式 |
|--------|----------|----------|----------|
| CLI Entry (`index.ts`) | Command Layer | 路由分发 | Commander action → dynamic import |
| Command Layer | ResourceHandler | 资源生命周期调用 | 方法调用 (scanLocal/push/pull/remove) |
| Command Layer | GitProvider | 平台操作 | 接口调用 (clone/authenticate/createPR) |
| Command Layer | Recall/Search | 知识检索 | 函数调用 (BM25 query + graph query) |
| Command Layer | Wiki Engine | 知识图谱构建 | 函数调用 (extract/aggregate/compile) |
| ResourceHandler | Utils/FS | 文件读写 | 同步/异步函数 |
| ResourceHandler | Utils/Git | 仓库操作 | simple-git API |
| GitProvider | 外部 CLI | 平台认证与 API | 子进程 (`gh` / `gf` CLI) |
| Recall/Search | search-index | BM25 检索 | 内存索引查询 |
| Recall/Search | code-knowledge-recall | 代码图谱检索 | JSON 文件读取 + 关键词匹配 |
| Wiki Engine | code-knowledge/ | AST 提取 | 多语言 extractor (TS/Python/Go) |
| Wiki Engine | adapters/ | 模板渲染 | Markdown 模板拼装 |
| Hook Dispatch | hook-handlers | 事件路由 | handler registry 匹配 + 执行 |
| Hook Dispatch | builtin-hooks | 内置处理器 | 数据驱动注册 |
| Import (`import.ts`) | Providers | 外部仓库拉取 | git clone + 文件扫描 |
| Import | repo-cache | 缓存管理 | LRU 文件缓存 |

## 核心链路

### 链路 1：Pull（团队知识同步到本地）

```
用户执行 `teamai pull`
    │
    ▼
pull.ts: loadConfig() → 读取 ~/.teamai/config.yaml
    │
    ▼
utils/git.ts: git pull team-repo (fast-forward)
    │
    ▼
resources/index.ts: 遍历 7 个 ResourceHandler
    │
    ▼
每个 handler.scanTeamForPull() → 扫描 team-repo 目录
    │
    ▼
每个 handler.pullItem() → 注入到 .claude/ .cursor/ .codex/ 等目录
    │
    ▼
rebuild-wiki-index.ts: 重建 BM25 搜索索引
    │
    ▼
本地 AI 工具获得最新团队知识
```

### 链路 2：Recall（知识检索）

```
AI 工具 hook 或用户执行 `teamai recall "<query>"`
    │
    ▼
recall.ts: 解析 query，加载搜索索引
    │
    ▼
utils/search-index.ts: BM25 全文检索 (skills + learnings + docs + rules)
    │
    ▼
code-knowledge-recall.ts: 代码知识图谱检索 (graph-index.json)
    │
    ▼
合并结果 → 按相关性排序 → 格式化输出
    │
    ▼
usage-tracker.ts: 记录 auto-upvote（命中即加分）
    │
    ▼
stdout → AI agent 获得团队上下文
```

### 链路 3：Hook Dispatch（AI 工具事件拦截）

```
AI 工具触发事件 (PreToolUse / PostToolUse / Stop)
    │
    ▼
hook-dispatch-cli.ts: 解析 stdin JSON (tool_name, event_type, input)
    │
    ▼
hook-dispatch.ts: buildHandlerRegistry()
    ├── builtin-hooks.ts: 内置 handler 注册
    └── hooks.ts: 团队自定义 handler 加载
    │
    ▼
按 tool_name + event_type 匹配 handler
    │
    ▼
执行 handler → 输出 JSON response (allow/block/modify)
    │
    ▼
AI 工具根据 response 决定行为
```

## 技术栈

| 维度 | 技术 | 说明 |
|------|------|------|
| 语言 | TypeScript 5.7+ | 严格类型检查，ESM-only 输出 |
| 运行时 | Node.js 20+ | 最低版本要求，利用原生 ESM 支持 |
| 构建 | tsup | 零配置 TypeScript 打包，输出 ESM bundle |
| 测试 | Vitest + @vitest/coverage-v8 | 单元测试 + 覆盖率 |
| CLI 框架 | Commander 12.1+ | 命令解析、选项声明、子命令路由 |
| Git 操作 | simple-git 3.27+ | Node.js Git 封装，用于仓库克隆/拉取/提交 |
| 平台 CLI | `gh` (GitHub) / `gf` (TGit) | Provider 通过子进程调用平台 CLI 完成认证与 API |
| 搜索引擎 | 自研 BM25 (search-index.ts) | 零依赖离线全文检索，无需向量数据库 |
| Schema 校验 | Zod 3.24+ | 配置文件与 API 输入校验 |
| 配置解析 | gray-matter + yaml + smol-toml | Markdown frontmatter / YAML / TOML 多格式支持 |
| 文件系统 | fs-extra 11+ | 跨平台文件操作增强 |
| 终端 UI | chalk 5 + ora 8 | 彩色输出 + spinner 进度指示 |
| CI/CD | GitHub Actions + Coding CI | 双流水线并行发布 npm + tnpm |
| 发布 | npm (public) + tnpm (internal) | `teamai-cli` / `@tencent/teamai-cli` 双包同源 |

<!-- teamai:referenced-doc-ids: [repos/github__m0Nst3r873__teamai-cli] -->