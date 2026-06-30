# src 模块 — 组件设计文档

## 🤖 AI 快速理解要点

| 维度 | 描述 |
|------|------|
| **核心职责** | 团队 AI 知识资源的同步、检索与管理 |
| **架构层级** | CLI 应用层（Commander 命令 → 资源/Provider 抽象） |
| **上游组件** | tsup 构建系统、用户 shell 环境 |
| **下游组件** | GitHub API、TGit API、本地 AI 工具目录 |
| **代码入口** | `src/index.ts`（Commander 注册所有命令） |
| **核心机制** | ResourceHandler 多态 + GitProvider 平台抽象 |
| **数据流向** | 团队 repo ↔ 本地 AI 工具目录（双向同步） |
| **技术栈** | TypeScript / Node 20 / Commander / Zod / ESM |

## 架构设计

```
┌─────────────────────────────────────────────────────────────────────┐
│                         CLI Entry (index.ts)                        │
│   init │ pull │ push │ recall │ import │ status │ hooks │ doctor   │
└────┬────────┬────────┬─────────┬────────┬───────────────────────────┘
     │        │        │         │        │
     ▼        ▼        ▼         ▼        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Resources Layer (resources/)                    │
│  ┌────────┐ ┌─────┐ ┌────┐ ┌────┐ ┌──────┐ ┌────┐ ┌────┐ ┌────┐ │
│  │ skills │ │rules│ │docs│ │ env│ │agents│ │wiki│ │hooks│ │mkt │ │
│  └───┬────┘ └──┬──┘ └─┬──┘ └─┬──┘ └──┬───┘ └─┬──┘ └─┬──┘ └─┬──┘ │
│      └─────────┴──────┴──────┴───────┴───────┴──────┴───────┘     │
│                    ResourceHandler (base.ts)                        │
│          scanLocalForPush / scanTeamForPull / pullItem / pushItem   │
└────────────────────────────────┬────────────────────────────────────┘
                                 │
          ┌──────────────────────┼──────────────────────┐
          ▼                      ▼                      ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
│  Providers Layer │  │   Utils Layer    │  │  Wiki Engine Layer   │
│  (providers/)    │  │   (utils/)       │  │  (wiki-engine/)      │
│                  │  │                  │  │                      │
│ ┌──────┐┌─────┐ │  │ git.ts           │  │ code-knowledge/      │
│ │GitHub││TGit │ │  │ search-index.ts  │  │   extractors/ (TS/   │
│ └──┬───┘└──┬──┘ │  │ cache-index.ts   │  │   Py/Go/Java/Rust)   │
│    └────┬──┘    │  │ claudemd.ts      │  │ manifest-compiler.ts │
│  GitProvider IF  │  │ fs.ts / logger   │  │ knowledge-reconciler │
└──────────────────┘  └──────────────────┘  │ call-chain-tracer    │
                                            └──────────────────────┘
```

### 核心子模块说明

| 子模块 | 路径 | 职责 |
|--------|------|------|
| **Commands** | `src/*.ts` (根层) | 各命令实现，lazy-load 注册于 Commander |
| **Resources** | `src/resources/` | 8 种资源类型的 CRUD，统一继承 `ResourceHandler` 抽象类 |
| **Providers** | `src/providers/` | Git 平台抽象（GitHub/TGit），封装认证、clone、PR/MR 创建 |
| **Wiki Engine** | `src/wiki-engine/` | 多语言 AST 提取 → 代码图谱 → manifest 编译 → 知识调解 |
| **Utils** | `src/utils/` | git 操作、BM25 搜索索引、缓存管理、CLAUDE.md 注入、日志 |
| **Domains** | `src/domains/` | 跨仓库知识域分组、AI 推荐、漂移检测 |
| **CI** | `src/ci/` | MR/PR 知识提取与 comment 集成 |
| **Auto-Recall** | `src/auto-recall.ts` | PostToolUse hook：错误检测 → 关键词提取 → 自动触发 recall |

## 接口设计

### 对外 CLI 接口

| 命令 | 子命令 | 说明 |
|------|--------|------|
| `teamai init` | — | 初始化配置（选择 Git 平台、克隆团队 repo） |
| `teamai pull` | — | 从团队 repo 拉取资源 → 注入本地 AI 工具 |
| `teamai push` | — | 扫描本地新增资源 → 提交 PR/MR 到团队 repo |
| `teamai status` | — | 显示本地与团队 repo 的资源 diff |
| `teamai recall <query>` | — | BM25 + 图谱搜索团队知识库 |
| `teamai import` | `--from-repo` / `--from-mr` / `--from-org` | 批量导入外部知识（代码仓库 / MR / 组织） |
| `teamai hooks` | `list` / `inject` / `remove` | 管理 AI 工具中的 teamai hooks |
| `teamai doctor` | — | 诊断配置与环境问题 |
| `teamai codebase` | `extract` / `lint` / `fix` | 代码知识提取与维护 |
| `teamai contribute` | — | 将 session 经验贡献至团队 repo |

### 核心编程接口

| 接口/类 | 路径 | 关键方法 |
|---------|------|----------|
| `GitProvider` | `providers/types.ts` | `cloneRepo` / `createPullRequest` / `authenticate` / `parseRepoInput` |
| `ResourceHandler` | `resources/base.ts` | `scanLocalForPush` / `scanTeamForPull` / `pullItem` / `pushItem` / `removeItem` / `diff` |
| `search-index` | `utils/search-index.ts` | `buildIndex(docsDir)` / `loadIndex()` / `search(query, topK)` |
| `cache-index` | `utils/cache-index.ts` | `getCacheStatus()` / `gcCache(maxBytes)` |
| `autoRecallFromInput` | `auto-recall.ts` | `(input: HookInput) → Promise<string \| null>` |
| `compileFromManifest` | `wiki-engine/manifest-compiler.ts` | `(manifestPath) → CompiledManifest` |

## 核心流程

### Pull 流程（团队 repo → 本地 AI 工具）

```
1. requireInit() — 加载 .teamai/config.yaml + 团队 repo 配置
2. git pull 团队 repo（确保最新）
3. 遍历已启用的资源类型：
   a. getHandler(type) → ResourceHandler 实例
   b. scanTeamForPull() → 识别新增/更新的资源项
   c. 过滤墓碑（已删除条目）
   d. pullItem() → 写入 ~/.claude/、~/.cursor/ 等工具目录
4. injectClaudeMdSection() — 更新项目 CLAUDE.md 的 teamai 注入段
5. autoPushTeamRepo() — 如有 votes 变更，自动同步回团队 repo
```

### Push 流程（本地 → 团队 repo PR/MR）

```
1. autoDetectInit() — 定位最近的 .teamai/ 配置
2. scanLocalForPush() — 收集本地新增/修改的资源
3. 交互式确认（askSelection），支持 --all 跳过
4. pushItem() — 复制资源到团队 repo 本地路径
5. pushRepoBranch() — git checkout -b → commit → push
6. getProvider() → GitProvider 实例
7. createPullRequest() — 创建 PR（GitHub）或 MR（TGit）
8. 返回 PR/MR URL
```

### Recall 流程（知识检索）

```
1. requireInit() — 加载配置
2. loadIndex() — 加载 search-index.json（不存在则 buildIndex 全量构建）
3. search(query, topK) — BM25 关键词匹配
4. queryCodeKnowledge() — 若 wiki 引擎启用，追加代码图谱匹配结果
5. formatResults() — 格式化输出（带 [teamai:recall:start/end] 定界符）
6. autoUpvote() — 写 votes 记录到本地及团队 repo
```

### Import 流程（外部仓库 → 团队知识库）

```
1. 检查增量缓存（repo-cache）— 跳过未变更 repo
2. git clone / fetch 目标仓库
3. codebase-extract — 多语言 AST 提取代码图谱
4. enrich-with-ai — AI 生成模块摘要与关键描述
5. domains/recommend — AI 推荐知识域分类
6. graph-aggregate — 汇总多 repo 图谱为统一索引
7. autoPushTeamRepo() — 推送至团队 repo
```

### Auto-Recall 流程（Hook 自动触发）

```
1. readStdin() — 解析 PostToolUse hook 的 JSON payload
2. parseHookInput() — 规范化不同 AI 工具的 hook 格式
3. 判断触发条件：
   a. containsError(output) — 工具输出含错误 → extractQuery()
   b. Grep/WebSearch/WebFetch tool — 提取搜索词
4. shouldSkipQuery() — session 级去重 + 限速（≤10 次/session）
5. recall search — 执行知识检索
6. 以 additionalContext JSON 写 STDOUT → hook runner 注入上下文
```

<!-- teamai:referenced-doc-ids: [evidence/code/github__Tencent__teamai-cli/overview, evidence/code/github__Tencent__teamai-cli/modules/src, evidence/code/github__Tencent__teamai-cli/interface, github__Tencent__teamai-cli] -->