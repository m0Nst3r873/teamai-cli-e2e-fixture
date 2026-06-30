# TeamAI CLI 技术架构总览

## 项目概述

TeamAI CLI 是一个团队级 AI 编码知识同步工具，以 Git 仓库为数据存储后端，将团队积累的 skills、rules、docs、hooks、agents 等资源统一管理并注入到各 AI 编码工具（Claude Code、Cursor、Codex、CodeBuddy）的本地配置中。同时提供代码库知识导入（AST 解析 → 知识图谱）和语义检索（BM25 多维评分）能力，使 AI 工具在编码时可引用团队沉淀的经验与规范。

## 架构图

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLI 入口层                                   │
│  index.ts (Commander 注册，lazy-import 各命令)                        │
└──────────┬──────────┬──────────┬──────────┬──────────┬──────────────┘
           │          │          │          │          │
           ▼          ▼          ▼          ▼          ▼
┌────────────────────────────────────────────────────────────────────┐
│                         命令层 (Commands)                            │
│  init │ pull │ push │ status │ recall │ import │ hooks │ aggregate  │
└───┬────────┬────────┬────────┬────────┬────────┬───────────────────┘
    │        │        │        │        │        │
    ▼        ▼        ▼        ▼        ▼        ▼
┌────────────────────┐  ┌──────────────┐  ┌──────────────────────────┐
│  资源处理层         │  │  Wiki 引擎    │  │  领域层 (Domains)         │
│  (resources/)      │  │  (wiki-engine)│  │  cluster/recommend/      │
│  Skills│Rules│Docs │  │  AST 解析     │  │  review/store            │
│  Hooks│Agents│Env  │  │  调用链追踪   │  │                          │
│  Wiki│Marketplace  │  │  知识合并     │  │                          │
└────────┬───────────┘  └──────┬───────┘  └────────────┬─────────────┘
         │                     │                       │
         ▼                     ▼                       ▼
┌────────────────────────────────────────────────────────────────────┐
│                        基础设施层 (Utils)                             │
│  git.ts │ search-index.ts │ cache-index.ts │ claudemd.ts │         │
│  ai-client.ts │ fs.ts │ tool-names.ts │ dedup.ts │ paths.ts       │
└────────┬──────────────┬───────────────────────────┬────────────────┘
         │              │                           │
         ▼              ▼                           ▼
┌─────────────────┐  ┌───────────────────┐  ┌───────────────────────┐
│  Provider 层     │  │  配置层            │  │  存储层                │
│  (providers/)   │  │  config.ts        │  │  ~/.teamai/ (本地)     │
│  GitHub │ TGit  │  │  types.ts (Zod)   │  │  Git 远端仓库 (团队)   │
│  registry.ts    │  │  teamai.yaml      │  │  graph-index.json      │
└─────────────────┘  └───────────────────┘  └───────────────────────┘
```

## 组件关系矩阵

| 上游组件 | 下游组件 | 关系类型 | 通信方式 |
|---------|---------|---------|---------|
| `index.ts` | 所有命令模块 | 注册/调度 | Commander lazy-import |
| `pull.ts` | `resources/*` | 遍历分发 | ResourceHandler.pull() 接口调用 |
| `push.ts` | `resources/*` + `providers/registry` | 收集变更 + 创建 MR | 接口调用 + Git 操作 |
| `recall.ts` | `search-index.ts` + `graph-index.json` | 查询 | BM25 评分 + 文件读取 |
| `import-repo.ts` | `wiki-engine/code-knowledge/` | 代码解析 | AST Extractor 调用 |
| `import-repo.ts` | `cache-index.ts` | 缓存写入 | 写锁保护 JSON 操作 |
| `resources/*.ts` | `utils/git.ts` / `utils/fs.ts` | 文件操作 | 函数调用 |
| `resources/hooks.ts` | `builtin-hooks.ts` | 合并 hooks | 数据合并 + 去重 |
| `providers/registry.ts` | `github/` 或 `tgit/` | 平台路由 | 根据 remote URL 自动分派 |
| `aggregate.ts` | `ai-client.ts` + `domains/` | AI 聚合 | LLM API 调用 + 域分类 |
| `config.ts` | 所有模块 | 配置注入 | 优先级链：CLI > 项目 > 用户 > 团队仓 > 默认 |

## 核心链路

### 链路 1：teamai pull（团队资源同步到本地）

```
用户执行 `teamai pull`
  → index.ts 路由到 pull.ts
  → git fetch 检测远端变更（增量 diff 快速路径）
  → config.ts 加载 teamai.yaml（角色过滤、资源配置）
  → resources/index.ts 遍历 8 类 ResourceHandler
  → 各 Handler 按类型处理：
      skills.ts  → 过滤角色 → 写入 ~/.claude/skills/
      rules.ts   → enforced 强制注入 → 写入 ~/.claude/rules/
      hooks.ts   → autoApply 合并 → 注入 settings.json
      docs.ts    → 写入 ~/.teamai/docs/
      agents.ts  → 写入 ~/.claude/agents/
  → search-index.ts 更新全文索引
  → 输出同步报告
```

### 链路 2：teamai recall（知识检索）

```
用户/Subagent 执行 `teamai recall "关键词"`
  → index.ts 路由到 recall.ts
  → 加载 search-index（预构建 BM25 索引）
  → 加载 graph-index.json（代码知识图谱）
  → 多维度并行评分：
      skills / learnings / docs / rules / codebase-graph
  → 按 score 排序 → 取 Top-K
  → 返回结构化摘要 + doc_ids（供 referenced-doc-ids 声明）
```

### 链路 3：teamai import --from-repo（代码库知识导入）

```
用户执行 `teamai import --from-repo <url>`
  → index.ts 路由到 import-repo.ts
  → git clone / fetch 目标仓库到缓存
  → wiki-engine/code-knowledge/extractors/ 按语言 AST 解析
      (支持 TS/Python/Go/Java/Rust)
  → 提取模块/函数/类/调用关系
  → knowledge-reconciler.ts 知识合并与去重
  → manifest-compiler.ts 编译知识清单
  → 写入 graph-index.json（文件写锁保护并发安全 #67）
  → 写入 docs/team-codebase/ 结构化文档
  → cache-index.ts 更新缓存哈希（增量导入支持）
```

## 技术栈

| 维度 | 技术 | 说明 |
|------|------|------|
| 语言 | TypeScript (ESM-only) | Node.js 20+ 生态对齐，chalk 5+ 要求 ESM |
| 运行时 | Node.js 20+ | LTS 版本，原生 ESM 支持 |
| CLI 框架 | Commander.js | 轻量、无额外依赖、lazy-import 友好 |
| 构建工具 | tsup | 零配置 TypeScript 打包，输出 ESM |
| 测试框架 | Vitest | 原生 ESM/TS 支持，与 tsup 生态一致 |
| Schema 验证 | Zod | 类型推断 + 运行时校验 + 配置默认值一体化 |
| Git 操作 | simple-git | Node.js Git 封装，支持 fetch/diff/clone 等 |
| 搜索引擎 | 自研 BM25 (search-index.ts) | 离线构建索引，查询时零文件扫描 |
| AST 解析 | 多语言 Extractor | TS/Python/Go/Java/Rust 分语言实现 |
| 数据存储 | Git 仓库 + JSON 文件 | 零基础设施、版本历史、复用平台 RBAC |
| 平台适配 | Provider 抽象 (GitHub/TGit) | 统一接口屏蔽平台差异 |
| 包发布 | npm (public) + tnpm (internal) | tag push 触发双通道 CI 并行发布 |
| CI/CD | GitHub Actions + Coding CI | validate → build → e2e → publish |
| 本地状态 | `~/.teamai/` | 配置、缓存、索引、文档的本地持久化目录 |

<!-- teamai:referenced-doc-ids: [github__Tencent__teamai-cli] -->