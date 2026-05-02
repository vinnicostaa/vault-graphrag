---
title: "Stack Atual + Bounded Contexts + Módulos do Produto"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: architecture
status: stable
---

# Stack Atual + Bounded Contexts + Módulos

Snapshot autoritativo do que está vivo em 2026-04-22. **Quando o `product-overview.md` ou os documentos do `Content/contents/` divergirem desta página, esta página vence** — porque ela reflete `web/CLAUDE.md`, `backend/CLAUDE.md`, `SESSION_CHARTER`, `AUDIT.md` e o código real.

---

## 1. Stack tecnológica

### Backend (Go)
| Camada | Tecnologia |
|---|---|
| Linguagem | Go 1.26 |
| Framework HTTP | Gin (abstraído via `contracts.HTTPRouter` + `contracts.Context`) |
| DB | PostgreSQL 18 + pgx/v5 + sqlc v1.30 (sem ORM) |
| Migrations | goose v3 via `embed.FS` |
| Cache | Redis 7 |
| Mensageria | NATS JetStream |
| Auth/IAM | Zitadel v4 (OIDC PKCE + introspection) |
| Billing | Stripe (`IPaymentGateway`) |
| Vídeo VOD | Mux (`IVideoProvider`) — webhook HMAC público |
| Vídeo Live | Livekit (`ILiveProvider`) |
| Email | **Resend** (`IEmailProvider`) — não AWS SES |
| SMS | Twilio (planejado, Sprint 2+) |
| Storage | MinIO (dev) + Cloudflare R2 ou S3 (prod) — D-029 ainda aberta |
| Busca | **PostgreSQL `tsvector`** (não OpenSearch) |
| Logging | zerolog wrapper |
| Observabilidade | OTel + Prometheus bridge — entregue Sprint 9 |

### Web (Bun workspaces — 6 apps + 2 packages)
SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS (com `prefix: "un-"`) · Kobalte · Axios · Zod · Biome.

| App | Porta | Função |
|---|---|---|
| auth | 3000 | PKCE entry point |
| cms | 3001 | Painel principal (síndico, empresa, morador) |
| lms | 3002 | Academia (pós-lançamento) |
| forum | 3003 | Fórum / Vizinhança |
| assembly | 3004 | Assembleia ao vivo |
| lp | 3005 | Landing page |

Packages: `ui` (Kobalte + CVA + UnoCSS shortcuts), `schemas` (Zod compartilhado).

### Mobile (Flutter)
- Flutter/Dart (FVM)
- Clean Architecture + Feature First
- DI: `get_it` + `injectable`
- Router: `go_router`
- Tokens: `flutter_secure_storage`
- Auth alvo: Zitadel PKCE
- Backend recebe `Authorization: Bearer`

### Deploy
- **Railway** (primário) — project ID `efe664b7-5352-48e7-ae09-78d2f286e0d5`
- AWS é plano futuro registrado em `design/deploy-config.md`

### CI
GitHub Actions (compilation + test + coverage + gosec + govulncheck).

---

## 2. Bounded Contexts (Backend Go)

| Módulo Go | Faixa migrations | Sprint origem | Status | Tabelas-chave |
|---|---|---|---|---|
| `identity` | 001-099 | 0-1 | ✅ produção | users, sessions, subscriptions, plans, feature_usages |
| `billing` | 100-199 | 1 | ✅ produção | subscriptions (FK identity), feature_usages, plans, plan_features |
| `institutional` | 200-299 | 2-3 | ✅ produção | condominiums, units, mandates, syndic_companies, timeline_entries, master_plan_actions, events, communications, surveys, governance_assessments |
| `commercial` | 300-399 | 4 | ✅ produção | companies, company_profiles, connect_me_requests, proposals, deals, execution_records, technical_activities, amendments, company_evaluations, neighborhood_partnerships, neighborhood_benefits |
| `content` | 400-499 | 5 | ✅ produção | videos, video_analytics, video_visibility_grants, marketing_agency_tokens, courses, lessons, certificates, forum_topics, ebooks |
| `assembly` | 500-599 | 7 | ✅ produção | assemblies, assembly_agendas, assembly_agenda_items, assembly_fractions, assembly_votings, assembly_votes, assembly_check_ins, assembly_speech_queue, proxies, assembly_homologations |

### Cross-cutting (não bounded contexts próprios — `internal/shared/`)
- `internal/shared/middleware/` — Recovery, RequestID, Logger, CORS, ErrorHandler, RateLimiter, Authenticate, RequirePermission (ABAC engine)
- `internal/shared/types/` — `AuthUser`, `IUserSyncer`, `SyncUserCommand/Output` (cruza bounded contexts sem violar dependency rule)
- `internal/shared/providers/` — database (pgx pool + goose), redis, nats, email, sms, storage, video, payment
- `internal/core/contracts/` — HTTPRouter, Context, HandlerFunc, IUseCase, IUnitOfWork, IDomainEventPublisher
- `internal/core/errors/` — AppError sentinels (ErrNotFound, ErrConflict, ErrValidation, ErrForbidden, ErrInternal, ErrBusiness)
- `pkg/safecast/` — Int32, Int16, Int32FromInt64 com clamp (criado em F10)
- `pkg/utils/id.go` — UUIDv7

---

## 3. Módulos do produto (visão de negócio — NÃO confundir com bounded contexts Go)

Os módulos do PRODUTO (apresentados ao usuário) **não mapeiam 1:1** com os bounded contexts Go. Tabela cruzada:

| Módulo do produto | Bounded Context Go primário | Bounded Contexts auxiliares |
|---|---|---|
| **Governança** (S1-S31, M1-M15) | institutional | content, identity |
| **Linha do Tempo** | institutional | (eventos NATS de commercial, assembly) |
| **Plano Diretor** | institutional | — |
| **Eventos** | institutional | — |
| **Comunicados** | institutional | — |
| **Connect Me** (4 direções) | commercial | identity (LGPD log) |
| **Propostas** | commercial | — |
| **Negócio Fechado** | commercial | institutional (auto-publish na timeline) |
| **Registro de Execução** | commercial | institutional (validação síndico → timeline) |
| **Validações Pendentes** (hub) | institutional | commercial, content |
| **Compliance** (C1-C11) | cross-domain (cross-cutting) | identity (audit), commercial (LGPD), institutional (timeline) |
| **Vizinhança** (4 jornadas) | commercial (`neighborhood_*`) | identity (Parceiro como persona) |
| **Banco de Talentos** | content (vídeo-currículo) + commercial (ficha profissional) | identity (Morador Pagante) |
| **Academia / LMS** | content | identity (gating por persona) |
| **Assembleia** (Live) | assembly | content (vídeo institucional na votação), commercial (propostas validadas), institutional (auto-publish ata) |
| **Painel Admin Master Síndico** | (a criar — não tem BC dedicado) | todos |
| **Marketing Link** | commercial | content (vídeos da empresa via agência) |

---

## 4. Cadeia de middleware BeyondCorp (fixa)

```
Recovery → RequestID → Logger → CORS → ErrorHandler
  → [rotas protegidas em /api/v1]:
      → RateLimiter (por IP + userId)
      → Authenticate (JWT → Zitadel introspection → cache Redis 5min → user sync → 1-device check)
      → RequirePermission(action, resource)  ← ABAC engine próprio
  → Handler (contracts.HandlerFunc, NÃO gin.HandlerFunc)
  → AuditLogger (DT-010 — Sprint 10)
```

### Rotas FORA de `/api/v1` (sem Authenticate)
- `GET /auth/login`, `GET /auth/callback`, `GET /auth/logout` — redirects browser
- `POST /webhooks/stripe` — HMAC validation
- `POST /webhooks/mux` — HMAC validation (movido de /api/v1 em F22 — A-022)

---

## 5. Padrões obrigatórios (memória `feedback_padroes_engenharia_exigidos`)

- **DDD** — bounded contexts, agregados (root), entities, value objects (imutáveis), domain events
- **Clean Architecture** — `infrastructure → application → domain` (dependency rule)
- **CQRS** — Commands ≠ Queries (arquivos separados)
- **SOLID** — todos os 5 princípios
- **TDD** — testes RED antes de código
- **SDD** — spec antes de código (`.kiro/specs/`); pular fases gera retrabalho
- **ACID** — UnitOfWork (`uow.Run(ctx, fn)`) para multi-write atômico
- **Saga** — para fluxos cross-resource externo (Mux upload, Livekit room) com compensação
- **Code Calisthenics** — 9 regras adaptadas
  - Proibido bloco `else` (memória `feedback_no_else`)
  - 1 nível de indent por função
  - Branded types para primitivos
  - First-class collections
  - 1 ponto por linha (Demeter)
  - Sem abreviações (`subscription`, não `sub`)
  - Componentes pequenos, máx 3-5 props
  - Sem getters/setters cegos

---

## 6. Padrões de transação

### UnitOfWork (Tx via `context.Context`)
```go
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    quota, err := uc.quotaRepository.FindByUserIDForUpdate(ctx, userID, feature, period)
    if err != nil { return err }
    if !quota.HasRemaining() { return apperrors.ErrBusiness.WithMessage("quota esgotada") }
    quota.Consume()
    return uc.quotaRepository.Save(ctx, quota)
})
```

### SELECT FOR UPDATE
Para concorrência crítica (votação, quotas, contadores). Ver A-026 (Commercial Vote contador) e A-030 (Commercial Vote session status).

### Outbox + Domain Events
Tabela `domain_events` commitada na mesma tx do estado; consumer NATS publica. Listeners idempotentes.

### Saga (compensação)
Para SDKs externos com side-effect (Mux Upload, Livekit Room). Padrão: try → register → compensate on failure (CancelDirectUpload / EndRoom). Exemplo em A-027 / A-028.

---

## 7. Decisões técnicas resolvidas (T1-T15)

Ver [[../regras-de-negocio/regras-canonicas]] §5 para tabela completa. Resumo:
- T1: AuthUser em `shared/types/`
- T2: Gin abstraído via contracts
- T3: AppError com sentinels + `errors.Is/As`
- T4: Tx em `context.Context`
- T5: goose + embed.FS
- T6: zerolog interface (sem SetGlobalLevel)
- T7: roles em **inglês** (Zitadel)
- T8: role_types **só no banco** (não no Zitadel)
- T9: Cookie `Domain=.mastersindico.com.br`, httpOnly, Secure, SameSite=Lax
- T10: Money `int64` centavos; fração ideal `NUMERIC(5,4)`
- T11: OIDC PKCE via `zitadel/oidc/v3`
- T12: `/auth/*` fora de `/api/v1`
- T13: Cache Redis introspection TTL 5min
- T14: `Authorization: Bearer` mobile fallback
- T15: Guard `403 NO_ROLE_ASSIGNED` antes do UPSERT

---

## 8. Stack descartada (lições do legacy `full-stack/`)

Documentada em `backend/.kiro/specs/master-sindico/reference/legacy-summary.md` + `antipatterns-to-avoid.md`. Resumo:

| Stack original | Por que descartado | Substituto atual |
|---|---|---|
| TypeScript + Fastify + Drizzle | Drizzle v1-beta instável; type drift | Go + Gin abstraído + sqlc |
| CASL ABAC | Complexidade desnecessária | Engine ABAC próprio |
| Awilix DI | Boilerplate + runtime overhead | DI manual em `main.go` (compile-time, type-safe) |
| JWT próprio + Argon2 + Arctic OAuth | Reinventar auth é antipattern | Zitadel cloud (MFA, reset, OAuth Google/Apple, introspection) |
| Mercado Pago (sugestão original) | Cliente preferiu Stripe (Stripe.com BR) | Stripe |
| AWS SES (sugestão original) | Custo + complexidade de setup BR | Resend |
| AWS ECS Fargate (sugestão original) | Complexidade de IaC vs Railway | Railway (primário) |
| OpenSearch | Operacionalmente caro para MVP | PostgreSQL `tsvector` |

> **Nota Q19-Q22**: requirements.md monolítico (Mar/2026) ainda referencia roles em pt-BR, Mercado Pago, AWS SES, OpenSearch. As specs vivas em `backend/.kiro/specs/master-sindico/requirements/*.md` consolidam as decisões finais. Ver [[../regras-de-negocio/quebras-detectadas]].

---

## 9. Marcos de entrega

| Marco | Data | Escopo |
|---|---|---|
| **Marco 1** | **2026-05-08** | Identity + Billing + Institutional I (MVP contratual) |
| **Marco 2** | **2026-06-20** | Commercial + Content (Connect Me, vídeos, marketplace, busca) |
| **Marco 3** | **2026-07-20** | Assembly + Academia LMS base + Polish |

Ver [[../regras-de-negocio/regras-canonicas]] §9.

---

## 10. Sprint atual

**Sprint 9 — Integrations** (iniciado 2026-04-22). Sprints 0-8 backend completos. 33 features fechadas (F1-F33). Sprint 10 pendente (Saga A-027/A-028, Zitadel managed em Railway 9.19, IAuditLogger DT-010, frontend SolidJS Sprint 1-8 telas, Flutter paridade).

Ver `backend/.kiro/SESSION_CHARTER.md` §10 para snapshot detalhado.

---

## Referências

- `backend/CLAUDE.md` — guia operacional do agente
- `backend/AGENTS.md` — contrato completo
- `backend/.kiro/steering/sdd-workflow.md` — ciclo 5-fase
- `backend/.kiro/steering/sdd-layers.md` — Domain → Application → Infrastructure
- `backend/.kiro/STATE.md` — D-0XX, DT-0XX vivos
- `backend/.kiro/AUDIT.md` — A-0XX abertos/resolvidos
- [[../regras-de-negocio/regras-canonicas]] — 10 regras + 12 decisões + T1-T15
- [[../02-Jornadas/Inventario-Telas]] — produto end-user
