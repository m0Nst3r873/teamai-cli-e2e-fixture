---
title: github__m0Nst3r873__teamai-cli config
domain: code-knowledge
source:
  - src/utils/cache-index.ts
  - src/utils/search-index.ts
  - src/auto-recall.ts
  - src/contribute-check.ts
  - src/doctor.ts
  - src/mr-hint.ts
  - src/types.ts
  - src/update.ts
  - src/providers/registry.ts
  - src/utils/ai-client.ts
  - src/providers/github/gh-cli.ts
---

# Config

- `process.env.TEAMAI_CACHE_DIR` ← src/utils/cache-index.ts:65 [EXTRACTED]
  ```
  return process.env.TEAMAI_CACHE_DIR ?? path.join(os.homedir(), '.teamai', 'cache', 'repos');
  ```
- `process.env.TEAMAI_CACHE_MAX_BYTES` ← src/utils/cache-index.ts:263 [EXTRACTED]
  ```
  const envVal = process.env.TEAMAI_CACHE_MAX_BYTES;
  ```
- `process.env.HOME` ← src/utils/search-index.ts:17 [EXTRACTED]
  ```
  return `${process.env.HOME ?? ''}/.teamai/search-index.json`;
  ```
- `process.env.TEAMAI_RECALL_DISABLED` ← src/auto-recall.ts:504 [EXTRACTED]
  ```
  if (process.env.TEAMAI_RECALL_DISABLED === '1') {
  ```
- `process.env.TEAMAI_EVAL_LOG_PATH` ← src/auto-recall.ts:605 [EXTRACTED]
  ```
  const evalLogPath = process.env.TEAMAI_EVAL_LOG_PATH;
  ```
- `process.env.TEAMAI_SEARCH_STRATEGY` ← src/auto-recall.ts:617 [EXTRACTED]
  ```
  strategy: process.env.TEAMAI_SEARCH_STRATEGY ?? 'keyword-v1',
  ```
- `process.env.CLAUDE_SESSION_ID` ← src/contribute-check.ts:57 [EXTRACTED]
  ```
  *   2. process.env.CLAUDE_SESSION_ID
  ```
- `process.env.SHELL` ← src/doctor.ts:129 [EXTRACTED]
  ```
  const shell = process.env.SHELL ?? '';
  ```
- `process.env.TEAMAI_MR_HINT_DISABLED` ← src/mr-hint.ts:410 [EXTRACTED]
  ```
  if (process.env.TEAMAI_MR_HINT_DISABLED === '1') return null;
  ```
- `process.env.TEAMAI_MR_HINT_CWD` ← src/mr-hint.ts:413 [EXTRACTED]
  ```
  const rawCwd = process.env.TEAMAI_MR_HINT_CWD ?? process.cwd();
  ```
- `TEAMAI_CONFIG_PATH` ← src/types.ts:272 [EXTRACTED]
  ```
  export const TEAMAI_CONFIG_PATH = `${TEAMAI_HOME}/config.yaml`;
  ```
- `TEAMAI_ENV_START` ← src/types.ts:292 [EXTRACTED]
  ```
  export const TEAMAI_ENV_START = '# [teamai:env:start]';
  ```
- `TEAMAI_ENV_END` ← src/types.ts:293 [EXTRACTED]
  ```
  export const TEAMAI_ENV_END = '# [teamai:env:end]';
  ```
- `DASHBOARD_DEFAULT_PORT` ← src/types.ts:422 [EXTRACTED]
  ```
  export const DASHBOARD_DEFAULT_PORT = 3721;
  ```
- `process.env.TEAMAI_WIKI_DISABLED` ← src/types.ts:669 [EXTRACTED]
  ```
  if (process.env.TEAMAI_WIKI_DISABLED === '1' || process.env.TEAMAI_WIKI_DISABLED === 'true') return false;
  ```
- `process.env.TEAMAI_WIKI_ENABLED` ← src/types.ts:670 [EXTRACTED]
  ```
  if (process.env.TEAMAI_WIKI_ENABLED === '0' || process.env.TEAMAI_WIKI_ENABLED === 'false') return false;
  ```
- `process.env.TEAMAI_HOOKS_DISABLED` ← src/types.ts:679 [EXTRACTED]
  ```
  return process.env.TEAMAI_HOOKS_DISABLED === '1' || process.env.TEAMAI_HOOKS_DISABLED === 'true';
  ```
- `process.env.TEAMAI_NPM_REGISTRY` ← src/update.ts:37 [EXTRACTED]
  ```
  const override = process.env.TEAMAI_NPM_REGISTRY?.trim();
  ```
- `process.env.TEAMAI_DEFAULT_PROVIDER` ← src/providers/registry.ts:46 [EXTRACTED]
  ```
  const override = process.env.TEAMAI_DEFAULT_PROVIDER?.trim();
  ```
- `process.env.PATH` ← src/utils/ai-client.ts:35 [EXTRACTED]
  ```
  *   3. `which <cmd>` —— 最终 fallback，使用 process.env.PATH 直接查找
  ```
- `process.env.GITHUB_TOKEN` ← src/providers/github/gh-cli.ts:109 [EXTRACTED]
  ```
  return process.env.GITHUB_TOKEN ?? process.env.GH_TOKEN ?? null;
  ```