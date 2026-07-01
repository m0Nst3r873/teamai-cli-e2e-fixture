---
title: github__Tencent__teamai-cli — src module
domain: code-knowledge
source: [src/]
---

# src

**821 facts** (component: 574, interface: 218, config: 24, error: 5)

**Depends on**: tsup.config.ts

## Core components

- `RepoInfo` ← src/providers/types.ts:17 (88 refs)
- `PrCreateOptions` ← src/providers/types.ts:25 (88 refs)
- `OrgRepoInfo` ← src/providers/types.ts:45 (88 refs)
- `GitProvider` ← src/providers/types.ts:62 (88 refs)
- `RepoNotFoundError` ← src/providers/types.ts:144 (88 refs)
- `ConfidenceFactor` ← src/wiki-engine/reconciler-v2-types.ts:5 (88 refs)
- `NumericConfidence` ← src/wiki-engine/reconciler-v2-types.ts:11 (88 refs)
- `fromLegacyConfidence` ← src/wiki-engine/reconciler-v2-types.ts:18 (88 refs)
- `labelFromScore` ← src/wiki-engine/reconciler-v2-types.ts:32 (88 refs)
- `buildConfidence` ← src/wiki-engine/reconciler-v2-types.ts:39 (88 refs)
- `ApiInterfaceMatch` ← src/wiki-engine/reconciler-v2-types.ts:48 (88 refs)
- `RuleCodeMatch` ← src/wiki-engine/reconciler-v2-types.ts:58 (88 refs)
- `ReconcileStaleWarning` ← src/wiki-engine/reconciler-v2-types.ts:67 (88 refs)
- `ReconcileLogEntry` ← src/wiki-engine/reconciler-v2-types.ts:78 (88 refs)
- `ReconcileStats` ← src/wiki-engine/reconciler-v2-types.ts:94 (88 refs)
- `ReconcileV2Extensions` ← src/wiki-engine/reconciler-v2-types.ts:109 (88 refs)
- `extractTypescript` ← src/wiki-engine/code-knowledge/extractors/typescript.ts:8 (88 refs)
- `ToolPathsSchema` ← src/types.ts:6 (87 refs)
- `ScopeEnum` ← src/types.ts:18 (87 refs)
- `Scope` ← src/types.ts:19 (87 refs)

## Config

- `process.env.TEAMAI_CACHE_DIR` ← src/utils/cache-index.ts
- `process.env.TEAMAI_CACHE_MAX_BYTES` ← src/utils/cache-index.ts
- `process.env.HOME` ← src/utils/search-index.ts
- `process.env.TEAMAI_API_TOKEN` ← src/api-key.ts
- `process.env.TEAMAI_RECALL_DISABLED` ← src/auto-recall.ts
- `process.env.TEAMAI_EVAL_LOG_PATH` ← src/auto-recall.ts
- `process.env.TEAMAI_SEARCH_STRATEGY` ← src/auto-recall.ts
- `process.env.CLAUDE_SESSION_ID` ← src/contribute-check.ts
- `process.env.SHELL` ← src/doctor.ts
- `process.env.TEAMAI_MR_HINT_DISABLED` ← src/mr-hint.ts

## Errors

- `RepoNotFoundError` ← src/providers/github/index.ts
- `TRANSCRIPT_INTERRUPT_PREFIX` ← src/dashboard-collector.ts
- `INFERRED` ← src/enrich-with-ai.ts
- `RepoNotAvailableError` ← src/source-http.ts
- `AggregateError` ← src/utils/ai-client.ts
