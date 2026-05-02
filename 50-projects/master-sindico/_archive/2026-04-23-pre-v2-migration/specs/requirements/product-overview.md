---
title: Product Overview — Decisões, Regras de Ouro, Marcos
version: 1.0
status: stable
type: requirement
domain: cross-domain
tags: [overview, decisions, golden-rules, milestones]
---

# Product Overview — Master Síndico

Visão transversal do produto: stack confirmada, decisões resolvidas (que **não** voltam a debate), regras de ouro invioláveis, marcos de entrega, decisões técnicas implementadas no Sprint 0 e 1.

## Visão do produto

Plataforma SaaS de **governança condominial** que conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local em um ecossistema integrado.

Não é ERP, não é app de comunicação, não é sistema financeiro. É **governança aplicada**:
- **Critério** → contratações baseadas em método, não em urgência
- **Registro** → memória institucional auditável
- **Reputação** → sistema de status baseado em evidência (Bronze → Diamante)
- **Integração** → ecossistema unificado de perfis, vídeos, cursos e contato

Identidade é **única** (1 CPF/CNPJ). Contextos de operação são **múltiplos**: mesmo usuário pode ser síndico no Condo X, morador no Y, e ter empresa cadastrada.

## Stack confirmada

- **Backend**: Go 1.26 + Gin (abstraído via `contracts.HTTPRouter`) — Clean Architecture + DDD + CQRS + Vertical Slices
- **Frontend Web**: SolidJS · TanStack Router/Query/Form · Rsbuild · UnoCSS · Axios · Kobalte · Zod · Biome
- **Mobile**: Flutter/Dart — Clean Architecture
- **Auth/IAM**: Zitadel — OIDC Authorization Code + PKCE + JWT introspection
- **Billing**: Stripe (`IPaymentGateway`) — trial, subscriptions, Customer Portal
- **Busca**: OpenSearch (Sprint 5)
- **Vídeo**: Mux (`IVideoProvider`) — upload, encoding, HLS, storage, analytics
- **SMS**: Twilio (`ISMSProvider`) — OTP, notificações
- **Email**: AWS SES (`IEmailProvider`)
- **Push**: FCM (`IPushProvider`)
- **Mensageria**: NATS JetStream (eventos assíncronos — quando necessário)
- **Banco**: PostgreSQL 18 (RDS em prod) via pgx/v5 + sqlc v1.30 (sem ORM)
- **Cache**: Redis 7 (ElastiCache em prod) via go-redis/v9
- **Migrations**: goose v3 via `embed.FS`
- **Logging**: zerolog (interface + context logger, sem `SetGlobalLevel`)
- **Infra**: AWS (ECS Fargate, RDS, ElastiCache, S3, SES, CloudFront, Route53, CloudWatch) + Railway (alternativa/complementar)

Arquitetura: **BeyondCorp adaptado** — Zero Trust, ABAC, tenant isolation estrito, 1 dispositivo por conta.

## Marcos de entrega

| Marco | Data | Escopo |
|---|---|---|
| **Marco 1** | **2026-05-08** | Identity + Billing + Institutional I (MVP contratual) |
| **Marco 2** | **2026-06-20** | Commercial + Content (Connect Me, vídeos, marketplace, busca) |
| **Marco 3** | **2026-07-20** | Assembly + Academia LMS base + Polish (observabilidade, performance) |

Priorização sempre protege o **Marco 1**. Ver `.kiro/STATE.md` para decisões em aberto.

## Decisões resolvidas (fonte de verdade — não revisitar)

| # | Decisão | Resolução |
|---|---|---|
| **1** | Escopo da Assembleia | Live Assembly com WebSocket real-time; Telão 2 áreas; fila de fala cronometrada. Modalidade online/híbrida **inativa** no MVP (só presencial). |
| **2** | Níveis de plano | `trial` · `base` · `plus` · `pro` para personas comerciais. Moradores: Base e Pagante. |
| **3** | Onboarding por persona | Sim — fluxos separados: Morador (4 telas), Síndico (6), Empresa (7), Parceiro (4). |
| **4** | Paywall | **Soft block** — dados permanecem, acesso a premium bloqueado. **Sem grace period.** |
| **5** | Connect Me — direções | 4 fluxos: Síndico→Empresa, Morador→Síndico, Empresa→Empresa (Plus/Pro), Síndico→Parceiro. |
| **6** | Banco de Talentos | **In-scope**. 11 telas morador, 7 funcionalidades empresa. Vídeo-currículo 90s, lock 90 dias. |
| **7** | Academia LMS | **In-app completa**. 5 áreas, 3 trilhas. Parceiro da Vizinhança **sem acesso**. |
| **8** | Painel Admin MS | **In-scope**. Gestão global de tenants, moderação, financeiro, broadcasts. |
| **9** | Connect Me Morador→Síndico | Formulário unidirecional **frio** (sem reply, sem histórico). |
| **10** | Quotas Connect Me | Por **ano**. N1: 2/ano · N2: 4/ano · N3: ilimitado · Base: 2/ano · Pagante: 4/ano. |
| **11** | Limite de condomínios | **15** por síndico. |
| **12** | Parceiro da Vizinhança | Persona com login. Aceita CPF (CNPJ não obrigatório). |

## Regras de ouro (invioláveis — aplica a todo código e spec)

1. **Separação de identidade** — nunca misture User (auth), Profile (domínio) e Subscription (comercial). Separação rígida por bounded context.
2. **Imutabilidade** — Timeline, Negócio Fechado confirmado, votos homologados: **nunca** editados, **nunca** deletados. Único mecanismo: adendo formal.
3. **Connect Me ≠ Chat** — formulário unidirecional. Sem histórico, sem reply, sem thread.
4. **Visibilidade de vídeos de empresa** — moradores **só** acessam durante votação de fornecedor em assembleia, se `autorizar_exibicao_votacao = true`. Acesso revogado após votação.
5. **Métricas privadas** — likes e comentários visíveis apenas ao dono do conteúdo.
6. **Backend como verdade única** — toda regra de negócio na API. Frontend é interface de consumo.
7. **Trava trimestral** — vídeos institucionais (empresa) e vídeo-currículo (morador): 1 atualização a cada 90 dias.
8. **1 dispositivo por conta** — fingerprint + IP tracking. Login em segundo device **invalida** sessão anterior.
9. **Tenant isolation** — usuários só acessam dados do próprio tenant. Cross-tenant apenas via grants explícitos (Connect Me, validações).
10. **LGPD by design** — dados revelados no Connect Me apenas após aceite. Log append-only com timestamp, IP, finalidade, versão dos termos.

## Decisões Técnicas Implementadas (T1–T15)

Estas decisões foram tomadas durante Sprint 0 + 1 e são **fonte de verdade** para sprints seguintes.

| # | Decisão | Resolução |
|---|---|---|
| T1 | Localização do `AuthUser` | `internal/shared/types/` — `shared/middleware/` não pode importar de `modules/` |
| T2 | Framework HTTP | Gin, mas abstraído via `contracts.HTTPRouter` + `contracts.Context` — módulos **não** importam Gin |
| T3 | Error handling | `AppError{Code, Message, Err}` com sentinels + `errors.Is/As`. ErrorHandler global intercepta `ctx.SetError(err)` |
| T4 | UnitOfWork | Tx injetada no `context.Context` via chave privada `txKey{}` — repositories extraem com `database.DBFromContext(ctx, pool)` |
| T5 | Migrations | goose v3 com `embed.FS` — migrations dentro do binário; `database.Migrate(ctx, pool)` no startup |
| T6 | Logger | zerolog wrapper com interface `Logger` — sem `SetGlobalLevel()` (mutação global quebra testes paralelos) |
| T7 | Roles no Zitadel | Roles primárias em inglês: `syndic`, `resident`, `enterprise`, `marketing`, `local_company`, `admin` |
| T8 | Role types | Sub-contexto operacional **apenas no banco** (`user_role_type` enum) — Zitadel não conhece |
| T9 | Cookie auth | `access_token` httpOnly, `Domain=.mastersindico.com.br`, `Secure`, `SameSite=Lax` |
| T10 | Valores monetários | `INTEGER` centavos no PostgreSQL, `int64` em Go — **nunca** `float` |
| T11 | Fluxo OIDC | Authorization Code Flow com PKCE via `zitadel/oidc/v3` (`rp.NewRelyingPartyOIDC`) |
| T12 | Rotas públicas de auth | `GET /auth/login`, `/callback`, `/logout` **fora** de `/api/v1` (sem middleware Authenticate) |
| T13 | Cache de introspection | Redis `ms:auth:{zitadelUserID}` com TTL 5min |
| T14 | Auth mobile | Fallback `Authorization: Bearer` (RFC 6750) além do cookie httpOnly |
| T15 | Guard clause sem role | Tokens sem role retornam `403 NO_ROLE_ASSIGNED` **antes** do UPSERT — evita user com `role=""` |

## Bounded Contexts

| Módulo | Responsabilidade | Sprint |
|---|---|---|
| `identity` | Usuários, sessões, autenticação Zitadel, 1-device | 0-1 (implementado) |
| `billing` | Planos, assinaturas, Stripe, trial, quotas | 1 |
| `institutional` | Condomínio, unidades, timeline, plano diretor, mandato | 2-3 |
| `commercial` | Connect Me, contratos, marketplace, banco de talentos | 4 |
| `content` | Vídeos Mux, busca OpenSearch, academia LMS base | 5 |
| `assembly` | Assembleias live, votações, atas | 7 |

## Critérios para mudar algo acordado

Decisões desta página **não são revisitadas** exceto por:
1. Requisito regulatório novo (LGPD, lei condominial) → altera spec, `.kiro/STATE.md` registra
2. Descoberta técnica impossibilitante (SDK quebrado, performance crítica) → `.kiro/STATE.md` abre decisão, João aprova
3. Feedback de usuário em staging/prod (pós-marco 1) → revisão deliberada

Para qualquer outra coisa: **respeitar o acordado**. "E se fosse X?" não é motivo para mudança — é conversa para `.kiro/STATE.md` ou nota pessoal.

## Referências

- **Full escopo** (Reqs 1-103): `requirements.md` monolítico (canônico até recortes por domínio estarem completos)
- **Personas, planos, quotas**: `personas-and-plans.md`
- **Matriz funcional**: `functional-matrix.md` (quando criado) ou `requirements.md` seção final
- **Lições do legacy**: `../reference/antipatterns-to-avoid.md`
- **Design e arquitetura**: `../design/` + `../design.md` monolítico
- **Roadmap operacional**: `../tasks/`
