---
title: "Output — Task-<N>: <título>"
type: note
tags: [pipeline-artifact, output]
task_id: <N>
role: worker
phase: output
status: in_progress
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
plan_ref: "`[[30-agents/planner/plans/<N>]]`"
commits: []
pointer_to_next: "`[[30-agents/_inbox/reviewer/<N>]]`"
---

# Output — Task-<N>: <título>

Artefato de execução gerado pelo role **Worker**. Referencia o plan oficial via `plan_ref`. Reviewer (Fase 2 do pipeline) consome este arquivo via ponteiro em `30-agents/_inbox/reviewer/<N>.md`.

## Resumo

<1-2 parágrafos do que foi feito, evidenciado pelos commits abaixo>

## Execução por Task do plan

### Task 1 — <nome do plan>

**Arquivos modificados**:
- `<path>` — `<mudança>`

**Commits**:
- `<sha>` — `<mensagem>`

**Gates verificados**:
- `go build ./...` → OK
- `go vet ./...` → OK
- `go test -race -count=1 ./<pacote>/...` → `<N>` passa
- Coverage: `<camada>: <pct>%` (target `<threshold>%`)
- `gosec -severity=high` → 0 findings
- `govulncheck` → 0 CVEs reachable

**Done criteria**: ✅ | ⚠️ | ❌ `<nota>`

### Task 2 — ...

### Task 3 — ...

## Desvios do plan (deviation rules do `/task-executor`)

- [ ] **Rule 1 (auto-fix bug)**: nenhum | `<descrição>` + commit `<sha>`
- [ ] **Rule 2 (auto-add missing critical)**: nenhum | `<descrição>`
- [ ] **Rule 3 (auto-fix blocking)**: nenhum | `<descrição>`
- [ ] **Rule 4 (architectural → parar)**: não disparou | PAUSOU em `<ponto>`, decisão necessária

Ver [[../../30-agents/autonomous-execution]] para definição de Rules.

## Must-haves atingidos

- [ ] Truth 1: <verificado como>
- [ ] Truth 2: <verificado como>
- [ ] Artefato 1: `<path>` existe com conteúdo esperado
- [ ] Key link 1: `<from>` → `<to>` testado em integração

## Dívidas descobertas (registrar)

- **DT-### (sugerida)**: `<descrição da dívida>` — gravar em `<projeto>/STATE.md` como DT.
- **A-### (sugerida)**: `<finding em AUDIT>` — gravar em `<projeto>/AUDIT.md` se severidade ≥ médio.

## Handoff para Reviewer

Ao marcar `status: completed`:
1. Escrever estado final neste arquivo (commits, gates, must-haves, dívidas).
2. Criar ponteiro em `30-agents/_inbox/reviewer/<N>.md` (só frontmatter + link canônico para este output) — **em segundo lugar** (ordem canônica do ajuste #9).

## Links

- [[_moc]]
- [[../../30-agents/worker/_moc]]
- [[../../30-agents/playbooks/plan-and-execute]]
- [[../../10-knowledge/methodology/sdd-workflow]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
