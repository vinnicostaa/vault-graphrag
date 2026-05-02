---
title: "Regras Canônicas — Master Síndico"
updated: 2026-04-22
maintainer: agent-3 (analista)
type: canon
status: stable
source: backend/.kiro/specs/master-sindico/requirements/product-overview.md + personas-and-plans.md + functional-matrix.md
---

# Regras Canônicas — Master Síndico

Consolidação cross-domínio. **Quando algo aqui divergir da spec viva em `backend/.kiro/`, a spec viva vence** — esta página replica para ter um único ponto de leitura no vault Obsidian. Atualizar quando spec mudar.

---

## 1. Visão de produto

Plataforma SaaS de **governança condominial** que conecta síndicos, moradores, empresas, agências de marketing e comércio local. Não é ERP, não é app de comunicação, não é sistema financeiro. É **governança aplicada** com 4 pilares:

- **Critério** — contratações por método, não urgência
- **Registro** — memória institucional auditável
- **Reputação** — sistema de status baseado em evidência (Bronze → Diamante)
- **Integração** — ecossistema unificado de perfis, vídeos, cursos e contato

Identidade é única (1 CPF/CNPJ); contextos de operação são múltiplos.

---

## 2. As 10 Regras de Ouro (invioláveis)

Aplicam-se a **todo** código, frontend, mobile e spec.

| # | Regra | Implicação prática |
|---|---|---|
| 1 | Separação de identidade — nunca misture User, Profile e Subscription | Bounded contexts: `identity` ≠ `institutional` ≠ `billing`. Relação por ID, não por ponteiro. |
| 2 | Imutabilidade — Timeline, Negócio Fechado, votos homologados nunca editados | Único mecanismo de mudança: **adendo** (Req 25). |
| 3 | Connect Me ≠ Chat — formulário unidirecional | Sem reply, sem thread, sem histórico. |
| 4 | Visibilidade de vídeos de empresa — moradores só durante votação de fornecedor | Flag `autorizar_exibicao_votacao=true` + `video_visibility_grants`; revogação automática ao fim da votação (NATS event). |
| 5 | Métricas privadas — likes/comentários visíveis apenas ao dono | Não há ranking público, sem competição de vaidade. |
| 6 | Backend como verdade única — toda regra de negócio na API | Frontend é UI; nunca calcula quotas, money, prazos. |
| 7 | Trava trimestral — vídeos institucionais e vídeo-currículo: 1 atualização/90 dias | Campo `last_updated_trimester`; bloqueia substituição antes do prazo. |
| 8 | 1 dispositivo por conta — fingerprint + IP | Login em 2º device invalida sessão anterior; `sessions.device_fingerprint`. |
| 9 | Tenant isolation — cross-tenant só via grants explícitos | `WHERE condominium_id = $tenantID` em toda query. |
| 10 | LGPD by design — log obrigatório de revelação de dados pessoais | `lgpd_logs` (timestamp, IP, finalidade, versão dos termos). Append-only, retenção 5 anos. |

---

## 3. As 12 Decisões Resolvidas (não revisitar)

| # | Tema | Decisão |
|---|---|---|
| 1 | Escopo da Assembleia | Live Assembly com WebSocket real-time, Telão 2 áreas, fila de fala cronometrada. **Modalidade online/híbrida inativa no MVP** (só presencial). |
| 2 | Níveis de plano | `trial / base / plus / pro` para personas comerciais. Moradores: Base + Pagante. |
| 3 | Onboarding por persona | Fluxos separados — Morador 4 telas, Síndico 6, Empresa 7, Parceiro 4. |
| 4 | Paywall | **Soft block** — dados permanecem, premium bloqueado. **Sem grace period.** |
| 5 | Connect Me — direções | 4 fluxos: Síndico→Empresa, Morador→Síndico, Empresa→Empresa (Plus/Pro), Síndico→Parceiro. |
| 6 | Banco de Talentos | **In-scope.** 11 telas morador, 7 funcionalidades empresa. Vídeo-currículo 90s, lock 90 dias. |
| 7 | Academia LMS | **In-app completa.** 5 áreas, 3 trilhas. Parceiro Vizinhança **sem acesso**. |
| 8 | Painel Admin MS | **In-scope.** Gestão global de tenants, moderação, financeiro, broadcasts. |
| 9 | Connect Me Morador→Síndico | Formulário unidirecional **frio** (sem reply, sem histórico). |
| 10 | Quotas Connect Me | Por **ano** — Morador Base 2/ano · Pagante 4/ano · Síndico N1 2/ano · N2 4/ano · N3 ilimitado. |
| 11 | Limite de condomínios por síndico | **15.** |
| 12 | Parceiro da Vizinhança | Persona com login. Aceita CPF (CNPJ não obrigatório). |

---

## 4. Personas, Planos e Trials

### Personas (1 CPF/CNPJ pode ter múltiplos vínculos)

| Persona | Tenant | Profile | Role Zitadel |
|---|---|---|---|
| Síndico | Operador do Condomínio | `institutional.SyndicProfile` | `syndic` |
| Morador | Pertencente ao Condomínio | `institutional.ResidentProfile` | `resident` |
| Empresa | Próprio Tenant (`company`) | `commercial.EnterpriseProfile` | `enterprise` |
| Agência de Marketing | Actor delegado (NÃO é tenant) | — | `marketing` (vínculo via token) |
| Parceiro da Vizinhança | Próprio Tenant (`company`, `is_neighborhood=true`) | `commercial.LocalCompanyProfile` | `local_company` |
| Admin Master Síndico | Global | — | `admin` |

### Role Types (sub-contexto operacional, **apenas no banco**, não no Zitadel — T8)

```
principal | auxiliary | assistant     # síndico
titular   | dependent                 # morador
administrator | operator              # empresa
```

### Planos por persona

**Morador:** `resident_base` (R$ 0) | `resident_paid` (R$ 9,90/mês)
**Síndico:** `syndic_n1` (Aprender, trial 15d) | `syndic_n2` (Atuar, trial 15d) | `syndic_n3` (Consolidar, trial 15d)
**Empresa:** `enterprise_plus` (trial 7d) | `enterprise_pro` (trial 7d)
**Agência:** `marketing_standard`
**Parceiro:** `local_company_standard` (trial 30d)

### Quotas (consolidado)

| Quota | Reset | Valores |
|---|---|---|
| Connect Me | Anual (1º jan) | Morador Base 2 · Pagante 4 · Sind. N1 2 · N2 4 · N3 ∞ · Empresa Plus E→E ilimitado · Pro ilimitado |
| Vídeos institucionais | Mensal (1º do mês) | Sind. N1 4 · N2 8 · N3 12 · Empresa Plus 8 · Pro 12 |
| Trava trimestral por vídeo | 90 dias | Substituição do mesmo vídeo (não do volume mensal) |
| Condomínios por síndico | Hard limit | **15** (Decision 11) |
| Dispositivos ativos por conta | Hard limit | **1** (Rule 8) |

---

## 5. Decisões Técnicas (T1–T15) — invariantes implementadas

| # | Decisão | Resolução |
|---|---|---|
| T1 | Localização do `AuthUser` | `internal/shared/types/` (middleware não importa de modules/) |
| T2 | Framework HTTP | Gin **abstraído** via `contracts.HTTPRouter` + `contracts.Context` |
| T3 | Error handling | `AppError{Code, Message, Err}` + sentinels + `errors.Is/As` |
| T4 | UnitOfWork | Tx no `context.Context` via chave privada `txKey{}` + `database.DBFromContext(ctx, pool)` |
| T5 | Migrations | goose v3 com `embed.FS`; `database.Migrate(ctx, pool)` no startup |
| T6 | Logger | zerolog wrapper interface `Logger` (sem `SetGlobalLevel()`) |
| T7 | Roles primárias | Inglês: `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin` |
| T8 | Role types | Sub-contexto operacional **só no banco** (`user_role_type` enum) — Zitadel não conhece |
| T9 | Cookie auth | `access_token` httpOnly, `Domain=.mastersindico.com.br`, Secure, SameSite=Lax |
| T10 | Money | `INTEGER` centavos no PG, `int64` em Go (nunca float). Fração ideal `NUMERIC(5,4)`. |
| T11 | Fluxo OIDC | Authorization Code + PKCE via `zitadel/oidc/v3` (`rp.NewRelyingPartyOIDC`) |
| T12 | Rotas públicas auth | `GET /auth/{login,callback,logout}` **fora** de `/api/v1` |
| T13 | Cache introspection | Redis `ms:auth:{zitadelUserID}` TTL 5min |
| T14 | Auth mobile | Fallback `Authorization: Bearer` (RFC 6750) além do cookie |
| T15 | Guard sem role | Tokens sem role → `403 NO_ROLE_ASSIGNED` **antes** do UPSERT |

---

## 6. Stack atual confirmada (2026-04-22)

> ⚠️ O `product-overview.md` ainda lista AWS SES / ECS Fargate / OpenSearch — **stale**. Realidade abaixo (corroborada por `web/CLAUDE.md`, `backend/CLAUDE.md`, AUDIT.md, SESSION_CHARTER.md). Ver [[quebras-detectadas]] item Q1.

| Camada | Tecnologia |
|---|---|
| Backend | Go 1.26 + Gin abstraído (Clean Arch + DDD + CQRS + Vertical Slices) |
| Web | SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS · Kobalte · Axios · Zod · Biome (Bun workspaces, 6 apps + 2 packages) |
| Mobile | Flutter/Dart (FVM, Clean Architecture, get_it+injectable, go_router) |
| Auth/IAM | Zitadel v4 (OIDC PKCE) |
| Billing | Stripe (`IPaymentGateway`) |
| Vídeo VOD | Mux (`IVideoProvider`) — webhook HMAC público |
| Vídeo live | Livekit (`ILiveProvider`) |
| Email | **Resend** (`IEmailProvider`) — não AWS SES |
| SMS | Twilio (planejado, Sprint 2+) |
| Storage | MinIO (dev) / Cloudflare R2 ou S3 (prod) — D-029 ainda aberta |
| Busca | **PostgreSQL `tsvector`** (não OpenSearch) |
| DB | PostgreSQL 18 + pgx/v5 + sqlc v1.30 (sem ORM) |
| Cache | Redis 7 |
| Mensageria | NATS JetStream |
| Migrations | goose v3 (embed.FS) |
| Logging | zerolog wrapper |
| Observabilidade | OTel (traces + metrics) + Prometheus bridge — entregue Sprint 9 |
| Deploy | **Railway** (primário) — não AWS ECS |
| CI | GitHub Actions |

---

## 7. Bounded contexts e numeração de migrations

| Módulo | Faixa | Sprint origem | Status |
|---|---|---|---|
| identity | 001-099 | 0-1 | ✅ produção |
| billing | 100-199 | 1 | ✅ produção |
| institutional | 200-299 | 2-3 | ✅ produção |
| commercial | 300-399 | 4 | ✅ produção |
| content | 400-499 | 5 | ✅ produção |
| assembly | 500-599 | 7 | ✅ produção |

---

## 8. Cadeia de middleware BeyondCorp (fixa)

```
Recovery → RequestID → Logger → CORS → ErrorHandler
  → [rotas protegidas]:
      → RateLimiter (por IP + userId)
      → Authenticate (JWT → Zitadel introspection → cache → user sync → 1-device check)
      → RequirePermission(action, resource)  ← ABAC engine próprio
  → Handler
  → AuditLogger (pendente — DT-010)
```

Rotas OIDC (`/auth/login`, `/auth/callback`, `/auth/logout`) e webhooks públicos (`/webhooks/stripe`, `/webhooks/mux`) são montados **fora** de `/api/v1`.

---

## 9. Marcos de entrega

| Marco | Data | Escopo |
|---|---|---|
| **Marco 1** | **2026-05-08** | Identity + Billing + Institutional I (MVP contratual) |
| **Marco 2** | **2026-06-20** | Commercial + Content (Connect Me, vídeos, marketplace, busca) |
| **Marco 3** | **2026-07-20** | Assembly + Academia LMS base + Polish (observabilidade, performance) |

Priorização sempre protege Marco 1.

---

## 10. Padrões de engenharia inegociáveis (memória `feedback_padroes_engenharia_exigidos`)

DDD · TDD · Clean Architecture · CQRS · SOLID · Code Calisthenics · ACID · Saga.

Code Calisthenics aplicado: **proibido bloco `else`** (memória `feedback_no_else`) — usar early return / strategy / default+override.

---

## Referências cruzadas

- [[quebras-detectadas]] — 18 inconsistências cross-doc (severidade alta, média, baixa)
- [[../sdd-gsd-aplicado]] — metodologia SDD + GSD aplicada ao projeto
- [[../_handoffs/2026-04-22-relatorio-agente3-analista]] — relatório completo da auditoria
- `backend/.kiro/specs/master-sindico/requirements/product-overview.md` — fonte canônica viva
- `backend/.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — 12 antipatterns do legacy
