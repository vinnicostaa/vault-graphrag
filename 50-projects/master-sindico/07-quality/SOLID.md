---
title: SOLID aplicado — Master Síndico (Go)
type: note
tags:
  - principle-applied
  - solid
  - master-sindico
  - quality
project: master-sindico
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - solid-master-sindico
  - solid-go-ms
---

# SOLID aplicado — Master Síndico v2 (Go)

Os 5 princípios SOLID aplicados ao backend Go 1.26 do MS, com foco em **DIP** (mais crítico — domínio puro) e **OCP** via Strategy/Registry. Especialização do canônico [[../../../10-knowledge/principles/solid]] com exemplos do stack real e detecção em CI.

> Cada regra **SOL-###** previne pelo menos um antipattern do catálogo [[antipatterns]] (AP-001..AP-026). Não duplica o knowledge global — linka.

---

## Sumário

SOLID em Go é diferente de Java/C# — não há herança, há composição via interfaces estruturais. No MS, **DIP é dogma**: domínio define interface (`IPaymentGateway`), infra implementa (Stripe adapter). O legado TS quebrou DIP importando Stripe direto no domain (AP-014); a reescrita corta isso na raiz. SRP previne God Objects (AP-001/AP-002). OCP via Strategy registry previne if/else hell (AP-011).

---

## Regras concretas

### SOL-001 — SRP: uma struct, uma razão para mudar

**Regra**: cada struct tem **um propósito coeso**. Aggregate só representa estado de domínio + invariantes (não persistência, não API call, não notificação). Use case faz uma ação. Repository só persiste. Provider só chama SDK externo.

**Por quê**: previne **AP-001** (`Subscription` 450 linhas) + **AP-002** (`Invoice` 550 linhas).

**Quando partir**:
- Struct > 200 linhas → review obrigatório.
- Struct com 3+ deps de tipos diferentes (ex: `Repo`, `Logger`, `Stripe`, `Email`, `PDF`) → separar.
- Métodos agrupam-se em 2+ "temas" claros (ex: `Subscription` com métodos billing + métodos quota) → extrair domain service ou novo aggregate.

**✅ Exemplo Go**:

```go
// internal/modules/billing/domain/entities/subscription.go (~200 linhas)
type Subscription struct {
    id           string
    customerID   string
    planID       string
    status       SubscriptionStatus
    trialEndsAt  *time.Time
    activatedAt  time.Time
    canceledAt   *time.Time
    stripeID     string
}

// Métodos: state machine + invariantes
func (s *Subscription) IsActive() bool { ... }
func (s *Subscription) IsInTrial() bool { ... }
func (s *Subscription) MarkCanceled(reason string) error { ... }
func (s *Subscription) MarkPastDue() error { ... }

// Quota e PDF NÃO vivem aqui:
// - Quota → application/use_cases/consume_quota_use_case.go + domain/services/quota_limits.go
// - PDF   → application/ports/IInvoicePDFGenerator + infrastructure/providers/pdf/
```

**❌ Anti-exemplo (AP-001)**:

```go
type Subscription struct { /* 30 campos */ }

func (s *Subscription) MarkCanceled() error { ... }
func (s *Subscription) GenerateInvoicePDF() ([]byte, error) { /* PDF concern */ }
func (s *Subscription) SendEmailNotification() error { /* email concern */ }
func (s *Subscription) ConsumeQuota(feature string) error { /* quota concern */ }
func (s *Subscription) SyncWithStripe(ctx context.Context) error { /* infra concern */ }
// 450 linhas...
```

**Detecção**:
- `funlen` + `wc -l domain/entities/*.go` em CI; warn ≥ 200, fail ≥ 300.
- `gocyclo -over 15` em domain/ → sinal de SRP violado.
- code review: PR template "esta struct/método tem mais de uma razão para mudar?".

---

### SOL-002 — OCP: extensibilidade via interface + Strategy

**Regra**: comportamento variante (por persona, por provider, por estratégia de moderação) é **interface** + **registry** (map). Adicionar variante = novo struct + entry no map. Não modifica código existente.

**Por quê**: previne **AP-011** (if/else hell). Permite Pluggable strategies sem refactor.

**✅ Exemplo Go — TrialPolicy (INV-017, BR-001)**:

```go
// domain/services/trial_policy.go
type TrialPolicy interface {
    TrialDays(persona enums.Persona) int
    Eligible(user *entities.User) bool
}

type SindicoTrial struct{}
func (SindicoTrial) TrialDays(_ enums.Persona) int           { return 15 }
func (SindicoTrial) Eligible(u *entities.User) bool          { return u.HasRole(enums.RoleSindico) }

type EmpresaTrial struct{}
func (EmpresaTrial) TrialDays(_ enums.Persona) int           { return 7 }
func (EmpresaTrial) Eligible(u *entities.User) bool          { return u.HasRole(enums.RoleEmpresa) }

type NoTrialPolicy struct{}
func (NoTrialPolicy) TrialDays(_ enums.Persona) int          { return 0 }
func (NoTrialPolicy) Eligible(_ *entities.User) bool         { return false }

// Registry — entry novo = nova persona suportada
var trialRegistry = map[enums.Persona]TrialPolicy{
    enums.PersonaSindico:  SindicoTrial{},
    enums.PersonaEmpresa:  EmpresaTrial{},
    enums.PersonaParceiro: ParceiroTrial{},
    enums.PersonaMorador:  NoTrialPolicy{},
}

func TrialDaysFor(p enums.Persona) int {
    policy, ok := trialRegistry[p]
    if !ok { return 0 }
    return policy.TrialDays(p)
}
```

**✅ Outro exemplo — IPaymentGateway (extensível)**:

```go
// domain/providers/payment_gateway.go
type IPaymentGateway interface {
    CreateCustomer(ctx context.Context, req CreateCustomerReq) (*CustomerResult, error)
    CreateSubscription(ctx context.Context, req CreateSubReq) (*SubResult, error)
    CancelSubscription(ctx context.Context, stripeID string, atPeriodEnd bool) error
    VerifyWebhookSignature(body []byte, signature string) error
}

// infrastructure/providers/stripe/stripe_gateway.go — adapter Stripe
// (futuro) infrastructure/providers/mercadopago/mp_gateway.go — adapter MP
// Trocar provider em prod = trocar 1 linha em main.go
```

**❌ Anti-exemplo (AP-011)**:

```go
func TrialDaysFor(p Persona) int {
    if p == PersonaSindico { return 15 }
    if p == PersonaEmpresa { return 7 }
    if p == PersonaParceiro { return 7 }
    if p == PersonaAgencia { return 14 }
    if p == PersonaMorador { return 0 }
    return 0
}
// Persona nova: editar este switch + esquecer um lugar = bug
```

**Detecção**:
- `gocyclo -over 8` em `domain/services/` (alta complexidade = if/else hell).
- `revive` rule `cognitive-complexity` ≤ 10.
- code review: switch com 5+ cases = sinal de Strategy faltando.

---

### SOL-003 — LSP: implementações honram precondition/postcondition

**Regra**: toda impl de interface respeita o contrato — **não** lança panic, **não** retorna mais erros que o contrato declara, **não** quebra garantias documentadas (ex: `Save` é idempotente OU não, mas todas as impls combinam).

**Por quê**: violar LSP = use case quebra com adapter novo. Garante que troca de adapter (Stripe → MP) é seguro.

**✅ Exemplo Go**:

```go
// application/ports/payment_gateway.go
// IPaymentGateway é o contrato do provider de pagamento.
// Todos os adapters DEVEM:
//   - retornar nil em sucesso
//   - retornar apperrors.ErrNotFound se ID não existe
//   - retornar apperrors.ErrConflict se idempotency-key conflita
//   - encapsular erro do SDK externo com fmt.Errorf("...: %w", err)
//   - NÃO lançar panic
//   - NÃO bloquear indefinidamente (timeout interno via gobreaker)
type IPaymentGateway interface {
    CreateCustomer(ctx context.Context, req CreateCustomerReq) (*CustomerResult, error)
    // ...
}

// infrastructure/providers/stripe/stripe_gateway.go
func (a *StripeAdapter) CreateCustomer(ctx context.Context, req CreateCustomerReq) (*CustomerResult, error) {
    result, err := a.cb.Execute(func() (any, error) {
        return a.client.Customers.New(ctx, mappers.ToStripeCustomerParams(req))
    })
    if err != nil {
        if isStripeIdempotencyConflict(err) {
            return nil, apperrors.ErrConflict.WithMessage("idempotency key conflict")
        }
        return nil, fmt.Errorf("stripe create customer: %w", err)
    }
    return mappers.FromStripeCustomer(result.(*stripe.Customer)), nil
}
```

**❌ Anti-exemplo**:

```go
func (a *StripeAdapter) CreateCustomer(ctx context.Context, req CreateCustomerReq) (*CustomerResult, error) {
    result, err := a.client.Customers.New(ctx, ...)
    if err != nil { panic(err) }                              // ❌ panic quebra contrato
    if result == nil { return nil, errors.New("ué nil") }     // ❌ erro fora do contrato
    return result, nil                                         // ❌ vaza tipo Stripe (LSP + AP-014)
}
```

**Detecção**:
- contract test (`internal/modules/<bc>/application/ports/contract_test.go`) que cada adapter passa, validando shape de erros.
- code review: `panic`/`log.Fatal` em adapter de provider = blocker.
- `revive` rule `defer-recover` desencoraja panic-recovery.

---

### SOL-004 — ISP: interfaces fatiadas por leitura/escrita ou capacidade

**Regra**: prefira interfaces pequenas com 1-3 métodos por capability. Use cases dependem da menor interface possível. Repository típico fatiado em `IXxxReader` + `IXxxWriter` quando query and write são consumidas por handlers diferentes.

**Por quê**: previne god-interfaces que forçam stubs pesados em testes. Permite que QueryHandler peça apenas leitura (sem mockar `Save`).

**✅ Exemplo Go**:

```go
// domain/repositories/i_user_repository.go
type IUserReader interface {
    FindByID(ctx context.Context, id string) (*entities.User, error)
    FindByCPF(ctx context.Context, cpfDigits string) (*entities.User, error)
    FindByZitadelID(ctx context.Context, zitadelID string) (*entities.User, error)
}

type IUserWriter interface {
    Save(ctx context.Context, u *entities.User) error
    SoftDelete(ctx context.Context, id string) error
}

// IUserRepository é a composição completa (apenas para impl)
type IUserRepository interface {
    IUserReader
    IUserWriter
}

// uso: query handler depende só do reader
type GetUserQueryHandler struct {
    reader IUserReader  // não precisa Save → stub fica trivial
}

// command depende dos dois (ou da composição)
type RegisterUserUseCase struct {
    users  IUserRepository
    // ...
}
```

**✅ Outro exemplo — ABAC engine usando matriz fatiada**:

```go
// shared/middleware/abac_engine.go
type IPermissionResolver interface {
    Decide(ctx ABACContext, action string) Decision
}

// Engine concreta implementa só isso; matriz é dado, não interface
```

**❌ Anti-exemplo (god-interface)**:

```go
type IUserRepository interface {
    FindByID(...) error
    FindByCPF(...) error
    FindByEmail(...) error
    Save(...) error
    Delete(...) error
    SoftDelete(...) error
    HardDelete(...) error
    Count(...) error
    ListPaginated(...) error
    ListByTenant(...) error
    Search(...) error
    Export(...) error
    // 20 métodos — toda query handler precisa stubar tudo
}
```

**Detecção**:
- code review: interface com > 8 métodos = warn.
- stub em teste com 80% de métodos retornando `panic("not implemented")` = sinal de ISP violado → fatiar.

---

### SOL-005 — DIP: domínio depende de abstração; infra implementa

**Regra**: **dogma do projeto**. Domain define interface (`IPaymentGateway`, `IVideoProvider`, `ILiveSFUProvider`, `IEmailProvider`, `IUserRepository`). Infrastructure implementa. **Wire-up** acontece em `module.go` (constructor injection — sem container DI mágico).

**Por quê**: previne **AP-014** (SDK externo no domínio — blocker absoluto) + **AP-003** (vazamento ORM). É o que mantém o domínio puro, testável e portável.

**✅ Exemplo Go (correto)**:

```go
// 1. Interface no domain
// internal/modules/billing/domain/providers/i_payment_gateway.go
package providers

import "context"

type CreateSubReq struct {
    CustomerID     string
    PriceID        string
    TrialEnd       *time.Time
    IdempotencyKey string
}

type SubResult struct {
    ID            string
    Status        string
    CurrentPeriodEnd time.Time
}

type IPaymentGateway interface {
    CreateCustomer(ctx context.Context, req CreateCustomerReq) (*CustomerResult, error)
    CreateSubscription(ctx context.Context, req CreateSubReq) (*SubResult, error)
    CancelSubscription(ctx context.Context, stripeID string, atPeriodEnd bool) error
    VerifyWebhookSignature(body []byte, signature string) error
}

// 2. Use case depende só da interface
// internal/modules/billing/application/use_cases/checkout_subscription_saga.go
type CheckoutSubscriptionSaga struct {
    gateway          providers.IPaymentGateway  // ← abstração
    subscriptionRepo repositories.ISubscriptionRepository
    unitOfWork       contracts.IUnitOfWork
    eventPublisher   events.IEventPublisher
}

// 3. Adapter implementa em infrastructure
// internal/modules/billing/infrastructure/providers/stripe/stripe_gateway.go
type StripeGateway struct {
    client  *stripe.Client
    cb      *gobreaker.CircuitBreaker
}

var _ providers.IPaymentGateway = (*StripeGateway)(nil)  // assertion estática

func (g *StripeGateway) CreateCustomer(ctx context.Context, req providers.CreateCustomerReq) (*providers.CustomerResult, error) {
    // SDK Stripe usado APENAS aqui
}

// 4. Wire-up em module.go
func NewModule(...) (*Module, error) {
    gateway := stripe.NewStripeGateway(cfg, log)
    saga := usecases.NewCheckoutSubscriptionSaga(gateway, subRepo, uow, ev)
    return &Module{...}, nil
}
```

**❌ Anti-exemplo (AP-014, blocker)**:

```go
// internal/modules/billing/domain/entities/subscription.go
import "github.com/stripe/stripe-go/v74"  // ❌

func (s *Subscription) Activate(ctx context.Context, client *stripe.Client) error {
    sub, err := client.Subscriptions.New(...)  // ❌ domínio chama SDK
    // ...
}
```

**Detecção**:
- CI script (`scripts/check_domain_purity.sh` em CA-006) bloqueia imports proibidos em `domain/`.
- code review: PR que adiciona dep externa em domain = blocker.
- `var _ IXxx = (*XxxImpl)(nil)` na linha 1 do impl em `infrastructure/providers/` — quebra de build se interface mudar.

---

### SOL-006 — Constructor injection padrão; sem global, sem DI container

**Regra**: dependências entram via construtor (`NewXxxUseCase(repoA, repoB, gateway, clock, log)`). Zero `init()` que mexe com state. Zero pacote de DI mágico (wire/fx/dig). Wire-up explícito em `module.go`.

**Por quê**: testes ficam triviais (passa stub no construtor). Ordem de inicialização é óbvia. Refactor seguro.

**✅ Exemplo Go**:

```go
// application/use_cases/cast_vote_use_case.go
type CastVoteUseCase struct {
    voteRepo    repositories.IVoteRepository
    sessionRepo repositories.IVoteSessionRepository
    clock       contracts.Clock
    unitOfWork  contracts.IUnitOfWork
    log         logger.Logger
}

func NewCastVoteUseCase(
    voteRepo repositories.IVoteRepository,
    sessionRepo repositories.IVoteSessionRepository,
    clock contracts.Clock,
    uow contracts.IUnitOfWork,
    log logger.Logger,
) *CastVoteUseCase {
    return &CastVoteUseCase{voteRepo, sessionRepo, clock, uow, log}
}

// teste fica trivial
func TestCastVote(t *testing.T) {
    voteRepo := &stubVoteRepo{}
    sessionRepo := &stubSessionRepo{...}
    clock := &FakeClock{Time: time.Date(2026, 5, 1, 0, 0, 0, 0, time.UTC)}
    uow := &fakeUoW{}
    uc := usecases.NewCastVoteUseCase(voteRepo, sessionRepo, clock, uow, logger.Noop)
    // ...
}
```

**❌ Anti-exemplo**:

```go
// global state — quebra testabilidade e ordem de init
var globalDB *pgxpool.Pool

func init() {
    globalDB, _ = pgxpool.New(...)
}

func (uc *UseCase) Execute(...) error {
    globalDB.Query(...)  // não posso testar sem DB real
}
```

**Detecção**: grep `func init()` em `internal/modules/` → fail (exceção: `internal/modules/*/infrastructure/database/generated/` que pode ter init benigno do sqlc).

---

### SOL-007 — `var _ IXxx = (*Impl)(nil)` para enforcement estático

**Regra**: toda implementação de interface declara assertion estática logo após a struct:

```go
type StripeGateway struct { ... }

var _ providers.IPaymentGateway = (*StripeGateway)(nil)
```

**Por quê**: build quebra **imediatamente** se a interface mudar e o impl não acompanhar. Sem essa linha, descobre só em runtime.

**Detecção**: code review obrigatório quando arquivo novo em `infrastructure/providers/` ou `infrastructure/database/repositories/`. Linter custom (planejado).

---

### SOL-008 — `errors.Is`/`errors.As` em vez de checagem por string

**Regra**: nunca `if err.Error() == "not found"`. Use sentinels exportados + `errors.Is(err, ErrXxx)`.

**Por quê**: SOLID + LSP. Substring de `Error()` quebra com encapsulação `%w`. Sentinel é parte do contrato (interface).

**✅ Exemplo Go**:

```go
sub, err := uc.subscriptionRepo.FindByID(ctx, id)
if errors.Is(err, apperrors.ErrNotFound) {
    return nil, apperrors.ErrNotFound.WithMessage("assinatura inexistente")
}
if err != nil { return nil, fmt.Errorf("find sub: %w", err) }
```

**❌ Anti-exemplo**:

```go
if err != nil && strings.Contains(err.Error(), "not found") { ... }  // ❌ frágil
```

**Detecção**: golangci-lint `errorlint: { errorf: true, asserts: true, comparison: true }`.

---

## Checklist de PR (cole no template)

```markdown
## SOLID

- [ ] SOL-001: nenhum aggregate > 200 linhas; nenhuma struct com 5+ deps de tipos heterogêneos
- [ ] SOL-002: variantes (persona, provider, strategy) via interface + registry — zero if/else 5+ cases
- [ ] SOL-003: impls de interface honram contrato (sem panic, sem erro fora do shape declarado)
- [ ] SOL-004: nenhuma interface com > 8 métodos; query handlers dependem de Reader, não da composição
- [ ] SOL-005: domain define interface; SDK externo só em `infrastructure/providers/<sdk>/`
- [ ] SOL-006: deps via construtor; zero `init()` mexendo com state; zero global mutável
- [ ] SOL-007: `var _ IXxx = (*Impl)(nil)` em todo arquivo de impl novo
- [ ] SOL-008: zero `err.Error() == "..."` ou `strings.Contains(err.Error(), ...)` — usar `errors.Is/As`
```

---

## Exceções aceitas (com justificativa)

| Exceção | Justificativa |
|---|---|
| `IPaymentGateway`, `IUnitOfWork`, `IModule` (prefixo `I`) | Convenção MS herdada quando interface coordena contrato amplo (múltiplos métodos com semântica de "porta") — ver [[../03-subprojects/backend/architecture]] §3 |
| Aggregate "exceção justificada" 200-300 linhas | Só com comentário no topo apontando ADR ou complexidade essencial; review obrigatório |
| Stub hand-rolled em testes (sem testify/mock) | Decisão explícita do projeto para transparência (ver [[../03-subprojects/backend/patterns]] §14) |
| Use case com 4-5 deps quando há Saga + UoW + Clock + Logger + Repo | Saga inerente a 3+ providers; aceito se documentado; ver SOL-001 limit "3+ tipos heterogêneos" |
| `init()` em `infrastructure/database/generated/` | Código gerado pelo sqlc; não editar |

---

## Vizinhos

- [[../../../10-knowledge/principles/solid]] — referência canônica atemporal
- [[../../../10-knowledge/principles/_moc]]
- [[../../../20-stacks/go/_moc]]
- [[CLEAN_CODE]] — naming + funções pequenas (SRP no nível função)
- [[CLEAN_ARCH]] — DIP em escala arquitetural
- [[CODE_CALISTHENICS]] — 9 regras (SRP via "entidades pequenas")
- [[PATTERNS]] — Strategy (OCP), DIP via ports
- [[antipatterns]] — AP-001/AP-002 (SRP), AP-011 (OCP), AP-014 (DIP)
- [[../03-subprojects/backend/patterns]] — patterns aplicados no código real
- [[../CLAUDE]] §5 — proibições
