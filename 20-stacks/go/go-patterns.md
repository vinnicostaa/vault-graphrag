---
title: Go — patterns e regras de implementação
type: template
tags: [go, patterns, stack, clean-architecture]
source: .kiro/steering/go-patterns.md
created: 2026-04-22
aliases: [Go patterns, Go rules]
---

# Go — patterns e regras de implementação

## Regras absolutas (violações reprovam PR)

- **Nunca** descartar erro silenciosamente. `_ = fn()` em operação falível é **proibido**.
- **Nunca** usar `float64` pra dinheiro — sempre `int64` (centavos).
- **Nunca** usar `float`/`double` pra fração ideal — `DECIMAL/NUMERIC(10,6)` no banco.
- **Nunca** importar SDK externo (Stripe, Mux, Twilio) diretamente em `domain/` — sempre via interface, impl em `infrastructure/providers/`.
- **Nunca** usar `panic` em produção — apenas `init()` com config irrecuperável.
- **Nunca** misturar commands e queries no mesmo use case (CQRS) — arquivos separados.
- **Nunca** chamar use case de outro bounded context diretamente — use domain events.
- **Nunca** usar `interface{}`/`any` onde tipo concreto serve.
- **Nunca** usar `zerolog.SetGlobalLevel()` — mutação global quebra testes paralelos.
- **Nunca** comparar erro por string (`err.Error() == "not found"`) — use `errors.Is`/`errors.As`.
- **Nunca** importar Gin em `core/` ou `domain/` — handlers usam `contracts.HandlerFunc`, não `gin.HandlerFunc`.
- Propagar erros com `fmt.Errorf("contexto: %w", err)`.
- `context.Context` sempre primeiro parâmetro em funções com I/O.

## Nomenclatura

| Tipo | Padrão | Exemplo |
|---|---|---|
| Arquivo | `snake_case.go` | `create_subscription_use_case.go` |
| Tipo/função exportada | `PascalCase` | `SubscriptionRepository`, `NewSubscription` |
| Interface | prefixo `I` | `ISubscriptionRepository`, `IPaymentGateway` |
| Variável local | nome completo | `unitOfWork`, `condominium` (**não** `uow`, `condo`) |
| Constante | `PascalCase` ou `UPPER_SNAKE` pra sentinels | `DefaultTimeout`, `StripeAPIVersion` |
| Teste | sufixo `_test.go`, função `TestXxx` | `subscription_test.go`, `TestActivate` |

**Um** use case por arquivo. **Um** handler por arquivo. **Um** provider por arquivo.

## Entidade (estrutura)

Estado privado, factories explícitas, métodos de domínio expressam intenção:

```go
type Subscription struct {
    id        string
    userID    string
    planID    string
    status    SubscriptionStatus
    expiresAt time.Time
}

// Factory para criação nova
func NewSubscription(userID, planID string, trialDays int) *Subscription {
    return &Subscription{
        id:        utils.NewUUIDv7(),
        userID:    userID,
        planID:    planID,
        status:    SubscriptionStatusTrialing,
        expiresAt: time.Now().AddDate(0, 0, trialDays),
    }
}

// Factory para hidratação do banco — nunca expõe setters
func ReconstructSubscription(id, userID, planID string, status SubscriptionStatus, expiresAt time.Time) *Subscription {
    return &Subscription{id: id, userID: userID, planID: planID, status: status, expiresAt: expiresAt}
}

// Getters explícitos
func (s *Subscription) ID() string                 { return s.id }
func (s *Subscription) Status() SubscriptionStatus { return s.status }
func (s *Subscription) IsActive() bool             { return s.status == SubscriptionStatusActive }
func (s *Subscription) IsExpired() bool            { return s.expiresAt.Before(time.Now()) }

// Métodos de domínio — validam invariantes
func (s *Subscription) Activate() error {
    if s.status != SubscriptionStatusTrialing {
        return apperrors.ErrBusiness.WithMessage("só pode ativar assinatura em trial")
    }
    s.status = SubscriptionStatusActive
    return nil
}
```

**Regra**: entidade **nunca** faz I/O. Sem DB, sem HTTP, sem logger (ou logger mínimo via parâmetro quando absolutamente necessário).

## Value Object

Imutável após criação. Validação na factory. Sem setters. Comparação por valor:

```go
type CPFVO struct{ value string }

func NewCPF(raw string) (CPFVO, error) {
    cleaned := strings.Map(func(r rune) rune {
        if unicode.IsDigit(r) { return r }
        return -1
    }, raw)
    if !isValidCPF(cleaned) {
        return CPFVO{}, apperrors.ErrValidation.WithMessage("CPF inválido")
    }
    return CPFVO{value: cleaned}, nil
}

func (c CPFVO) Value() string             { return c.value }
func (c CPFVO) Masked() string            { return c.value[:3] + ".***.***-" + c.value[9:] }
func (c CPFVO) Equals(other CPFVO) bool   { return c.value == other.value }
```

## Use Case (CQRS)

Commands e queries **separados, arquivos diferentes**. Único método `Execute`. Deps via construtor:

```go
// --- COMMAND ---
type CreateSubscriptionCommand struct {
    UserID   string
    PlanID   string
    Interval string // "monthly" | "yearly"
}

type CreateSubscriptionUseCase struct {
    subscriptionRepository ISubscriptionRepository
    paymentGateway         IPaymentGateway
    unitOfWork             IUnitOfWork
    eventBus               IEventBus
    logger                 *zerolog.Logger
}

// Compile-time interface check
var _ contracts.IUseCase[CreateSubscriptionCommand, *SubscriptionDTO] = (*CreateSubscriptionUseCase)(nil)

func (uc *CreateSubscriptionUseCase) Execute(ctx context.Context, cmd CreateSubscriptionCommand) (*SubscriptionDTO, error) {
    var output *SubscriptionDTO
    err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        existing, err := uc.subscriptionRepository.FindActiveByUserID(ctx, cmd.UserID)
        if err != nil && !errors.Is(err, apperrors.ErrNotFound) {
            return fmt.Errorf("FindActiveByUserID: %w", err)
        }
        if existing != nil {
            return apperrors.ErrConflict.WithMessage("usuário já tem assinatura ativa")
        }
        // ... resto do fluxo
        return nil
    })
    if err != nil {
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    return output, nil
}

// --- QUERY (arquivo separado) ---
type GetSubscriptionQuery struct{ UserID string }

type GetSubscriptionUseCase struct {
    subscriptionRepository ISubscriptionRepository
}

func (uc *GetSubscriptionUseCase) Execute(ctx context.Context, q GetSubscriptionQuery) (*SubscriptionDTO, error) {
    subscription, err := uc.subscriptionRepository.FindActiveByUserID(ctx, q.UserID)
    if err != nil {
        return nil, fmt.Errorf("FindActiveByUserID: %w", err)
    }
    return mappers.ToSubscriptionDTO(subscription), nil
}
```

## Handler (contracts, nunca gin direto)

**Não importar Gin**. Usar `contracts.Context` e `contracts.HandlerFunc`:

```go
// ❌ PROIBIDO
func (h *SubscriptionHandler) Create(c *gin.Context) { ... }

// ✅ CORRETO
func (h *SubscriptionHandler) Create(ctx contracts.Context) {
    var cmd CreateSubscriptionCommand
    if err := ctx.DecodeJSON(&cmd); err != nil {
        ctx.SetError(apperrors.ErrValidation.Wrap(err))
        return
    }
    cmd.UserID = ctx.AuthUser().ID  // middleware Authenticate

    output, err := h.createSubscriptionUseCase.Execute(ctx.Request().Context(), cmd)
    if err != nil {
        ctx.SetError(err)  // ErrorHandler global
        return
    }
    ctx.JSON(http.StatusCreated, server.ApiSuccess(output))
}
```

**Responsabilidade única**: parse → use case → serializar. Sem regra de negócio.

## Provider externo (SDK isolado)

SDK vive **apenas** em `infrastructure/providers/`. Domínio consome via interface:

```go
// domain/providers/i_payment_gateway.go
type IPaymentGateway interface {
    CreateSubscription(ctx context.Context, input CreateSubscriptionInput) (*SubscriptionOutput, error)
    CancelSubscription(ctx context.Context, stripeSubID string) error
    VerifyWebhookSignature(payload []byte, header, secret string) (Event, error)
}

// infrastructure/providers/stripe_gateway.go
type StripeGateway struct {
    client *stripe.Client
    logger *zerolog.Logger
}

var _ domain.IPaymentGateway = (*StripeGateway)(nil)

func (g *StripeGateway) CreateSubscription(ctx context.Context, input domain.CreateSubscriptionInput) (*domain.SubscriptionOutput, error) {
    params := &stripe.SubscriptionParams{
        Customer: stripe.String(input.CustomerID),
        Items:    []*stripe.SubscriptionItemsParams{{Price: stripe.String(input.PriceID)}},
    }
    params.SetIdempotencyKey(fmt.Sprintf("subscription:%s:%d", input.CustomerID, time.Now().Unix()))

    sub, err := g.client.Subscriptions.New(params)
    if err != nil {
        return nil, fmt.Errorf("stripe.Subscriptions.New: %w", mapStripeError(err))
    }

    return &domain.SubscriptionOutput{ID: sub.ID, Status: string(sub.Status)}, nil
}

// Mapeia erros do SDK → AppError do domínio
func mapStripeError(err error) error {
    var stripeErr *stripe.Error
    if !errors.As(err, &stripeErr) {
        return apperrors.ErrInternal.Wrap(err)
    }
    switch stripeErr.Code {
    case stripe.ErrorCodeResourceMissing:    return apperrors.ErrNotFound.Wrap(err)
    case stripe.ErrorCodeRateLimit:          return apperrors.ErrRateLimited.Wrap(err)
    case stripe.ErrorCodeCardDeclined, stripe.ErrorCodeIncorrectCVC:
        return apperrors.ErrValidation.WithMessage(stripeErr.Msg)
    default:                                 return apperrors.ErrExternal.Wrap(err)
    }
}
```

**Regra dura**: antes de codar provider externo, puxar docs via **Context7 MCP**. Ver [[../../30-agents/mcp-integration]].

## Early returns / guard clauses

Aplicação da regra #1 de [[../../10-knowledge/principles/code-calisthenics]]:

```go
// ✅ CORRETO — guard clauses, caminho feliz no nível 0
func (uc *UseCase) Execute(ctx context.Context, cmd Cmd) error {
    if user == nil {
        return apperrors.ErrNotFound
    }
    if !user.IsActive() {
        return apperrors.ErrBusiness.WithMessage("usuário inativo")
    }
    if !user.HasPermission() {
        return apperrors.ErrForbidden
    }
    // caminho feliz
    return nil
}
```

## Error handling — sentinels + wrap

```go
// core/errors/errors.go
var (
    ErrNotFound      = &AppError{Code: "NOT_FOUND",      StatusCode: 404}
    ErrValidation    = &AppError{Code: "VALIDATION",     StatusCode: 400}
    ErrUnauthorized  = &AppError{Code: "UNAUTHORIZED",   StatusCode: 401}
    ErrForbidden     = &AppError{Code: "FORBIDDEN",      StatusCode: 403}
    ErrConflict      = &AppError{Code: "CONFLICT",       StatusCode: 409}
    ErrBusiness      = &AppError{Code: "BUSINESS_ERROR", StatusCode: 400}
    ErrRateLimited   = &AppError{Code: "RATE_LIMITED",   StatusCode: 429}
    ErrExternal      = &AppError{Code: "EXTERNAL_ERROR", StatusCode: 502}
    ErrInternal      = &AppError{Code: "INTERNAL",       StatusCode: 500}
)

// Propagação com contexto
return nil, fmt.Errorf("FindByID: %w", apperrors.ErrNotFound.WithMessage("subscription id="+id))

// Inspeção
if errors.Is(err, apperrors.ErrNotFound) { ... }

var validationErr *apperrors.ValidationError
if errors.As(err, &validationErr) {
    // validationErr.Fields disponível
}
```

**Regra**: handler **nunca** escreve resposta de erro diretamente. Propaga via `ctx.SetError(err)` → ErrorHandler global mapeia `AppError` → HTTP status + response body.

## Context e cancellation (pgx specifics)

- `context.Context` sempre primeiro parâmetro em I/O
- **Atenção pgx**: cancelamento de context **não faz rollback automático** de transação em execução. Timeout explícito em queries que podem ser longas:

```go
func (r *ReportRepository) GenerateHeavy(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    // ...
}
```

- Background jobs recebem context do parent, não criam `context.Background()` isolado — perdem cancelamento do shutdown.

## Logger (zerolog contextual)

- **Nunca** `zerolog.SetGlobalLevel()` — quebra testes paralelos.
- Logger injetado via construtor ou extraído do contexto:

```go
ctx = logger.WithContext(ctx)

log := zerolog.Ctx(ctx)
log.Info().Str("subscription_id", sub.ID()).Msg("subscription created")
```

- **Nunca** logar: senhas, tokens completos (só prefixo), CPF/CNPJ sem mascarar, PII. Ver [[../../10-knowledge/security/security-principles]].
- Nível em produção: `info`. `debug` só em dev via env `LOG_LEVEL`.

## Compile-time interface checks

Toda implementação de interface do domínio declara assertion:

```go
var _ domain.ISubscriptionRepository = (*SubscriptionRepository)(nil)
var _ domain.IPaymentGateway        = (*StripeGateway)(nil)
var _ contracts.IUseCase[Cmd, *DTO] = (*MyUseCase)(nil)
```

Interface muda → build quebra no compile-time, não em runtime.

## Links

- [[_moc]]
- [[../../10-knowledge/architecture/clean-architecture-layers]] — as 4 camadas aplicadas
- [[../../10-knowledge/patterns/transaction-patterns]] — UoW + Saga + locks em Go
- [[../../10-knowledge/security/security-principles]] — BeyondCorp stack
- [[../../10-knowledge/principles/code-calisthenics]] — regras com exemplos Go
- [[../../10-knowledge/principles/do-dont-checklist]] — checklist consolidado
- [[../postgres/postgresql-conventions]] — pgx pool, sqlc, goose
