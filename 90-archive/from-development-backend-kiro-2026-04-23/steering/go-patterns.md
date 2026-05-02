---
inclusion: always
---

# Go — Padrões e Regras de Implementação

## Regras Absolutas (violações reprovam PR)

- **Nunca** descarte erros silenciosamente. `_ = fn()` em operações falíveis é **proibido**.
- **Nunca** use `float64` para dinheiro. Sempre `int64` (centavos).
- **Nunca** use `float`/`double` para fração ideal. Sempre `DECIMAL/NUMERIC(10,6)` no PostgreSQL.
- **Nunca** importe SDK externo (Stripe, Mux, Twilio, stripe-go, zitadel-go) diretamente no `domain/`. Sempre via interface, implementação isolada em `infrastructure/providers/`.
- **Nunca** use `panic` em código de produção. Apenas em `init()` para falha irrecuperável de config (config Validate retorna erro → `panic` aceitável no bootstrap).
- **Nunca** misture commands e queries no mesmo use case (CQRS). Arquivos separados.
- **Nunca** chame use cases de outro bounded context diretamente. Use domain events.
- **Nunca** use `interface{}` / `any` onde tipo concreto serve.
- **Nunca** use `zerolog.SetGlobalLevel()` — mutação global quebra testes paralelos. Logger via injeção ou `zerolog.Ctx(ctx)`.
- **Nunca** compare erro por string: `err.Error() == "not found"` é proibido. Use `errors.Is` / `errors.As`.
- **Nunca** importe Gin em `core/` ou `domain/` — handlers usam `contracts.HandlerFunc`, não `gin.HandlerFunc`.
- Propague erros com `fmt.Errorf("contexto: %w", err)`.
- `context.Context` é sempre o primeiro parâmetro em funções com I/O.

## Nomenclatura

| Tipo | Padrão | Exemplo |
|---|---|---|
| Arquivo | `snake_case.go` | `create_subscription_use_case.go` |
| Tipo/função exportada | `PascalCase` | `SubscriptionRepository`, `NewSubscription` |
| Interface | prefixo `I` | `ISubscriptionRepository`, `IPaymentGateway` |
| Variável local | nome completo | `unitOfWork` (não `uow`), `condominium` (não `condo`) |
| Constante | `PascalCase` ou `UPPER_SNAKE` para sentinels | `DefaultTimeout`, `StripeAPIVersion` |
| Teste | sufixo `_test.go`, função `TestXxx` | `subscription_test.go`, `TestActivate` |

Um use case por arquivo. Um handler por arquivo. Um provider por arquivo.

## Estrutura de Entidade

Estado privado, factories explícitas, métodos de domínio expressam intenção de negócio.

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
func ReconstructSubscription(
    id, userID, planID string,
    status SubscriptionStatus,
    expiresAt time.Time,
) *Subscription {
    return &Subscription{id: id, userID: userID, planID: planID, status: status, expiresAt: expiresAt}
}

// Getters explícitos
func (s *Subscription) ID() string                  { return s.id }
func (s *Subscription) Status() SubscriptionStatus  { return s.status }
func (s *Subscription) IsActive() bool              { return s.status == SubscriptionStatusActive }
func (s *Subscription) IsExpired() bool             { return s.expiresAt.Before(time.Now()) }

// Métodos de domínio — validam invariantes, não setam estado sem checar
func (s *Subscription) Activate() error {
    if s.status != SubscriptionStatusTrialing {
        return apperrors.ErrBusiness.WithMessage("só pode ativar assinatura em trial")
    }
    s.status = SubscriptionStatusActive
    return nil
}
```

**Regra**: entidade **nunca** faz I/O. Sem DB, sem HTTP, sem logger (ou logger mínimo via parâmetro quando absolutamente necessário).

## Estrutura de Value Object

Imutáveis após criação. Validação na factory. Sem setters. Comparação por valor.

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

## Estrutura de Use Case (CQRS)

Commands e queries **separados**. Um método `Execute`. Dependências via construtor.

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

func NewCreateSubscriptionUseCase(
    repo ISubscriptionRepository,
    gateway IPaymentGateway,
    uow IUnitOfWork,
    bus IEventBus,
    logger *zerolog.Logger,
) *CreateSubscriptionUseCase {
    return &CreateSubscriptionUseCase{
        subscriptionRepository: repo,
        paymentGateway:         gateway,
        unitOfWork:             uow,
        eventBus:               bus,
        logger:                 logger,
    }
}

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
        // ...resto do fluxo
        return nil
    })
    if err != nil {
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    return output, nil
}

// --- QUERY (separada, arquivo separado) ---

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

Handlers **não importam Gin**. Usam `contracts.Context` e `contracts.HandlerFunc`.

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

    cmd.UserID = ctx.AuthUser().ID  // do middleware Authenticate

    output, err := h.createSubscriptionUseCase.Execute(ctx.Request().Context(), cmd)
    if err != nil {
        ctx.SetError(err)  // propaga para ErrorHandler global
        return
    }

    ctx.JSON(http.StatusCreated, server.ApiSuccess(output))
}
```

Responsabilidade única do handler: **parse → chamar use case → serializar**. Sem regra de negócio.

## Provider Externo (SDK isolado)

SDK externo vive **apenas** em `infrastructure/providers/`. Domínio consome via interface.

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

    return &domain.SubscriptionOutput{
        ID:     sub.ID,
        Status: string(sub.Status),
    }, nil
}

// Mapeia erros do SDK para AppError do domínio
func mapStripeError(err error) error {
    var stripeErr *stripe.Error
    if !errors.As(err, &stripeErr) {
        return apperrors.ErrInternal.Wrap(err)
    }
    switch stripeErr.Code {
    case stripe.ErrorCodeResourceMissing:
        return apperrors.ErrNotFound.Wrap(err)
    case stripe.ErrorCodeRateLimit:
        return apperrors.ErrRateLimited.Wrap(err)
    case stripe.ErrorCodeCardDeclined, stripe.ErrorCodeIncorrectCVC:
        return apperrors.ErrValidation.WithMessage(stripeErr.Msg)
    default:
        return apperrors.ErrExternal.Wrap(err)
    }
}
```

**Regra dura**: antes de codar provider externo, puxar docs via **Context7 MCP** (`resolve-library-id` → `get-library-docs`). Treino do modelo pode estar desatualizado.

## Early Return / Guard Clauses (sem if/else hell)

```go
// ❌ PROIBIDO — aninhamento profundo
func (uc *UseCase) Execute(ctx context.Context, cmd Cmd) error {
    if user != nil {
        if user.IsActive() {
            if user.HasPermission() {
                // lógica feliz
            } else {
                return errForbidden
            }
        } else {
            return errInactive
        }
    } else {
        return errNotFound
    }
}

// ✅ CORRETO — guard clauses
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
    // caminho feliz, nível 0
    return nil
}
```

## Error Handling — Sentinels + Wrap

```go
// core/errors/errors.go
var (
    ErrNotFound      = &AppError{Code: "NOT_FOUND", StatusCode: 404}
    ErrValidation    = &AppError{Code: "VALIDATION", StatusCode: 400}
    ErrUnauthorized  = &AppError{Code: "UNAUTHORIZED", StatusCode: 401}
    ErrForbidden     = &AppError{Code: "FORBIDDEN", StatusCode: 403}
    ErrConflict      = &AppError{Code: "CONFLICT", StatusCode: 409}
    ErrBusiness      = &AppError{Code: "BUSINESS_ERROR", StatusCode: 400}
    ErrRateLimited   = &AppError{Code: "RATE_LIMITED", StatusCode: 429}
    ErrExternal      = &AppError{Code: "EXTERNAL_ERROR", StatusCode: 502}
    ErrInternal      = &AppError{Code: "INTERNAL", StatusCode: 500}
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

**Regra**: handler nunca escreve resposta de erro diretamente. Propaga via `ctx.SetError(err)` → `ErrorHandler` global mapeia `AppError` para HTTP status + response body.

## Context e Cancellation (pgx specifics)

- `context.Context` sempre primeiro parâmetro em I/O
- **Atenção**: cancelamento de context no pgx **não faz rollback automático** de transação em execução. Use `context.WithTimeout` explícito em repository queries que podem ser longas:

```go
func (r *ReportRepository) GenerateHeavy(ctx context.Context) error {
    ctx, cancel := context.WithTimeout(ctx, 30*time.Second)
    defer cancel()
    // ...
}
```

- Funções que rodam em background (jobs, goroutines) recebem `context.Context` de parent, não criam `context.Background()` isolado — perdem cancelamento do shutdown.

## Logger (zerolog contextual)

- **Nunca** `zerolog.SetGlobalLevel()` — muta estado global, quebra testes paralelos.
- Prefira logger injetado no construtor ou extraído do contexto:

```go
// Middleware injeta logger com request_id e user_id no contexto
ctx = logger.WithContext(ctx)

// Em qualquer use case/repository
log := zerolog.Ctx(ctx)
log.Info().Str("subscription_id", sub.ID()).Msg("subscription created")
```

- **Nunca** logue: senhas, tokens completos (só prefixo), CPF/CNPJ sem mascarar, PII do usuário.
- Nível de log em produção: `info`. `debug` só em desenvolvimento (via env `LOG_LEVEL`).

## BeyondCorp Middleware Stack

Ordem **obrigatória** (ver `steering/security.md` para cada etapa):

```
Recovery → RequestID → Logger → CORS → ErrorHandler
  → [rotas protegidas /api/v1]:
      → RateLimiter (por IP + userId)
      → Authenticate (cookie OU Bearer → introspection → cache Redis → user sync → 1-device)
      → RequirePermission(action, resource) — ABAC
  → Handler
  → AuditLogger (Sprint 3)
```

- `ctx.AuthUser()` e `ctx.Session()` — handlers leem do contexto, nunca do cookie direto.
- Cookie httpOnly no domínio `.mastersindico.com.br`, `Secure` em prod, `SameSite=Lax`.
- Mobile: header `Authorization: Bearer <token>` (RFC 6750). Middleware suporta ambos.

## Imutabilidade (regras críticas de domínio)

- `timeline_entries`: sem `deleted_at`, sem UPDATE após publicação. Adendo = novo registro.
- `audit_trail`: append-only. Retenção 5 anos (LGPD).
- `assembly_votes` após homologação: imutável.
- `closed_deals` após dupla assinatura: `is_immutable=true`.
- Status do Plano Diretor: **apenas** via evento `TimelinePublished`.

## Tenant Isolation

- Toda query de dados de condomínio inclui `WHERE condominium_id = $tenantID`.
- ABAC rejeita request onde `tenantID` do token ≠ recurso solicitado.
- Cross-tenant **apenas** via grants explícitos (Connect Me, validações).

## Compile-time Interface Checks

Toda implementação de interface do domínio declara assertion:

```go
var _ domain.ISubscriptionRepository = (*SubscriptionRepository)(nil)
var _ domain.IPaymentGateway        = (*StripeGateway)(nil)
var _ contracts.IUseCase[Cmd, *DTO] = (*MyUseCase)(nil)
```

Se a interface muda, o build quebra no compile-time, não em runtime.
