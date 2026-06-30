---
title: github__Tencent__teamai-cli — src module
domain: code-knowledge
source: [src/]
---

# src

**772 facts** (component: 540, interface: 207, config: 21, error: 4)

**Depends on**: tsup.config.ts

## Core components

- `ConfidenceFactor` ← src/wiki-engine/reconciler-v2-types.ts:5 (90 refs)
- `NumericConfidence` ← src/wiki-engine/reconciler-v2-types.ts:11 (90 refs)
- `fromLegacyConfidence` ← src/wiki-engine/reconciler-v2-types.ts:18 (90 refs)
- `labelFromScore` ← src/wiki-engine/reconciler-v2-types.ts:32 (90 refs)
- `buildConfidence` ← src/wiki-engine/reconciler-v2-types.ts:39 (90 refs)
- `ApiInterfaceMatch` ← src/wiki-engine/reconciler-v2-types.ts:48 (90 refs)
- `RuleCodeMatch` ← src/wiki-engine/reconciler-v2-types.ts:58 (90 refs)
- `ReconcileStaleWarning` ← src/wiki-engine/reconciler-v2-types.ts:67 (90 refs)
- `ReconcileLogEntry` ← src/wiki-engine/reconciler-v2-types.ts:78 (90 refs)
- `ReconcileStats` ← src/wiki-engine/reconciler-v2-types.ts:94 (90 refs)
- `ReconcileV2Extensions` ← src/wiki-engine/reconciler-v2-types.ts:109 (90 refs)
- `extractTypescript` ← src/wiki-engine/code-knowledge/extractors/typescript.ts:8 (90 refs)
- `RepoInfo` ← src/providers/types.ts:17 (89 refs)
- `PrCreateOptions` ← src/providers/types.ts:25 (89 refs)
- `OrgRepoInfo` ← src/providers/types.ts:45 (89 refs)
- `GitProvider` ← src/providers/types.ts:62 (89 refs)
- `RepoNotFoundError` ← src/providers/types.ts:144 (89 refs)
- `ToolPathsSchema` ← src/types.ts:6 (88 refs)
- `ScopeEnum` ← src/types.ts:19 (88 refs)
- `Scope` ← src/types.ts:20 (88 refs)

## Config

- `process.env.TEAMAI_CACHE_DIR` ← src/utils/cache-index.ts
- `process.env.TEAMAI_CACHE_MAX_BYTES` ← src/utils/cache-index.ts
- `process.env.HOME` ← src/utils/search-index.ts
- `process.env.TEAMAI_RECALL_DISABLED` ← src/auto-recall.ts
- `process.env.TEAMAI_EVAL_LOG_PATH` ← src/auto-recall.ts
- `process.env.TEAMAI_SEARCH_STRATEGY` ← src/auto-recall.ts
- `process.env.CLAUDE_SESSION_ID` ← src/contribute-check.ts
- `process.env.SHELL` ← src/doctor.ts
- `process.env.TEAMAI_MR_HINT_DISABLED` ← src/mr-hint.ts
- `process.env.TEAMAI_MR_HINT_CWD` ← src/mr-hint.ts

## Errors

- `RepoNotFoundError` ← src/providers/github/index.ts
- `TRANSCRIPT_INTERRUPT_PREFIX` ← src/dashboard-collector.ts
- `INFERRED` ← src/enrich-with-ai.ts
- `AggregateError` ← src/utils/ai-client.ts
