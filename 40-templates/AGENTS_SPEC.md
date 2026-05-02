---
title: AGENTS.md SPEC — contrato dos agentes operacionais
type: spec
tags:
  - template
  - spec
  - agents
  - operational
created: '2026-04-22'
updated: '2026-04-27'
---
# AGENTS_SPEC.md

Template e guia para produzir um `AGENTS.md` de qualquer projeto. Extraído de um corpo real de coding guidelines multi-stack (Go backend + SolidJS web + Flutter mobile) e destilado em formato replicável.

Este arquivo tem três camadas:
1. **Princípios** — o porquê; regras que não mudam entre projetos
2. **Estrutura canônica** — seções que todo `AGENTS.md` deve ter
3. **Templates** — snippets prontos pra copiar, parametrizados por stack

---

## 1. Princípios

### 1.1 Papel do agente

`AGENTS.md` define **comportamento**, não só convenções. Cabeçalho obrigatório declarando o papel em camadas:

- Arquiteto de Soluções Sênior
- Desenvolvedor Sênior no stack do projeto
- DevOps Sênior
- Especialista em Cibersegurança aplicável (LGPD/GDPR/HIPAA/PCI conforme domínio)
- Engenheiro de Software com foco em `<arquitetura-alvo>` (DDD / Clean Arch / Hexagonal / conforme projeto)

O agente **não apenas executa** — guia tecnicamente enquanto o dev escreve. Sempre:
1. Explica antes de codar
2. Mostra o código
3. Justifica decisões técnicas
4. Aponta alternativas quando existem
5. Relaciona com boas práticas reais
6. Identifica riscos e inconsistências

Em revisão de código: como sênior. Avaliar arquitetura, legibilidade, padrões, SOLID/DDD, consistência, segurança. **Não avança se a base estiver ruim** — corrige primeiro.

### 1.2 Filosofia geral

- Corretude e clareza **acima de** performance (exceto onde especificado)
- Comentários explicam o **porquê**, nunca o quê
- Preferir editar arquivos existentes a criar novos
- Evitar adições criativas não solicitadas
- Nomes completos — sem abreviações (`subscription`, não `sub`; `queue`, não `q`)
- Toda regra de negócio na camada de aplicação/domínio. UI é consumo.

### 1.3 Evitar

- `if/else` hell — early returns, guard clauses
- Código procedural sem coesão — cada função uma responsabilidade
- Abstrações sem dois usuários reais (YAGNI)
- `try/catch` que engole erros sem contexto
- Funções com >3–5 parâmetros — consolidar em struct/objeto
- Arquivos com múltiplos use cases ou handlers — um por arquivo

### 1.4 Análise contínua

Em toda sessão:
- Detectar inconsistências no projeto
- Avaliar viabilidade técnica das decisões ativas
- Identificar riscos **antes** de implementar
- Sugerir melhorias quando identificar débito técnico
- Comparar com padrões reais de system design

---

## 2. Estrutura canônica do AGENTS.md

Seções obrigatórias, nesta ordem:

```
1.  Papel do Agente                  ← quem você é neste projeto
2.  Projeto                          ← domínio, stack, arquitetura em uma linha
3.  Filosofia Geral                  ← princípios invariantes
4.  Análise Contínua                 ← o que avaliar toda sessão
5.  Regras por Stack                 ← uma seção por camada (backend/web/mobile/...)
6.  Arquitetura Cross-Cutting        ← DDD, vertical slices, bounded contexts
7.  Banco de Dados                   ← naming, PKs, soft-delete, tipos numéricos
8.  Segurança                        ← rate-limit, audit, webhooks, regulatório
9.  Testes                           ← estratégia, thresholds, tipos prioritários
10. Pull Request                     ← formato do título, body, release notes
11. Plano de Execução                ← sprints ou roadmap (opcional)
```

---

## 3. Templates por seção

### 3.1 Papel do Agente (template)

```markdown
## Papel do Agente

Você atua como:
- Arquiteto de Soluções Sênior
- Desenvolvedor Fullstack Sênior (<linguagens/frameworks>)
- DevOps Sênior
- Especialista em Cibersegurança (<regulação aplicável>)
- Engenheiro de Software com foco em <arquitetura>

Seu papel **não é apenas executar** — é guiar tecnicamente enquanto o desenvolvedor escreve o código.

Sempre:
1. Explique antes de codar
2. Mostre o código
3. Justifique decisões técnicas
4. Aponte alternativas quando existirem
5. Relacione com boas práticas reais
6. Identifique riscos e inconsistências
```

### 3.2 Projeto (template)

```markdown
## Projeto

<descrição em uma linha>. Stack: **<lang1>** · **<framework-web>** · **<framework-mobile>** · **<auth>** · **<pagamento>** · **<storage>** · **<cache>** · **<queue>** · **<banco>**.

Arquitetura: **<nome-do-modelo>** — <princípios-duros em uma linha>.
```

### 3.3 Backend em linguagem tipada — Clean Architecture + DDD + CQRS

Padrão aplicável a Go, Rust, C#, Kotlin, Java, TypeScript tipado, Scala, etc.

```
cmd/<api>/main.go                    ← entry: bootstrap, DI wire-up, graceful shutdown

internal/ (ou src/)
  core/
    contracts/                       ← interfaces base: IUseCase, IRepository, IUnitOfWork, IDomainEvent
    errors/                          ← DomainError + sentinels (NotFound, Unauthorized, Forbidden, Conflict, Business)

  modules/
    <bounded-context>/
      domain/
        entities/                    ← rich domain models; estado privado, getters/métodos expostos
        value_objects/               ← imutáveis, auto-validados na factory (Create/From/New)
        enums/
        repositories/                ← interfaces (ports); impl vai em infrastructure/
        services/                    ← domain services puros; sem I/O
        events/                      ← IDomainEvent implementations
      application/
        use_cases/                   ← CQRS: commands e queries em arquivos separados; um Execute por caso
        mappers/                     ← entity → DTO de resposta
      infrastructure/
        database/                    ← repo impl (driver nativo ou geração de código tipada)
        http/
          routes/                    ← handlers thin (parse → use case → serialize)
          middleware/                ← hooks do módulo
        providers/                   ← adapters externos (gateway, OIDC validator, etc.)
        jobs/                        ← cron do módulo

  shared/
    config/                          ← config carregado + validado no startup
    middleware/                      ← cadeia cross-módulos (auth, rate limit, etc.)
    providers/                       ← interfaces globais: IEmailProvider, ICacheProvider, IPaymentGateway...
    types/                           ← ApiResponse, PaginatedResponse, AuthUser, etc.

pkg/ (ou lib/)
  logger/                            ← structured logger wrapper
  money/                             ← value object Money (centavos em inteiro de 64 bits)
  utils/                             ← id generator (UUIDv7), validators, parsers
```

Regras absolutas:

- **Dependency rule**: `infrastructure → application → domain`. Domain não importa nada das outras camadas.
- **Handlers não importam o framework HTTP diretamente**. Abstrair via interface (`contracts.HTTPRouter`/`HandlerFunc`). O adapter do framework vive num módulo isolado.
- **Um use case por arquivo**. Um handler por recurso HTTP.
- **Interfaces prefixadas com `I`** em linguagens que separam interface de struct (Go/TS/Java). Em Rust/C# usar convenção idiomática.
- **Compile-time assertion** que struct implementa interface (ex: Go `var _ contracts.IUseCase[I,O] = (*MyUseCase)(nil)`).
- **Erros são tipos concretos**, nunca strings. Usar `errors.Is/As` (ou equivalente). Handler global mapeia para HTTP status.
- **`context.Context`** (ou equivalente) como primeiro parâmetro de função com I/O.

### 3.4 Module Pattern (template)

```go
// Contrato que todo módulo implementa
type Module interface {
    Register(router HTTPRouter, container DIContainer) error
    Bootstrap(container DIContainer) error // cron, event subs, startup tasks
}
```

`main.go` só instancia módulos e chama `Register/Bootstrap`. Não conhece internals.

### 3.5 Use Case / CQRS (template)

```go
// Command (escrita)
type CreateXCommand struct {
    Field1 string
    Field2 Money
}

// Use Case (dependências via construtor — DI)
type CreateXUseCase struct {
    xRepository IXRepository
    unitOfWork  IUnitOfWork
    logger      Logger
}

func (uc *CreateXUseCase) Execute(ctx Context, cmd CreateXCommand) (*XDTO, error) {
    // sem acesso direto a banco, sem HTTP, sem SDK externo aqui
}
```

Commands e queries **em arquivos separados**. Nunca um use case mistura os dois.

### 3.6 Entity (template)

Estado interno **privado**. Getters e métodos de domínio explícitos.

```go
type Session struct {
    id        string
    userID    string
    expiresAt time.Time
}

func (s *Session) IsExpired() bool      { return s.expiresAt.Before(time.Now()) }
func (s *Session) Extend(d Duration)    { s.expiresAt = s.expiresAt.Add(d) }
func (s *Session) Invalidate()          { s.expiresAt = time.Now() }
```

Nunca expõe campo público diretamente.

### 3.7 Value Object (template)

Imutável, auto-validado no construtor.

```go
type Email struct{ value string }

func NewEmail(raw string) (Email, error) {
    if !isValidEmail(raw) {
        return Email{}, NewValidationError("email inválido")
    }
    return Email{value: strings.ToLower(raw)}, nil
}

func (e Email) Value() string { return e.value }
```

Em Dart: `factory Email.parse(String raw)` com `throw FormatException`. Em TypeScript: branded type `type Email = string & { __brand: "Email" }` + parser Zod.

### 3.8 Repository Interface (template)

Interface no domínio. Impl na infra. Use case depende só da interface.

```go
// domain/repositories/x_repository.go
type IXRepository interface {
    FindByID(ctx Context, id string) (*X, error)
    FindAllByOwnerID(ctx Context, ownerID string) ([]*X, error)
    Save(ctx Context, x *X) error
    DeleteByID(ctx Context, id string) error
}
```

### 3.9 Unit of Work (template)

Atomicidade cross-repo numa transação.

```go
type IUnitOfWork interface {
    Run(ctx Context, fn func(ctx Context) error) error
    GetDB(ctx Context) *Conn // tx se dentro de Run, pool caso contrário
}
```

Use cases chamam `unitOfWork.Run(ctx, func(ctx Context) error { ... })` quando escrevem em múltiplos repos.

### 3.10 Domain Events (template)

**Comunicação entre bounded contexts** — nunca chamada direta entre use cases de contextos diferentes.

```go
type IDomainEvent interface {
    EventName() string
    OccurredAt() time.Time
}

type EntityCreated struct {
    EntityID   string
    OwnerID    string
    occurredAt time.Time
}
```

Publicação: bus in-memory no MVP; broker (NATS/Kafka/RabbitMQ) quando escalar.

### 3.11 Provider Interfaces (template)

Toda integração externa via interface. **Trocar provider = nova implementação, zero mudança no domínio**.

```go
// Correto
type IPaymentGateway interface {
    CreateSubscription(ctx Context, input CreateSubscriptionInput) (*Subscription, error)
    CancelSubscription(ctx Context, subscriptionID string) error
}

// Errado: use case importa SDK vendor diretamente
```

Interfaces típicas: `IPaymentGateway`, `IEmailProvider`, `IStorageProvider`, `IVideoProvider`, `ILiveProvider`, `ISMSProvider`, `IPushProvider`, `ICacheProvider`, `IOIDCProvider`, `ISearchProvider`.

### 3.12 HTTP Handler (template)

Thin. Parse → use case → serialize. Sem lógica de negócio.

```go
func (h *XHandler) Create(c Context) {
    var cmd CreateXCommand
    if err := c.DecodeJSON(&cmd); err != nil {
        c.SetError(NewValidationError(err)); return
    }
    output, err := h.createX.Execute(c.Request().Context(), cmd)
    if err != nil {
        c.SetError(err); return // error handler global escreve resposta
    }
    c.JSON(200, ApiSuccess(output))
}
```

### 3.13 Error Handling (template)

Erros de domínio são tipos concretos com código + status. Mapeados por handler global.

```go
type DomainError struct {
    Message    string
    Code       string
    StatusCode int
}

var (
    ErrNotFound     = &DomainError{Code: "NOT_FOUND",     StatusCode: 404}
    ErrUnauthorized = &DomainError{Code: "UNAUTHORIZED",  StatusCode: 401}
    ErrForbidden    = &DomainError{Code: "FORBIDDEN",     StatusCode: 403}
    ErrConflict     = &DomainError{Code: "CONFLICT",      StatusCode: 409}
    ErrBusiness     = &DomainError{Code: "BUSINESS_ERROR",StatusCode: 400}
)
```

Regras:
- Nunca descartar erro (`_ = fn()` em op falível é proibido)
- Propagar com contexto: `fmt.Errorf("operação X: %w", err)`
- Nunca comparar string de erro — `errors.Is/As`
- `panic` só em `init()` para config irrecuperável

### 3.14 Middleware Stack (template)

Ordem fixa, não-negociável:

```
Recovery → RequestID → Logger → CORS → ErrorHandler → RateLimiter
  → Authenticate → Authorize(ABAC/RBAC) → Handler → AuditLogger
```

`Authenticate` popula contexto com usuário. `Authorize` avalia `(userID, tenantID, role, plan, action)` e rejeita 403 se negar. **Toda query multi-tenant filtra por `tenantID` vindo do contexto** — nunca do request body.

### 3.15 Imutabilidade (regras canônicas)

Tabelas imutáveis por design (sem `deleted_at`, sem UPDATE):
- `audit_trail` — append-only, retenção conforme regulação
- `<evento-publicado>` — ex: entradas de linha do tempo; modificação só via adendo (novo registro linkado)
- `<registro-votação-homologada>` — imutável após aprovação
- `<contrato-fechado-assinado>` — `is_immutable=true` flag

Estado derivado atualizado **apenas via evento**, nunca por update manual.

### 3.16 Frontend web (template)

```
apps/
  <app-1>/         ← um SPA por bounded context ou função (auth, admin, marketing, etc.)
  <app-N>/
packages/
  ui/              ← design system (primitivos + variantes + tokens)
  schemas/         ← validação compartilhada (Zod/Yup/io-ts)
  core/            ← SDK stateless (http client, helpers puros) — opcional
```

Regras:
- **Auth context vive em cada app**, não no core. Core é stateless.
- **Credentials no cookie httpOnly** (`credentials: 'include'` em toda chamada). Nunca `localStorage`/`sessionStorage`.
- Server state: TanStack Query (ou equivalente). UI state local: signals/stores nativos do framework.
- Forms: lib tipada + validator schema-based.
- Componentes acessíveis: lib headless (Kobalte/Radix/Ariakit) + CVA/variance-authority.
- **Paleta imutável**: mudança de tokens de cor requer aprovação explícita.

Visibility engine: frontend exibe o que backend retorna. **Nunca calcula quotas, prazos, permissões** — só mostra flags.

### 3.17 Mobile (template)

Feature-first + Clean Architecture:

```
lib/ (ou src/)
  app/              ← bootstrap, DI, router
  core/             ← network (http client + interceptors), auth, storage seguro
  features/
    <feature>/
      data/         ← repository impl, datasources
      domain/       ← entities, use cases, repository interfaces
      presentation/ ← state mgmt (Bloc/Riverpod/etc.), screens, widgets
```

Regras:
- **Dependency rule**: `presentation → domain ← data`. Features cruzam via `core/` ou `domain/entities/` de outra feature — nunca importando `data/` ou `presentation/` alheia.
- **Erros nunca propagam como exceção além de `data/`**. Datasources lançam exceção tipada; `RepositoryImpl` é o único lugar com `try/catch` — converte em `Failure` e retorna `Either<Failure, T>`.
- **Tokens em secure storage nativo** (Keychain iOS / Keystore Android). Nunca em SharedPreferences/UserDefaults.
- DI com anotações semânticas:
  - Singleton/LazySingleton → Repositories, Datasources, HTTP client, NetworkInfo
  - Factory/Injectable → state mgmt containers e use cases (fresh por resolução; scopados à página)
  - Module → terceiros que não aceitam anotação
- **Certificate pinning** em produção.
- **Detecção de jailbreak/root** básica. Reportar ao backend, não bloquear sem justificativa.
- LGPD/GDPR: modal de consentimento antes de revelar dados pessoais; opção exportar/deletar; zero analytics sem opt-in.

### 3.18 DDD + Vertical Slices (regras cross-cutting)

- **Cada sprint entrega vertical slice completo**: backend + web + mobile juntos.
- **Bounded contexts** listados no `AGENTS.md` do projeto. Nunca misturar lógica entre contextos.
- **Entidades nunca importam infra**. Dependências apontam para dentro.
- **Eventos de domínio** são o único mecanismo de comunicação entre contextos.
- **CQRS dentro de cada módulo**: commands e queries separados.

### 3.19 Banco de Dados (regras universais)

- **PKs**: UUIDv7 (ordenável por tempo, amigável a índices)
- **Nomenclatura**: `snake_case` em SQL, `camelCase`/`PascalCase` no código (conforme linguagem)
- **Timestamps**: `created_at`, `updated_at`, `deleted_at` (soft delete) — **exceto tabelas imutáveis por design**
- **Valores monetários**: inteiro de 64 bits (centavos). **Nunca float.**
- **Valores fracionais com regra de negócio** (ex: fração ideal, proporção de voto): `DECIMAL/NUMERIC(P,S)`. **Nunca `FLOAT`/`DOUBLE`.**
- **Migrations** versionadas, embarcadas no binário quando possível (`embed.FS` em Go, resources em JVM, etc.). **Nunca destrutivas em produção** — sem `DROP TABLE`/`DROP COLUMN`.
- **Numeração de migration particionada por módulo** em projetos com múltiplos contextos (ex: módulo A 001–099, módulo B 100–199).

### 3.20 Segurança (regras universais)

- **Rate limiting** em todas as rotas de auth e endpoints caros.
- **CAPTCHA** em tentativas suspeitas de login/cadastro.
- **Signed URLs com TTL curto** para arquivos privados.
- **Verificação de assinatura HMAC** em todo webhook vendor (Stripe, Mux, GitHub, etc.).
- **Audit trail append-only** de ações críticas: `actor_id`, `action`, `resource_type`, `resource_id`, `tenant_id`, `metadata`, `ip`, `user_agent`.
- **Logs LGPD/GDPR**: `timestamp`, `ip`, `consent_version`, `data_revealed_at` sempre que dados pessoais são expostos.
- **Anti-fraude**: bloquear múltiplos cadastros do mesmo IP + CAPTCHA em suspeita.
- **1 dispositivo ativo por conta** (quando aplicável ao modelo): fingerprint + IP, login em novo device invalida anterior.
- **Secrets nunca em código** — sempre em env vars ou secret manager. Lint pré-commit detecta.
- **Tenant isolation**: filtro `WHERE tenant_id = $ctx.tenantID` em **toda** query. Cross-tenant apenas via grant explícito.

### 3.21 Testes

Meta de cobertura: **~90% global**, mas **não inicialmente**. Testes entram após a base estar sólida.

Thresholds por camada (valores típicos):

| Camada | Alvo | Motivo |
|---|---|---|
| `domain/` (entities, VOs, services) | **95%** | Regras de negócio — todos os paths |
| `application/` (use cases, orquestração) | **90%** | Máquina de estados de negócio |
| `infrastructure/repositories/` (único try/catch) | **85%** | Success + failure paths |
| `infrastructure/datasources/` | **80%** | Mock de I/O, erros de rede |
| `presentation/` (handlers, screens) | **70–75%** | Coberto também por integração/E2E |
| `core/` / `pkg/` (shared) | **90%** | Infra reusável |
| **Global** | **≥ 80–85%** | Sanidade |

Tipos prioritários:
1. **Integration tests** — fluxos completos por bounded context, contra DB real
2. **Property-based testing** para regras críticas (quotas, ABAC, cálculos monetários, invariantes)
3. **Benchmarks** para hot paths (WebSocket, queries críticas, loops de processamento)
4. **Golden tests** para componentes de design system (pixel-perfect)
5. **E2E** nos fluxos principais de negócio (por persona/role)

Regras:
- **Red-Green-Refactor sempre**. Teste antes do código.
- **Arrange / Act / Assert** explícito.
- **Mocks em helper compartilhado** — estender, não redeclarar.
- **Testes independentes** — sem ordem, sem estado compartilhado mutável.
- **Zero `any`** em teste. Usar generics do lib de testing.

### 3.22 Pull Request (template)

- **Título**: imperativo, capitalizado, sem prefixo convencional (`fix:`, `feat:`), sem pontuação final. Prefixo opcional com domínio quando escopo claro (`auth: Add PKCE flow`).
- **Body**: seção `Release Notes:` obrigatória no final:
  - `- Added <user-facing change>`
  - `- Fixed <user-facing bug>`
  - `- Improved <user-facing improvement>`
  - `- N/A` para mudança interna

```
Release Notes:

- N/A
```

### 3.23 Code Calisthenics (9 regras, adaptáveis)

Cross-linguagem:

1. **Um nível de indent por método** — guard clauses, extrair helpers
2. **Sem `else`** — ternário ou early return; `else if` encadeado → Strategy/Map
3. **Envolver primitivos** — `UserId`, `Email`, `Money` como tipos próprios com validação
4. **First-class collections** — `VoteCollection` em vez de `List<Vote>` solto; invariantes junto
5. **Um ponto por linha** (Lei de Demeter) — `user.profile.address.city` viola; método no agregado (`user.cityName()`)
6. **Não abreviar** — idiomáticas OK (`i` em loop curto, `ctx`, `e` em catch/stack trace)
7. **Classes/widgets pequenos** — teto ~100 linhas; acima: extrair sub-componentes
8. **Máx 3–5 parâmetros** — consolidar em struct/config/props object
9. **Sem setters em entidades** — imutabilidade + `copyWith`/builder

---

## 4. Catálogo de Runtime Antipatterns (OP-###)

Catálogo cross-linguagem de **antipatterns operacionais** (comportamento em produção: concorrência, idempotência, cache, acoplamento, observabilidade, segurança operacional, performance). Cada entrada tem ID estável (`OP-XXX`), sintoma, fix preferido e alternativas, e severidade. Reviewer cita o ID no review (ver §9). Usar este catálogo como ponto de partida do antipattern-list de cada projeto — adicionar novos `OP-XXX` específicos do domínio no próprio `AGENTS.md`.

> **Distinção importante** (reconciliação 2026-04-24, D-vault-012): este catálogo `OP-###` cobre **runtime antipatterns**. Code smells clássicos Fowler/GoF (Long Method, God Object, Primitive Obsession, etc.) estão em `10-knowledge/antipatterns/AP-001..AP-023` — catálogo distinto e não-colidente. MOC navegacional em `10-knowledge/runtime-antipatterns/_moc.md`.

**Severidade:**
- **Critical** — segurança, perda de dados, duplicação financeira. Corrigir antes de merge.
- **High** — acoplamento que cresce, bug em produção, manutenibilidade grave.
- **Medium** — DX ruim, dívida técnica acumulável.

### Concorrência

#### OP-001 — Race condition em contador / quota

- **Sintoma:** `read → compute → write` sem lock. Threads concorrentes sobrescrevem valores.
- **Fix preferido:** UPSERT com incremento atômico no banco (`INSERT ... ON CONFLICT DO UPDATE SET count = count + 1 RETURNING count`). Equivalente NoSQL: `UpdateExpression SET count = count + 1` / `$inc`.
- **Alternativa:** distributed lock (Redis `SETNX` com TTL) quando preciso aplicar regra antes do write.
- **Nunca:** `SELECT FOR UPDATE` em hot path sem justificativa explícita — performance destroi.
- **Severity:** Critical

#### OP-002 — Check-then-act não atômico

- **Sintoma:** `if (not exists) insert` fora de transação. Segunda chamada entre `if` e `insert` quebra invariante.
- **Fix preferido:** constraint `UNIQUE` no banco + tratar violação (`ErrConflict`) em vez de pré-checar.
- **Alternativa:** `INSERT ... WHERE NOT EXISTS` ou `INSERT ... ON CONFLICT DO NOTHING`.
- **Severity:** High

### Idempotência

#### OP-003 — Webhook sem idempotência

- **Sintoma:** handler processa `event.id` sem checar se já foi processado. Provider reenvia → transação financeira duplicada.
- **Fix preferido:** tabela `webhook_events_processed(event_id PK, status, processed_at)`. Primeira linha do handler: `if repo.isProcessed(event.id) return 200`. Marcar como processado ao fim.
- **Complemento:** diferenciar erros retentáveis (timeout, DB down → rethrow → provider retenta) de não-retentáveis (validação, regra de negócio → marcar `failed` + retornar 200).
- **Severity:** Critical

#### OP-004 — Operação crítica sem retry strategy

- **Sintoma:** falhou → throw → provider ou worker retenta eternamente; ou falhou → swallow → perde o evento.
- **Fix preferido:** classificar erro (retentável vs não-retentável) + exponential backoff com teto + dead-letter queue após N tentativas.
- **Severity:** High

### Cache

#### OP-005 — Cache sem versionamento quando regra muda

- **Sintoma:** cache key `abilities:{userId}`. Admin muda permissões do plano, usuário continua com abilities antigas até TTL expirar.
- **Fix preferido:** hash das regras do plano como parte da key: `abilities:{userId}:{planPermissionsHash}`. Mudança de plano produz hash novo → cache miss automático.
- **Alternativa:** publicar evento `PlanPermissionsChanged` e invalidar pattern `abilities:*:{oldHash}`.
- **Severity:** Critical (segurança — permissões stale)

#### OP-006 — Cache falha quebra fluxo principal

- **Sintoma:** `cacheProvider.get()` lança exceção, fluxo quebra. Cache down = produto down.
- **Fix preferido:** try/catch em torno de `get`/`set`. Em falha: log warn + seguir para fonte de verdade (banco). Cache é otimização, não dependência dura.
- **Severity:** High (disponibilidade)

#### OP-007 — Cache key com JSON serializado gigante

- **Sintoma:** `search:${JSON.stringify(filters)}`. Filtros complexos explodem a key > 1KB. Redis aceita, mas fica patológico.
- **Fix preferido:** hash determinístico (MD5/SHA1) dos filtros: `search:{indexName}:{filtersHash}:{page}:{limit}`.
- **Severity:** Medium

#### OP-008 — Autorização avaliada apenas no cache hit

- **Sintoma:** cache retorna objeto com `role`, código valida ABAC contra `cached.role`. Se role mudou no banco, acesso pode ser indevido.
- **Fix preferido:** (a) incluir permissionsHash na cache key (OP-005) **ou** (b) sempre re-validar autorização contra source-of-truth mesmo em cache hit.
- **Severity:** Critical (segurança)

### Acoplamento / design

#### OP-009 — Vazamento de domínio entre módulos

- **Sintoma:** módulo B tem `switch (someEnumDoModuloA)`. Mudança no enum de A quebra B.
- **Fix preferido:** B define seu próprio enum/tipo; `A → B` passa por mapper explícito na camada de aplicação.
- **Severity:** High

#### OP-010 — Feature importa internals de outra feature

- **Sintoma:** `features/cart/presentation/` importa `features/auth/data/`. Quebra Feature Isolation.
- **Fix preferido:** compartilhar **só** via `core/` ou via `domain/entities/` pública da feature. Nunca `data/` ou `presentation/` alheia.
- **Severity:** High

#### OP-011 — Lógica de negócio em serviço de infra

- **Sintoma:** `PermissionService` na infra tem regras de ownership hardcoded (`if userRole === 'admin' grant all`).
- **Fix preferido:** mover regras para config (seed de permissões no banco) ou para um `OwnershipRulesProvider` testável. Infra executa, domínio decide.
- **Severity:** Medium

#### OP-012 — Fan-out desnecessário (N queries quando 1 basta)

- **Sintoma:** `Promise.all([repo1.find, repo2.find, ...repo5.find])` — todos buscam pelo mesmo id, 4 retornam null.
- **Fix preferido:** coluna discriminadora (`profile_type` em `users`) → 1 query direcionada. Ou `UNION ALL` com coluna identificando origem.
- **Severity:** Medium (performance)

### Duplicação

#### OP-013 — Switch/case gigante replicado em N lugares

- **Sintoma:** mesmo `switch` por tipo aparece em `Create*UseCase`, `Update*UseCase`, `GetPublic*UseCase`. Adicionar novo tipo = tocar N arquivos.
- **Fix preferido:** **Strategy Pattern** — `Map<Type, Strategy>`. Cada use case pede a strategy certa e delega. Novo tipo = nova classe, zero edit em existentes (Open/Closed).
- **Severity:** High

#### OP-014 — Construtor com 10+ parâmetros posicionais

- **Sintoma:** entidade/use case com 27 params. Trocar ordem por engano quebra silenciosamente.
- **Fix preferido:** **Builder Pattern** (`.withX().withY().build()`) **ou** objeto de configuração (`new Entity({x, y, z})`). Validação centralizada no `build()`.
- **Severity:** Medium

#### OP-015 — Validação repetida em N métodos

- **Sintoma:** `createA`, `createB`, `createC` cada um valida unicidade de documento na mão.
- **Fix preferido:** helper privado `validateDocumentUniqueness(doc, type)` que recebe tipo e delega pro repo certo. DRY.
- **Severity:** Medium

### Null-safety / validação

#### OP-016 — Force-unwrap / non-null assertion em toda mapeamento

- **Sintoma:** `raw.street!` / `raw.city!` em `toDomain`. Se qualquer campo for `null`, explode em runtime.
- **Fix preferido:** validar **todos** os campos obrigatórios antes do mapeamento; retornar `null` ou lançar erro tipado se incompleto. **Nunca** desabilitar lint de null safety em arquivo inteiro (`biome-ignore`, `@SuppressWarnings`, `// @ts-ignore`).
- **Severity:** Critical (NPE em produção)

#### OP-017 — Mapeamento parcial falha silenciosamente

- **Sintoma:** `hasAddress = raw.street && raw.zipCode`; se `city` for null, monta objeto `Address` meio preenchido.
- **Fix preferido:** validar **todos** os campos obrigatórios antes. Se falta qualquer um, retornar `null` (não há address) em vez de objeto quebrado.
- **Severity:** High

### Estado / invariantes

#### OP-018 — Transição de state machine sem guards

- **Sintoma:** `VALID_TRANSITIONS[from].includes(to)` sem validação adicional. Permite `paused → active` sem payment method.
- **Fix preferido:** cada transição chama guard function que valida invariantes (`canPause(): assert hasPaymentMethod`). Biblioteca de state machine (XState, Stately) ou guards manuais no método.
- **Severity:** Medium

#### OP-019 — Mutação de campo derivado sem validação

- **Sintoma:** `renewPeriod(start, end)` aceita `start >= end`, cria período inválido.
- **Fix preferido:** validar invariante no setter (`if (start >= end) throw ValidationError`). Value Object `Period { start, end }` com construtor validador elimina a classe inteira de bugs.
- **Severity:** Medium

### Observabilidade

#### OP-020 — Erro engolido silenciosamente

- **Sintoma:** `_ = fn()`, `catch {}` vazio, `return []` quando dep falta. Usuário vê "nada" em vez de erro claro.
- **Fix preferido:** propagar erro tipado. Se precisa mesmo retornar vazio por design (fail-open), logar explicitamente com nível `error` + razão.
- **Severity:** High (debug impossível em prod)

#### OP-021 — Ação de admin sem audit trail

- **Sintoma:** `if (role === 'admin') allow all` — sem log, sem trilha.
- **Fix preferido:** todo bypass de permissão gera entrada em `audit_trail` com `actor_id`, `action`, `resource`, `reason`, `ip`, `user_agent`, `timestamp`.
- **Severity:** High (compliance, LGPD/GDPR/SOX)

#### OP-022 — Log com PII / payload sensível sem sanitização

- **Sintoma:** `logger.info({webhook: event.data})` — payload contém cartão, token, CPF.
- **Fix preferido:** função `sanitize(payload)` remove campos sensíveis antes de logar. Lista allow de campos logáveis, não block.
- **Severity:** Critical (vazamento de dados, LGPD)

### Segurança

#### OP-023 — Webhook sem validação de assinatura HMAC

- **Sintoma:** handler aceita qualquer POST como válido. Atacante forja eventos.
- **Fix preferido:** provider expõe `verifyWebhookSignature(rawBody, signature, secret)`. Primeira linha do handler. Sem signature válida → 401.
- **Severity:** Critical

#### OP-024 — Rate limit ausente em endpoint caro ou exposto

- **Sintoma:** webhook, endpoint de busca, login — sem rate limit. Alvo de DoS / brute force.
- **Fix preferido:** rate limiter na cadeia de middleware (global + override por rota). Em webhook: key por `signature` ou `IP`.
- **Severity:** High

### Performance

#### OP-025 — Coluna com query frequente sem índice

- **Sintoma:** `findByUserId` executa full scan em tabela grande.
- **Fix preferido:** índice composto `(user_id, deleted_at)` (ou os campos que a query usa). Medir com `EXPLAIN ANALYZE` antes e depois.
- **Severity:** Medium (fica High em escala)

#### OP-026 — Endpoint de lista sem paginação default

- **Sintoma:** `GET /items` retorna tudo quando cliente não passa `limit`.
- **Fix preferido:** default de `limit` (20-50) no handler. Teto máximo (p. ex. 100). Cliente pedir mais que teto → clampar.
- **Severity:** Medium

---

## 5. Tabela DO / DON'T universal

Cross-linguagem. Adaptar a sintaxe ao stack, mas a regra vale em qualquer. Reviewer escaneia PR contra essa tabela.

| ✅ DO | ❌ DON'T |
|---|---|
| Retornar `Result<T, E>` / `Either<E, T>` / tuple `(T, error)` | Lançar exceção através de 3+ camadas |
| Resolver deps via container DI / injeção no construtor | Instanciar com `new`/`make()` dentro do consumidor |
| Referenciar tokens do design system (`colors.primary`, `theme.spacing.md`) | Hardcode hex, pixel, `Color(0xFF...)` |
| Referenciar rotas por constante (`Routes.login`) | String literal espalhada (`'/login'`) |
| `const`/`final`/`readonly`/`val` por padrão | `let`/`var` sem razão de mutabilidade |
| Um use case / handler / query por arquivo | Múltiplos propósitos no mesmo arquivo |
| Mirror `lib/` ↔ `test/` 1:1 | Pasta `test/` flat ou descompassada |
| Mocks em helper compartilhado | Remock no topo de cada teste |
| Guard clause + early return | `if/else` aninhado com 3+ níveis |
| Ternário ou pattern-matching | `else if` encadeado com 4+ ramos (use Strategy) |
| `Option<T>` / `T?` explícito | Optional mascarado (`undefined` as any) |
| Value Object para primitivo com invariante (`Email`, `Money`) | String/int solto com validação em N pontos |
| First-class collection (`VoteCollection`) quando há invariante | `List<Vote>` público + invariante espalhada |
| `context.Context`/`cancellationToken` como primeiro param de I/O | I/O sem suporte a cancelamento |
| Logger estruturado com campos nomeados | `print()`/`console.log` em produção |
| Comentário explica **porquê** + invariante oculta | Comentário reescreve o que o código já diz |
| Bilingual `///` (EN + lang do negócio) em APIs públicas | Um idioma só em API exportada, quando projeto é bilíngue |
| `const` constructor / struct literal imutável | Setter em entidade (use `copyWith`/`with`) |
| `errors.Is/As` / `instanceof`/`match` em error types | Comparação por string (`err.message === "..."`) |
| Validar request body com schema (Zod/Yup/JSONSchema) | Acessar `req.body.x` sem validar |
| Transação em UnitOfWork quando escreve em 2+ repos | Rodar writes fora de tx e "torcer" pra funcionar |
| `UPSERT` / atomic increment para contador | Read-modify-write sem lock |
| `INSERT ... ON CONFLICT` + constraint `UNIQUE` | Pré-check `exists()` antes de `insert` |
| `credentials: 'include'` + cookie httpOnly | Token em `localStorage`/`sessionStorage` |
| Secure storage nativo (Keychain/Keystore) no mobile | `SharedPreferences`/`UserDefaults` pra token |
| Verificar assinatura HMAC em webhook | Aceitar body cru como confiável |
| Sanitizar payload antes de logar | Logar raw event/payload com PII |

---

## 6. Padrões arquiteturais cross-módulo

Recorrem em praticamente todo sistema multi-módulo. Declarar a escolha no `AGENTS.md` do projeto evita drift.

### 6.1 Provider-agnostic services (Strategy Pattern para externos)

Todo integrador externo vive atrás de uma interface no domínio/aplicação. Trocar provider = nova implementação, zero mudança no domínio.

```
domain/
  providers/
    IPaymentGateway.ts    ← contrato (createSubscription, cancel, refund, verifyWebhookSig, parseEvent)
    IEmailProvider.ts
    IStorageProvider.ts
    IVideoProvider.ts
    IOIDCProvider.ts

infrastructure/
  providers/
    stripe/StripeGateway.ts   ← impl Stripe
    pix/PixGateway.ts         ← impl PIX
    resend/ResendEmail.ts
```

Regras:
- **Entity de negócio é agnóstica ao provider**. Ex: `Subscription` tem `providerSubscriptionId: string | null` e `provider: string`, mas nunca `stripeCustomerId`, `stripeStatus`, etc. diretamente como nomes específicos em toda a classe.
- **Separe `Subscription` (contrato de serviço)** de **`Transaction` (evento financeiro específico)** — uma subscription produz N transactions ao longo do tempo, cada uma ligada a 1 provider event.
- **Mapeamento de status do provider → status do domínio** fica no adapter (`mapStripeStatus(raw) → SubscriptionStatus`).

### 6.2 Authorization como módulo central (não dentro de billing/auth)

`PermissionService` usado por todo request. **Não** pertence a `billing` nem a `auth` — pertence a um módulo próprio `authorization` (ou `core/authorization`).

```
modules/authorization/
  domain/
    services/
      PermissionService.ts    ← can(userId, feature, action)
      FunctionalMatrix.ts     ← estrutura central ABAC
    entities/
      Action.ts, Feature.ts, PlanCode.ts
  infrastructure/
    providers/
      CaslAdapter.ts / OpaAdapter.ts   ← quando usa engine externa
```

Regra dura: **nenhum módulo de negócio implementa sua própria autorização**. Todo check vai por `PermissionService.can(...)`.

### 6.3 Functional Matrix (ABAC declarativa)

Estrutura central `Map<PlanCode, Map<FeatureCode, Action[]>>` que cruza plano × feature × ação. Hardcode no código em projeto pequeno; seed em tabela `plan_permissions` em projeto com admin UI.

```
resident_base      → search       = ['read']
                   → videos       = ['read']
                   → connect_me   = ['send', 0]                    ← último elemento = quota
resident_paid      → connect_me   = ['send', 4]
syndic_n2          → connect_me   = ['send', 2]
                   → assemblies   = ['create', 'read', 'update']
enterprise_pro     → connect_me   = ['send', 20]
                   → courses      = ['create']
```

Benefícios:
- Matriz é testável isoladamente (property-based testing: `todo plan × feature retorna resultado determinístico`).
- Admin UI vira CRUD em tabela única.
- Checagem é 1 lookup `O(1)` em Map hidratado.

### 6.4 Abstract + Impl para toda dep swappable

Em todo `core/` que depende de third-party:

```
core/network/
  network_info.ts           ← abstract (contrato)
  network_info_impl.ts      ← impl usando connectivity_plus / navigator.onLine / etc.

core/storage/
  secure_storage.ts
  secure_storage_impl.ts    ← impl usando Keychain/Keystore/flutter_secure_storage
```

Trocar lib = escrever novo `_impl`, zero mudança em domain/app.

---

## 7. Convenções de documentação

Três práticas que vivem no código e no SPEC do projeto.

### 7.1 "Why this file exists" em cabeçalho

Todo arquivo não-trivial começa com comentário curto respondendo três perguntas:
1. **Why it exists** — o que trouxe a existência (problema que resolve)
2. **What it does** — 1-3 linhas do comportamento
3. **Design decision** — decisão não-óbvia que explica uma escolha surpreendente

Aplicar em: pontos de entrada (`main`, `app.ts`), DI config, router, módulos core (`error/`, `network/`), entidades de domínio complexas. **Não aplicar** em arquivos triviais (DTOs, constants puras) — vira ruído.

### 7.2 Bilingual EN + idioma do negócio em APIs exportadas

Em projetos bilíngues (ex: produto pt-BR):

```
/// Authenticates the user with email and password.
/// Autentica o usuário com e-mail e senha.
Future<Either<Failure, UserEntity>> login(...);
```

- Aplicar em: APIs públicas (métodos exportados, interfaces públicas, entidades do domínio).
- Não aplicar em: internals, `// comments` inline de linha única, testes.
- Ordem fixa: **EN sempre em cima**. Evita drift de qual é a "fonte".

### 7.3 Tabela de escolhas de pacote com justificativa

README.md ou `docs/packages.md` lista cada dep third-party com "por que foi escolhida":

```markdown
| Package | Why chosen |
|---|---|
| flutter_bloc  | Event/state split. Traceability pro debug. |
| get_it + injectable | DI sem widget-tree coupling. Injectable gera boilerplate. |
| go_router     | Official Flutter team. Declarative, deep-link nativo. |
| dartz         | Industry-standard `Either` pra Dart. |
```

Quando trocar uma dep, **atualizar** a tabela com a nova justificativa. Drift aqui = fonte de briga técnica eterna.

---

## 8. Convenções cross-stack que todo AGENTS.md reforça

- **Idioma**: UI copy, log messages, commit messages em `<idioma-do-negócio>`. Identifiers e filenames em **inglês**.
- **snake_case** em filenames e colunas SQL; **PascalCase** em tipos; **camelCase** em variáveis/funções (ou conforme idioma).
- **Um use case por arquivo**.
- **Money em inteiro**. Sempre.
- **UUIDv7 em IDs**. Ordenação temporal + aleatoriedade.
- **Context/cancellation token** como primeiro parâmetro de I/O.
- **Concurrency justificada** — cada goroutine/task/worker tem um "por que aqui é async".

---

## 9. Checklist de revisão (Reviewer usa isso)

Ao revisar PR, Reviewer valida em ordem de prioridade:

1. **Dependency rule** violada? (bloqueador)
2. **Erro descartado silenciosamente**? `_ = fn()`, `catch {}` vazio (bloqueador)
3. **Padrões idiomáticos** da linguagem (early return, tipos corretos, `context.Context`)
4. **DDD/Clean Arch**: estado privado? VO imutável? CQRS respeitado?
5. **Segurança**: webhook assinado? validação server-side? PII em log?
6. **Antipatterns listados** no `AGENTS.md` do projeto
7. **Coverage** por camada bate o threshold
8. **Lint + vet + gosec/equivalente** passam

Output do Reviewer segue formato JSON:

```json
{
  "approved": true,
  "confidence": 0-100,
  "critical_issues": [{"location": "file:line", "rule": "<id>", "description": "..."}],
  "high_issues": [],
  "medium_issues": [],
  "suggestions": [],
  "verify_criteria_met":    ["..."],
  "verify_criteria_failed": [],
  "reasoning": "3-5 linhas"
}
```

---

## 10. Como replicar em um novo projeto

1. Copiar este SPEC como seed.
2. Preencher seção "Projeto" com stack real.
3. Listar bounded contexts reais na seção "DDD + Vertical Slices".
4. Escolher thresholds de coverage específicos (os daqui são defaults).
5. Adicionar regras regulatórias aplicáveis (LGPD/GDPR/HIPAA/PCI/SOX).
6. Listar providers reais e as interfaces correspondentes.
7. Definir o stack de middleware com a ordem exata.
8. Ajustar templates de erro/entity/VO à sintaxe da linguagem.
9. Publicar na raiz do projeto como `AGENTS.md`.
10. Revalidar a cada sprint — `AGENTS.md` vive, não fossiliza.
