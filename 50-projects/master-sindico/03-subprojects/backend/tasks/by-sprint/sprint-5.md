---
title: Sprint 5 — Backend (Content — Video Mux)
type: sprint-spec
tags: [master-sindico, sprint-5, backend, tasks]
project: master-sindico
stack: backend
sprint: 5
period: "2026-05-12 → 2026-05-18"
status: completed
milestone_target: m2
created: 2026-04-24
---

# Sprint 5 Backend — Content (Video Mux + Agências)

**Período**: 2026-05-12 → 2026-05-18 (retrospectivo — hardening Saga 22/04)
**Status**: ✅ concluído
**Objetivo**: Entregar o módulo content — upload de vídeos via Mux (signed URL), agências de marketing como entidade, paywall 25% preview, visibilidade condicional (votação fornecedor ativa), webhook Mux público.

## Escopo desta sprint (backend)

Content é a fundação de conteúdo de M2 (institucionais) + M3 (banco de talentos). Upload Mux precisa Saga pra garantir que o asset órfão seja compensado se o Create do DB falhar (A-027, fechado via commit `c32ab5a`). Webhook público (`/webhooks/mux` fora de `/api/v1` — A-022 fechado).

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| backend-5.1 | `Video` aggregate + states (`uploading→processing→ready→blocked`) | ✅ | `internal/modules/content/domain/entities/video.go` | — | D-029 |
| backend-5.2 | Mux SDK integration (`muxinc/mux-go`) — upload ticket + direct upload + thumbnail | ✅ | `internal/modules/content/infrastructure/providers/mux/mux_provider.go` | — | D-029 |
| backend-5.3 | Webhook Mux HMAC-SHA256 antes de parse + rota pública `/webhooks/mux` | ✅ | `internal/modules/content/infrastructure/http/routes/mux_webhook_handler.go` | A-022 | — |
| backend-5.4 | `UploadVideoSaga` com compensação `CancelUpload` em falha (A-027) | ✅ | `internal/modules/content/application/use_cases/upload_video.go` | A-027 | commit `c32ab5a` |
| backend-5.5 | `MarketingAgency` aggregate + delegation para Empresa (switch contexto) | ✅ | `internal/modules/commercial/domain/entities/marketing_agency.go` | — | — |
| backend-5.6 | Paywall 25% preview (signed URL TTL curto p/ preview + full em vote window) | ✅ | `internal/modules/content/application/use_cases/get_signed_playback.go` | — | — |
| backend-5.7 | Visibilidade gate: vídeo empresa só público durante votação fornecedor ativa | ✅ | `internal/modules/content/application/use_cases/check_video_access.go` | — | D-030 |
| backend-5.8 | Lock 90d em vídeo empresa + bypass 4-eyes (D-054) | ✅ parcial | idem acima | — | D-054 |

## Dependências

- Sprint anterior: commercial ([[sprint-4]]) para gate votação fornecedor.
- Cross-stack: nenhuma.
- Decisões: D-029, D-030; D-054 (lock 90d bypass) adicionado retroativo na Fase 5.

## Entregáveis

- `POST /api/v1/videos/upload-ticket` retorna signed upload URL Mux.
- `POST /webhooks/mux` processa `video.asset.*` idempotente.
- `GET /api/v1/videos/:id/playback` retorna HLS signed URL (TTL ≤ 1h).
- Agência pode gerenciar N empresas via delegation.

## Gates (`<verify>`)

- `go build ./... && go vet ./... && go test -race -count=1 ./...` ✅
- Saga A-027/A-028 testadas com falha simulada Mux provider ✅

## Pós-sprint

- **O que deu certo**: Saga UploadVideo + StartLiveSession compartilham template genérico; HMAC before parse reforçou pattern.
- **Bloqueadores**: nenhum crítico.
- **Dívidas criadas**: P-APP-005 SSRF Mux thumbnail custom URL (A-DC-PEN-007) — Sprint 10.
- **Próxima sprint**: [[sprint-6|Sprint 6 — Infra]].

## Links

- [[sprint-4]]
- [[sprint-6]]
- [[../../../../_moc]]
- [[../../../../STATE]]
- [[../../../11-audit/AUDIT]]
