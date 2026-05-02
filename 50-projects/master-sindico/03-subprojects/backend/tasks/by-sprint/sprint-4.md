---
title: Sprint 4 — Backend (Commercial + Connect Me)
type: sprint-spec
tags: [master-sindico, sprint-4, backend, tasks]
project: master-sindico
stack: backend
sprint: 4
period: "2026-05-05 → 2026-05-11"
status: completed
milestone_target: m2
created: 2026-04-24
---

# Sprint 4 Backend — Commercial (Connect Me + Company + Proposals)

**Período**: 2026-05-05 → 2026-05-11 (retrospectivo — materializado em sessão autônoma 22/04)
**Status**: ✅ concluído
**Objetivo**: Subir o módulo commercial com Connect Me (4 fluxos), Company, Proposals + Votação de Fornecedor. Base do marco M2 (Connect Me + Conteúdo).

## Escopo desta sprint (backend)

Entrega a infraestrutura de marketplace B2B2C. Connect Me tem 4 direções (Síndico→Empresa, Morador→Síndico, Empresa→Empresa Plus/Pro, Síndico→Parceiro). `ConnectMeRequest.MarkExpired` depende de cron noturno (A-FASE10-DEV-007). Votação de fornecedor é onde ficou a race TOCTOU A-030 (fechada).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-4.1 | `Company` aggregate + profile pública + `ActivationStatus` | ✅ | `internal/modules/commercial/domain/entities/company.go` | — | D-027 |
| backend-4.2 | `ConnectMeRequest` + 4 direções + LGPD consent version | ✅ | `internal/modules/commercial/domain/entities/connect_me_request.go` | A-FASE10-DEV-007 | D-028 |
| backend-4.3 | `Proposal` aggregate + states (rascunho→enviado→aprovado→publicado→bloqueado) | ✅ | `internal/modules/commercial/domain/entities/proposal.go` | P-INV-006 | — |
| backend-4.4 | `SupplierVoteSession` + `SupplierVote` + UNIQUE constraint + `FindSessionByIDForUpdate` | ✅ | `internal/modules/commercial/domain/entities/supplier_vote*.go` | A-026, A-030 | — |
| backend-4.5 | `CastVoteUseCase` dentro de `unitOfWork.Run` (resolve race) | ✅ | `internal/modules/commercial/application/use_cases/cast_vote.go` | A-026 | — |
| backend-4.6 | `CompanyEvaluation` + UNIQUE(condominium_id, company_id, evaluator_id) + 409 correto | ✅ | `internal/modules/commercial/domain/entities/company_evaluation.go` | A-029 | — |
| backend-4.7 | Quotas Connect Me por tier/role (snapshot via `FeatureUsage`) | ✅ | `internal/modules/billing/.../feature_usage.go` | A-FASE8-FU-001 | D-017 |
| backend-4.8 | `Deal` + 4-eyes policy signing (parcial M1; completar M2) | ✅ parcial | `internal/modules/commercial/domain/entities/deal.go` | — | D-054 |

## Dependências

- Sprint anterior: institutional completo ([[sprint-3]]); ABAC + quotas ([[sprint-1]]).
- Cross-stack: nenhuma.
- Decisões: D-027/D-028; INV-032 (UNIQUE supplier_vote).

## Entregáveis

- `POST /api/v1/connect-me` síndico cria solicitação (quota checked).
- `POST /api/v1/supplier-vote-sessions/:id/votes` voto fornecedor idempotente.
- `POST /api/v1/company-evaluations` avaliação com 409 se duplicata.
- Marketplace listagem por cidade/categoria (index invertido placeholder, OpenSearch em Sprint 6+).

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- Race conditions fechadas: A-025, A-026, A-029, A-030 ✅

## Pós-sprint

- **O que deu certo**: UNIQUE constraint + `FindForUpdate` virou pattern para qualquer contador; padrão repetido em Sprint 7 (assembly vote).
- **Bloqueadores**: cron scheduler (MarkExpired) continua pendente.
- **Dívidas criadas**: motor de Reputação Bronze→Diamante (A-FASE10-DEV-006) — M2.
- **Próxima sprint**: [[sprint-5|Sprint 5 — Content (Video Mux)]].

> Nota histórica: sprint executado retrospectivamente (as entidades existiam de prompts anteriores; o hardening e resolução de races A-025..A-030 foram materializados na sessão autônoma 22/04 — ver [[../../../SESSION_CHARTER#F21-F22]]).

## Links

- [[sprint-3]]
- [[sprint-5]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
