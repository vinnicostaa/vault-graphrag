---
title: Sprint 7 — Backend (Assembly foundation)
type: sprint-spec
tags: [master-sindico, sprint-7, backend, tasks]
project: master-sindico
stack: backend
sprint: 7
period: "2026-05-26 → 2026-06-01"
status: completed
milestone_target: m3
created: 2026-04-24
---

# Sprint 7 Backend — Assembly foundation

**Período**: 2026-05-26 → 2026-06-01 (retrospectivo)
**Status**: ✅ concluído
**Objetivo**: Estabelecer a fundação de assembleias — aggregates (Assembly, AgendaItem, Vote, Proxy, Minutes), state machine de votação, UNIQUE constraint resolvendo TOCTOU, Livekit SDK real com retry + reconciliação.

## Escopo desta sprint (backend)

M3 backend (julho) mas entregue adiantado. Livekit retry/reconciliação fecha A-033/A-034. Vote UNIQUE (`UNIQUE(agenda_item_id, voter_id)` em `00504_create_votes.sql`) resolve A-025. Fração ideal em `AgendaItem` usa `decimal.Decimal` (não float).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-7.1 | `Assembly` aggregate + state machine (planned→open→in-progress→closed→homologated) | ✅ | `internal/modules/assembly/domain/entities/assembly.go` | Q-020 | D-088, INV-118 |
| backend-7.2 | `AgendaItem` + tipo de quórum + fração ideal `decimal.Decimal` (não float) | ✅ | `internal/modules/assembly/domain/entities/agenda_item.go` | A-DC-PEN-006 | — |
| backend-7.3 | `Vote` + UNIQUE(agenda_item_id, voter_id) + `isUniqueViolation → ErrConflict` | ✅ | `internal/modules/assembly/infrastructure/db/migrations/00504_create_votes.sql` | A-025 | — |
| backend-7.4 | `Proxy` (procuração) aggregate + homologação escalonada D-053 | ✅ | `internal/modules/assembly/domain/entities/proxy.go` | A-DC-PEN-009 | D-053 |
| backend-7.5 | `Minutes` (ata) aggregate + imutabilidade pós-publish (INV-119) | ✅ | `internal/modules/assembly/domain/entities/minutes.go` | Q-020 | D-088 |
| backend-7.6 | Livekit SDK real + `retryLivekit[T]` backoff 100/300/900ms | ✅ | `internal/modules/assembly/infrastructure/providers/livekit/livekit_provider.go` | A-033, A-034 | commit `76d8e98` |
| backend-7.7 | `StartLiveSessionSaga` com compensação `EndRoom` em falha | ✅ | `internal/modules/assembly/application/use_cases/start_live_session.go` | A-028 | commit `c32ab5a` |
| backend-7.8 | Edital 8d CHECK constraint + validator `POST /assemblies` | 🟡 parcial Sprint 10 | — | Q-027 | D-088, INV-118 |
| backend-7.9 | PBT fração ideal AgendaItem (primeiro PBT do projeto — F6) | ✅ | `agenda_item_pbt_test.go` | — | — |

## Dependências

- Sprint anterior: infra + OTel ([[sprint-6]]).
- Cross-stack: nenhuma.
- Decisões: D-053 (homologação escalonada); D-088 (edital 8d + ata/timeline imutável DB); INV-118/119/120.

## Entregáveis

- `POST /api/v1/assemblies` síndico cria assembleia.
- `POST /api/v1/assemblies/:id/agenda-items` monta pauta.
- `POST /api/v1/agenda-items/:id/votes` voto idempotente.
- `POST /api/v1/assemblies/:id/live-sessions` inicia sessão Livekit.
- Ata gerada pós-close + homologação 4-eyes.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- PBT fração ideal + PBT vote count ✅
- A-025 (Assembly Vote TOCTOU) fechado ✅

## Pós-sprint

- **O que deu certo**: retry helper `retryLivekit[T]` genérico virou pattern reutilizável; Saga StartLiveSession + UploadVideo compartilham template.
- **Bloqueadores**: Edital 8d CHECK constraint + validator — pendente migration Sprint 10 (Q-027).
- **Dívidas criadas**: ata/timeline imutabilidade DB-level (INV-119/120) — migration Sprint 10.
- **Próxima sprint**: [[sprint-8|Sprint 8 — Security hardening]].

## Links

- [[sprint-6]]
- [[sprint-8]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
