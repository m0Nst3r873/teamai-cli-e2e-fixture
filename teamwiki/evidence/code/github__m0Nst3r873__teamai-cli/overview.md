---
title: github__m0Nst3r873__teamai-cli overview
domain: code-knowledge
---

# github__m0Nst3r873__teamai-cli

**1395 facts** extracted from 151 files.
Graph: 768 nodes, 987 edges.

## Module Structure

| Module | Facts | Components | Interfaces |
|--------|-------|------------|------------|
| src | 759 | 530 | 205 |

## Dependencies

- **src** → tsup.config.ts

## Key Dependency Paths

- buildHandlerRegistry (src/hook-handlers.ts): buildHandlerRegistry → createDispatcher → deriveSessionId → normalizeToolName → filterRulesByKnowledgeNamespaces → loadTeamConfig → expandHome → MAX_LOG_BYTES → loadRolesManifest → extractToml → isKeyFile → mapKindToEvidenceType → createGit → injectClaudeMdSection → resolveRegistryForPackage → resolveEffectiveUpdatePolicy → ToolPathsSchema → RepoNotFoundError
- getHandler (src/resources/index.ts): getHandler → ResourceHandler → ToolPathsSchema → RepoNotFoundError → expandHome → MAX_LOG_BYTES → ensureSkillFrontmatter → RulesHandler → DocsHandler → EnvHandler
- getAllHandlers (src/resources/index.ts): getAllHandlers → ResourceHandler → ToolPathsSchema → RepoNotFoundError → expandHome → MAX_LOG_BYTES → ensureSkillFrontmatter → RulesHandler → DocsHandler → EnvHandler
- AgentsHandler (src/resources/agents.ts): AgentsHandler → ResourceHandler → ToolPathsSchema → RepoNotFoundError → expandHome → MAX_LOG_BYTES
- DocsHandler (src/resources/docs.ts): DocsHandler → ResourceHandler → ToolPathsSchema → RepoNotFoundError → expandHome → MAX_LOG_BYTES
