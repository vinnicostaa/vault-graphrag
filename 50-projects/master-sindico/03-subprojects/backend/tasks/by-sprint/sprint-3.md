---
title: Sprint 3 — Backend (Institutional completa)
type: sprint-spec
tags: [master-sindico, sprint-3, backend, tasks]
project: master-sindico
stack: backend
sprint: 3
period: "2026-04-28 → 2026-05-04"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 3 Backend — Institutional completa

**Período**: 2026-04-28 → 2026-05-04 (1 semana)
**Status**: ✅ concluído
**Objetivo**: Fechar o módulo institutional com Events, Comunicados, Polls (enquetes) e entidades de Compliance (dossiê, avaliação obrigatória, documentos) — tudo o que o M1 de morador precisa para ter conteúdo real além de timeline.

## Escopo desta sprint (backend)

Backend segue solo. Sprint denso de 7 novas aggregates. Polls ficam com `soft_close` agendado por cron (A-FASE10-DEV-007 — cron scheduler não existe ainda). Compliance documents tem `RetentionPolicy` (LGPD 5y) + soft-delete.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-3.1 | `Event` aggregate + `EventResponse` (ciência/confirma presença) | ✅ | `internal/modules/institutional/domain/entities/event*.go` | A-031 (falso positivo) | D-022 |
| backend-3.2 | `Announcement` (Comunicados) + `AnnouncementRead` tracking | ✅ | `internal/modules/institutional/domain/entities/announcement*.go` | — | D-023 |
| backend-3.3 | `Poll` (enquetes) + `PollOption` + `PollResponse` + `MarkExpired` cron-hook | ✅ | `internal/modules/institutional/domain/entities/poll*.go` | A-FASE10-DEV-007 | D-024 |
| backend-3.4 | `Compliance.Document` aggregate + RetentionPolicy LGPD 5y | ✅ | `internal/modules/compliance/domain/entities/document.go` | — | D-025 |
| backend-3.5 | `Compliance.AuditLogEntry` + `CreateAuditLog` persistência (BC-only) | ✅ | `internal/modules/compliance/application/.../compliance_use_cases.go` | A-FASE10-DEV-008 (DT-010) | DT-010 |
| backend-3.6 | `ManagementEvaluation` (avaliação obrigatória do morador; 3 perguntas 1-10 anônima) | ✅ | `internal/modules/institutional/domain/entities/management_evaluation.go` | — | D-026 |
| backend-3.7 | Handlers HTTP + testes (happy + 401/403/409) p/ todas as 7 aggregates | ✅ | `internal/modules/institutional/infrastructure/http/routes/` | — | — |
| backend-3.8 | `UpsertEventResponse` atômico `INSERT ... ON CONFLICT DO UPDATE` (resolve TOCTOU claim) | ✅ | `internal/modules/institutional/.../event_response_repository.go` | A-031 | — |

## Dependências

- Sprint anterior: Condomínio/Membership/Timeline ([[sprint-2]]).
- Cross-stack: nenhuma.
- Decisões: D-022..D-026; `IAuditLogger` cross-BC (DT-010) registrado para tratamento Sprint 10.

## Entregáveis

- 7 aggregates institucionais com CRUD + listagem paginada + ABAC.
- `GET /api/v1/condominiums/:id/events|announcements|polls` paginado.
- Avaliação obrigatória como gate (morador não pode acessar outras features até avaliar — regra aplicada em middleware `MandatoryEvaluationGate`).
- Audit log de compliance persistido (parcial — apenas BC compliance).

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- Coverage institutional ≥ 85% domain, ≥ 90% application ✅

## Pós-sprint

- **O que deu certo**: reuso de patterns (sqlc + ABAC gates + handler smoke tests) acelerou; Upsert atômico virou padrão.
- **Bloqueadores**: cron scheduler inexistente — `MarkExpired` de Poll depende de trigger externo (A-FASE10-DEV-007).
- **Dívidas criadas**: DT-010 (IAuditLogger precisa expandir para billing/commercial/identity — Sprint 10 BE-004).
- **Próxima sprint**: [[sprint-4|Sprint 4 — Commercial]].

## Links

- [[sprint-2]]
- [[sprint-4]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
