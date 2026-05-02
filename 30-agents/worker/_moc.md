---
title: MOC — Worker (Fase 1 instrumentada)
type: moc
tags: [moc, agents, worker, pipeline]
created: 2026-04-24
updated: 2026-04-24
---

# Worker — pipeline operacional (Fase 1)

Pasta operacional do role **Worker**. Instrumentada em D-vault-015.a (Fase 1).

## Papel

Agente que **executa o plan produzindo código/artefato**:

- Lê ponteiro em `30-agents/_inbox/worker/<task-id>.md` → resolve para `planner/plans/<task-id>.md`
- Segue ordem canônica SDD ([[../../10-knowledge/methodology/sdd-workflow]]): migration → codegen → domain → application → infra → routes → wire-up → testes
- Commits atômicos por task
- Aplica deviation rules 1-3 (auto-fix bugs, auto-add missing critical, auto-fix blocking) — ver [[../autonomous-execution#Antipatterns que quebram autonomous mode]]
- Rule 4 (architectural change) → **pausar** + escalar
- Produz **output** em `outputs/<task-id>.md` usando [[../../40-templates/pipeline/output-template]]
- Mantém `active-task.md` atualizado (single file, current state)
- Escreve **ponteiro** em `_inbox/reviewer/<task-id>.md` ao marcar `status: completed`

## Sub-pastas operacionais

- `outputs/<task-id>.md` — **estado oficial de execução** (commits, gates, deviation log, must-haves)
- `active-task.md` — **single file** com task atualmente em execução; overwrite a cada nova task

## Regras duras

1. **Um Worker ativo por task-id** (ajuste #8). `active-task.md` tem 1 task; nova task só após commit final da anterior.
2. **Ordem canônica de handoff** (ajuste #9):
   - (a) Escrever `outputs/<task-id>.md` com `status: completed` + todos os dados finais.
   - (b) Atualizar `active-task.md` (zera ou aponta para próxima).
   - (c) Escrever `_inbox/reviewer/<task-id>.md` (ponteiro para o output).
3. **Path canônico obrigatório**. Output fora de `worker/outputs/` é ignorado.
4. **Zero lógica de negócio em deviation Rule 4** (architectural). Sempre pausa para humano — não decide arquitetura.
5. **Gates todos verdes** antes de marcar `completed`: `go build`, `go vet`, `go test -race`, coverage threshold, `gosec`, `govulncheck`.

## Referência externa (biblioteca GSD)

[[../gsd-library/family-executor]] — 4 agents desta família (executor, code-fixer, code-reviewer, integration-checker). Nosso Worker **consolida** a essência; reviewer temáticos delegados para role Reviewer (Fase 2).

## Crash recovery

Se Worker crasha durante execução:
- `active-task.md` fica stale (sem `status: completed`).
- Próxima sessão: ler `active-task.md`; se `updated` > 2h, considerar task abandonada → registrar em AUDIT + re-planejar ou retomar.

## Status

🟢 **Fase 1 instrumentada** (D-vault-015.a). Execução depende de skill `/task-executor` + conhecimento do ciclo SDD.

## Vizinhos

- [[../_moc]]
- [[../_inbox/_moc]] — fila de entrada (lê ponteiro; estado em `planner/plans/`)
- [[../planner/_moc]] — origem do plan
- [[../reviewer/_moc]] — destino do handoff ao completar
- [[../gsd-library/family-executor]]
- [[../../10-knowledge/methodology/sdd-workflow]]
- [[../../10-knowledge/methodology/gsd-get-shit-done]]
- [[../../40-templates/pipeline/output-template]]
- [[../../40-templates/pipeline/_moc|pipeline templates]]
- [[../../00-meta/STATE]] — D-vault-015.a
