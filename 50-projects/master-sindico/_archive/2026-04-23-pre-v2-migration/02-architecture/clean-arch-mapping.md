---
title: Clean Architecture Mapping — Master Síndico
type: project
tags: [master-sindico, architecture, clean-architecture]
project: master-sindico
created: 2026-04-22
---

# Clean Architecture Mapping — Master Síndico

Como Clean Architecture aterrissa no código real do backend Go.

Complementa [[../07-quality/CLEAN_ARCH]] (regras de arquitetura) e [[system-overview]].

---

## Regra de ouro

```
infrastructure/ ──▶ application/ ──▶ domain/

    (seta aponta PARA DENTRO)
```

Violação = compile error (via CI grep no domínio).

---

## Mapa: arquivo físico → camada Clean Arch

| Camada Clean Arch | Nosso código | Exemplo |
|---|---|---|
| **Entities** | `internal/modules/<ctx>/domain/entities/` | `Condominium`, `User`, `Assembly` |
| **Value Objects** | `internal/modules/<ctx>/domain/value_objects/` | `Email`, `CPF`, `FracaoIdeal` |
| **Domain Services** | `internal/modules/<ctx>/domain/services/` | `QuorumCalculator` |
| **Repository Interfaces** | `internal/modules/<ctx>/domain/interfaces.go` | `IUserRepository` |
| **Domain Events** | `internal/modules/<ctx>/domain/events/` | `UserCreatedEvent` |
| **Use Cases (Interactors)** | `internal/modules/<ctx>/application/use_cases/` | `CreateUserUseCase` |
| **DTOs** | `internal/modules/<ctx>/application/dtos.go` | `CreateUserDTO` |
| **Mappers** | `internal/modules/<ctx>/application/mappers/` | `UserToDTO` |
| **Event Handlers** | `internal/modules/<ctx>/application/event_handlers/` | `OnUserCreated` |
| **Repository Impl** | `internal/modules/<ctx>/infrastructure/database/` | `UserRepository` struct |
| **sqlc queries** | `internal/modules/<ctx>/infrastructure/database/queries/*.sql` | `GetUserByID` |
| **Provider Adapters** | `internal/modules/<ctx>/infrastructure/providers/` | `StripeAdapter` |
| **HTTP Handlers** | `internal/modules/<ctx>/infrastructure/http/routes/` | `UserHandler` |
| **Module wire-up** | `internal/modules/<ctx>/module.go` | `IdentityModule.Register` |
| **Framework Adapter** | `internal/server/gin_adapter.go` | `GinContext`, `GinRouter` |
| **Middleware** | `internal/shared/middleware/` | `ABAC`, `Authenticate` |
| **Shared Providers** | `internal/shared/providers/` | `pgxpool + UoW + Migrator` |
| **Contracts (abstrações)** | `internal/core/contracts/` | `IModule`, `IUseCase`, `HTTPRouter`, `Context` |
| **Error types** | `internal/core/errors/` | `AppError`, `ValidationError` |

---

## Camadas detalhadas

### `domain/` — puro, zero deps externas

```go
// internal/modules/identity/domain/entities/user.go
package entities

import (
    "context"
    "time"
    "errors"
    "github.com/mastersindico/backend/internal/modules/identity/domain/value_objects"
)

type User struct {
    id        UserID
    email     value_objects.Email
    cpf       value_objects.CPF
    createdAt time.Time
    // campos privados — getters retornam VOs
}

func NewUser(email value_objects.Email, cpf value_objects.CPF) (*User, error) {
    if email.IsZero() { return nil, ErrInvalidEmail }
    return &User{
        id:    uuid7.New(),
        email: email,
        cpf:   cpf,
        createdAt: time.Now().UTC(),
    }, nil
}

func ReconstructUser(...) *User { ... }  // sem validação, usado no repo

func (u *User) ChangeEmail(new value_objects.Email) error {
    if u.locked { return ErrUserLocked }
    u.email = new
    return nil
}

func (u User) Email() value_objects.Email { return u.email }  // getter via VO
```

Regras:
- **Sem import** de `gin`, `pgx`, `stripe-go`, `fmt` pra logar. Só stdlib mínimo.
- Factories `NewX` (valida) e `ReconstructX` (hidrata).
- Getters via VO; setters específicos com regra (`ChangeEmail`, não `SetEmail`).

### `application/use_cases/` — orquestração

```go
// internal/modules/identity/application/use_cases/create_user_use_case.go
package use_cases

type CreateUserCommand struct {
    Email string
    CPF   string
    Password string
}

type CreateUserResult struct {
    UserID string
}

type CreateUserUseCase struct {
    userRepo domain.IUserRepository
    uow      contracts.IUnitOfWork
    eventBus contracts.IEventBus
    log      logger.Logger
}

func (uc *CreateUserUseCase) Execute(ctx context.Context, cmd CreateUserCommand) (CreateUserResult, error) {
    email, err := value_objects.NewEmail(cmd.Email)
    if err != nil { return CreateUserResult{}, apperrors.ErrValidation.WithMessage(err.Error()) }
    cpf, err := value_objects.NewCPF(cmd.CPF)
    if err != nil { return CreateUserResult{}, apperrors.ErrValidation.WithMessage(err.Error()) }

    user, err := entities.NewUser(email, cpf)
    if err != nil { return CreateUserResult{}, err }

    err = uc.uow.Run(ctx, func(ctx context.Context) error {
        return uc.userRepo.Save(ctx, user)
    })
    if err != nil { return CreateUserResult{}, err }

    uc.eventBus.Publish(ctx, events.NewUserCreated(user))

    return CreateUserResult{UserID: user.ID().String()}, nil
}
```

Regras:
- Importa **apenas** `domain/` do próprio módulo + `core/contracts` + `core/errors`.
- 1 command por arquivo; 1 query por arquivo.
- Sem lógica de negócio (vai nas entidades).

### `infrastructure/database/` — adapters

```go
// internal/modules/identity/infrastructure/database/user_repository_impl.go
type UserRepository struct {
    pool *pgxpool.Pool
    q    *sqlc.Queries
}

func (r *UserRepository) Save(ctx context.Context, u *entities.User) error {
    db := database.DBFromContext(ctx, r.pool)
    _, err := r.q.UpsertUser(ctx, db, sqlc.UpsertUserParams{
        ID:        u.ID().UUID(),
        Email:     u.Email().String(),
        Cpf:       u.CPF().Digits(),
        CreatedAt: u.CreatedAt(),
    })
    return mapErr(err)  // UNIQUE → ErrConflict, etc.
}
```

Regras:
- Importa sqlc, pgx, mappers.
- **Retorna entidade** (não row).
- Tradução de erro DB → AppError via `mapErr()`.

### `infrastructure/http/routes/` — HTTP

```go
// internal/modules/identity/infrastructure/http/routes/user_handler.go
type UserHandler struct {
    createUserUseCase *use_cases.CreateUserUseCase
    log               logger.Logger
}

func (h *UserHandler) Create(ctx contracts.Context) error {
    var req CreateUserRequest
    if err := ctx.DecodeJSON(&req); err != nil {
        return apperrors.ErrValidation.WithMessage("payload inválido")
    }
    result, err := h.createUserUseCase.Execute(ctx, use_cases.CreateUserCommand{
        Email: req.Email, CPF: req.CPF, Password: req.Password,
    })
    if err != nil { return err }
    return ctx.JSON(201, result)
}
```

Regras:
- Recebe `contracts.Context`, não `*gin.Context`.
- Apenas decodifica, chama use case, responde.
- **Sem lógica de negócio aqui.**

### `module.go` — wire-up explícito

```go
// internal/modules/identity/module.go
type Module struct {
    deps Deps
}

type Deps struct {
    Pool    *pgxpool.Pool
    Redis   *redis.Client
    Zitadel auth.IZitadel
    UoW     contracts.IUnitOfWork
    Log     logger.Logger
}

func NewModule(deps Deps) contracts.IModule {
    return &Module{deps: deps}
}

func (m *Module) Register(router contracts.HTTPRouter) {
    userRepo := database.NewUserRepository(m.deps.Pool)
    sessionRepo := database.NewSessionRepository(m.deps.Pool)

    createUserUC := &use_cases.CreateUserUseCase{
        UserRepo: userRepo, UoW: m.deps.UoW, EventBus: m.deps.EventBus, Log: m.deps.Log,
    }

    handler := &routes.UserHandler{CreateUserUseCase: createUserUC, Log: m.deps.Log}

    api := router.Group("/api/v1")
    api.POST("/users", handler.Create, middleware.Authenticate, middleware.ABAC("user", "create"))
}
```

Regras:
- DI explícito, sem reflection/container mágico.
- Cada módulo expõe `Register(HTTPRouter)`.
- `main.go` monta todos e chama `Register`.

### `cmd/api/main.go` — entry

```go
func main() {
    cfg := config.MustLoad()
    pool := database.MustConnect(cfg)
    redis := cache.MustConnect(cfg)
    zitadel := auth.MustZitadel(cfg)
    uow := database.NewUoW(pool)
    eventBus := events.NewInProcess()
    log := logger.New(cfg)

    deps := shared.Deps{Pool: pool, Redis: redis, Zitadel: zitadel, UoW: uow, EventBus: eventBus, Log: log}

    modules := []contracts.IModule{
        identity.NewModule(deps.ToIdentity()),
        billing.NewModule(deps.ToBilling()),
        institutional.NewModule(deps.ToInstitutional()),
        commercial.NewModule(deps.ToCommercial()),
        content.NewModule(deps.ToContent()),
        assembly.NewModule(deps.ToAssembly()),
    }

    srv := server.New(cfg, modules)
    srv.Start()
}
```

---

## Contratos centrais (`core/contracts`)

```go
type Context interface {
    Request() *http.Request
    Response() http.ResponseWriter
    JSON(status int, payload any) error
    DecodeJSON(into any) error
    Param(name string) string
    Query(name string) string
    User() AuthUser
    RequestID() string
    Tenant() TenantID
    // context.Context compatibility
    context.Context
    RawBody() io.Reader  // A-024: elevar pra interface (Sprint 10)
}

type HTTPRouter interface {
    Group(path string) HTTPRouter
    GET(path string, handler HandlerFunc, middleware ...HandlerFunc)
    POST(path string, handler HandlerFunc, middleware ...HandlerFunc)
    // ...
}

type IModule interface {
    Register(HTTPRouter)
}

type IUnitOfWork interface {
    Run(ctx context.Context, fn func(ctx context.Context) error) error
}
```

Troca de framework (Gin → Echo, por exemplo) é **contained** ao `server/` — modules não mudam.

---

## Violações que CI deve pegar

```bash
# domain não importa externo
! grep -rn '"github.com/gin-gonic\|jackc/pgx\|stripe-go\|stripe/stripe-go\|zitadel\|mux' \
    internal/modules/*/domain/

# application não importa infrastructure
! grep -rn 'internal/modules/.*/infrastructure' \
    internal/modules/*/application/

# infrastructure/http não importa database diretamente (vai via use case)
! grep -rn 'internal/modules/.*/infrastructure/database' \
    internal/modules/*/infrastructure/http/
```

---

## Links

- [[_moc]]
- [[system-overview]]
- [[modules-layout]]
- [[cqrs-layout]]
- [[../07-quality/CLEAN_ARCH]]
- [[../../../10-knowledge/architecture/clean-architecture-layers]]
