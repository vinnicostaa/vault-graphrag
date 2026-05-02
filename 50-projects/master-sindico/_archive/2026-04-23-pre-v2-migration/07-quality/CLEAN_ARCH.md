---
title: CLEAN_ARCH — Aplicação ao Master Síndico
type: project
tags: [master-sindico, clean-architecture, ddd, cqrs]
project: master-sindico
created: 2026-04-22
---

# CLEAN_ARCH — Master Síndico

Aplicação concreta de Clean Architecture ao backend Go do Master Síndico. Universal em [[../../../10-knowledge/architecture/clean-architecture-layers]].

---

## Regra de dependência (a mais importante)

```
infrastructure → application → domain
```

Setas apontam **para dentro**. **Nunca** o oposto.

Controle em CI (grep no CI):

```bash
# Falha se domain importa framework externo
! grep -rn '"github.com/gin-gonic\|jackc/pgx\|stripe-go\|zitadel' internal/modules/*/domain/
```

---

## Camadas (topo → base)

### `main.go` + DI

- `cmd/api/main.go` monta dependências **explicitamente** (sem container mágico)
- Composição: config → providers → modules → server
- Sem reflection/container pesado (rejeitado Uber FX — ver ADR-0022 *(a criar)*)

### `infrastructure/` (adapters)

- **http/handlers** — recebem `contracts.Context` (não `*gin.Context`); decodam, chamam use case, respondem
- **http/routes** — registram handlers no `contracts.HTTPRouter`; aplicam middleware específico
- **database** — sqlc generated + mappers (DTO ↔ entidade)
- **providers** — adapters pra SDK externo (Stripe, Mux, Livekit, Zitadel, Twilio, SES, FCM)
- **jobs** — background jobs (cleanup sessão expirada, reconciliação Mux)
- **http/websocket** — assembleia live

**Regra**: esta camada importa `application/` e `domain/` livremente, **nunca o contrário**.

### `application/` (use cases)

- **use_cases** — 1 arquivo por command; 1 arquivo por query
- **mappers** — entity ↔ DTO (request/response)
- **event_handlers** — consome domain events, aciona side-effects

**Regra**: importa **apenas** `domain/` do próprio módulo e `core/contracts` + `core/errors`.

### `domain/` (núcleo)

- **entities** — agregados com estado privado, factory `NewX` + `ReconstructX`, métodos de domínio
- **value_objects** — CPF, CNPJ, Email, Password, Money, FracaoIdeal, ReputacaoTier (imutáveis, com `Validate()`)
- **interfaces** — `IRepository`, `IXYZProvider` (portas)
- **events** — domain events (`CondominiumCreated`, `VoteCast`, etc.)
- **services** — domain services quando regra cruza múltiplas entidades (ex: `QuorumCalculatorService`)

**Regra**: **zero** imports de framework externo. Só `time`, `errors`, `fmt`, `strings`, `encoding/json`, etc. da stdlib. E `context` (padrão Go para propagar cancelamento).

### `core/contracts`

Interfaces abstratas que isolam framework:
- `HTTPRouter`, `Context`, `HandlerFunc`, `Group`, `Use`
- `IUseCase[Cmd, Result]`
- `IRepository` (por domínio)
- `IUnitOfWork` / `UnitOfWork.Run(ctx, fn)`

### `core/errors`

- `AppError` — erro tipado com código, mensagem pt-BR, detalhes
- Sentinels: `ErrValidation`, `ErrNotFound`, `ErrConflict`, `ErrForbidden`, `ErrUnauthorized`, `ErrInternal`
- `ValidationError` — detalha campos inválidos
- **Sem deps de Gin** — é canônico; handler global em `server/` converte pra HTTP status

---

## Fluxo de uma request (exemplo: criar unidade)

```
Cliente HTTP
  ↓ POST /api/v1/condominiums/:id/units { payload }
GinRouter (cmd/api/main.go)
  ↓ GinAdapter envolve gin.Context em contracts.Context
middleware.BeyondCorp (ordem: RequestID → Recovery → Authenticate → ABAC → TenantIsolation → RateLimit)
  ↓
handler (infrastructure/http/routes/unit_handler.go)
  ↓ DecodeJSON(ctx, &req)
  ↓ ctx.User() obtido do middleware
  ↓ cmd := CreateUnitCommand{ CondominiumID, Number, FloorArea, FracaoIdeal, ... }
  ↓ result, err := uc.Execute(ctx, cmd)
use case (application/use_cases/create_unit_use_case.go)
  ↓ domain validation via VO (FracaoIdeal.Validate())
  ↓ unitOfWork.Run(ctx, func(ctx) {
  ↓   cond := condoRepo.FindByID(ctx, cmd.CondominiumID)
  ↓   cond.AddUnit(unit)  # invariante: soma de fracaoIdeal ≤ 100%
  ↓   condoRepo.Save(ctx, cond)
  ↓   timelineRepo.AppendEntry(ctx, "UnitCreated", metadata)
  ↓ })
  ↓ eventBus.Publish(UnitCreatedEvent{...})  # fora da tx
repository (infrastructure/database/condominium_repository_impl.go)
  ↓ sqlc query tipada
  ↓ mapper: row → entidade
mappers (infrastructure/database/mappers.go)
  ↓ detecção de erro (unique violation, FK violation)
  ↓ retorno de AppError tipado
handler (response)
  ↓ ctx.JSON(201, mappers.UnitToDTO(result))
```

---

## Organização por módulo (bounded context)

Todos os módulos seguem a mesma estrutura — **consistência absoluta**:

```
internal/modules/<ctx>/
├── domain/
│   ├── entities/
│   ├── value_objects/
│   ├── events/
│   ├── services/
│   └── interfaces.go           # IRepository, ISyncer, etc.
├── application/
│   ├── use_cases/
│   │   ├── create_x_use_case.go    # Command
│   │   ├── update_x_use_case.go    # Command
│   │   ├── list_x_use_case.go      # Query
│   │   └── get_x_use_case.go       # Query
│   ├── event_handlers/
│   ├── mappers/
│   └── dtos.go
├── infrastructure/
│   ├── database/
│   │   ├── queries/*.sql           # fonte sqlc
│   │   ├── sqlc/                   # generated
│   │   ├── <entity>_repository_impl.go
│   │   └── mappers.go
│   ├── http/
│   │   ├── routes/
│   │   │   └── <entity>_handler.go
│   │   └── middleware/             # específico do módulo
│   ├── providers/                  # SDK externo (Stripe, Mux)
│   └── jobs/
└── module.go                       # IModule impl: Register(router, container)
```

---

## CQRS neste projeto

- **Commands** mudam estado; retornam ID ou void
- **Queries** só leem; nunca mudam estado
- **1 arquivo por use case** (command **ou** query, não misturar)
- **Read model separado** quando query performance exige (ex: materialized view para reputação)

Exemplo:

```
application/use_cases/
├── create_company_use_case.go      # Command
├── update_company_use_case.go      # Command
├── approve_company_use_case.go     # Command
├── list_companies_by_condominium_use_case.go  # Query
├── get_company_by_id_use_case.go   # Query
└── search_companies_use_case.go    # Query (OpenSearch)
```

---

## UoW vs Saga (regra)

### Usar UoW (Unit of Work)
Quando **tudo** cabe na mesma tx do **mesmo bounded context**.

```go
err := uow.Run(ctx, func(ctx context.Context) error {
    cond := condoRepo.FindByID(ctx, id)
    cond.AddUnit(unit)
    return condoRepo.Save(ctx, cond)
})
```

### Usar Saga
Quando atravessa **serviço externo** com estado próprio (Mux, Livekit, Stripe).

```go
// Upload vídeo (A-027 fix):
uploadURL, err := videoProvider.CreateUploadURL(ctx, params)
if err != nil { return err }

if err := videoRepo.Create(ctx, video); err != nil {
    // compensação: cancelar upload no Mux
    if cErr := videoProvider.CancelUpload(ctx, uploadURL.ID); cErr != nil {
        logger.Error("saga compensation failed", "cause", cErr, "original", err)
    }
    return err
}
return nil
```

---

## Anti-padrões que **bloqueiam merge** neste projeto

| Antipattern | Detecção |
|---|---|
| Domínio importando Gin/pgx/Stripe | grep no CI |
| `else` block em handler | `gofmt` + linter customizado |
| `_ = fn()` em I/O crítica | code review; checklist |
| `SELECT *` em query nova | `grep '\*' queries/**/*.sql` (exceções: COUNT(*)) |
| Lógica de negócio em handler | code review |
| Handler retornando tipo do ORM | mapper obrigatório |
| Use case com 2+ retornos sem erro contextualizado | code review |
| Abstração sem 2+ impls reais | code review (YAGNI) |

---

## Testes por camada

| Camada | Tipo de teste | Meta coverage |
|---|---|---|
| `domain/entities` | Unit + PBT | 95% |
| `domain/value_objects` | Unit + PBT | 95% |
| `domain/services` | Unit | 95% |
| `application/use_cases` | Unit com stubs; mocktail proibido | 90% |
| `infrastructure/database` | Integration (testcontainers) | 85% |
| `infrastructure/http/routes` | Integration | 75% |
| `infrastructure/providers` | Integration com sandbox/test mode | 85% |
| `shared/middleware` | Unit + integration | 90% |
| `core/` + `pkg/` | Unit | 95% |

---

## Links

- [[_moc]]
- [[CLEAN_CODE]]
- [[CODE_CALISTHENICS]]
- [[PATTERNS]]
- [[../02-architecture/clean-arch-mapping]]
- [[../02-architecture/modules-layout]]
- [[../../../10-knowledge/architecture/clean-architecture-layers]]
- [[../../../10-knowledge/patterns/_moc]]
