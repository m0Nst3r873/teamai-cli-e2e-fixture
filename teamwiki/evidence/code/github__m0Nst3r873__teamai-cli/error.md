---
title: github__m0Nst3r873__teamai-cli error
domain: code-knowledge
source:
  - src/providers/github/index.ts
  - src/enrich-with-ai.ts
  - src/utils/ai-client.ts
---

# Error

- `RepoNotFoundError` ← src/providers/github/index.ts:44 [EXTRACTED]
  ```
  throw new RepoNotFoundError(repo);
  ```
- `INFERRED` ← src/enrich-with-ai.ts:150 [INFERRED]
  ```
  confidence: 'INFERRED' as const,
  ```
- `AggregateError` ← src/utils/ai-client.ts:204 [EXTRACTED]
  ```
  throw new AggregateError(errors, `${errors.length} AI task(s) failed`);
  ```