---
title: Sprint 2 — Backend (Institutional base)
type: sprint-spec
tags: [master-sindico, sprint-2, backend, tasks]
project: master-sindico
stack: backend
sprint: 2
period: "2026-04-21 → 2026-04-27"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 2 Backend — Institutional base (Condomínio + Timeline + Plano Diretor)

**Período**: 2026-04-21 → 2026-04-27 (1 semana; overlap com refactor Sprint 3)
**Status**: ✅ concluído
**Objetivo**: Subir os blocos fundamentais do domínio institucional — Condomínio, Unit (com lacuna conhecida), Timeline append-only, Plano Diretor esqueleto — base para todos os fluxos de morador/síndico.

## Escopo desta sprint (backend)

Backend segue à frente. Web/mobile ainda sem entregas (não existiam). Timeline é append-only por design (INV-119, INV-120 reforçados depois via D-088). Unit foi entregue **sem** `unit_type` nem `fracao_ideal` — gap crítico identificado em Fase 7d e tratado no Sprint 10 backend (BE-001).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-2.1 | `Condominium` aggregate + CNPJ/endereço + ViaCEP lookup | ✅ | `internal/modules/institutional/domain/entities/condominium.go` | — | D-018 |
| backend-2.2 | `Membership` aggregate (user ↔ condomínio, role síndico/morador) | ✅ | `internal/modules/institutional/domain/entities/membership.go` | — | D-019 |
| backend-2.3 | `Unit` aggregate básico (sem `unit_type` nem `fracao_ideal` — GAP) | ✅ parcial | `internal/modules/institutional/domain/entities/unit.go` | — | D-020 |
| backend-2.4 | `TimelineEntry` append-only + INV immutable ao nível aplicação | ✅ | `internal/modules/institutional/domain/entities/timeline_entry.go` | — | D-021 |
| backend-2.5 | Repo sqlc para condominium/unit/membership/timeline | ✅ | `internal/modules/institutional/infrastructure/db/queries/*.sql` | A-021 | — |
| backend-2.6 | `RegisterMembershipUseCase` (strategy por role; ver F4) | ✅ | `internal/modules/institutional/application/use_cases/register_membership.go` | — | — |
| backend-2.7 | `CreateCondominiumUseCase` + onboarding síndico inicial | ✅ | idem pasta acima | — | — |
| backend-2.8 | `PlanoDiretor` esqueleto (atividades, kanban states) | ✅ | `internal/modules/institutional/domain/entities/master_plan*.go` | A-020 | — |
| backend-2.9 | Integration test testcontainers (F7, Sprint 9 retroativo) | ✅ retroativo | `internal/modules/institutional/.../condominium_test.go` | — | — |

## Dependências

- Sprint anterior: ABAC para ativar gates por tier/role ([[sprint-1]]).
- Cross-stack: nenhuma.
- Decisões: D-018..D-021; INV-119/120 vieram depois (Fase 9).

## Entregáveis

- `POST /api/v1/condominiums` síndico cria condomínio.
- `POST /api/v1/condominiums/:id/memberships` morador solicita vínculo.
- `GET /api/v1/condominiums/:id/timeline` lista append-only.
- `POST /api/v1/condominiums/:id/timeline` síndico registra entrada.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- Integration tests (testcontainers) institutional passando ✅ (validado retroativamente Sprint 9)

## Pós-sprint

- **O que deu certo**: append-only timeline funciona; membership flow simples suporta M1.
- **Bloqueadores**: Unit sem `unit_type`/`fracao_ideal` virou gap crítico — só foi identificado em Fase 7d (ADR-0032/D-056).
- **Dívidas criadas**: BE-001 (Unit gap crítico) — Sprint 10 backend.
- **Próxima sprint**: [[sprint-3|Sprint 3 — Institutional completa]].

> Nota histórica: executado em 21→27/04/2026 cronologicamente, mas o grosso do polish (F4 strategy pattern em register_membership, integration test F7) foi materializado na sessão autônoma 22/04. STATE/SESSION_CHARTER documenta honestamente.

## Links

- [[sprint-1]]
- [[sprint-3]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
