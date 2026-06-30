已完成源码分析，下面输出组件设计文档。

---

# src 模块 — 组件设计文档

## 🤖 AI 快速理解要点

| 维度 | 说明 |
|------|------|
| **核心职责** | 团队知识同步 + 检索 + 代码图谱 |
| **架构层级** | CLI 命令层 → 资源抽象层 → 平台适配层 |
| **上游组件** | 用户终端 / AI 工具 Hook 事件 |
| **下游组件** | GitHub API / TGit API / 本地文件系统 |
| **代码入口** | `src/index.ts`（Commander 注册） |
| **核心机制** | ResourceHandler 多态 + BM25 检索 |
| **数据流向** | 远端 repo → 本地资源 → 搜索索引 → AI |
| **技术栈** | TypeScript / Node 20 / Commander / tsup |

---

## 架构设计

```
┌──────────────────────────────────────────────────────────────────────┐
│                        CLI 命令入口 (index.ts)                        │
│  pull │ push │ recall │ import │ init │ status │ hook-dispatch │ ...  │
└───┬───────┬───────┬───────────┬──────────────────────┬───────────────┘
    │       │       │           │                      │
    ▼       ▼       │           ▼                      ▼
┌────────────────┐  │  ┌─────────────────────┐  ┌──────────────────────┐
│  resources/    │  │  │   import 子系统      │  │  hook-dispatch 子系统 │
│                │  │  │                     │  │                      │
│ ┌────────────┐ │  │  │ import-repo.ts      │  │ hook-dispatch-cli.ts │
│ │ base.ts    │ │  │  │ import-org.ts       │  │ hook-dispatch.ts     │
│ │(抽象基类)  │ │  │  │ import-mr.ts        │  │ hook-handlers.ts     │
│ └─────┬──────┘ │  │  │ import-iwiki.ts     │  │  ├─ auto-recall      │
│       │        │  │  └─────────┬───────────┘  │  ├─ mr-hint          │
│  skills.ts     │  │            │              │  ├─ contribute-check  │
│  rules.ts      │  │            ▼              │  └─ usage-tracker     │
│  docs.ts       │  │  ┌─────────────────────┐  └──────────────────────┘
│  env.ts        │  │  │  wiki-engine/       │
│  wiki.ts       │  │  │                     │
│  agents.ts     │  │  │ code-knowledge/     │
│  hooks.ts      │  │  │  extractors/        │
│ index.ts       │  │  │   ts│py│go│java│rust │
│ (getHandler)   │  │  │  code-graph.ts      │
└────────┬───────┘  │  │ core/               │
         │          │  │  graph-index.schema  │
         ▼          │  │  wiki-protocol.ts    │
┌────────────────┐  │  └─────────────────────┘
│  providers/    │  │
│                │  ▼
│ ┌────────────┐ │  ┌─────────────────────────┐
│ │ types.ts   │ │  │  recall + search 子系统  │
│ │(GitProvider)│ │  │                         │
│ └─────┬──────┘ │  │ recall.ts (命令入口)     │
│       │        │  │ auto-recall.ts (hook入口)│
│ github/        │  │ code-knowledge-recall.ts │
│  gh-cli.ts     │  │                         │
│  gh-org.ts     │  │       ┌─────────┐       │
│  mr-fetch.ts   │  │       │ BM25    │       │
│                │  │       │ Engine  │       │
│ tgit/          │  │       └─────────┘       │
│  gf-cli.ts     │  │ utils/search-index.ts   │
│  gf-org.ts     │  └─────────────────────────┘
│  mr-fetch.ts   │
│                │
│ registry.ts    │  ┌─────────────────────────┐
│ (工厂+探测)   │  │  utils/                 │
└────────────────┘  │  cache-index.ts (GC)    │
                    │  search-index.ts (BM25) │
                    │  git.ts / fs.ts         │
                    │  logger.ts              │
                    └─────────────────────────┘
```

### 核心子模块说明

| 子模块 | 职责 | 核心抽象 |
|--------|------|----------|
| **resources/** | 7 种资源类型的 CRUD + diff + 同步 | `ResourceHandler` 抽象基类，子类实现 `scanLocal/scanTeam/push/pull/remove` |
| **providers/** | Git 平台操作的统一适配（clone/PR/MR/auth） | `GitProvider` 接口 + `registry.ts` 工厂自动探测 |
| **recall + search** | 团队知识 BM25 检索 + 图谱加权 | `search-index.ts` 索引引擎，`code-knowledge-recall.ts` 图谱邻居加权 |
| **wiki-engine/** | 代码知识提取：AST → CodeFact → GraphIndex | 多语言 extractor + `buildCodeGraph()` |
| **hook-dispatch** | AI 工具事件分发（PostToolUse 等） | `HookHandler` 接口 + `createDispatcher()` |
| **utils/** | 文件/Git/缓存/日志通用工具 | `CacheIndex`（磁盘 GC）、`buildIndex/search`（BM25） |
| **domains/** | 仓库域（domain）知识管理 | schema + store + cluster + recommend |

---

## 接口设计

### CLI 对外命令接口

| 命令 | 入口文件 | 说明 |
|------|----------|------|
| `teamai init` | `init.ts` | 初始化团队仓库配置 |
| `teamai pull` | `pull.ts` | 从团队仓库拉取资源到本地 |
| `teamai push` | `push.ts` | 将本地资源推送到团队仓库 |
| `teamai recall <query>` | `recall.ts` | 检索团队知识库 |
| `teamai import --from-repo <url>` | `import-repo.ts` | 从代码仓库提取知识 |
| `teamai import --from-org <org>` | `import-org.ts` | 批量导入组织所有仓库 |
| `teamai import --from-mr <url>` | `import-mr.ts` | 从 MR 提取 learnings |
| `teamai import --cache-status/--cache-gc` | `import.ts` | 缓存状态查询 / 垃圾回收 |
| `teamai status` | `status.ts` | 查看本地与远端 diff |
| `teamai hook-dispatch <event>` | `hook-dispatch-cli.ts` | AI 工具 hook 事件分发 |
| `teamai doctor` | `doctor.ts` | 环境诊断 |

### 编程接口（模块 export）

| 接口 | 方法 | 路径 | 说明 |
|------|------|------|------|
| `getHandler` | `(type) → ResourceHandler` | `resources/index.ts` | 按类型获取资源处理器单例 |
| `getAllHandlers` | `() → ResourceHandler[]` | `resources/index.ts` | 获取全部 7 个 handler |
| `getProvider` | `(name?) → GitProvider` | `providers/registry.ts` | 按名称获取 Git 平台实例 |
| `getProviderFromUrl` | `(url) → GitProvider` | `providers/registry.ts` | 从 URL 自动探测平台 |
| `buildIndex` | `(opts) → Promise<void>` | `utils/search-index.ts` | 构建 BM25 全文索引 |
| `loadIndex` | `(path?) → Promise<SearchIndex>` | `utils/search-index.ts` | 加载已有索引 |
| `search` | `(query, index, limit?) → SearchResult[]` | `utils/search-index.ts` | BM25 检索 |
| `queryCodeKnowledge` | `(query, opts) → Promise<CodeKnowledgeResult[]>` | `code-knowledge-recall.ts` | 代码图谱检索 |
| `buildCodeGraph` | `(facts) → GraphIndex` | `wiki-engine/code-knowledge/code-graph.ts` | CodeFact → 图索引 |
| `createDispatcher` | `(config) → Dispatcher` | `hook-dispatch.ts` | 创建 hook 事件分发器 |
| `autoRecallFromInput` | `(input) → Promise<string \| null>` | `auto-recall.ts` | hook 触发的自动检索 |

---

## 核心流程

### 流程 1：知识同步（pull）

```
1. 用户执行 `teamai pull`
2. 读取 ~/.teamai/config.yaml 获取团队仓库 URL
3. providers/registry.ts 根据 URL 探测平台（GitHub/TGit）
4. 通过 GitProvider.cloneRepo() 克隆/更新团队仓库（带 cache-index 缓存）
5. 遍历 getAllHandlers()，对每个 handler 调用 scanTeamForPull()
6. 对比本地 tombstone（已删除标记），过滤已主动移除的资源
7. 调用 handler.pullItem() 将资源写入 ~/.claude/ / ~/.cursor/ 等目标路径
8. 调用 buildIndex() 重建 ~/.teamai/search-index.json
```

### 流程 2：知识检索（recall）

```
1. 用户执行 `teamai recall "如何部署"`
2. recall.ts 加载 user-scope + project-scope 两个 search-index.json
3. 调用 search(query, index) 执行 BM25 token 匹配
   - 标题权重 ×3，tag 权重 ×2，投票加成
4. 并行调用 queryCodeKnowledge(query) 检索代码图谱
   - BM25 匹配 wiki 页面 + graph-index 邻居节点加权
5. 合并两路结果，按 score 降序排列
6. formatResults() 输出结构化 Markdown 到 STDOUT（供 AI 读取）
```

### 流程 3：代码知识导入（import --from-repo）

```
1. 用户执行 `teamai import --from-repo <url>`
2. providers/registry.ts 探测平台 + clone（使用 cache-index 避免重复克隆）
3. codebase-extract.ts 扫描源码目录
4. wiki-engine/code-knowledge/extractors/<lang>.ts 按语言提取 CodeFact[]
   - 每个 fact 包含：name, type, file, line, dependencies, exports
5. buildCodeGraph(facts) 构建 GraphIndex（节点 + DEPENDS_ON 边）
6. writeCodeGraph() 写入 teamwiki/graph-index.json
7. enrich-with-ai.ts（可选）调用 LLM 生成组件 wiki 页面
8. 写入 teamwiki/pages/*.md
9. 重建 search-index 纳入新知识
```

### 流程 4：Hook 事件分发（PostToolUse）

```
1. AI 工具（Claude Code）调用 `teamai hook-dispatch PostToolUse --tool claude --matcher Bash`
2. hook-dispatch-cli.ts 读取 STDIN（JSON 格式的工具输出）
3. createDispatcher() 加载 hook-handlers.ts 中注册的 handler 列表
4. 根据 event + matcher 路由到匹配的 handler：
   - Bash 输出含 error → auto-recall handler → 检索相关知识 → 返回提示
   - MR 相关操作 → mr-hint handler → 返回提交规范提醒
   - 任何工具调用 → usage-tracker → 记录到 events.jsonl
5. handler 返回值（string | null）输出到 STDOUT，AI 工具读取并注入上下文
```

### 流程 5：缓存生命周期管理

```
1. import/clone 时 → cache-index.ts 记录 {key, size, last_used} 到 .cache-index.json
2. 用户执行 `teamai import --cache-status` → 展示当前缓存占用
3. 用户执行 `teamai import --cache-gc` → 按规则驱逐：
   - staleDays: 超过 N 天未使用的条目
   - maxBytes: 总大小超限时 LRU 驱逐
4. 环境变量 TEAMAI_CACHE_DIR / TEAMAI_CACHE_MAX_BYTES 可覆盖默认值
```

<!-- teamai:referenced-doc-ids: [github__m0Nst3r873__teamai-cli] -->