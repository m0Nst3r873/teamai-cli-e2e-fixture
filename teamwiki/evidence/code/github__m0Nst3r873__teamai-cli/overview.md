---
title: github__m0Nst3r873__teamai-cli overview
domain: code-knowledge
source: [https://github.com/m0Nst3r873/teamai-cli.git]
---

# Codebase 概览

## 项目概述

TeamAI CLI 是一个 Git 原生的团队 AI 经验共享框架，帮助团队统一管理和同步 Skills、Rules、Docs、Env 等资源到 20+ 主流 AI 编程工具（Claude Code、Cursor、Codex、CodeBuddy 等）。通过一个中心化的 Git 仓库作为 "team repo"，实现团队知识的版本化管理和自动分发。

核心能力：
- 🔄 **资源同步**：通过 `push`/`pull` 在本地和 team repo 之间双向同步 skills、rules、docs、env、agents、hooks、wiki 七种资源
- 📥 **知识检索（Recall）**：BM25 全文搜索引擎 + 代码知识图谱，让 AI agent 在编码前检索团队经验
- 🔌 **多 Provider 抽象**：统一接口同时支持 GitHub 和 TGit（腾讯工蜂），按仓库 URL 自动选择
- 🪝 **Hook 系统**：团队声明式 hooks 自动注入 AI 工具的 PreToolUse/PostToolUse/Stop 事件
- 📊 **用量追踪**：session 级别的 AI 工具使用数据收集与聚合
- 🏷️ **角色体系（Roles）**：按角色（如 hai_dev）分发不同的资源集，支持多角色叠加
- 🌐 **跨团队订阅（Source）**：团队间可订阅公开的 skills，实现跨组织技能复用

## 技术栈

| 维度 | 技术 |
|------|------|
| 语言 | **TypeScript** 5.7+ |
| 运行时 | **Node.js** 20+ |
| 构建 | **tsup**（ESM 输出） |
| 测试 | **Vitest** 2.1+（含 `@vitest/coverage-v8`） |
| CLI 框架 | **Commander** 12.1+ |
| Schema 校验 | **Zod** 3.24+ |
| Git 操作 | **simple-git** 3.27+ |
| 配置解析 | **YAML** 2.6+、**gray-matter** 4.0+、**smol-toml** 1.3+ |
| 文件系统 | **fs-extra** 11.2+ |
| UI 交互 | **chalk** 5.3+、**ora** 8.1+ |
| CI | GitHub Actions + Coding CI（.coding-ci.yaml） |
| 包发布 | npm（public `teamai-cli`）+ tnpm（internal `@tencent/teamai-cli`） |

## 目录结构与模块职责

```
teamai-cli/
├── src/
│   ├── index.ts                        # CLI 入口，注册所有命令（commander）
│   ├── types.ts                        # 全局类型定义 + Zod schema
│   ├── config.ts                       # 配置加载（teamai.yaml + config.yaml + state）
│   │
│   ├── ┌─ 资源同步系统 ──────────────────────────────────────────┐
│   ├── │ resources/
│   ├── │   ├── base.ts                 # ResourceHandler 抽象基类            │
│   ├── │   ├── index.ts                # Handler 注册表（getHandler/getAll） │
│   ├── │   ├── skills.ts               # Skills 同步处理                     │
│   ├── │   ├── rules.ts                # Rules 同步处理                      │
│   ├── │   ├── docs.ts                 # Docs 同步处理                       │
│   ├── │   ├── env.ts                  # Env 变量同步处理                    │
│   ├── │   ├── agents.ts               # Agents 同步处理                     │
│   ├── │   ├── hooks.ts                # Hooks 同步处理                      │
│   ├── │   ├── wiki.ts                 # Wiki 同步处理                       │
│   ├── │   ├── agent-format.ts         # Agent 文件格式规范化                │
│   ├── │   └── marketplace.ts          # Marketplace 资源发现                │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ Git Provider 抽象 ─────────────────────────────────────┐
│   ├── │ providers/
│   ├── │   ├── types.ts                # GitProvider 接口定义                │
│   ├── │   ├── registry.ts             # Provider 注册与选择                 │
│   ├── │   ├── index.ts                # Provider 导出                       │
│   ├── │   ├── github.ts               # GitHub Provider 实现               │
│   ├── │   └── tgit.ts                 # TGit（工蜂）Provider 实现          │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 知识检索与搜索 ─────────────────────────────────────────┐
│   ├── │ recall.ts                     # recall 命令主逻辑                   │
│   ├── │ utils/search-index.ts         # BM25 搜索引擎（索引构建/查询）     │
│   ├── │ code-knowledge-recall.ts      # 代码知识图谱检索                   │
│   ├── │ codebase-extract.ts           # 代码库元信息提取                   │
│   ├── │ rebuild-wiki-index.ts         # Wiki 索引重建                      │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ Hook 分发系统 ──────────────────────────────────────────┐
│   ├── │ hook-dispatch-cli.ts          # Hook CLI 入口                      │
│   ├── │ hook-handlers.ts              # buildHandlerRegistry 构建          │
│   ├── │ hook-dispatch.ts              # createDispatcher 事件分发          │
│   ├── │ builtin-hooks.ts              # 内建 hook 定义                     │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 领域知识（Domains）──────────────────────────────────────┐
│   ├── │ domains/
│   ├── │   ├── schema.ts               # 领域 schema 定义                   │
│   ├── │   ├── store.ts                # 领域数据持久化                     │
│   ├── │   ├── cluster.ts              # 领域聚类                           │
│   ├── │   ├── recommend.ts            # 领域推荐                           │
│   ├── │   ├── review.ts               # 领域审核                           │
│   ├── │   └── index.ts                # 领域模块导出                       │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 命令实现 ───────────────────────────────────────────────┐
│   ├── │ init.ts                       # teamai init 命令                   │
│   ├── │ push.ts                       # teamai push 命令                   │
│   ├── │ pull.ts                       # teamai pull（含 pull-skip 逻辑）   │
│   ├── │ status.ts                     # teamai status 命令                 │
│   ├── │ import-local.ts               # teamai import 命令（本地/批量导入）│
│   ├── │ env-commands.ts               # teamai env 子命令集                │
│   ├── │ skill-cmd.ts                  # teamai skill 子命令                │
│   ├── │ roles-cmd.ts                  # teamai roles 子命令                │
│   ├── │ tags.ts                       # teamai tags 命令                   │
│   ├── │ aggregate.ts                  # teamai dashboard/aggregate         │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 工具函数（Utils）────────────────────────────────────────┐
│   ├── │ utils/
│   ├── │   ├── git.ts                  # Git 操作封装                       │
│   ├── │   ├── fs.ts                   # 文件系统工具                       │
│   ├── │   ├── logger.ts               # 日志系统                           │
│   ├── │   ├── ai-client.ts            # AI 模型调用客户端                  │
│   ├── │   ├── claudemd.ts             # CLAUDE.md 文件操作                 │
│   ├── │   ├── cache-index.ts          # 缓存索引管理                       │
│   ├── │   ├── repo-cache.ts           # 仓库缓存                           │
│   ├── │   ├── repo-url.ts             # 仓库 URL 解析                      │
│   ├── │   ├── dedup.ts                # 去重工具                           │
│   ├── │   ├── path-safety.ts          # 路径安全校验                       │
│   ├── │   ├── tool-names.ts           # AI 工具名标准化                    │
│   ├── │   ├── source-conflict.ts      # 跨源冲突检测                       │
│   ├── │   ├── pre-push-sync.ts        # Push 前同步检查                    │
│   ├── │   ├── hook-output.ts          # Hook 输出格式化                    │
│   ├── │   ├── session-id.ts           # Session ID 生成                    │
│   ├── │   ├── prompt.ts               # 交互式提示                         │
│   ├── │   ├── tags.ts                 # Tag 工具                           │
│   ├── │   ├── gf-cli.ts              # gf CLI 管理（TGit 专用）           │
│   ├── │   ├── iwiki-client.ts         # iWiki API 客户端                   │
│   ├── │   └── team-codebase-paths.ts  # 代码库路径解析                     │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 其他模块 ──────────────────────────────────────────────┐
│   ├── │ builtin-agents.ts             # 内建 agent 定义                    │
│   ├── │ builtin-skills.ts             # 内建 skill 定义                    │
│   ├── │ session-collector.ts          # Session 数据收集                   │
│   ├── │ usage-tracker.ts              # 用量统计                           │
│   ├── │ iwiki-dual.ts                 # iWiki 双写（local + remote）       │
│   ├── │ repo-list/                    # 仓库列表管理（schema + store）     │
│   ├── └─────────────────────────────────────────────────────────────────────┘
│   │
│   └── __tests__/                      # 122 个测试文件
│       ├── *.test.ts                   # 单元/集成测试
│       ├── e2e/e2e.test.ts             # 端到端测试
│       └── fixtures/                   # 测试夹具
│
├── skills/                             # 随包发布的内建 skills
├── agents/                             # 随包发布的内建 agents
├── docs/                               # 使用文档
├── tsup.config.ts                      # 构建配置
├── vitest.config.ts                    # 测试配置
├── .coding-ci.yaml                     # Coding CI 配置（tnpm 发布）
└── .github/workflows/                  # GitHub Actions CI
```

## 数据与配置

```
~/.teamai/                              # 用户级数据目录（TEAMAI_CACHE_DIR 可覆盖）
├── config.yaml                         # 本地配置（repo URL, scope, primaryRole 等）
├── team-repo/                          # team repo 的 git clone
│   ├── teamai.yaml                     # 团队配置（sharing、provider、roles 等）
│   ├── skills/                         # 团队共享 skills
│   ├── rules/                          # 团队共享 rules
│   ├── docs/                           # 团队共享文档
│   ├── agents/                         # 团队共享 agents
│   ├── hooks/                          # 团队声明式 hooks
│   └── votes/                          # 成员 recall 投票数据
├── state.json                          # 同步状态（lastPull 时间、commit hash）
├── search-index.json                   # BM25 搜索索引
├── docs/                               # 拉取到本地的 docs 副本
├── learnings/                          # 团队经验沉淀
└── votes/                              # 本地投票缓存

<project>/.teamai/                      # 项目级配置（scope=project 时）
├── config.yaml                         # 项目级本地配置
└── team-repo/                          # 项目级 team repo clone
```

## 核心数据流

### 1. 资源推送（teamai push）
```
用户执行 teamai push
    │
    ├─ 1. loadLocalConfig() → 读取 ~/.teamai/config.yaml
    ├─ 2. loadTeamConfig() → 读取 team-repo/teamai.yaml
    ├─ 3. getProvider() → 按配置选择 GitHub/TGit provider
    ├─ 4. getAllHandlers().scanLocalForPush() → 扫描各类本地资源
    │   └─ 对比 team repo，生成 ResourceDiff
    ├─ 5. 用户确认推送内容
    ├─ 6. handler.pushItem() → 复制资源到 team-repo 目录
    ├─ 7. git commit + git push → 推送到远端分支
    └─ 8. provider.createPullRequest() → 创建 PR/MR
         └─ ✅ 返回 PR URL
```

### 2. 资源拉取（teamai pull）
```
用户执行 teamai pull（或 hook 自动触发）
    │
    ├─ 1. loadLocalConfig() → 读取配置 + state
    ├─ 2. git pull → 更新 team-repo 本地副本
    │   └─ 检查 lastPull commit，无变更则 skip
    ├─ 3. getAllHandlers().scanTeamForPull() → 扫描 team 资源
    ├─ 4. handler.pullItem() → 逐资源注入本地 AI 工具目录
    │   ├─ Skills → ~/.claude/skills/, .cursor/skills/, ...
    │   ├─ Rules → ~/.claude/rules/, .cursorrules, ...
    │   ├─ Agents → ~/.claude/agents/
    │   └─ Hooks → ~/.claude/settings.json hooks 字段
    ├─ 5. buildIndex() → 重建搜索索引
    └─ ✅ 更新 state.json（lastPull 时间 + commit）
```

### 3. 知识检索（teamai recall）
```
AI Agent 调用 teamai recall "<query>"
    │
    ├─ 1. loadIndex() → 加载 search-index.json
    │   └─ 不存在或过期 → buildIndex() 全量重建
    ├─ 2. search(query, index) → BM25 全文检索
    │   ├─ 搜索范围：skills + learnings + docs + rules
    │   └─ 支持 user-scope + project-scope 合并搜索
    ├─ 3. queryCodeKnowledge() → 代码知识图谱检索
    │   └─ 搜索 teamwiki/ 下的结构化代码知识
    ├─ 4. formatResults() → 格式化为 AI 可消费的输出
    │   └─ 包含 doc_id、scope、type 标签
    └─ 5. autoUpvote() → 自动记录引用投票
         └─ ✅ 输出到 STDOUT（AI agent 读取）
```

### 4. Hook 事件分发
```
AI 工具触发 PreToolUse/PostToolUse 事件
    │
    ├─ 1. hook-dispatch-cli 接收事件 JSON
    ├─ 2. buildHandlerRegistry() → 构建 handler 映射表
    │   ├─ 内建 handlers: auto-recall, mr-hint, contribute-check, todowrite-hint
    │   └─ 团队自定义 handlers: 从 team-repo/hooks/ 加载
    ├─ 3. createDispatcher(registry) → 生成事件分发器
    ├─ 4. dispatcher.dispatch(event) → 匹配 + 执行 handler
    │   └─ 按 tool_name / event_type 路由到对应 handler
    └─ ✅ 输出 hook response（allow/deny/message）
```

## 关键接口与抽象

### ResourceHandler（资源处理基类）

```typescript
abstract class ResourceHandler {
  abstract readonly type: ResourceType;
  abstract scanLocalForPush(teamConfig: TeamaiConfig, localConfig: LocalConfig): Promise<ResourceItem[]>;
  abstract scanTeamForPull(teamConfig: TeamaiConfig, localConfig: LocalConfig): Promise<ResourceItem[]>;
  abstract pushItem(item: ResourceItem, teamConfig: TeamaiConfig, localConfig: LocalConfig): Promise<void>;
  abstract pullItem(item: ResourceItem, teamConfig: TeamaiConfig, localConfig: LocalConfig): Promise<void>;
  abstract removeItem(name: string, teamConfig: TeamaiConfig, localConfig: LocalConfig): Promise<string[]>;
}
```

实现子类：`SkillsHandler`、`RulesHandler`、`DocsHandler`、`EnvHandler`、`WikiHandler`、`AgentsHandler`、`HooksHandler`

### GitProvider（Git 托管平台抽象）

```typescript
interface GitProvider {
  readonly name: string;  // 'github' | 'tgit'
  parseRepoInput(input: string): RepoInfo;
  isAuthenticated(): boolean;
  authenticate(): Promise<string>;
  ensureInstalled(): Promise<void>;
  cloneRepo(repo: string, localPath: string): void;
  createRepo(owner: string, repo: string): Promise<void>;
  createPullRequest(opts: PrCreateOptions): Promise<string>;
  getMrDetails?(url: string): Promise<MrDetails>;
  listOrgRepos?(org: string, opts?: ListReposOptions): Promise<OrgRepoInfo[]>;
}
```

实现类：`GitHubProvider`（gh CLI + REST API）、`TGitProvider`（gf CLI + netrc）

### 搜索索引接口

```typescript
// BM25 全文搜索
function buildIndex(sources: IndexSource[]): SearchIndex;
function search(query: string, index: SearchIndex, options?: SearchOptions): SearchResult[];
function loadIndex(indexPath: string): SearchIndex | null;
```

## 配置系统

### 配置优先级

```
环境变量（最高）
  ↓ TEAMAI_CACHE_DIR, TEAMAI_DEFAULT_PROVIDER, TEAMAI_SEARCH_STRATEGY
项目级配置（<project>/.teamai/config.yaml）
  ↓ scope: project 时生效
用户级配置（~/.teamai/config.yaml）
  ↓ 默认 scope: user
团队配置（team-repo/teamai.yaml）
  ↓ 由团队管理员维护，pull 时自动同步
```

### Scope 检测逻辑

当项目目录下存在 `.teamai/config.yaml` 时识别为 project scope，否则 fallback 到 user scope（`~/.teamai/`）。`detectProjectConfig()` 从 CWD 向上逐级查找 `.teamai/` 目录。

### 关键配置结构示例

```yaml
# teamai.yaml（团队配置）
provider: github          # 'github' | 'tgit'
scope: user               # 'user' | 'project'
sharing:
  skills: {}
  rules:
    enforced: ['security-rules', 'code-style']
  docs:
    localDir: '~/.teamai/docs'
  env:
    injectShellProfile: true
  hooks:
    autoApply: true
    requireTeamScripts: false
roles:
  - id: hai_dev
    skills: ['hai-*']
    rules: ['hai-rules']
```

```yaml
# ~/.teamai/config.yaml（本地配置）
repo:
  url: 'https://github.com/team/teamai-repo.git'
  localPath: '~/.teamai/team-repo'
primaryRole: 'hai_dev'
additionalRoles: []
scope: user
```

## 性能与可靠性

| 设计 | 机制 | 说明 |
|------|------|------|
| 增量同步 | state.json commit 对比 | pull 时对比 lastPull commit，无变更直接 skip |
| 搜索索引缓存 | search-index.json | 预构建 BM25 索引，recall 时直接加载 |
| 仓库缓存 | utils/repo-cache.ts | 避免重复 clone，按 URL hash 缓存 |
| 去重机制 | utils/dedup.ts | 跨源资源去重，避免重复注入 |
| 冲突检测 | utils/source-conflict.ts | 多源订阅时检测同名资源冲突 |
| 路径安全 | utils/path-safety.ts | 防止路径遍历攻击 |
| 批量导入并发控制 | import-local.ts | graph-index.json 锁机制防止并发写入竞争 |
| Hook 超时 | hook-dispatch.ts | Hook handler 执行有超时保护，避免阻塞 AI 工具 |
| 静默降级 | pull --silent | Hook 触发的静默 pull 不打断用户工作流 |

## 架构决策与权衡

- **为什么选择 Git 作为同步载体而不是中心化服务**：Git 天然支持版本控制、分支审查（PR/MR）、离线工作，且不需要额外的服务端基础设施；权衡是需要处理 merge 冲突和多分支同步。

- **为什么用 ResourceHandler 抽象基类而非函数式插件**：七种资源类型的生命周期一致（scan → push/pull → remove），基类强制接口一致性并共享 tombstone 逻辑；权衡是新增资源类型需要实现完整接口。

- **为什么 BM25 而非向量检索**：BM25 零依赖、离线可用、可解释性好、对关键词匹配效果稳定；权衡是语义相关度不如 embedding，通过代码知识图谱（Graph RAG）补充语义能力。

- **为什么 Provider 抽象而非只支持 GitHub**：腾讯内部使用 TGit（工蜂），外部用户用 GitHub，必须双轨；权衡是维护两套 API 调用逻辑和认证流程。

- **为什么 ESM-only 而不是 CJS 兼容**：Node.js 20+ 已全面支持 ESM，chalk 5、ora 8 等依赖均为 ESM-only，无历史包袱；权衡是不支持 Node.js 18 以下版本。

## 已知限制与演进方向

- **搜索索引全量重建**：当前 `buildIndex()` 为全量扫描，大型团队（1000+ 文件）时可能耗时较长；后续可改为增量索引更新。

- **单 team repo 限制**：每个 scope 只支持绑定一个 team repo，多团队场景需通过 source 订阅间接实现；后续可支持多 repo 直接绑定。

- **Hook handler 无沙箱**：团队 hooks 在用户本地直接执行，依赖 `requireTeamScripts` 配置做安全约束；后续可引入沙箱执行环境。

- **Provider 可选方法不一致**：`getMrDetails()`、`listOrgRepos()` 等方法为可选实现（`?`），GitHub/TGit 功能覆盖不对等；后续可补齐 TGit 缺失 API。

- **离线场景体验不完整**：git pull 失败时静默降级，但不会提示用户本地缓存可能过期；后续可增加 stale 提示机制。

## 测试覆盖

| 层级 | 文件数 | 说明 |
|------|--------|------|
| 单元测试 | ~110 | 覆盖各 handler、utils、provider、schema |
| 集成测试 | ~10 | 跨模块流程（import-repo、hooks-e2e、contribute-check-e2e 等） |
| E2E 测试 | 2 | `e2e.test.ts` + `test/e2e.mjs`，需真实 Git 仓库 + token |
| 测试文件总数 | 122 | `src/__tests__/` 目录 |
| 源代码文件 | 153 | `src/` 目录（不含测试） |

E2E 测试通过 GitHub Actions CI 运行，需配置 `TEAMAI_TEST_REPO_URL` + `TEAMAI_TEST_TOKEN`，使用 `concurrency: e2e-fixture-repo` 串行避免状态污染。

## 备注

- ✅ 目录结构、接口签名、配置格式均来自源码实际内容
- ✅ 数据流基于 recall.ts 文件内注释及 providers/types.ts 接口注释
- ⚠️ 测试覆盖率百分比未实际运行 `--coverage`，仅提供文件数量统计
- ⚠️ 性能数据（如索引重建耗时）为基于代码结构的推断，未做实际 benchmark

<!-- teamai:referenced-doc-ids: [evidence/code/github__m0Nst3r873__teamai-cli/overview, evidence/code/github__m0Nst3r873__teamai-cli/modules/src, evidence/code/github__m0Nst3r873__teamai-cli/dependency-paths, evidence/code/github__m0Nst3r873__teamai-cli/config] -->