# Design — Master Síndico

> **⚠️ Em migração**: este arquivo monolítico é **fonte canônica atual** para design arquitetural completo. Arquivos em `design/*.md` vão ser criados incrementalmente conforme cada Sprint toca um tema específico. Quando todos estiverem completos, este arquivo será deprecated.
>
> **Índice dos recortes**: [`design/README.md`](design/README.md)


## Filosofia Arquitetural: BeyondCorp Adaptado

O Master Síndico implementa uma arquitetura **Zero Trust** adaptada do modelo BeyondCorp do Google:

- **Identidade como perímetro:** Zitadel é o root resolver. Todo acesso — web, mobile, API — passa por ele primeiro. Não existe "dentro da rede" confiável por padrão.
- **ABAC granular:** cada request avalia `(userId, tenantId, role, plan, action)` antes de qualquer operação de negócio.
- **Tenant isolation estrito:** dados de um tenant nunca vazam para outro. Cross-tenant apenas via grants explícitos (Connect Me, validações).
- **1 dispositivo ativo por conta:** fingerprint + IP tracking. Login em segundo device invalida sessão anterior.
- **Audit trail imutável:** todas as ações críticas são registradas em append-only log com retenção de 5 anos (LGPD).
- **Cookie httpOnly no domínio raiz:** `.mastersindico.com.br` — todos os subdomínios compartilham sessão automaticamente sem localStorage.

---

## Stack Tecnológica

| Camada | Tecnologia | Justificativa |
|--------|-----------|---------------|
| Backend | Go (Gin) | Performance, binário único, concorrência nativa para WebSocket |
| Frontend Web | SolidJS + TanStack + Rsbuild + UnoCSS + Axios + Kobalte | Reatividade fina, sem VDOM overhead |
| Mobile | Dart (Flutter) + Clean Architecture | Código único iOS/Android, performance nativa |
| Auth/IAM | Zitadel | OIDC cloud-native, multi-tenant nativo, MFA, ABAC |
| Billing | Stripe | SDK maduro, Customer Portal self-service, trial nativo |
| Vídeo VOD | Mux | Upload direto (client → Mux), encoding, HLS streaming |
| Conferência ao vivo | Livekit | WebRTC self-hosted, salas para assembly online/híbrida |
| Storage de objetos | MinIO | Self-hosted em Railway; migração para Cloudflare R2 em prod |
| Email transacional | Resend | Templates HTML pt-BR, SPF/DKIM managed |
| SMS | Twilio | OTP, notificações transacionais |
| Mensageria | NATS JetStream | Pub/Sub persistente para eventos assíncronos (Sprint 10+) |
| Banco | PostgreSQL 18 (pgx + sqlc + goose) | ACID, soft delete, migrations embedded |
| Cache | Redis 7 | Sessões, quotas, snapshots de assembleia |
| Infra | Railway | Containers, PostgreSQL managed, Redis managed, deploy via GitHub Actions |

---

## Arquitetura do Backend (Go — Clean Architecture + DDD + CQRS)

Cada bounded context é um módulo autocontido. Dependências sempre apontam para dentro: `infrastructure → application → domain`. O `main.go` apenas instancia e registra os módulos — não conhece detalhes internos.

```
cmd/
  api/
    main.go                    ← entry point: bootstrap Gin, DI wire-up, graceful shutdown

internal/
  core/
    contracts/                 ← IUseCase, IRepository, IUnitOfWork, IDomainEvent, IModule
                               ← HTTPRouter, HandlerFunc, Context (abstrações framework-agnostic)
    errors/                    ← AppError (Code + Message + Unwrap), sentinels, ValidationError com campos

  modules/
    identity/                  ← Bounded Context: usuários, sessões, autenticação Zitadel
      domain/
        entities/              ← User, Session (estado privado, getters explícitos)
        value_objects/         ← Email, CPF, CNPJ, Password (imutáveis, auto-validados)
        enums/                 ← UserRole, SessionStatus
        repositories/          ← IUserRepository, ISessionRepository (interfaces)
        services/              ← domain services puros (sem I/O)
        events/                ← UserCreated, SessionRevoked (IDomainEvent)
        providers/             ← IZitadelProvider (interface — implementação em shared/providers/auth/)
      application/
        use_cases/             ← um arquivo por use case (CQRS: commands e queries separados)
        mappers/               ← entidade → DTO
      infrastructure/
        database/              ← implementações dos repositories (pgx)
        http/routes/           ← handlers via contracts.HandlerFunc (sem import Gin)
        providers/             ← adapters específicos do módulo
        jobs/                  ← session cleanup, verification cleanup

    billing/                   ← Bounded Context: planos, assinaturas, Stripe, trial, quotas
    institutional/             ← Bounded Context: condomínio, unidades, timeline, plano diretor
    commercial/                ← Bounded Context: connect me, contratos, marketplace
    content/                   ← Bounded Context: vídeos Mux, busca OpenSearch
    assembly/                  ← Bounded Context: assembleias, votações, atas

  server/                      ← Gin engine, health checks, error handler global, GinAdapter
                               ← Nota: fica em internal/server/ (não pkg/) — é específico desta app

  shared/
    config/                    ← Config struct + Viper loader (com Validate())
    middleware/                ← BeyondCorp stack (authenticate, authorize, recovery, cors, logger, request_id)
                               ← Importa apenas shared/types/ — NUNCA modules/
    providers/
      auth/                    ← ZitadelProvider (implementação concreta de IZitadelProvider)
      cache/                   ← interface Cache + implementação Redis (go-redis/v9)
      database/                ← pgxpool + UnitOfWork (tx via context key privada) + Migrator (goose v3)
    types/                     ← AuthUser, ApiResponse, PaginatedResponse (sem deps de modules/)

pkg/
  logger/                      ← zerolog wrapper (interface Logger — sem SetGlobalLevel)
  money/                       ← Value Object Money (int64 centavos)
  utils/                       ← UUIDv7, validators, request parser
```

### Module Pattern

```go
// contracts/module.go — interface real implementada
type IModule interface {
    Name() string
    Register(router HTTPRouter) error   // HTTPRouter é framework-agnostic
    Bootstrap() error                   // cron jobs, event subscriptions
}

// HTTPRouter abstrai gin.RouterGroup — trocar de framework = novo adapter, zero mudança nos módulos
type HTTPRouter interface {
    Group(relativePath string) HTTPRouter
    GET(path string, handlers ...HandlerFunc)
    POST(path string, handlers ...HandlerFunc)
    PUT(path string, handlers ...HandlerFunc)
    PATCH(path string, handlers ...HandlerFunc)
    DELETE(path string, handlers ...HandlerFunc)
    Use(middleware ...HandlerFunc)
}

// HandlerFunc e Context são framework-agnostic
// GinAdapter em internal/server/gin_adapter.go converte gin.Context ↔ contracts.Context
type HandlerFunc func(ctx Context)
```

O `main.go` chama `srv.RegisterModules(modules...)` e `srv.BootstrapModules(modules...)` — não conhece detalhes internos de nenhum módulo.

### CQRS — Use Cases

Commands (escrita) e queries (leitura) são structs e arquivos separados. Cada use case tem um único método `Execute`. Dependências chegam via construtor — sem DI framework, injeção manual no `main.go`.

```go
// application/use_cases/create_session_command.go
type CreateSessionCommand struct {
    UserID            string
    IPAddress         string
    DeviceFingerprint string
    UserAgent         string
}

type CreateSessionUseCase struct {
    sessionRepository domain.ISessionRepository
    unitOfWork        contracts.IUnitOfWork
    logger            logger.Logger
}

// Implementa contracts.IUseCase[CreateSessionCommand, *SessionDTO]
func (uc *CreateSessionUseCase) Execute(ctx context.Context, cmd CreateSessionCommand) (*SessionDTO, error)
```

Handlers em `infrastructure/http/routes/` usam `contracts.HandlerFunc` — sem import Gin. Responsabilidade única: parsear → chamar use case → serializar.

### Middleware Stack (BeyondCorp)

Todo request passa pela seguinte cadeia antes de chegar ao handler:

```
Request
  → Recovery (panic → 500, sem vazar stack trace)
  → RequestID (X-Request-ID gerado ou propagado)
  → Logger (zerolog structured — sem SetGlobalLevel)
  → CORS
  → ErrorHandler (intercepta c.Error() após Next())
  → [rotas protegidas]:
      → RateLimiter (por IP + userId) — Sprint 1
      → Authenticate:
          JWT extractor (cookie httpOnly "access_token")
          → Zitadel introspection (IZitadelProvider)
          → Cache Redis ms:session:{zitadelSessionID} (TTL 5min)
          → User sync UPSERT (primeiro login)
          → Session validator (1 device: fingerprint + IP)
          → AuthUser injetado em ctx via SetValue(AuthUserKey)
      → RequirePermission(action, resource) — ABAC engine
  → Handler
  → AuditLogger — Sprint 3
```

**Regras críticas de implementação:**
- `shared/middleware/` importa apenas `shared/types/` — nunca `modules/`
- `AuthUser` vive em `shared/types/` (não em `modules/identity/`)
- `recovery.go` escreve resposta diretamente com `c.JSON` — não delega para ErrorHandler (panic interrompe a chain)
- `ErrorHandler` intercepta `c.Error()` após `ctx.Next()` — handlers propagam erros, nunca escrevem resposta de erro diretamente

### Money Value Object

```go
type Money struct {
    amount   int64  // centavos — nunca float
    currency string // "BRL"
}
```

Fração ideal da assembleia usa `DECIMAL/NUMERIC` no PostgreSQL — nunca `float`.

---

## Autenticação Multi-Subdomain (BeyondCorp)

```
auth.mastersindico.com.br   ← Zitadel (OIDC flows)
app.mastersindico.com.br    ← SolidJS (CMS principal)
api.mastersindico.com.br    ← Go API

Set-Cookie: session=...; Domain=.mastersindico.com.br; Secure; HttpOnly; SameSite=Lax
```

Todos os subdomínios leem o cookie automaticamente. Cada app faz `GET /v1/auth/me` no mount — cookie vai junto. Sem localStorage, sem sincronização de estado entre apps.

### Fluxo de Auth

```
1. Usuário acessa app.mastersindico.com.br
2. Sem cookie → redirect para auth.mastersindico.com.br (Zitadel)
3. Zitadel autentica → emite JWT com claims (userId, tenantId, role, plan)
4. Backend valida JWT → verifica 1 device → verifica ABAC → processa
5. Cookie httpOnly setado no domínio raiz
6. Próximas requests: cookie vai automaticamente para todos os subdomínios
```

---

## Estrutura de Tenants (Multi-Tenant)

O ecossistema opera com **2 tipos de tenant** + actors delegados:

### Tenant: Condomínio
- Isolamento total de dados por `condominium_id`
- Síndico é o gatekeeper — nada entra na Timeline sem validação dele
- Empresa não publica direto — tudo passa por validação
- Empresa Síndica Vinculada: actor delegado com permissões parametrizáveis

### Tenant: Empresa
- Estrutura organizacional própria (Administrador + Operador Técnico)
- Opera across tenants (múltiplos condomínios)
- Agência de Marketing: actor delegado restrito a vídeos e analytics

### Actors Delegados (não são tenants)
- **Empresa Síndica:** vinculada por token, permissões configuráveis
- **Agência de Marketing:** acesso restrito a vídeos e analytics
- **Administradora:** opera exclusivamente no módulo Assembleia

---

## Data Model (PostgreSQL)

### Convenções
- UUIDs como PKs
- `snake_case`
- `created_at`, `updated_at`, `deleted_at` (soft delete) em todas as tabelas
- `DECIMAL/NUMERIC` para fração ideal (nunca float)
- `INTEGER` (centavos) para valores monetários

### Identity Domain

```sql
users:
  id UUID PK, email TEXT UNIQUE, password_hash TEXT
  name TEXT, phone TEXT, user_type ENUM
  status ENUM(pending_verification/active/suspended/deleted)
  email_verified_at TIMESTAMP
  device_fingerprint TEXT, last_ip TEXT  -- BeyondCorp: 1 device

sessions:
  id UUID PK, user_id UUID FK
  token TEXT, ip_address TEXT, user_agent TEXT
  device_fingerprint TEXT, expires_at TIMESTAMP

plans:
  id UUID PK, name TEXT, slug TEXT, type ENUM
  price_monthly INTEGER, price_yearly INTEGER  -- centavos
  stripe_product_id TEXT, stripe_price_monthly_id TEXT, stripe_price_yearly_id TEXT
  trial_days INTEGER, features JSONB, quotas JSONB

subscriptions:
  id UUID PK, user_id UUID FK, plan_id UUID FK
  status ENUM(trial/active/past_due/canceled/paused)
  stripe_subscription_id TEXT, stripe_customer_id TEXT
  current_period_start TIMESTAMP, current_period_end TIMESTAMP
  cancel_at_period_end BOOLEAN

onboarding_progress:
  id UUID PK, user_id UUID FK, persona ENUM
  current_step INTEGER, data JSONB, completed_at TIMESTAMP
```

### Profile Domain

```sql
syndic_profiles:
  id UUID PK, user_id UUID UNIQUE
  sindico_type ENUM, cpf TEXT UNIQUE, birth_date DATE
  professional_address JSONB, city TEXT, state TEXT
  years_of_experience INTEGER, governance_markers TEXT[]
  status ENUM(incomplete/active)

enterprise_profiles:
  id UUID PK, user_id UUID UNIQUE
  cnpj TEXT UNIQUE, razao_social TEXT, nome_fantasia TEXT
  categoria_principal ENUM, subcategorias ENUM[]
  compliance_markers TEXT[], status ENUM

resident_profiles:
  id UUID PK, user_id UUID UNIQUE
  cpf TEXT UNIQUE, birth_date DATE, status ENUM

local_company_profiles:
  id UUID PK, user_id UUID UNIQUE
  document TEXT UNIQUE  -- CPF ou CNPJ
  nome_fantasia TEXT, categoria ENUM
  neighborhood TEXT, city TEXT, cep TEXT, status ENUM
```

### Institutional Domain

```sql
condominiums:
  id UUID PK, created_by UUID FK
  name TEXT, cnpj TEXT, address JSONB
  condominium_type ENUM, total_units INTEGER
  public_id TEXT UNIQUE  -- 8 chars alfanumérico
  qr_code_url TEXT

mandates:
  id UUID PK, condominium_id UUID FK, sindico_id UUID FK
  mandate_type ENUM, start_date DATE, end_date DATE
  status ENUM(ativo/encerrado/transferido)

units:
  id UUID PK, condominium_id UUID FK
  number TEXT, block TEXT
  fracao_ideal NUMERIC(10,6)  -- NUNCA float
  titular_id UUID, status ENUM

timeline_entries:
  id UUID PK, condominium_id UUID FK, author_id UUID FK
  entry_type ENUM(7 types), title TEXT, description TEXT
  published_at TIMESTAMP
  is_amended BOOLEAN, original_entry_id UUID
  attachments JSONB
  -- SEM deleted_at: imutável por design

master_plan_actions:
  id UUID PK, condominium_id UUID FK
  title TEXT, tipo_atividade ENUM(26)
  area_impactada ENUM(26), tipo_risco ENUM(9)
  status ENUM(7)  -- atualizado APENAS via evento da Timeline
  due_date DATE, budget_estimated INTEGER

validation_items:
  id UUID PK, condominium_id UUID FK
  source_type TEXT, source_id UUID, company_id UUID
  status ENUM(pending/approved/rejected/adjustment_requested)
  reviewed_by UUID, rejection_reason TEXT

audit_trail:
  id UUID PK, actor_id UUID, action TEXT
  resource_type TEXT, resource_id UUID
  tenant_id UUID, metadata JSONB
  created_at TIMESTAMP
  -- Append-only, NUNCA atualizado ou deletado
  -- Retenção: 5 anos (LGPD)
```

### Commercial Domain

```sql
connect_me_requests:
  id UUID PK, flow_type ENUM(4 types)
  requester_id UUID, target_id UUID
  status ENUM, description TEXT, urgency ENUM
  data_revealed_at TIMESTAMP, data_revealed_ip TEXT
  consent_version TEXT, auto_closed_at TIMESTAMP

proposals:
  id UUID PK, company_id UUID, condominium_id UUID
  connect_me_request_id UUID
  title TEXT, scope TEXT, estimated_cost INTEGER
  status ENUM, version INTEGER, attachments JSONB

closed_deals:
  id UUID PK, condominium_id UUID, company_id UUID
  agreed_cost INTEGER, start_date TIMESTAMP, end_date TIMESTAMP
  is_immutable BOOLEAN DEFAULT false  -- true após dupla assinatura
  company_signed_at TIMESTAMP, sindico_signed_at TIMESTAMP
  timeline_entry_id UUID

partners:
  id UUID PK, user_id UUID, name TEXT, category ENUM
  neighborhood TEXT, city TEXT, cep TEXT
  plan_type ENUM, status ENUM
```

### Content Domain

```sql
videos:
  id UUID PK, owner_id UUID, tenant_type TEXT
  title TEXT, video_type ENUM(12 types)
  mux_asset_id TEXT, mux_playback_id TEXT
  status ENUM(processing/ready/failed/hidden)
  autorizar_exibicao_votacao BOOLEAN DEFAULT false
  can_be_replaced_after TIMESTAMP  -- trava trimestral
  view_count INTEGER

curricula:
  id UUID PK, morador_id UUID UNIQUE
  status ENUM, areas_of_interest TEXT[]
  availability ENUM, contract_types TEXT[]
  lgpd_consented_at TIMESTAMP
```

### Assembly Domain

```sql
assemblies:
  id UUID PK, condominium_id UUID
  assembly_type ENUM, scheduled_date TIMESTAMP
  quorum_type ENUM, speech_time_minutes INTEGER
  modalidade ENUM(presencial/hibrida/online)
  status ENUM, contingency_active BOOLEAN
  transparency_score INTEGER

assembly_votes:
  id UUID PK, assembly_id UUID, agenda_item_id UUID
  morador_id UUID, unit_id UUID
  vote_value TEXT
  fracao_ideal_weight NUMERIC(10,6)  -- NUNCA float
  is_proxy BOOLEAN, is_presencial_assistido BOOLEAN
  voted_at TIMESTAMP, is_homologated BOOLEAN
  UNIQUE(assembly_id, agenda_item_id, morador_id)
```

---

## Adapter Layer (Interfaces Go)

```go
type IPaymentGateway interface {
    CreateCustomer(ctx context.Context, data CustomerData) (Customer, error)
    CreateSubscription(ctx context.Context, data SubscriptionData) (Subscription, error)
    CancelSubscription(ctx context.Context, providerID string) error
    VerifyWebhookSignature(payload, signature string) bool
    ParseWebhookEvent(payload string) (WebhookEvent, error)
}
// Implementação: StripeGateway — troca de provider = nova impl, zero mudança no domínio

type IVideoProvider interface {
    CreateAsset(ctx context.Context, inputURL string) (Asset, error)
    GetPlaybackURL(playbackID string) string
    GetThumbnailURL(playbackID string) string
    DeleteAsset(ctx context.Context, assetID string) error
}

type IStorageProvider interface {
    Upload(ctx context.Context, bucket, key string, data io.Reader) (string, error)
    GetSignedURL(bucket, key string, ttl time.Duration) (string, error)
    Delete(ctx context.Context, bucket, key string) error
}

type IEmailProvider interface {
    Send(ctx context.Context, to, subject, templateID string, data map[string]any) error
}

type ISMSProvider interface {
    Send(ctx context.Context, to, message string) error
}

type ISearchProvider interface {
    Index(ctx context.Context, indexName string, documents []map[string]any) error
    Search(ctx context.Context, indexName, query string, filters map[string]any) ([]map[string]any, error)
    Delete(ctx context.Context, indexName, documentID string) error
}
```

Troca de provider (ex: Stripe → outro) = apenas nova implementação da interface. Nenhum domínio interno sabe qual provider está sendo usado.

---

## OpenSearch — 7 Índices

| Índice | Searchable | Filterable | Visibilidade |
|--------|-----------|------------|--------------|
| companies | name, fantasy_name, description, categories, city | category, city, plan_type, rating_avg | Plan-based |
| sindicos | name, city, state, sindico_type, governance_markers | city, state, sindico_type, plan_type | — |
| videos | title, description, video_type | video_type, company_id, status | — |
| partners | name, category, neighborhood, city | category, neighborhood, city | — |
| curricula | areas_of_interest, availability, contract_types | areas_of_interest, cnh_type, salary_range | Companies Plus/Pro |
| courses | title, description, category | category, level, type | — |
| forum_topics | title, content, category, tags | category, status | — |

---

## WebSocket Architecture (Assembleia)

```
Client (Telão / App Morador / Presidente)
    │ WSS + JWT auth
    ▼
Gin WebSocket (gorilla/websocket)
    │ Room: assembly:{id}
    ├── assembly:{id}:telao     → presença, quorum, item atual, fila
    ├── assembly:{id}:voting    → contagem votos, status
    ├── assembly:{id}:speech    → fila de fala, timer
    ├── assembly:{id}:checkin   → presença real-time
    └── assembly:{id}:control   → ações do presidente

Fluxo:
1. Ação → PostgreSQL (TX atômica)
2. Goroutine broadcaster → Redis pub/sub
3. Todas instâncias Go → push para WebSocket clients
```

---

## Frontend Web (SolidJS)

### Estrutura do Monorepo

```
apps/
  auth/          ← Zitadel OIDC flows (login, registro, verificação)
  lp/            ← Landing page (em andamento)
  cms/           ← Painel síndico + empresa + morador
  lms/           ← Academia (Marco 4+)
  forum/         ← Fórum (Marco 4+)
  assembly/      ← Assembleia ao vivo (Marco 4+)
packages/
  ui/            ← Design system (Kobalte + UnoCSS)
  schemas/       ← Zod schemas compartilhados
  core/          ← SDK stateless
    src/
      http/client.ts      ← fetch wrapper, credentials: include
      auth/client.ts      ← getSession(), signOut() — sem estado
      auth/types.ts       ← User, Session, Tenant
```

### Auth Context (por app, não no core)

```typescript
// apps/cms/src/context/auth.tsx
const [user, setUser] = createSignal<User | null>(null)
onMount(async () => {
  const session = await getSession() // do packages/core
  setUser(session?.user ?? null)
})
```

### Design System (IMUTÁVEL)

```css
--primary:    oklch(0.715 0.120 84.2)  /* Gold CTA */
--secondary:  oklch(0.218 0.055 256.1) /* Navy */
--accent:     oklch(0.871 0.129 82.0)  /* Gold Light */
--background: oklch(0.976 0.003 264.5) /* Light */
--background: oklch(0.183 0.031 263.4) /* Dark */
```

Fontes: **Manrope** (body) · **Michroma** (headings, letter-spacing: 0.05em)

---

## Mobile (Flutter — Clean Architecture)

```
lib/
  core/
    network/        ← HTTP client (Dio + interceptors)
    auth/           ← Zitadel OIDC integration
    storage/        ← Secure storage (flutter_secure_storage)
  features/
    auth/
      data/         ← repositories impl, datasources
      domain/       ← entities, use cases, repository interfaces
      presentation/ ← BLoC/Cubit, screens, widgets
    governance/
    commercial/
    content/
    assembly/
```

Cookie compartilhado com web via `Domain=.mastersindico.com.br`. Mobile faz `GET /v1/auth/me` no startup — cookie vai junto.

---

## Infra de Deploy (Railway)

| Serviço | Uso |
|---------|-----|
| Railway (containers) | Go API, Zitadel, Livekit, MinIO |
| Railway PostgreSQL 18 | Banco principal (managed, backups automáticos) |
| Railway Redis 7 | Cache + sessões + quotas + assembly snapshots |
| Railway NATS JetStream | Mensageria assíncrona (Sprint 10+) |
| MinIO (container Railway) | Storage de objetos (documentos, exports, assets) |
| Cloudflare R2 (prod futuro) | Migração MinIO → R2 para escala e CDN global |
| Resend | Email transacional (DNS gerenciado, templates pt-BR) |
| GitHub Actions | CI/CD: build → test → deploy Railway via CLI |
| Cloudflare (DNS + WAF) | DNS *.mastersindico.com.br, proteção DDoS |

---

## Segurança (BeyondCorp)

- JWT com expiração + refresh token (Zitadel)
- Cookie httpOnly no domínio raiz — sem localStorage para tokens
- 1 dispositivo ativo por conta (fingerprint + IP)
- Rate limiting em todas as rotas de auth
- CAPTCHA para tentativas suspeitas
- Tenant isolation via middleware ABAC
- Cross-tenant apenas via grants explícitos
- LGPD: log de compartilhamento com timestamp, IP, finalidade, consentimento
- Audit trail append-only, retenção 5 anos
- Signed URLs para acesso a arquivos privados (TTL curto)
- Webhook signature verification para Stripe e Mux
