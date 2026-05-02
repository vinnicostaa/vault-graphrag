---
inclusion: always
---

# SDD — Spec Driven Development por Camada

## Filosofia

Cada feature começa com spec antes de código. A spec define o contrato — o código implementa. Nunca o contrário.

Spec files ficam em `.kiro/specs/master-sindico/`:
- `requirements.md` — o quê e por quê
- `design.md` — como (arquitetura, interfaces, fluxos)
- `tasks.md` — lista de implementação com status

---

## Camada 1: Domain

**O que vai aqui:** regras de negócio puras, sem I/O, sem dependências externas.

### Entities

Representam conceitos do domínio com identidade e ciclo de vida.

```go
// Regras:
// - Estado privado, getters explícitos
// - Métodos de domínio expressam intenção de negócio
// - Factory function New*() ou Reconstruct*() para hidratação
// - Nunca importa nada de infrastructure/ ou application/

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

// Factory para hidratação do banco
func ReconstructSubscription(id, userID, planID string, status SubscriptionStatus, expiresAt time.Time) *Subscription {
    return &Subscription{id: id, userID: userID, planID: planID, status: status, expiresAt: expiresAt}
}

// Métodos de domínio expressam intenção
func (s *Subscription) Activate() error {
    if s.status != SubscriptionStatusTrialing {
        return errors.ErrBusiness.WithMessage("só pode ativar assinatura em trial")
    }
    s.status = SubscriptionStatusActive
    return nil
}

func (s *Subscription) IsExpired() bool { return s.expiresAt.Before(time.Now()) }
func (s *Subscription) ID() string      { return s.id }
func (s *Subscription) Status() SubscriptionStatus { return s.status }
```

### Value Objects

Imutáveis, auto-validados, sem identidade.

```go
// Regras:
// - Imutável após criação
// - Validação na factory, nunca depois
// - Sem setters
// - Comparação por valor, não por referência

type CPFVO struct{ value string }

func NewCPF(raw string) (CPFVO, error) {
    cleaned := strings.ReplaceAll(strings.ReplaceAll(raw, ".", ""), "-", "")
    if !isValidCPF(cleaned) {
        return CPFVO{}, errors.ErrValidation.WithMessage("CPF inválido")
    }
    return CPFVO{value: cleaned}, nil
}

func (c CPFVO) Value() string  { return c.value }
func (c CPFVO) Masked() string { return c.value[:3] + ".***.***-" + c.value[9:] }
func (c CPFVO) Equals(other CPFVO) bool { return c.value == other.value }
```

### Repository Interfaces

Contrato de persistência — sem implementação.

```go
// Regras:
// - Apenas interface, sem implementação
// - Métodos retornam entidades de domínio, nunca tipos do banco
// - context.Context sempre primeiro parâmetro

type ISubscriptionRepository interface {
    FindByID(ctx context.Context, id string) (*Subscription, error)
    FindActiveByUserID(ctx context.Context, userID string) (*Subscription, error)
    Save(ctx context.Context, subscription *Subscription) error
    Delete(ctx context.Context, id string) error
}
```

### Domain Events

Comunicação entre bounded contexts.

```go
// Regras:
// - Imutável após criação
// - Nome no passado: SubscriptionActivated, not ActivateSubscription
// - Contém apenas dados necessários para handlers

type SubscriptionActivated struct {
    SubscriptionID string
    UserID         string
    PlanID         string
    occurredAt     time.Time
}

func (e SubscriptionActivated) EventName() string  { return "subscription.activated" }
func (e SubscriptionActivated) OccurredAt() time.Time { return e.occurredAt }
```

---

## Camada 2: Application

**O que vai aqui:** orquestração de casos de uso. Coordena domain + infrastructure via interfaces.

### Use Cases (CQRS)

```go
// Regras:
// - Um arquivo por use case
// - Commands (escrita) e Queries (leitura) são structs separadas
// - Único método Execute
// - Dependências via construtor (DI)
// - Sem acesso direto a banco, sem HTTP, sem providers externos

// --- COMMAND (escrita) ---

type ActivateSubscriptionCommand struct {
    SubscriptionID string
    UserID         string
}

type ActivateSubscriptionUseCase struct {
    subscriptionRepository ISubscriptionRepository
    unitOfWork             IUnitOfWork
    eventBus               IEventBus
    logger                 *zerolog.Logger
}

func NewActivateSubscriptionUseCase(
    repo ISubscriptionRepository,
    uow IUnitOfWork,
    bus IEventBus,
    logger *zerolog.Logger,
) *ActivateSubscriptionUseCase {
    return &ActivateSubscriptionUseCase{
        subscriptionRepository: repo,
        unitOfWork:             uow,
        eventBus:               bus,
        logger:                 logger,
    }
}

func (uc *ActivateSubscriptionUseCase) Execute(ctx context.Context, cmd ActivateSubscriptionCommand) error {
    return uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        subscription, err := uc.subscriptionRepository.FindByID(ctx, cmd.SubscriptionID)
        if err != nil {
            return fmt.Errorf("FindByID: %w", err)
        }

        if subscription.UserID() != cmd.UserID {
            return errors.ErrForbidden
        }

        if err := subscription.Activate(); err != nil {
            return fmt.Errorf("Activate: %w", err)
        }

        if err := uc.subscriptionRepository.Save(ctx, subscription); err != nil {
            return fmt.Errorf("Save: %w", err)
        }

        uc.eventBus.Publish(SubscriptionActivated{
            SubscriptionID: subscription.ID(),
            UserID:         cmd.UserID,
            PlanID:         subscription.PlanID(),
            occurredAt:     time.Now(),
        })

        return nil
    })
}

// --- QUERY (leitura) ---

type GetSubscriptionQuery struct {
    UserID string
}

type SubscriptionDTO struct {
    ID        string `json:"id"`
    PlanID    string `json:"plan_id"`
    Status    string `json:"status"`
    ExpiresAt string `json:"expires_at"`
}

type GetSubscriptionUseCase struct {
    subscriptionRepository ISubscriptionRepository
}

func (uc *GetSubscriptionUseCase) Execute(ctx context.Context, query GetSubscriptionQuery) (*SubscriptionDTO, error) {
    subscription, err := uc.subscriptionRepository.FindActiveByUserID(ctx, query.UserID)
    if err != nil {
        return nil, fmt.Errorf("FindActiveByUserID: %w", err)
    }
    return mappers.ToSubscriptionDTO(subscription), nil
}
```

### Mappers

```go
// Regras:
// - Funções puras, sem estado
// - Entidade → DTO (nunca o contrário no mapper de application)
// - Nunca expõe tipos internos do banco

func ToSubscriptionDTO(s *entities.Subscription) *SubscriptionDTO {
    return &SubscriptionDTO{
        ID:        s.ID(),
        PlanID:    s.PlanID(),
        Status:    string(s.Status()),
        ExpiresAt: s.ExpiresAt().Format(time.RFC3339),
    }
}
```

---

## Camada 3: Infrastructure

**O que vai aqui:** implementações concretas. Banco, HTTP, providers externos.

### Repository Implementation

```go
// Regras:
// - Implementa interface do domain
// - Usa sqlc para queries tipadas
// - Mapeia tipos do banco → entidades de domínio (nunca expõe tipos sqlc)
// - Usa DBFromContext para suporte a transações

type SubscriptionRepository struct {
    pool *pgxpool.Pool
}

var _ domain.ISubscriptionRepository = (*SubscriptionRepository)(nil) // compile-time check

func (r *SubscriptionRepository) FindByID(ctx context.Context, id string) (*entities.Subscription, error) {
    q := billingdb.New(database.DBFromContext(ctx, r.pool))

    row, err := q.GetSubscriptionByID(ctx, id)
    if err != nil {
        if errors.Is(err, pgx.ErrNoRows) {
            return nil, apperrors.ErrNotFound.WithMessage("assinatura não encontrada")
        }
        return nil, apperrors.NewInternal(err)
    }

    return mapRowToSubscription(row), nil
}

// Mapper banco → domínio (privado, nunca exposto)
func mapRowToSubscription(row billingdb.Subscription) *entities.Subscription {
    return entities.ReconstructSubscription(
        row.ID,
        row.UserID,
        row.PlanID,
        entities.SubscriptionStatus(row.Status),
        row.ExpiresAt.Time,
    )
}
```

### HTTP Handler

```go
// Regras:
// - Responsabilidade única: parse → use case → serialize
// - Sem lógica de negócio
// - Propaga erros via ctx.Error(err) — nunca escreve resposta de erro diretamente
// - Usa contracts.HandlerFunc, não gin.HandlerFunc

func (h *SubscriptionHandler) Activate(c *gin.Context) {
    userID := c.GetString("userID") // setado pelo middleware Authenticate

    var cmd ActivateSubscriptionCommand
    if err := c.ShouldBindJSON(&cmd); err != nil {
        c.JSON(http.StatusBadRequest, server.ValidationError(err))
        return
    }
    cmd.UserID = userID

    if err := h.activateSubscriptionUseCase.Execute(c.Request.Context(), cmd); err != nil {
        c.Error(err) // propaga para o error handler global
        return
    }

    c.JSON(http.StatusOK, server.ApiSuccess(nil))
}
```

### Provider Externo

```go
// Regras:
// - Implementa interface do domain
// - Isola o SDK externo — domínio nunca conhece stripe-go, mux-go, etc.
// - Mapeia tipos do SDK → tipos do domínio

type StripeGateway struct {
    client *stripe.Client
    logger *zerolog.Logger
}

var _ domain.IPaymentGateway = (*StripeGateway)(nil)

func (g *StripeGateway) CreateSubscription(ctx context.Context, input domain.CreateSubscriptionInput) (*domain.SubscriptionOutput, error) {
    params := &stripe.SubscriptionParams{
        Customer: stripe.String(input.CustomerID),
        Items: []*stripe.SubscriptionItemsParams{
            {Price: stripe.String(input.PriceID)},
        },
    }

    sub, err := g.client.Subscriptions.New(params)
    if err != nil {
        return nil, fmt.Errorf("stripe.CreateSubscription: %w", mapStripeError(err))
    }

    return &domain.SubscriptionOutput{
        ID:     sub.ID,
        Status: string(sub.Status),
    }, nil
}
```

---

## Camada 4: Routes (HTTP)

**O que vai aqui:** registro de rotas, agrupamento, middleware específico do módulo.

```go
// Regras:
// - Registra rotas no grupo do módulo
// - Aplica middleware específico (ex: authorize para ABAC)
// - Não instancia use cases — recebe via DI

func RegisterSubscriptionRoutes(router *gin.RouterGroup, handler *SubscriptionHandler) {
    subscriptions := router.Group("/subscriptions")
    {
        subscriptions.POST("/activate", handler.Activate)
        subscriptions.GET("/me", handler.GetMySubscription)
        subscriptions.DELETE("/cancel", handler.Cancel)
    }

    // Webhook não passa pelo middleware de auth
    router.POST("/webhooks/stripe", handler.HandleStripeWebhook)
}
```

---

## Ordem de Implementação por Feature

```
1. Migration SQL (goose)
   └── internal/shared/providers/database/migrations/000NN_*.sql

2. sqlc query
   └── internal/modules/{ctx}/infrastructure/database/queries/*.sql
   └── sqlc generate

3. Domain entity / value object
   └── internal/modules/{ctx}/domain/entities/*.go
   └── internal/modules/{ctx}/domain/value_objects/*.go

4. Repository interface
   └── internal/modules/{ctx}/domain/repositories/i_*_repository.go

5. Domain event (se necessário)
   └── internal/modules/{ctx}/domain/events/*.go

6. Use case
   └── internal/modules/{ctx}/application/use_cases/*_use_case.go

7. Mapper
   └── internal/modules/{ctx}/application/mappers/*_mapper.go

8. Repository implementation
   └── internal/modules/{ctx}/infrastructure/database/*_repository.go

9. Provider externo (se necessário)
   └── internal/modules/{ctx}/infrastructure/providers/*_provider.go

10. Handler
    └── internal/modules/{ctx}/infrastructure/http/routes/*_handler.go

11. Wire-up
    └── internal/modules/{ctx}/module.go
    └── cmd/api/main.go (se novo módulo)

12. Validação
    └── go build ./...
    └── go vet ./...
    └── go test ./internal/modules/{ctx}/... -count=1
```
