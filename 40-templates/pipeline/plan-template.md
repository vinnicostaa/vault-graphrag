---
title: "<Título descritivo da task>"
type: note
tags: [pipeline-artifact, plan]
task_id: <N>
role: planner
phase: plan
status: pending
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
pointer_to_next: "`[[30-agents/_inbox/worker/<N>]]`"
requirements_refs: []
---

# Plan — Task-<N>: <título>

Plano canônico gerado pelo role **Planner**. Este artefato é **source of truth** da decomposição da task. Worker consome este arquivo via ponteiro em `30-agents/_inbox/worker/<N>.md`.

## Contexto

<1-2 parágrafos: o problema, o requisito/briefing, decisões prévias relevantes>

## Fontes consultadas (research-loop)

- `[[vault_ref_1]]` — o que foi extraído
- Context7: `<lib>@<versão>` topic `<topic>` — data da consulta
- Engineering blog / docs oficial: <URL> + data

(seguir [[../../30-agents/playbooks/research-loop]] nas 5 fontes)

## Decisão de plano

<o que será feito, resumo executivo em 3-5 linhas>

## Tasks (2-3 — GSD shipping small)

### Task 1 — <nome curto>

**Arquivos** (create/modify/not-touch):
- `<path>` — `<motivo>`

**Ação específica**: <descrição do que fazer, não como>

**Gates esperados** (`<verify>` declarativo, ver [[../../10-knowledge/methodology/sdd-workflow]]):
- Build: `<comando>` passa
- Lint/vet: `<comando>` sem warning
- Testes: `<N>` testes novos, verdes com `-race`
- Coverage: ≥ `<threshold>` em camada `<x>`
- Security: `gosec -severity=high` 0 findings

**Done quando**: <critério de aceite observável>

### Task 2 — ...

### Task 3 — ...

## Must-haves (goal-backward)

- **Truths observáveis**:
  - Usuário pode <X>
  - <invariante preservado>
- **Artefatos criados**:
  - `<path>`: `<provê X>`
- **Key links** (pontos onde mais provavelmente quebra):
  - `<from>` → `<to>` via `<padrão>`

## Dependências de outros plans

- `task_id: <M>` — bloqueante / paralelo / não-dep

## Registro de decisão

Se este plan envolve decisão não-trivial (escolha de lib, pattern, contrato público): registrar em `<projeto>/STATE.md` como D-### com fontes + rationale + data alvo de reavaliação. Ver [[../../30-agents/playbooks/research-loop]].

## Links

- [[_moc]]
- [[../../30-agents/planner/_moc]]
- [[../../30-agents/playbooks/plan-and-execute]]
- [[../../10-knowledge/methodology/sdd-workflow]]
