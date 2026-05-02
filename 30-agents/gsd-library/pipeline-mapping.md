---
title: Pipeline Mapping — nosso pipeline × GSD agents
type: note
tags: [agents, gsd, pipeline, mapping, crosswalk]
created: 2026-04-24
updated: 2026-04-24
aliases: [pipeline-mapping, crosswalk planner worker reviewer, gsd mapping]
---

# Pipeline Mapping

Correspondência entre os **papéis canônicos do nosso pipeline** (Planner / Worker / Reviewer / Orchestrator) e os **33 agents especializados** do ecosistema [gsd-build/get-shit-done](https://github.com/gsd-build/get-shit-done).

> **Regra**: esta tabela mostra **afinidade funcional**, não equivalência 1:1. Nosso Planner ≠ gsd-planner — eles **compartilham intenção** (decompor trabalho em plano executável) mas têm contratos diferentes. Biblioteca GSD é inspiração/referência, não template de copy-paste.

---

## 📋 Planner — decompõe trabalho em plano executável

Papel: traduzir spec/requisito em tasks executáveis com dependências e gates. No nosso pipeline vive em [[../planner/_moc]].

| GSD agent | Afinidade | Insight aproveitável |
|---|---|---|
| [[card-gsd-planner]] | **primária** | PLAN.md como prompt (não documento que vira prompt); 2-3 tasks/plan; waves de paralelização; `<verify>` declarativo |
| [[card-gsd-plan-checker]] | **primária** | plan validation **antes** de execução — evita retrabalho |
| [[card-gsd-roadmapper]] | secundária | planejamento macro (milestones, fases) vs tático (sprint) |
| [[card-gsd-framework-selector]] | secundária | decisão de stack como plan step discreto |
| [[card-gsd-eval-planner]] | tangencial | plano de avaliação antes de execução (útil em tasks de ML/research) |

## 🔨 Worker — executa o plano produzindo artefato

Papel: rodar as tasks do plano, commitar atomicamente, reportar progresso. No nosso pipeline vive em [[../worker/_moc]].

| GSD agent | Afinidade | Insight aproveitável |
|---|---|---|
| [[card-gsd-executor]] | **primária** | commits atômicos por task; deviation rules (auto-fix bugs, add missing critical functionality, block on architectural changes); TDD como fluxo RED→GREEN→REFACTOR |
| [[card-gsd-code-fixer]] | **primária** | auto-fix dentro do escopo atual; escalonar architectural changes |
| [[card-gsd-integration-checker]] | secundária | validação de integração antes de ship |
| [[card-gsd-doc-writer]] | secundária | worker também escreve docs de output (summary, release notes) |

## 👁 Reviewer — segundo par de olhos, valida após execução

Papel: auditoria independente em contexto limpo; detecta alucinação/desvio; aprova ou rejeita. No nosso pipeline vive em [[../reviewer/_moc]] e usa [[../playbooks/dual-check]].

| GSD agent | Afinidade | Insight aproveitável |
|---|---|---|
| [[card-gsd-verifier]] | **primária** | verificação funcional + cobertura de must-haves |
| [[card-gsd-code-reviewer]] | **primária** | review de diff/PR, mas **em contexto limpo** (não viu plano) |
| [[card-gsd-eval-auditor]] | secundária | valida output de avaliações (ex.: benchmarks) |
| [[card-gsd-nyquist-auditor]] | secundária | auditoria de *sampling rate* (insight importado de Signal Processing) |
| [[card-gsd-security-auditor]] | secundária | STRIDE + threat model como task discreta |
| [[card-gsd-ui-auditor]] | específica UI | auditoria visual/a11y |
| [[card-gsd-ui-checker]] | específica UI | checagem funcional de UI |
| [[card-gsd-doc-verifier]] | docs | valida veracidade da documentação |

## 🎛 Orchestrator — coordena o pipeline

Papel: despachar tasks pros agents certos, manter STATE consistente, tratar checkpoints. No nosso pipeline vive em [[../orchestrator/_moc]].

| GSD agent | Afinidade | Insight aproveitável |
|---|---|---|
| [[card-gsd-intel-updater]] | secundária | atualiza STATE com decisões recém-tomadas pelos outros agents |
| [[card-gsd-debug-session-manager]] | secundária | orquestra ciclo de debug (isolamento + hipótese + teste) como subpipeline |

> Nota: GSD não tem um "orchestrator agent" explícito — o orquestrador é o **operador humano** ou comando slash (`/gsd-plan-phase`, `/gsd-execute-phase`). Nosso Orchestrator pode adotar esse mesmo modelo (não precisa de agent dedicado) ou virar uma skill.

## 🧩 Support — research & síntese prévia ao pipeline

Papel: alimentar planner/worker/reviewer com contexto confiável. Não é "papel" fixo do pipeline mas é insumo essencial.

| GSD agent | Afinidade | Para qual papel alimenta |
|---|---|---|
| [[card-gsd-advisor-researcher]] | research genérico | Planner |
| [[card-gsd-ai-researcher]] | research sobre estado-da-arte IA | Planner |
| [[card-gsd-assumptions-analyzer]] | quebra premissas ocultas | Planner, Reviewer |
| [[card-gsd-codebase-mapper]] | mapa do código existente | Worker (antes de editar) |
| [[card-gsd-domain-researcher]] | research de domínio de negócio | Planner |
| [[card-gsd-pattern-mapper]] | identifica patterns aplicáveis | Planner, Reviewer |
| [[card-gsd-phase-researcher]] | research fase-específica | Planner |
| [[card-gsd-project-researcher]] | history do projeto | Planner, Orchestrator |
| [[card-gsd-research-synthesizer]] | síntese N researchers | Planner (consome saída dos outros researchers) |
| [[card-gsd-user-profiler]] | contexto do usuário final | Planner |
| [[card-gsd-ui-researcher]] | research UI (TOC classifica em audit) | Planner UI |
| [[card-gsd-doc-classifier]] | triagem de docs pré-ingestão | Support |
| [[card-gsd-doc-synthesizer]] | síntese cross-doc | Support |
| [[card-gsd-debugger]] | investigação post-mortem | Worker (Rule 1 - bug auto-fix) e Reviewer (análise causal) |

## Padrões agregados (emergentes da biblioteca)

1. **Especialização por fonte, não por função genérica**. O GSD não tem "um researcher" — tem 10. Cada um é especializado em uma fonte/escopo (AI, domain, codebase, patterns, users...). Para nosso pipeline: resistir à tentação de ter um "Support" genérico; pensar em agents temáticos.
2. **Checker antes do Executor** ([[card-gsd-plan-checker]]). Valida o plano antes de gastar tokens/tempo executando. Nosso equivalente: Gate 0 do SDD + dual-check no plan.
3. **Synthesizer no fim do research** ([[card-gsd-research-synthesizer]]). Não é mais research — é agregador. Pattern: quando se usa múltiplos researchers paralelos, **sempre** há um synthesizer no fim.
4. **Session manager para operações longas** ([[card-gsd-debug-session-manager]]). Debugging não termina num único prompt. Pattern aplicável ao nosso Orchestrator.
5. **Auditors especializados, não 1 reviewer genérico**. Security, UI, Nyquist (sampling/coverage), Eval — cada um com *lens* própria. Para nosso Reviewer: manter abertura para reviewers temáticos (security review já existe como `security-review` skill no vault-raiz do Claude Code).
6. **Goal-backward methodology** (extraído de gsd-planner): derivar truths → artifacts → wiring → key-links. É equivalente ao `<verify>` declarativo do SDD + `must_haves` do PLAN.md. Aplicável imediatamente.

## Vizinhos

- [[_moc]] — MOC da gsd-library
- [[../_moc]] — MOC do pipeline 30-agents
- [[../planner/_moc]] · [[../worker/_moc]] · [[../reviewer/_moc]] · [[../orchestrator/_moc]]
- [[../playbooks/dual-check]] · [[../playbooks/research-loop]] · [[../playbooks/plan-and-execute]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
- [[../../10-knowledge/methodology/sdd-workflow]]
