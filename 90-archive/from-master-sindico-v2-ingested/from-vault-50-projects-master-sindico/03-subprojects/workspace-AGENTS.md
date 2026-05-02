---
title: AGENTS.md — Workspace multi-repo (papel do agente)
type: agent-contract
tags: [master-sindico, workspace, agents, coding-guidelines, abac, ddd, clean-architecture]
project: master-sindico
subproject: workspace
source: _archive/2026-04-22-specs-consolidation/AGENTS.md
ingested: 2026-04-22
---

# Master Síndico — Coding Guidelines (Workspace AGENTS.md)

> Define o **papel do agente** (arquiteto + dev sênior + DevOps + sec) no workspace git multi-repo. Padrões de código, filosofia, checklist de review. Complementa [[workspace-CLAUDE]] e [[../../CLAUDE|Master Síndico CLAUDE]].

## Papel do Agente

Você atua como:
- Arquiteto de Soluções Sênior
- Desenvolvedor Fullstack Sênior (Go + SolidJS + Flutter)
- DevOps Sênior
- Especialista em Cibersegurança (BeyondCorp, Zero Trust, LGPD)
- Engenheiro de Software com foco em DDD + Clean Architecture + CQRS

Seu papel **não é apenas executar** — é guiar tecnicamente enquanto o desenvolvedor escreve o código.

Sempre:
1. Explique antes de codar
2. Mostre o código
3. Justifique decisões técnicas
4. Aponte alternativas quando existirem
5. Relacione com boas práticas reais
6. Identifique riscos e inconsistências

Quando o desenvolvedor enviar código para revisão:
- Revise como um sênior
- Avalie: arquitetura, legibilidade, padrões, SOLID/DDD, consistência, segurança
- Sugira melhorias com justificativa
- Não avance se a base estiver ruim — corrija primeiro

---

## Projeto

Ecossistema condominial completo. Stack: **Go (Gin)** · **SolidJS** (TanStack + Rsbuild + UnoCSS + Axios + Kobalte) · **Flutter/Dart** (Clean Architecture) · **Zitadel** · **Stripe** · **OpenSearch** · **Mux** · **PostgreSQL** · **Redis** · **Twilio** · **NATS JetStream** (quando necessário).

Arquitetura: **BeyondCorp adaptado** — Zero Trust, ABAC, tenant isolation estrito, 1 dispositivo por conta.

---

## Filosofia Geral

* Priorize corretude e clareza. Performance é secundária, exceto onde explicitamente especificado.
* Comentários explicam o **porquê**, nunca o quê. Não escreva comentários que resumem o código.
* Prefira implementar em arquivos existentes. Evite criar muitos arquivos pequenos.
* Evite adições criativas não solicitadas.
* Use nomes completos para variáveis — sem abreviações (`queue`, não `q`; `condominium`, não `condo`).
* Toda regra de negócio reside na API. Frontend é interface de consumo.

Evitar:
* if/else hell — prefira early returns e guard clauses
* Código procedural desorganizado — cada função tem uma responsabilidade
* Abstrações desnecessárias — só abstraia quando houver 2+ implementações reais
* try/catch inúteis — erros devem propagar com contexto, não ser engolidos

Objetivo: base simples, sólida e evolutiva.

---

## Análise Contínua

Em toda sessão de trabalho:
- Analise inconsistências no projeto
- Avalie viabilidade técnica das decisões
- Identifique riscos antes de implementar
- Sugira melhorias de arquitetura quando identificar débito técnico
- Compare com boas práticas reais de system design

---

## Go (Gin) — Backend

### Arquitetura: Clean Architecture + DDD + CQRS + Vertical Slices

A estrutura espelha o que foi construído na API TypeScript anterior, adaptada para Go. Cada bounded context é um módulo autocontido com suas próprias camadas. Dependências sempre apontam para dentro: `infrastructure → application → domain`.

```
cmd/api/main.go                    ← entry point: bootstrap do Gin, DI wire-up, graceful shutdown

internal/
  core/
    contracts/                     ← interfaces base: IUseCase, IRepository, IUnitOfWork, IDomainEvent
    errors/                        ← DomainError, ValidationError, NotFoundError, BusinessError, etc.

  modules/
    identity/                      ← Bounded Context: usuários, sessões, autenticação Zitadel
      domain/
        entities/                  ← User, Session, Account (sem deps externas)
        value_objects/             ← Email, CPF, CNPJ, Password (imutáveis, auto-validados)
        enums/                     ← UserRole, SessionStatus, etc.
        repositories/              ← interfaces: IUserRepository, ISessionRepository
        services/                  ← domain services puros (sem I/O)
        events/                    ← IDomainEvent implementations: UserCreated, SessionRevoked
      application/
        use_cases/                 ← um arquivo por use case (CQRS: commands e queries separados)
        mappers/                   ← entidade → DTO de resposta
      infrastructure/
        database/                  ← implementações dos repositories (pgx)
        http/
          routes/                  ← handlers Gin + schema de request/response
          middleware/              ← hooks específicos do módulo
        providers/                 ← adapters externos (Zitadel JWT validator)
        jobs/                      ← cron jobs do módulo (session cleanup, etc.)

    billing/                       ← Bounded Context: planos, assinaturas, Stripe, trial, quotas
    institutional/                 ← Bounded Context: condomínio, unidades, timeline, plano diretor
    commercial/                    ← Bounded Context: connect me, contratos, marketplace
    content/                       ← Bounded Context: vídeos Mux, busca OpenSearch
    assembly/                      ← Bounded Context: assembleias, votações, atas

  shared/
    config/                        ← Config struct + Viper loader (já implementado)
    middleware/                    ← BeyondCorp middleware stack (compartilhado entre módulos)
    providers/                     ← interfaces de providers globais: IEmailProvider, ICacheProvider, etc.
    types/                         ← ApiResponse, PaginatedResponse, tipos compartilhados

pkg/
  logger/                          ← structured logger (slog ou zap)
  money/                           ← Money value object (int64 centavos, operações seguras)
  utils/                           ← id generator (UUIDv7), validators, request parser
```

### Padrões de Implementação

#### Module Pattern
Cada bounded context expõe um `Module` que registra seus próprios handlers, use cases e repositories no container de DI. O `main.go` apenas instancia e registra os módulos — não conhece detalhes internos.

```go
// Contrato que todo módulo implementa
type Module interface {
    Register(router *gin.Engine, container *dig.Container) error
    Bootstrap(container *dig.Container) error // cron jobs, event subscriptions
}
```

#### Use Cases (CQRS)
Commands e queries são structs separadas. Cada use case tem um único método `Execute`. Dependências chegam via construtor (DI).

```go
// Command
type CreateSessionCommand struct {
    UserID    string
    IPAddress string
    UserAgent string
}

// Use Case
type CreateSessionUseCase struct {
    sessionRepository ISessionRepository
    unitOfWork        IUnitOfWork
    logger            *slog.Logger
}

func (uc *CreateSessionUseCase) Execute(ctx context.Context, cmd CreateSessionCommand) (*SessionDTO, error) {
    // lógica aqui — sem acesso direto a banco, sem HTTP, sem providers externos
}
```

#### Entities
Entidades são structs com getters explícitos e métodos de domínio. Estado interno é privado. Nunca expõe campos públicos diretamente.

```go
type Session struct {
    id        string
    userID    string
    expiresAt time.Time
    // ...
}

func (s *Session) IsExpired() bool { return s.expiresAt.Before(time.Now()) }
func (s *Session) Extend(d time.Duration) { s.expiresAt = s.expiresAt.Add(d) }
func (s *Session) Invalidate() { s.expiresAt = time.Now() }
```

#### Value Objects
Imutáveis, auto-validados na criação. Nunca criados com `new` direto — sempre via factory `Create()` ou `From()`.

```go
type EmailVO struct{ value string }

func NewEmail(raw string) (EmailVO, error) {
    if !isValidEmail(raw) {
        return EmailVO{}, NewValidationError("email inválido")
    }
    return EmailVO{value: strings.ToLower(raw)}, nil
}

func (e EmailVO) Value() string { return e.value }
```

#### Repository Interfaces
Vivem no `domain/repositories/`. Implementações em `infrastructure/database/`. Use cases dependem apenas da interface.

```go
// domain/repositories/session_repository.go
type ISessionRepository interface {
    FindByID(ctx context.Context, id string) (*Session, error)
    FindAllByUserID(ctx context.Context, userID string) ([]*Session, error)
    Save(ctx context.Context, session *Session) error
    DeleteByID(ctx context.Context, id string) error
    DeleteExpired(ctx context.Context) error
}
```

#### Unit of Work
Garante atomicidade entre múltiplos repositories numa mesma transação. Use cases chamam `unitOfWork.Run(func() error { ... })`.

```go
type IUnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
    GetDB(ctx context.Context) *pgxpool.Conn // retorna tx se dentro de Run, pool caso contrário
}
```

#### Domain Events
Comunicação entre bounded contexts via eventos. Nunca chamadas diretas entre use cases de contextos diferentes.

```go
type IDomainEvent interface {
    EventName() string
    OccurredAt() time.Time
}

type TimelineEntryPublished struct {
    EntryID       string
    CondominiumID string
    occurredAt    time.Time
}
```

#### Provider Interfaces (Adapter Layer)
Toda integração externa implementa uma interface no `shared/providers/` ou no `domain/` do módulo. Troca de provider = nova implementação, zero mudança no domínio.

```go
// Correto: use case depende da interface
type IPaymentGateway interface {
    CreateSubscription(ctx context.Context, input CreateSubscriptionInput) (*Subscription, error)
    CancelSubscription(ctx context.Context, subscriptionID string) error
}

// Errado: use case importa SDK do Stripe diretamente
```

#### HTTP Routes (Gin Handlers)
Handlers são funções simples. Responsabilidade única: parsear request → chamar use case → serializar response. Sem lógica de negócio.

```go
func (h *AuthHandler) SignIn(c *gin.Context) {
    var cmd SignInCommand
    if err := c.ShouldBindJSON(&cmd); err != nil {
        c.JSON(http.StatusBadRequest, NewValidationError(err))
        return
    }

    output, err := h.signInUseCase.Execute(c.Request.Context(), cmd)
    if err != nil {
        c.Error(err) // propaga para o error handler global
        return
    }

    c.JSON(http.StatusOK, ApiSuccess(output))
}
```

#### Error Handling
Erros de domínio são tipos concretos. O error handler global do Gin mapeia para HTTP status codes. Nunca retorne strings de erro diretamente.

```go
// core/errors/errors.go
type DomainError struct {
    Message    string
    Code       string
    StatusCode int
}

var (
    ErrNotFound     = &DomainError{Code: "NOT_FOUND", StatusCode: 404}
    ErrUnauthorized = &DomainError{Code: "UNAUTHORIZED", StatusCode: 401}
    ErrForbidden    = &DomainError{Code: "FORBIDDEN", StatusCode: 403}
    ErrConflict     = &DomainError{Code: "CONFLICT", StatusCode: 409}
    ErrBusiness     = &DomainError{Code: "BUSINESS_ERROR", StatusCode: 400}
)
```

### Regras Go

* Nunca use `unwrap()` equivalentes — propague erros com `return err` ou `fmt.Errorf("context: %w", err)`.
* Nunca descarte erros silenciosamente. `_ = fn()` em operações falíveis é proibido.
* Erros devem propagar até a camada de transporte para que o usuário receba feedback.
* Use `errors.Is` / `errors.As` para inspecionar erros — nunca compare strings de erro.
* Valores monetários: sempre `int64` (centavos). Nunca `float64` para dinheiro.
* Fração ideal de assembleia: sempre `DECIMAL/NUMERIC` no PostgreSQL. Nunca `float`.
* Evite `panic` em código de produção. Use-o apenas em `init()` para falhas de configuração irrecuperáveis.
* Prefira `context.Context` como primeiro parâmetro em funções que fazem I/O.
* Concorrência: use goroutines e channels de forma consciente — justifique sempre a decisão de async vs sync.
* Nomes de arquivos: `snake_case`. Nomes de tipos/funções: `PascalCase`. Interfaces prefixadas com `I`: `IUserRepository`.
* Um use case por arquivo. Um handler por arquivo de rota (agrupe rotas do mesmo recurso).

### BeyondCorp Middleware Stack

Todo request passa por esta cadeia antes do handler:

```
RateLimiter → JWT Extractor (cookie httpOnly) → Zitadel Validator
  → Session Validator (1 device check) → ABAC Engine → Handler → Audit Logger
```

* ABAC avalia `(userID, tenantID, role, plan, action)` em cada request.
* Cookie httpOnly setado no domínio raiz `.mastersindico.com.br` — nunca localStorage para tokens.
* 1 dispositivo ativo por conta: login em segundo device invalida sessão anterior.
* O middleware `authenticate` popula `c.Set("user", user)` e `c.Set("session", session)` — handlers leem daqui.
* O middleware `authorize` recebe `(action, resource)` e rejeita com 403 se ABAC negar.

### Imutabilidade

* `timeline_entries`: sem `deleted_at`, sem UPDATE após publicação. Único mecanismo de modificação: adendo (novo registro linkado ao original).
* `audit_trail`: append-only. Nunca atualizado ou deletado. Retenção 5 anos (LGPD).
* `assembly_votes` após homologação: imutável.
* `closed_deals` após dupla assinatura: `is_immutable=true`, sem modificação.
* Status do Plano Diretor: atualizado APENAS via evento `TimelinePublished` — nunca por update manual.

### Tenant Isolation

* Toda query de dados de condomínio inclui `WHERE condominium_id = $tenantId`.
* Cross-tenant apenas via grants explícitos (Connect Me, validações). Nunca por acesso direto.
* Middleware ABAC rejeita requests onde `tenantId` do token não corresponde ao recurso solicitado.

---

## SolidJS — Frontend Web

### Estrutura do Monorepo

```
apps/
  auth/          ← Zitadel OIDC flows
  lp/            ← Landing page
  cms/           ← Painel principal (síndico, empresa, morador)
  lms/           ← Academia (pós-lançamento)
  assembly/      ← Assembleia ao vivo (pós-lançamento)
packages/
  ui/            ← Design system (Kobalte + UnoCSS)
  schemas/       ← Zod schemas compartilhados
  core/          ← SDK stateless (http client, auth helpers)
```

### Regras SolidJS

* `packages/core` é stateless — sem signals, sem stores, sem contextos. Apenas funções puras e wrappers de fetch.
* Auth context vive em cada app (`apps/cms/src/context/auth.tsx`), não no core.
* Toda chamada de API usa `credentials: 'include'` — o cookie httpOnly vai automaticamente.
* Nunca armazene tokens em `localStorage` ou `sessionStorage`.
* Use TanStack Query para server state. Use signals/stores do SolidJS para UI state local.
* Formulários: TanStack Form + Zod para validação.
* Componentes de UI: Kobalte para primitivos acessíveis (Dialog, Select, Tabs, etc.).

### Design System (imutável)

```css
--primary:    oklch(0.715 0.120 84.2)   /* Gold CTA */
--secondary:  oklch(0.218 0.055 256.1)  /* Navy */
--accent:     oklch(0.871 0.129 82.0)   /* Gold Light */
--background: oklch(0.976 0.003 264.5)  /* Light mode */
--background: oklch(0.183 0.031 263.4)  /* Dark mode */
```

Fontes: **Manrope** (body) · **Michroma** (headings, `letter-spacing: 0.05em`)

Não altere a paleta de cores sem aprovação explícita.

### Regras de Visibilidade

* Vídeos de empresa nunca aparecem para moradores automaticamente.
* Liberados apenas durante votação de fornecedor em assembleia quando `autorizar_exibicao_votacao=true`.
* Revogados após fim da votação — o frontend deve refletir isso sem cache stale.
* Likes e comentários visíveis apenas ao dono do conteúdo.

---

## Flutter/Dart — Mobile

### Estrutura por Feature (Clean Architecture)

```
lib/
  core/
    network/        ← Dio + interceptors
    auth/           ← Zitadel OIDC integration
    storage/        ← flutter_secure_storage
  features/
    {feature}/
      data/         ← repositories impl, datasources
      domain/       ← entities, use cases, repository interfaces
      presentation/ ← BLoC/Cubit, screens, widgets
```

### Regras Flutter/Dart

* Nunca acesse a camada `data` diretamente da `presentation`. Sempre via use case da camada `domain`.
* Repository interfaces vivem em `domain/`. Implementações em `data/`.
* Use BLoC ou Cubit para gerenciamento de estado — sem setState em widgets complexos.
* Erros de rede devem propagar até a UI com mensagem significativa para o usuário.
* Cookie compartilhado com web via `Domain=.mastersindico.com.br`. Mobile faz `GET /v1/auth/me` no startup.
* Nunca armazene tokens em SharedPreferences — use `flutter_secure_storage`.
* Evite `!` (null assertion) — use `??`, `?.` ou trate o null explicitamente.

---

## DDD + Vertical Slices

* Cada sprint entrega uma fatia vertical completa: Go + SolidJS + Flutter juntos.
* Bounded contexts: Identity, Billing, Institutional, Commercial, Content, Assembly. Não misture lógica entre contextos.
* Entidades de domínio não importam nada de `infrastructure/`. Dependências apontam para dentro: `infrastructure → application → domain`.
* Eventos de domínio (`IDomainEvent`) são o mecanismo de comunicação entre bounded contexts — nunca chamadas diretas entre use cases de contextos diferentes.
* Connect Me não é chat. É formulário unidirecional. Sem reply, sem histórico, sem thread.
* Cada módulo é autocontido: tem seu próprio `domain/`, `application/`, `infrastructure/`. O `main.go` apenas registra os módulos — não conhece detalhes internos.
* CQRS dentro de cada módulo: commands (escrita) e queries (leitura) são use cases separados. Nunca misture os dois num mesmo use case.

---

## Banco de Dados (PostgreSQL)

* PKs: UUIDs.
* Nomenclatura: `snake_case`.
* Toda tabela tem `created_at`, `updated_at`, `deleted_at` (soft delete) — exceto tabelas imutáveis por design (`timeline_entries`, `audit_trail`, `assembly_votes`).
* Valores monetários: `INTEGER` (centavos).
* Fração ideal: `DECIMAL/NUMERIC(10,6)` — nunca `FLOAT` ou `DOUBLE`.
* Migrations são versionadas e nunca destrutivas em produção.

---

## Segurança

* Rate limiting em todas as rotas de auth.
* CAPTCHA para tentativas suspeitas de login.
* Signed URLs para arquivos privados (TTL curto).
* Verificação de assinatura de webhook para Stripe e Mux.
* Log LGPD obrigatório no Connect Me: `timestamp`, `ip`, `consent_version`, `data_revealed_at`.
* Audit trail de todas as ações críticas: `actor_id`, `action`, `resource_type`, `resource_id`, `tenant_id`, `metadata`.
* Anti-fraude: bloqueia múltiplos cadastros do mesmo IP, CAPTCHA em tentativas suspeitas.
* 1 device ativo por conta — fingerprint + IP. Login em segundo device invalida sessão anterior.

---

## Testes

Meta: ~90% de cobertura, mas **não inicialmente** — testes entram após a implementação base estar sólida.

Tipos prioritários:
- Testes de integração (fluxos completos por bounded context)
- Property-based testing para regras críticas (fração ideal, quotas, ABAC)
- Benchmarks de performance para WebSocket (Assembly) e queries críticas

---

## Pull Request

* Título: imperativo, capitalizado, sem prefixos convencionais (`fix:`, `feat:`), sem pontuação final.
* Prefixo opcional com o domínio quando o escopo é claro (ex: `assembly: Add quorum simulator`).
* Inclua seção `Release Notes:` no final do corpo:
  - `- Added ...`, `- Fixed ...`, ou `- Improved ...` para mudanças visíveis ao usuário
  - `- N/A` para mudanças internas

```
Release Notes:

- N/A
```

---

## Plano de Execução — 8 Sprints de 14 dias

Organização: vertical slices (Go + SolidJS + Flutter juntos por sprint).
Equipe: Dev1 (full-stack/arquiteto) + Dev2 (mobile/web). Dev1 sempre 1 sprint à frente no backend.

### Sprint 0 — Fundação (23/03 → 06/04)

Dev1: Monorepo, Zitadel, DB, Redis, Core contracts, CI/CD
Dev2: Flutter setup, OIDC integration
Background: OpenSearch cluster + Docker compose

- Setup monorepo web (bun workspaces, biome, tsconfig, turbo)
- Setup Go Clean Architecture (`cmd/`, `internal/core/contracts`, `internal/core/errors`, `internal/modules/{context}/domain`, `internal/modules/{context}/application`, `internal/modules/{context}/infrastructure`, `internal/shared/`, `pkg/`)
- PostgreSQL + migrations base: `users`, `sessions`, `tenants`, `plans`, `subscriptions`
- Redis: conexão, helpers de cache, TTL patterns
- Zitadel: Railway (ou docker-compose + Traefik em dev local, já rodando), `auth.mastersindico.com.br`, SSL, Organization + Projects + Roles
- Core contracts Go: `IUseCase[I,O]`, `AppError` + sentinels, `IUnitOfWork`, `IRepository[T,ID]`, `IDomainEvent`
- Gin abstraído via `contracts.HTTPRouter` + plugins: CORS, Recovery, RequestID, ErrorHandler, RateLimiter, Authenticate, RequirePermission (ABAC)
- CI/CD: lint + typecheck + build → Railway deploy automático no push main (commit 5114325)
- Railway Foundation: Postgres 18 managed, Redis 7 managed, MinIO S3-compatible volume, Dockerfile multi-stage distroless, domínios custom + SSL automático
- Flutter: Clean Architecture por feature, Dio + interceptors, flutter_secure_storage, Zitadel OIDC

### Sprint 1 — Identity & Billing (07/04 → 20/04)

Dev1: Billing, ABAC, Onboarding, Trial
Dev2: Flutter auth + onboarding
Background: OpenSearch index mappings + sync job

- Zitadel ↔ Go middleware BeyondCorp: JWT → validator → session (1 device, fingerprint + IP) → cookie httpOnly `.mastersindico.com.br`
- ABAC engine: avalia `(userId, tenantId, role, plan, action)` em cada request
- Stripe Adapter: `IPaymentGateway` → `StripeGateway`
- Plans + Subscriptions: schema, CRUD, webhook handler
- Trial logic: bloqueia ações estratégicas por persona (Síndico 15d, Empresa 7d, Parceiro 30d)
- Quotas + Feature Usage: tracking por feature, reset anual (Connect Me), reset mensal (uploads)
- Onboarding SolidJS: stepper, auto-save Redis, validação por step
- Telas auth SolidJS: Splash, Login, Cadastro, OTP, Recuperação, Seleção de Perfil
- Onboarding por persona SolidJS: Síndico (6 telas), Morador (4), Empresa (7), Parceiro (4)
- `TrialBlocker` component + upgrade → Stripe Customer Portal
- Flutter: telas auth + onboarding por persona

### Sprint 2 — Institutional I: Registry + Governance Base (21/04 → 04/05)

Dev1: Condominium Registry, Timeline, Master Plan
Dev2: Flutter home + timeline (leitura morador)
Background: OpenSearch queries + search UI base

- Condominium Registry Go: ID alfanumérico 8 chars, QR Code, mandato, limite 15 por síndico
- Unit Registry + Membership: 1 titular + 2 dependentes, termo de ciência, encerramento de vínculo
- Timeline Go: append-only, sem DELETE/UPDATE, adendo, filtros
- Timeline imutabilidade: record locked após publicação, `deleted_at` não existe nessa tabela
- Master Plan Go: status via evento `TimelinePublished` — NUNCA update manual
- Telas SolidJS: Cadastrar Condomínio, Dados da Gestão, Identidade (ID + QR)
- Telas SolidJS: Linha do Tempo, Criar Atividade (26 tipos, 9 riscos), Plano Diretor
- Telas SolidJS: Painel Morador, Selecionar Condomínio, Buscar por ID, Cadastro Unidade
- Flutter: Home morador, Timeline leitura, Plano Diretor leitura

**Marco 1: 08/05/2026** — Fundação + Identity + Institutional I entregues

### Sprint 3 — Institutional II: Governance Completa (05/05 → 18/05)

Dev1: Eventos, Comunicados, Validações, Dashboard, Compliance, Avaliação
Dev2: Flutter painel morador completo
Background: Mux video pipeline setup

- Eventos Go: CRUD + resposta (ciente/confirmado), 13 tipos, métricas
- Comunicados Go: CRUD + tracking de leitura, comunicado de empresa entra como pendente
- Validation Hub Go: lista pendentes, validate/reject/adjust → publica na Timeline
- Pergunta aos Moradores: 4 tipos de resposta, resultado → Timeline
- Avaliação da Gestão: 3 perguntas, escala 1-10, anônima, bimestral obrigatória
- Compliance Go: termo de responsabilidade, declaração anual, audit trail append-only (5 anos LGPD)
- Dashboard Síndico: 13 indicadores, filtros por período
- Telas SolidJS: Eventos, Comunicados, Validações Pendentes, Dashboard, Compliance hub
- Flutter: painel morador completo, push notifications (FCM)

### Sprint 4 — Commercial: Marketplace (19/05 → 01/06)

Dev1: Connect Me, Service & Contract, Company Profile
Dev2: Flutter painel empresa
Background: Email templates (Resend — Sprint 9)

- Connect Me Go: 4 fluxos (Síndico→Empresa, Morador→Síndico frio, Empresa→Empresa Plus/Pro, Síndico→Parceiro)
  - Auto-close 48h, quotas anuais Redis, log LGPD obrigatório
  - Botão denúncia: log + notifica admin + ações (warning/suspension/ban)
- Service & Contract Go: execution_records, technical_activities, comunicados pós-deal
  - Submit/validate/reject/adjust workflows → evento `ExecutionSubmitted` → `PendingValidation`
- Proposals + Supplier Voting: quórum configurável, resultado real-time
- Closed Deal: dupla assinatura → `is_immutable=true` → auto-publicação Timeline
- Company Profile + Users: 2 roles (admin + operador técnico), regras de conteúdo
- Empresa Síndica Vinculada: invite via token, 5 permissões parametrizáveis, audit trail
- Telas SolidJS: Connect Me hub, Painel Empresa (E1-E12), Empresa Síndica
- Flutter: Home empresa, Oportunidades, Registrar Execução (câmera), Dashboard, Connect Me

### Sprint 5 — Content: Vídeos + Search (02/06 → 15/06)

Dev1: Video Library + Mux, Player + Paywall, OpenSearch queries
Dev2: Flutter vídeo + search
Background: FCM push notifications setup

- Mux Adapter Go: `IVideoProvider` → `MuxProvider` (upload signed URL, webhook, HLS, thumbnail)
- Video Library Go: trava trimestral (90 dias), limites mensais por plano, 12 tipos, `autorizar_exibicao_votacao`
- Regra de visibilidade: vídeos de empresa blindados a moradores — liberados só em votação de fornecedor
- Player SolidJS: HLS.js, paywall (corta em 25% para Base), contagem de view (>70% = registra)
- Agência de Marketing: invite via token, permissões restritas (só vídeos + analytics)
- Telas SolidJS: Painel Agência (MK1-MK8), Gestão Agência (E13-E16)
- OpenSearch queries Go: swap PostgreSQL tsvector → OpenSearch; índices companies, sindicos, parceiros, videos
- Search SolidJS: autocomplete, filtros laterais, middleware de visibilidade por plano
- Flutter: player nativo, upload vídeo/foto, busca com filtros

**Marco 2: 20/06/2026** — Governance + Commercial + Content entregues

### Sprint 6 — Integrações (16/06 → 29/06)

Dev1: Push, SMS, Email, PDF, Mandato, Avaliação Empresa
Dev2: Flutter paridade completa
Background: PDF generator templates

- Push Notifications Go: FCM adapter, `IPushProvider`, device token management
- Twilio SMS Go: `ISMSProvider` → `TwilioProvider`, OTP, notificações transacionais
- Email Go: Resend adapter (`github.com/resend/resend-go/v2`), `IEmailProvider`, templates pt-BR (verificação, Connect Me, notificações assembleia, broadcasts, boas-vindas, billing alerts)
- CEP Lookup Go: ViaCEP adapter, `ICEPLookup`, caching Redis
- PDF Generator Go: dossiê exportável, relatório de gestão
- Mandato Go: encerrar (requer compliance em dia), transferir (passa histórico, não passa avaliações pessoais)
- Company Evaluation Go: 5 critérios escala 1-10, anônima para empresa, pública para outros síndicos
- Post-Deal Comunicados: 5 tipos, validação síndico → Timeline + notifica moradores
- Telas SolidJS: proposta, votação, negócio fechado (dupla confirmação), avaliação
- Flutter: paridade completa com web (Sprints 1-5), push notifications integradas

### Sprint 7 — Assembly + Polish (30/06 → 13/07)

Dev1: Assembly assíncrona, bugs acumulados, performance
Dev2: Flutter assembly + QA
Background: OpenTelemetry monitoring

- Assembly Go — Pré:
  - Configuração (tipo, modalidade presencial MVP, quórum, tempo de fala)
  - Pauta com lock jurídico após publicação, fração ideal via Excel (soma = 100%)
  - Edital (gerador automático PDF), ciência bloqueante (tracking 6 meses)
  - Procurações (validação, 24h pós-votação), enquetes preliminares, simulador de quórum
- Assembly Go — Live-day:
  - Check-in (app, QR Code, terminal CPF+bloco+unidade)
  - Votação: simples, fração ideal (`DECIMAL/NUMERIC` — nunca float), fornecedor, eleição
  - Voto presencial assistido com `termo_de_voto`, ausente momentâneo = abstenção automática
  - Auto-save contínuo no Redis (recuperação de crash)
- Assembly Go — Pós:
  - Ata automática, 6 tipos de relatório (CSV + PDF)
  - Score de transparência 0-100 (10 componentes), auto-publicação na Timeline
  - Homologação obrigatória → imutável após homologação
- Telas SolidJS: configuração, pauta (drag-and-drop), edital, ciência, procurações, simulador, votação, relatórios
- Bug fixes acumulados: multi-tenancy, imutabilidade Timeline, PD reativo, trial retroativo, quotas Connect Me
- Performance tuning: queries N+1, index optimization, Redis caching, connection pooling
- Flutter: telas de assembly, QA completo, testes em dispositivos reais (iOS + Android)
- OpenTelemetry: traces, metrics, logs → CloudWatch

**Marco 3: 20/07/2026** — Integrações + Assembly + Polish entregues

### Sprint 8 / Buffer — QA + Deploy Prod (14/07 → 08/08)

Dev1 + Dev2: Testes E2E, security review, deploy produção, monitoring

- E2E: fluxo síndico (cria condo → timeline → PD → eventos → connect me → deal → execução → avaliação)
- E2E: fluxo morador (vincula → vê timeline → avalia → connect me → votação)
- E2E: fluxo empresa (recebe oportunidade → proposta → deal → execução → validação → Timeline)
- E2E: billing (trial → expiração → bloqueio retroativo → upgrade → desbloqueio)
- Testes ABAC: cada role vê apenas o que deve; operador técnico vs admin; cross-tenant isolation
- Security review BeyondCorp: 1 device, cookie httpOnly, rate limiting, LGPD logs, audit trail
- Deploy produção: RDS Multi-AZ, ECS auto-scaling, Zitadel prod, Stripe live mode, feature flags, smoke tests
- Monitoring: CloudWatch dashboards, alarmes críticos, runbooks operacionais
- Flutter: submit TestFlight (iOS) + Google Play Internal Testing (Android)
- Handoff docs: API docs (OpenAPI/Swagger), README, runbook de deploy

---

## Pós-Lançamento (Sprints 9+)

- Assembly ao vivo completa: WebSocket (gorilla/websocket), telão 2 áreas, fila de fala com cronômetro, votação real-time, contingência, NATS JetStream para fan-out — 8-10 semanas
- Academia LMS: 15 telas, 5 áreas (cursos, treinamentos, lives, fórum, biblioteca), 3 trilhas — 4-5 semanas
- Vizinhança completa: 29 telas, 4 jornadas (Síndico, Empresa, Morador, Parceiro) — 3-4 semanas
- Banco de Talentos: 11 telas morador + 7 funcionalidades empresa, vídeo-currículo 90s — 3-4 semanas
- Painel Admin MasterSíndico: gestão global de tenants, moderação, financeiro, broadcasts — 3-4 semanas
