---
title: get-shit-done — TOC curado
type: moc
tags: [sources, get-shit-done, gsd, agents, sdd, toc]
created: 2026-04-22
---

# GSD — Índice curado

Navegação por **tópico** (não por layout do repo). 346 MDs em `raw/` após limpeza de i18n e admin. Destilados em `_notes/`.

## Leitura obrigatória (ordem)

1. [[raw/README|README]] — landing
2. [[raw/docs/ARCHITECTURE|docs/ARCHITECTURE]] — modelo mental
3. [[raw/docs/AGENTS|docs/AGENTS]] — visão geral do ecosistema de agents
4. [[raw/docs/USER-GUIDE|docs/USER-GUIDE]] — fluxo do usuário
5. [[raw/docs/FEATURES|docs/FEATURES]] + [[raw/docs/COMMANDS|docs/COMMANDS]]

## Por tópico

### 🤖 Agentes especializados (33 em `raw/agents/`)

Cada agent é um **contrato** (system prompt + ferramentas + responsabilidade). Mapeamento pro nosso pipeline Planner/Worker/Reviewer em `_notes/mapping-planner-worker-reviewer.md` (a criar).

**Research & análise:**
- [[raw/agents/gsd-advisor-researcher|advisor-researcher]]
- [[raw/agents/gsd-ai-researcher|ai-researcher]]
- [[raw/agents/gsd-assumptions-analyzer|assumptions-analyzer]]
- [[raw/agents/gsd-codebase-mapper|codebase-mapper]]
- [[raw/agents/gsd-domain-researcher|domain-researcher]]
- [[raw/agents/gsd-pattern-mapper|pattern-mapper]]
- [[raw/agents/gsd-phase-researcher|phase-researcher]]
- [[raw/agents/gsd-project-researcher|project-researcher]]
- [[raw/agents/gsd-research-synthesizer|research-synthesizer]]
- [[raw/agents/gsd-user-profiler|user-profiler]]

**Planning & roadmap:**
- [[raw/agents/gsd-planner|planner]]
- [[raw/agents/gsd-plan-checker|plan-checker]]
- [[raw/agents/gsd-roadmapper|roadmapper]]
- [[raw/agents/gsd-framework-selector|framework-selector]]
- [[raw/agents/gsd-intel-updater|intel-updater]]

**Executor & code:**
- [[raw/agents/gsd-executor|executor]]
- [[raw/agents/gsd-code-fixer|code-fixer]]
- [[raw/agents/gsd-code-reviewer|code-reviewer]]
- [[raw/agents/gsd-integration-checker|integration-checker]]

**Verifier / audit:**
- [[raw/agents/gsd-verifier|verifier]]
- [[raw/agents/gsd-eval-auditor|eval-auditor]]
- [[raw/agents/gsd-eval-planner|eval-planner]]
- [[raw/agents/gsd-nyquist-auditor|nyquist-auditor]]
- [[raw/agents/gsd-security-auditor|security-auditor]]
- [[raw/agents/gsd-ui-auditor|ui-auditor]]
- [[raw/agents/gsd-ui-checker|ui-checker]]
- [[raw/agents/gsd-ui-researcher|ui-researcher]]

**Debug & docs:**
- [[raw/agents/gsd-debugger|debugger]]
- [[raw/agents/gsd-debug-session-manager|debug-session-manager]]
- [[raw/agents/gsd-doc-classifier|doc-classifier]]
- [[raw/agents/gsd-doc-synthesizer|doc-synthesizer]]
- [[raw/agents/gsd-doc-verifier|doc-verifier]]
- [[raw/agents/gsd-doc-writer|doc-writer]]

### 🔄 Workflows (81 em `raw/get-shit-done/workflows/`)

São os **procedimentos executáveis** do GSD — equivalentes aos `30-agents/playbooks/` nosso. Organização por ciclo SDD:

**Ciclo de 5 fases:**
- [[raw/get-shit-done/workflows/discuss-phase|discuss-phase]] + [[raw/get-shit-done/workflows/discuss-phase-power|-power]] + [[raw/get-shit-done/workflows/discuss-phase-assumptions|-assumptions]]
- [[raw/get-shit-done/workflows/plan-phase|plan-phase]] + [[raw/get-shit-done/workflows/plan-review-convergence|plan-review-convergence]] + [[raw/get-shit-done/workflows/plan-milestone-gaps|milestone-gaps]]
- [[raw/get-shit-done/workflows/execute-phase|execute-phase]] + [[raw/get-shit-done/workflows/execute-plan|execute-plan]]
- [[raw/get-shit-done/workflows/verify-phase|verify-phase]] + [[raw/get-shit-done/workflows/verify-work|verify-work]]
- [[raw/get-shit-done/workflows/ship|ship]]

**Review / audit:**
- [[raw/get-shit-done/workflows/code-review|code-review]] + [[raw/get-shit-done/workflows/code-review-fix|-fix]]
- [[raw/get-shit-done/workflows/audit-fix|audit-fix]] + [[raw/get-shit-done/workflows/audit-milestone|-milestone]] + [[raw/get-shit-done/workflows/audit-uat|-uat]]
- [[raw/get-shit-done/workflows/review|review]] + [[raw/get-shit-done/workflows/eval-review|eval-review]]
- [[raw/get-shit-done/workflows/forensics|forensics]]

**Research & discovery:**
- [[raw/get-shit-done/workflows/research-phase|research-phase]]
- [[raw/get-shit-done/workflows/discovery-phase|discovery-phase]]
- [[raw/get-shit-done/workflows/map-codebase|map-codebase]]
- [[raw/get-shit-done/workflows/explore|explore]]
- [[raw/get-shit-done/workflows/spike|spike]] + [[raw/get-shit-done/workflows/spike-wrap-up|-wrap-up]]
- [[raw/get-shit-done/workflows/sketch|sketch]] + [[raw/get-shit-done/workflows/sketch-wrap-up|-wrap-up]]

**Project / milestone:**
- [[raw/get-shit-done/workflows/new-project|new-project]] + [[raw/get-shit-done/workflows/new-milestone|new-milestone]]
- [[raw/get-shit-done/workflows/new-workspace|new-workspace]] + [[raw/get-shit-done/workflows/list-workspaces|list-workspaces]] + [[raw/get-shit-done/workflows/remove-workspace|remove-workspace]]
- [[raw/get-shit-done/workflows/milestone-summary|milestone-summary]] + [[raw/get-shit-done/workflows/complete-milestone|complete-milestone]]
- [[raw/get-shit-done/workflows/graduation|graduation]]

**Specialized phases:**
- [[raw/get-shit-done/workflows/secure-phase|secure-phase]]
- [[raw/get-shit-done/workflows/ui-phase|ui-phase]] + [[raw/get-shit-done/workflows/ui-review|ui-review]]
- [[raw/get-shit-done/workflows/ai-integration-phase|ai-integration-phase]]
- [[raw/get-shit-done/workflows/ultraplan-phase|ultraplan-phase]]

**Meta & sessão:**
- [[raw/get-shit-done/workflows/autonomous|autonomous]]
- [[raw/get-shit-done/workflows/pause-work|pause-work]] + [[raw/get-shit-done/workflows/resume-project|resume-project]]
- [[raw/get-shit-done/workflows/session-report|session-report]]
- [[raw/get-shit-done/workflows/extract_learnings|extract_learnings]]
- [[raw/get-shit-done/workflows/profile-user|profile-user]]

**Task mgmt:**
- [[raw/get-shit-done/workflows/add-phase|add-phase]], [[raw/get-shit-done/workflows/insert-phase|insert]], [[raw/get-shit-done/workflows/remove-phase|remove]]
- [[raw/get-shit-done/workflows/add-todo|add-todo]] + [[raw/get-shit-done/workflows/check-todos|check-todos]]
- [[raw/get-shit-done/workflows/add-tests|add-tests]]
- [[raw/get-shit-done/workflows/node-repair|node-repair]]

### 📝 Templates (em `raw/get-shit-done/templates/`)

Artefatos do ciclo GSD — comparar com spec-kit/templates e 40-templates/ nosso.

- [[raw/get-shit-done/templates/AI-SPEC|AI-SPEC]] — spec quando task envolve AI
- [[raw/get-shit-done/templates/claude-md|claude-md]] — modelo de CLAUDE.md
- [[raw/get-shit-done/templates/copilot-instructions|copilot-instructions]]
- [[raw/get-shit-done/templates/context|context]] — context file
- [[raw/get-shit-done/templates/continue-here|continue-here]] — continuation
- [[raw/get-shit-done/templates/DEBUG|DEBUG]]
- [[raw/get-shit-done/templates/debug-subagent-prompt|debug-subagent-prompt]]
- [[raw/get-shit-done/templates/discovery|discovery]]
- [[raw/get-shit-done/templates/discussion-log|discussion-log]]
- [[raw/get-shit-done/templates/milestone|milestone]] + [[raw/get-shit-done/templates/milestone-archive|-archive]]
- [[raw/get-shit-done/templates/phase-prompt|phase-prompt]]
- [[raw/get-shit-done/templates/planner-subagent-prompt|planner-subagent-prompt]]
- [[raw/get-shit-done/templates/project|project]]
- [[raw/get-shit-done/templates/requirements|requirements]]
- [[raw/get-shit-done/templates/research|research]]
- [[raw/get-shit-done/templates/retrospective|retrospective]]
- [[raw/get-shit-done/templates/roadmap|roadmap]]
- [[raw/get-shit-done/templates/SECURITY|SECURITY]]
- [[raw/get-shit-done/templates/spec|spec]]
- [[raw/get-shit-done/templates/state|state]]
- [[raw/get-shit-done/templates/summary|summary]] + [[raw/get-shit-done/templates/summary-complex|-complex]] + [[raw/get-shit-done/templates/summary-minimal|-minimal]] + [[raw/get-shit-done/templates/summary-standard|-standard]]
- [[raw/get-shit-done/templates/UAT|UAT]]
- [[raw/get-shit-done/templates/UI-SPEC|UI-SPEC]]
- [[raw/get-shit-done/templates/user-profile|user-profile]] + [[raw/get-shit-done/templates/user-setup|user-setup]]
- [[raw/get-shit-done/templates/VALIDATION|VALIDATION]]
- [[raw/get-shit-done/templates/verification-report|verification-report]]
- Subdirs: `raw/get-shit-done/templates/codebase/` + `raw/get-shit-done/templates/research-project/`

### 🧩 Contexts (modo operacional)

- [[raw/get-shit-done/contexts/dev|dev]]
- [[raw/get-shit-done/contexts/research|research]]
- [[raw/get-shit-done/contexts/review|review]]

### 📚 References (docs conceituais em `raw/get-shit-done/references/`)

- [[raw/get-shit-done/references/agent-contracts|agent-contracts]] — ★ como escrever um agent
- [[raw/get-shit-done/references/ai-evals|ai-evals]] + [[raw/get-shit-done/references/ai-frameworks|ai-frameworks]]
- [[raw/get-shit-done/references/artifact-types|artifact-types]]
- [[raw/get-shit-done/references/autonomous-smart-discuss|autonomous-smart-discuss]]
- [[raw/get-shit-done/references/checkpoints|checkpoints]]
- [[raw/get-shit-done/references/common-bug-patterns|common-bug-patterns]]
- [[raw/get-shit-done/references/context-budget|context-budget]]
- [[raw/get-shit-done/references/continuation-format|continuation-format]]
- [[raw/get-shit-done/references/debugger-philosophy|debugger-philosophy]]
- [[raw/get-shit-done/references/decimal-phase-calculation|decimal-phase-calculation]]
- [[raw/get-shit-done/references/doc-conflict-engine|doc-conflict-engine]]
- [[raw/get-shit-done/references/domain-probes|domain-probes]]
- [[raw/get-shit-done/references/executor-examples|executor-examples]]
- [[raw/get-shit-done/references/gate-prompts|gate-prompts]] + [[raw/get-shit-done/references/gates|gates]]
- [[raw/get-shit-done/references/git-integration|git-integration]] + [[raw/get-shit-done/references/git-planning-commit|git-planning-commit]]
- [[raw/get-shit-done/references/mandatory-initial-read|mandatory-initial-read]]
- [[raw/get-shit-done/references/model-profiles|model-profiles]] + [[raw/get-shit-done/references/model-profile-resolution|resolution]]
- [[raw/get-shit-done/references/phase-argument-parsing|phase-argument-parsing]]
- Subdir: `raw/get-shit-done/references/few-shot-examples/`

### 🛠️ Commands (slash commands do GSD — 85 em `raw/commands/gsd/`)

Referência de vocabulário (nomes, sintaxe, semântica). **Não replicar tudo** — extrair só os que fazem sentido pro automation-agents.

Alta prioridade: `autonomous`, `fast`, `next`, `do`, `plan-phase`, `review`, `ship`, `new-project`, `milestone-summary`, `pause-work`, `resume-work`, `sync-skills`, `intel`, `graphify`, `from-gsd2`.

### 📦 SDK (em `raw/sdk/`)

- [[raw/sdk/README|README do SDK]]
- [[raw/sdk/docs/caching|caching]] — estratégia de cache do SDK
- [[raw/sdk/HANDOVER-GOLDEN-PARITY|HANDOVER golden-parity]]
- [[raw/sdk/HANDOVER-PARITY-DOCS|HANDOVER parity-docs]]
- [[raw/sdk/HANDOVER-QUERY-LAYER|HANDOVER query-layer]]
- Prompts: `raw/sdk/prompts/agents/`, `raw/sdk/prompts/templates/`, `raw/sdk/prompts/workflows/`
- Source docs: `raw/sdk/src/query/QUERY-HANDLERS.md`, `raw/sdk/src/golden/fixtures/`

### 📖 Docs gerais (em `raw/docs/`)

- [[raw/docs/AGENTS|AGENTS]] — visão geral
- [[raw/docs/ARCHITECTURE|ARCHITECTURE]] — ★
- [[raw/docs/BETA|BETA]]
- [[raw/docs/CLI-TOOLS|CLI-TOOLS]]
- [[raw/docs/COMMANDS|COMMANDS]]
- [[raw/docs/CONFIGURATION|CONFIGURATION]]
- [[raw/docs/context-monitor|context-monitor]]
- [[raw/docs/FEATURES|FEATURES]]
- [[raw/docs/gsd-sdk-query-migration-blurb|gsd-sdk-query-migration-blurb]]
- [[raw/docs/INVENTORY|INVENTORY]]
- [[raw/docs/manual-update|manual-update]]
- [[raw/docs/USER-GUIDE|USER-GUIDE]]
- [[raw/docs/workflow-discuss-mode|workflow-discuss-mode]]
- Subdirs: [[raw/docs/skills/discovery-contract|skills/discovery-contract]], [[raw/docs/superpowers/specs/2026-04-17-ultraplan-phase-design|superpowers/specs/2026-04-17-ultraplan-phase-design]]

## Prioridade de destilação

**P0 — destilar em `_notes/`:**
1. Mapeamento `gsd-*` agents → Planner / Worker / Reviewer nosso → `_notes/mapping-planner-worker-reviewer.md`
2. `docs/ARCHITECTURE` + `references/agent-contracts` → contexto de `[[../../30-agents/_moc]]`
3. Workflows do ciclo de 5 fases → `[[../../10-knowledge/methodology/sdd-workflow]]`
4. `templates/claude-md` + nosso `40-templates/CLAUDE.md.template` — reconciliar

**P1:**
5. Templates de SPEC/STATE/UAT → comparar e reconciliar com `40-templates/`
6. `references/context-budget` + `context-monitor` → `[[../../30-agents/autonomous-execution]]`
7. `references/gates` + `gate-prompts` → conceito de gates no pipeline nosso

## Vizinhos no grafo

- [[_index]]
- [[_notes/_moc]]
- [[../spec-kit/_toc]] — fonte complementar (specs)
- [[../../30-agents/_moc]] — destino
- [[../../10-knowledge/methodology/sdd-workflow]]
- [[../../40-templates/_moc]]
