---
title: github__Tencent__teamai-cli overview
domain: code-knowledge
source: [https://github.com/Tencent/teamai-cli.git]
---

# Codebase 概览

## 项目概述

TeamAI CLI 是一个 Git 原生的团队 AI 经验共享框架，用于在 20+ AI 编程工具（Claude Code、Cursor、Codex、CodeBuddy 等）之间统一管理和同步 Skills、Rules、Docs、Env 等团队资源。通过 Git 仓库作为 single source of truth，实现团队知识的沉淀、检索与分发。

核心能力：
- 🔄 **多工具同步**：一次 push/pull，自动分发到 Claude Code、Cursor、Codex、CodeBuddy、Gemini CLI 等 20+ AI 工具
- 📥 **知识导入**：从 Git 仓库批量导入代码库文档，构建知识图谱索引
- 🔍 **智能召回**：基于搜索索引的团队知识检索（skills + learnings + docs + rules + codebase graph）
- 🏷️ **角色与域**：按角色（roles）分发资源，按域（domains）组织知识聚类
- 🪝 **Hooks 管理**：团队声明式 hooks + 内置 hooks 统一管理，自动注入 AI 工具配置
- 📊 **用量追踪**：Session 级别的 AI 工具使用数据收集与仪表盘

## 技术栈

| 维度 | 技术 |
|------|------|
| 语言 | **TypeScript** 5.7+ |
| 运行时 | **Node.js** 20+ |
| 模块系统 | ESM（`"type": "module"`） |
| 构建 | **tsup** 8.3 |
| 测试 | **Vitest** 2.1 + @vitest/coverage-v8 |
| CLI 框架 | **Commander** 12.1 |
| Schema 验证 | **Zod** 3.24 |
| Git 操作 | **simple-git** 3.27 |
| 配置解析 | **gray-matter** 4.0 / **yaml** 2.6 / **smol-toml** 1.3 |
| 终端 UI | **chalk** 5.3 / **ora** 8.1 |
| 文件操作 | **fs-extra** 11.2 |
| CI | GitHub Actions + Coding CI（`.coding-ci.yaml`） |
| 发布 | npm（public）+ tnpm（`@tencent/teamai-cli`） |

## 目录结构与模块职责

```
teamai-cli/
├── src/
│   ├── index.ts                        # CLI 入口，注册所有命令（commander）
│   ├── types.ts                        # 全局类型定义与 Zod Schema
│   │
│   ├── ┌─ 核心命令 ──────────────────────────────────────────┐
│   ├── │ init.ts                       # teamai init：初始化配置、克隆团队仓         │
│   ├── │ push.ts                       # teamai push：推送本地资源到团队仓（MR 流程） │
│   ├── │ pull.ts                       # teamai pull：拉取团队仓资源并注入本地工具    │
│   ├── │ status.ts                     # teamai status：对比本地与远端差异            │
│   ├── │ recall.ts                     # teamai recall：知识库检索                   │
│   ├── │ import-local.ts               # teamai import：批量导入代码/文档             │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 辅助命令 ──────────────────────────────────────────┐
│   ├── │ env-commands.ts               # teamai env：环境变量管理                    │
│   ├── │ skill-cmd.ts                  # teamai skill：技能管理                      │
│   ├── │ tags.ts                       # teamai tags：标签管理                       │
│   ├── │ roles-cmd.ts                  # teamai roles：角色管理                      │
│   ├── │ aggregate.ts                  # teamai contribute：经验沉淀聚合             │
│   ├── │ session-collector.ts          # teamai dashboard：使用数据收集              │
│   ├── │ rebuild-wiki-index.ts         # teamai wiki：Wiki 索引重建                  │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 资源处理器（resources/）────────────────────────────┐
│   ├── │ resources/
│   ├── │   ├── base.ts                 # 资源处理基类                               │
│   ├── │   ├── index.ts                # 资源注册表                                 │
│   ├── │   ├── skills.ts               # Skills 资源处理                            │
│   ├── │   ├── rules.ts                # Rules 资源处理                             │
│   ├── │   ├── docs.ts                 # Docs 资源处理                              │
│   ├── │   ├── hooks.ts                # Hooks 资源处理                             │
│   ├── │   ├── agents.ts               # Agents 资源处理                            │
│   ├── │   ├── wiki.ts                 # Wiki 资源处理                              │
│   ├── │   ├── env.ts                  # Env 资源处理                               │
│   ├── │   ├── marketplace.ts          # Marketplace 资源处理                       │
│   ├── │   └── agent-format.ts         # Agent 格式转换                             │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 域管理（domains/）──────────────────────────────────┐
│   ├── │ domains/
│   ├── │   ├── index.ts                # 域入口                                     │
│   ├── │   ├── schema.ts               # 域 Schema 定义                             │
│   ├── │   ├── store.ts                # 域持久化存储                               │
│   ├── │   ├── cluster.ts              # 知识聚类算法                               │
│   ├── │   ├── recommend.ts            # 知识推荐                                   │
│   ├── │   └── review.ts               # 域审查                                     │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 工具库（utils/）────────────────────────────────────┐
│   ├── │ utils/
│   ├── │   ├── git.ts                  # Git 操作封装                               │
│   ├── │   ├── fs.ts                   # 文件系统辅助                               │
│   ├── │   ├── logger.ts               # 日志（verbose/silent 模式）                │
│   ├── │   ├── ai-client.ts            # AI 模型调用客户端                          │
│   ├── │   ├── search-index.ts         # 全文搜索索引                               │
│   ├── │   ├── cache-index.ts          # 缓存索引管理                               │
│   ├── │   ├── claudemd.ts             # CLAUDE.md 文件操作                         │
│   ├── │   ├── repo-url.ts             # 仓库 URL 解析                              │
│   ├── │   ├── repo-cache.ts           # 仓库级缓存                                 │
│   ├── │   ├── tool-names.ts           # IDE 工具名规范化                            │
│   ├── │   ├── iwiki-client.ts         # iWiki API 客户端                           │
│   ├── │   ├── pre-push-sync.ts        # Push 前同步逻辑                            │
│   ├── │   ├── hook-output.ts          # Hook 输出格式化                            │
│   ├── │   ├── path-safety.ts          # 路径安全校验                               │
│   ├── │   ├── source-conflict.ts      # 源冲突检测                                 │
│   ├── │   ├── dedup.ts                # 去重工具                                   │
│   ├── │   ├── prompt.ts               # 交互提示                                   │
│   ├── │   ├── session-id.ts           # Session ID 生成                            │
│   ├── │   ├── gf-cli.ts              # 工蜂 CLI 封装                              │
│   ├── │   ├── tags.ts                 # 标签工具                                   │
│   ├── │   └── team-codebase-paths.ts  # 代码库路径解析                             │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   ├── ┌─ 独立功能模块 ──────────────────────────────────────┐
│   ├── │ config.ts                     # 配置加载与解析                             │
│   ├── │ builtin-skills.ts             # 内置 Skills 注册                           │
│   ├── │ builtin-agents.ts             # 内置 Agents 注册                           │
│   ├── │ iwiki-dual.ts                 # iWiki 双向同步                             │
│   ├── │ usage-tracker.ts              # 使用量追踪                                 │
│   ├── │ codebase-extract.ts           # 代码库知识提取                             │
│   ├── │ repo-list/                    # 多仓库列表管理                             │
│   ├── │   ├── store.ts                # 仓库列表存储                               │
│   ├── │   └── schema.ts              # 仓库列表 Schema                            │
│   ├── └─────────────────────────────────────────────────────────────┘
│   │
│   └── __tests__/                      # 单元测试 + E2E 测试（70+ 测试文件）
│
├── skills/                             # 随包分发的内置 Skills
├── agents/                             # 随包分发的内置 Agents
├── docs/                               # 项目文档（usage-guide, providers, ci-e2e-setup）
├── test/e2e.mjs                        # 独立 E2E 测试脚本
├── tsup.config.ts                      # 构建配置
├── vitest.config.ts                    # 测试配置
├── .github/workflows/                  # GitHub Actions CI/CD
└── .coding-ci.yaml                     # Coding CI（tnpm 发布）
```

## 数据与配置

```
~/.teamai/                              # 用户级数据根目录
├── config.yaml                         # 本地用户配置（scope、repo URL、provider）
├── team-repo/                          # 团队仓库本地克隆
│   ├── teamai.yaml                     # 团队共享配置（sharing rules、hooks policy）
│   ├── skills/<role>/                  # 按角色组织的 Skills
│   ├── rules/                          # 团队 Rules
│   ├── docs/                           # 团队 Docs
│   ├── learnings/                      # 团队 Learnings（经验沉淀）
│   └── agents/                         # 团队 Agents
├── docs/                               # 本地 docs 副本
├── learnings/                          # 本地 learnings 副本
└── graph-index.json                    # 知识图谱索引

<project>/.teamai/                      # 项目级配置（scope=project 时）
├── config.yaml                         # 项目级配置
└── team-repo/                          # 项目级团队仓库克隆

<tool-config>/                          # 各 AI 工具配置目录
├── ~/.claude/skills/                   # Claude Code Skills 注入点
├── ~/.claude/rules/                    # Claude Code Rules 注入点
├── ~/.cursor/rules/                    # Cursor Rules 注入点
└── ...                                 # 其他 20+ 工具
```

## 核心数据流

### 1. Pull（团队资源同步到本地）
```
用户执行 teamai pull（或 hook 自动触发）
    │
    ├─ 1. git fetch + diff：检测团队仓是否有更新
    │   └─ 无变化且非 --force → 跳过（快速路径）
    ├─ 2. 加载 teamai.yaml：确定 sharing 策略、hooks policy
    ├─ 3. 逐类型分发资源：
    │   ├─ Skills → 按 role 过滤 → 写入 tool paths
    │   ├─ Rules → enforced 规则强制注入 → 写入 tool paths
    │   ├─ Docs  → 同步到 ~/.teamai/docs/
    │   ├─ Hooks → autoApply? → 注入 AI 工具 settings.json
    │   ├─ Agents → 写入 tool agents 目录
    │   └─ Env   → 注入 shell profile
    ├─ 4. 更新搜索索引 + 缓存标记
    └─ ✅ 同步完成，本地 AI 工具可立即使用新资源
```

### 2. Push（本地资源提交到团队仓）
```
用户执行 teamai push [--skill <path>]
    │
    ├─ 1. 扫描本地变更：对比 local vs team-repo
    │   └─ --all 跳过确认 / 交互选择要推送的资源
    ├─ 2. 创建特性分支：feature/push-<timestamp>
    ├─ 3. 提交变更到分支
    ├─ 4. 创建 MR/PR：
    │   ├─ GitHub → gh pr create
    │   └─ TGit   → gf API 创建 MR
    └─ ✅ MR 已创建，等待 review 合入
```

### 3. Recall（知识检索）
```
AI 工具执行 teamai recall "<关键词>"
    │
    ├─ 1. 加载搜索索引（search-index + graph-index）
    ├─ 2. 多维度检索：
    │   ├─ Skills 匹配
    │   ├─ Learnings 匹配
    │   ├─ Docs 匹配
    │   ├─ Rules 匹配
    │   └─ Codebase Graph 匹配
    ├─ 3. 评分排序 → 返回 Top-K 摘要 + doc_ids
    └─ ✅ 返回结构化检索结果
```

### 4. Import（代码库知识导入）
```
用户执行 teamai import <repo-url>
    │
    ├─ 1. 克隆/更新目标仓库
    ├─ 2. AST 解析：提取模块、函数、类、接口
    ├─ 3. 构建知识图谱：
    │   ├─ 节点：文件、模块、符号
    │   └─ 边：import、call、extend 关系
    ├─ 4. 生成文档：组件设计文档 + 架构总览
    ├─ 5. 写入 graph-index.json + docs/team-codebase/
    └─ ✅ 知识库更新完成，recall 可立即检索
```

## 关键接口与抽象

```typescript
// 资源处理器基类（src/resources/base.ts）
interface ResourceHandler {
  type: string;                          // 资源类型标识
  pull(config: TeamConfig): Promise<void>;   // 从团队仓拉取并注入
  push(config: TeamConfig): Promise<void>;   // 从本地推送到团队仓
  status(config: TeamConfig): Promise<DiffResult>; // 对比差异
}
// 实现：SkillsHandler, RulesHandler, DocsHandler, HooksHandler,
//       AgentsHandler, WikiHandler, EnvHandler, MarketplaceHandler
```

```typescript
// 工具路径配置（src/types.ts）
const ToolPathsSchema = z.object({
  skills: z.string().optional(),     // AI 工具的 skills 目录
  rules: z.string().optional(),      // AI 工具的 rules 目录
  settings: z.string().optional(),   // AI 工具的 settings 文件
  claudemd: z.string().optional(),   // CLAUDE.md 路径
  wiki: z.string().optional(),       // Wiki 目录
  agents: z.string().optional(),     // Agents 目录
});
```

```typescript
// 团队共享配置（src/types.ts）
const SharingConfigSchema = z.object({
  skills: z.object({}).default({}),
  rules: z.object({
    enforced: z.array(z.string()).default([]),  // 强制注入的规则列表
  }).default({}),
  docs: z.object({
    localDir: z.string().default('~/.teamai/docs'),
  }).default({}),
  env: z.object({
    injectShellProfile: z.boolean().default(true),
    shellProfilePath: z.string().optional(),
  }).default({}),
  hooks: z.object({
    autoApply: z.boolean().default(true),       // 自动注入 hooks
    requireTeamScripts: z.boolean().default(false),
  }).optional(),
});
```

```typescript
// Scope 枚举
type Scope = 'user' | 'project';
// user: 资源安装到 ~/（跨项目共享）
// project: 资源安装到 .teamai/（项目独立）
```

## 配置系统

**优先级（高→低）：**
1. 命令行参数（`--repo`, `--scope`, `--force` 等）
2. 项目级配置（`<project>/.teamai/config.yaml`）
3. 用户级配置（`~/.teamai/config.yaml`）
4. 团队仓配置（`team-repo/teamai.yaml`）
5. 内置默认值（Zod Schema defaults）

**Scope 检测逻辑：**
- 显式指定 `--scope project` → 使用项目目录
- 存在 `<cwd>/.teamai/config.yaml` → project scope
- 否则 → user scope（默认）

**关键配置结构（teamai.yaml）：**
```yaml
provider: github | tgit          # Git 平台（自动检测）
scope: user | project            # 资源安装位置
sharing:
  skills: {}
  rules:
    enforced: [security, code-style]  # 强制规则
  docs:
    localDir: ~/.teamai/docs
  env:
    injectShellProfile: true
  hooks:
    autoApply: true              # 自动注入 AI 工具
    requireTeamScripts: false    # 是否限制为团队脚本
sources:                         # 跨团队订阅
  - name: other-team
    repo: git@github.com:other/repo.git
```

## 性能与可靠性

| 设计点 | 实现方式 | 说明 |
|--------|----------|------|
| 增量同步 | git fetch + diff 快速路径 | 无变更时 pull 近乎零开销 |
| 缓存索引 | `cache-index.ts` + `repo-cache.ts` | 避免重复解析未变更文件 |
| 批量导入并发 | graph-index.json 写锁 | 解决多进程 race condition（#67） |
| 搜索性能 | 预构建 search-index | recall 查询无需遍历文件系统 |
| 超时控制 | simple-git 超时配置 | 网络不稳定时不会无限挂起 |
| 幂等操作 | pull/push 均幂等 | 重复执行不会破坏状态 |
| 去重 | `dedup.ts` | 跨源订阅时避免资源重复 |
| 降级策略 | provider fallback（gh → token） | 认证方式不可用时自动降级 |

## 架构决策与权衡

- **为什么用 Git 仓库而不是数据库/SaaS**：团队已有 Git 工作流，零额外基础设施；版本历史天然可追溯；权限复用 Git 平台（GitHub/TGit）的 RBAC。代价是不适合高频写入场景。

- **为什么 ESM-only 而不是 CJS 双模式**：Node.js 20+ 生态已全面拥抱 ESM；tsup 直接输出 ESM 简化构建；chalk 5+ 等依赖强制 ESM。代价是不兼容 Node.js <18。

- **为什么 Commander 而不是 yargs/oclif**：Commander 轻量、无子依赖、lazy-import 友好（每个命令 `await import()`）；oclif 过重不适合快速迭代的 CLI。代价是没有内置 plugin 体系。

- **为什么用 Zod 做 Schema 验证**：类型推断 + 运行时验证一体化；`.default()` 天然支持配置默认值；错误信息可读性好。代价是包体增加 ~50KB。

- **为什么 Provider 抽象层分离 GitHub/TGit**：腾讯内部用 TGit（工蜂），外部用 GitHub；统一接口后 push/pull 逻辑不感知平台差异。代价是每新增平台需实现完整 provider。

## 已知限制与演进方向

- **单仓限制**：当前每个 scope 仅绑定一个团队仓库；跨团队资源共享需通过 `sources` 订阅，配置较繁琐。→ 未来可能支持多仓直连。

- **搜索精度**：recall 基于关键词匹配 + 简单评分，缺少语义向量检索能力。→ 可集成 embedding 模型提升召回质量。

- **知识图谱深度**：`import` 命令的 AST 解析目前侧重 TypeScript/JavaScript，对其他语言支持有限。→ 可扩展 tree-sitter 多语言 parser。

- **冲突解决**：多人同时 push 相同资源时依赖 Git merge，缺少资源级细粒度锁。→ 可引入乐观锁或 advisory lock 机制。

- **离线场景**：pull/push 强依赖网络可达；离线时无法同步也无法 recall（本地索引可用但可能过期）。→ 可增加离线模式 + 自动重试队列。

## 测试覆盖

| 层级 | 文件数 | 说明 |
|------|--------|------|
| 单元测试 | 70+ | 覆盖各模块核心逻辑（resources、utils、domains、commands） |
| 集成测试 | 5+ | 跨模块交互（import-repo-merge、cross-repo-edges、hooks-e2e） |
| E2E 测试 | 2 | `src/__tests__/e2e/e2e.test.ts` + `test/e2e.mjs`（需 token） |
| 覆盖率工具 | @vitest/coverage-v8 | `npm run test:coverage` |
| CI 验证 | GitHub Actions + Coding CI | validate → build → e2e → publish |

E2E 测试覆盖命令链：init → push → pull → source → env → tags → roles → contribute → dashboard → uninstall。需配置 `TEAMAI_TEST_REPO_URL` + `TEAMAI_TEST_TOKEN` 方可在 CI 运行。

## 备注

- ✅ 项目概述、技术栈、命令列表、类型定义 — 基于 package.json / index.ts / types.ts 源码
- ✅ CI/CD 流程、Provider 机制 — 基于 docs/ 文档
- ✅ 配置系统 — 基于 types.ts 中的 Zod Schema 定义
- ⚠️ 资源处理器具体接口方法 — 基于文件命名与 pull/push 调用模式推断
- ⚠️ 域管理（cluster/recommend）实现细节 — 基于文件名推断
- ⚠️ 性能设计中的具体参数（超时值、并发数）— 基于 commit 信息与文件名推断

<!-- teamai:referenced-doc-ids: [] -->