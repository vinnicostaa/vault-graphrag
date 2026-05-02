---
inclusion: always
---

# Testes — TDD, Estratégia, Padrões, Thresholds

## Filosofia: TDD por default

**Testes vêm antes da implementação.** Ciclo **Red → Green → Refactor**. Não é "testes depois quando sobrar tempo" — é metodologia obrigatória.

```
1. RED       — escreve teste que falha (descreve o comportamento desejado)
2. GREEN     — implementa o mínimo para o teste passar (sem over-engineer)
3. REFACTOR  — melhora o código mantendo os testes verdes
```

**Exceções aceitas** (raras, documentadas em `.kiro/STATE.md`):
- Spike exploratório para validar viabilidade de SDK externo antes de contrato estável. Spike **sempre** descartado; implementação final nasce do zero com testes primeiro.
- Refatoração puramente estética (rename, move) sem mudança de comportamento — testes existentes cobrem.
- Migration SQL: o teste é a migration rodar up/down repetidamente em integração.

Tudo fora disso: teste primeiro.

### Por que TDD (não "testes depois")

- **Força design testável**: acoplamento e dependências implícitas aparecem já no primeiro teste. Use case que é difícil de testar revela falha de design.
- **Specs executáveis**: o teste é a documentação viva da intenção.
- **Refactor seguro**: com suite verde, refatoração é mecânica em vez de arriscada.
- **Regressão zero**: o bug que te pegou uma vez tem teste daí em diante.

O full-stack legacy não fazia TDD — resultado foram bugs silenciosos (PermissionService comentado, race conditions em quotas, vazamentos de domínio). Não vamos repetir.

## Pirâmide de testes (ordem de valor)

```
           /\
          /E2\          (Sprint 8) — fluxos completos (síndico cria condo, morador vota)
         /----\
        / INT  \        testcontainers-go: Postgres 18 + Redis 7 reais
       /--------\
      /   UNIT   \      tabela-driven + PBT para regras críticas
     /------------\
    /   CONTRACT   \    testes de contrato HTTP (httptest + MatchedBy)
   /________________\
```

- **Unit** — base. Rápidos (<100ms cada). Mocks para dependências externas. Cobrem lógica de domínio e use cases.
- **Contract (HTTP)** — valida request/response do handler. Não duplica regra de negócio (isso é no teste do use case).
- **Integration** — banco real via testcontainers. Cobrem repository + migration + sqlc em conjunto.
- **E2E** — fluxos críticos end-to-end no Sprint 8. Playwright MCP no frontend, HTTP client no backend.

## Thresholds por camada (gate duro em CI)

| Camada | Coverage mínimo | Racional |
|---|---|---|
| `internal/modules/*/domain/` | **95%** | Regras de negócio — testa todos os caminhos |
| `internal/modules/*/application/` | **90%** | Use cases — testa commands, queries, erros |
| `internal/modules/*/infrastructure/database/` | **85%** | Repositories — happy path + erros de pgx |
| `internal/modules/*/infrastructure/http/` | **75%** | Handlers — contratos HTTP |
| `internal/modules/*/infrastructure/providers/` | **85%** | Providers externos — mapeamento de erros SDK |
| `internal/shared/middleware/` | **90%** | Segurança — todas as branches |
| `internal/shared/providers/*` | **90%** | Infra compartilhada |
| `internal/core/` | **95%** | Contracts e errors — invariantes |
| `pkg/**` | **90%** | Helpers (logger, utils, money) |
| **Global mínimo** | **85%** | Sanidade |

`.coverage.yml` na raiz do backend:

```yaml
totalCoverageThreshold: 85

packages:
  "github.com/master-sindico/backend/internal/core":                                       95
  "github.com/master-sindico/backend/internal/modules/identity/domain":                    95
  "github.com/master-sindico/backend/internal/modules/identity/application":               90
  "github.com/master-sindico/backend/internal/modules/identity/infrastructure/database":   85
  "github.com/master-sindico/backend/internal/modules/identity/infrastructure/http":       75
  "github.com/master-sindico/backend/internal/modules/identity/infrastructure/providers":  85
  "github.com/master-sindico/backend/internal/shared/middleware":                          90
  "github.com/master-sindico/backend/internal/shared/providers":                           90
  "github.com/master-sindico/backend/pkg":                                                 90

exclude:
  - "**/generated/**"        # sqlc gerado
  - "**/mocks/**"            # mockery gerado
  - "cmd/**"                 # main packages
```

Gate em CI:

```bash
go test -count=1 -coverprofile=coverage.out ./...
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
```

Threshold não atingido → PR bloqueado. Exceção documentada em `.kiro/STATE.md` com prazo de pagamento.

## Ferramentas

```go
import (
    "testing"
    "github.com/stretchr/testify/assert"
    "github.com/stretchr/testify/require"
    "github.com/stretchr/testify/mock"

    "pgregory.net/rapid"   // PBT — preferido sobre gopter

    "github.com/testcontainers/testcontainers-go"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
    "github.com/testcontainers/testcontainers-go/modules/redis"
)
```

**Mocks**: `mockery` CLI, gerados em `application/mocks/`. Nunca escritos à mão para interfaces com 3+ métodos.

## Ciclo TDD aplicado — exemplo (CreateSubscriptionUseCase)

### RED — escreve o teste antes do código

```go
func TestCreateSubscriptionUseCase_Execute(t *testing.T) {
    t.Run("creates subscription when none exists", func(t *testing.T) {
        repo := new(MockSubscriptionRepository)
        gw := new(MockPaymentGateway)
        uow := new(MockUnitOfWork)
        logger := zerolog.Nop()

        uow.On("Run", mock.Anything, mock.AnythingOfType("func(context.Context) error")).
            Run(func(args mock.Arguments) {
                fn := args.Get(1).(func(context.Context) error)
                _ = fn(context.Background())
            }).Return(nil)
        repo.On("FindActiveByUserID", mock.Anything, "user-1").
            Return(nil, apperrors.ErrNotFound)
        gw.On("CreateSubscription", mock.Anything, mock.Anything).
            Return(&domain.SubscriptionOutput{ID: "sub_stripe_1"}, nil)
        repo.On("Save", mock.Anything, mock.AnythingOfType("*entities.Subscription")).
            Return(nil)

        uc := NewCreateSubscriptionUseCase(repo, gw, uow, nil, &logger)
        dto, err := uc.Execute(context.Background(),
            CreateSubscriptionCommand{UserID: "user-1", PlanID: "plan-plus", Interval: "monthly"})

        require.NoError(t, err)
        assert.NotEmpty(t, dto.ID)
        repo.AssertExpectations(t)
        gw.AssertExpectations(t)
    })
}
```

Roda: **falha** porque `CreateSubscriptionUseCase` não existe. Esperado.

### GREEN — implementação mínima para passar

Cria `CreateSubscriptionUseCase` com o mínimo necessário. Nada além. Nenhuma feature "enquanto estou aqui".

### REFACTOR — com teste verde, melhora design

Renomeia variáveis para clareza, extrai helper, elimina duplicação. Testes verdes confirmam que comportamento não mudou.

## Table-driven (padrão para casos múltiplos)

```go
func TestCreateSubscriptionUseCase_Execute(t *testing.T) {
    tests := []struct {
        name    string
        cmd     CreateSubscriptionCommand
        setup   func(repo *MockSubscriptionRepository, gw *MockPaymentGateway, uow *MockUnitOfWork)
        wantErr error
    }{
        {
            name: "creates subscription when none exists",
            cmd:  CreateSubscriptionCommand{UserID: "user-1", PlanID: "plan-plus", Interval: "monthly"},
            setup: func(repo *MockSubscriptionRepository, gw *MockPaymentGateway, uow *MockUnitOfWork) {
                uow.On("Run", mock.Anything, mock.AnythingOfType("func(context.Context) error")).
                    Run(func(args mock.Arguments) {
                        fn := args.Get(1).(func(context.Context) error)
                        _ = fn(context.Background())
                    }).Return(nil)
                repo.On("FindActiveByUserID", mock.Anything, "user-1").Return(nil, apperrors.ErrNotFound)
                gw.On("CreateSubscription", mock.Anything, mock.Anything).
                    Return(&domain.SubscriptionOutput{ID: "sub_stripe_1"}, nil)
                repo.On("Save", mock.Anything, mock.AnythingOfType("*entities.Subscription")).Return(nil)
            },
            wantErr: nil,
        },
        {
            name: "fails when active subscription exists",
            cmd:  CreateSubscriptionCommand{UserID: "user-1", PlanID: "plan-plus", Interval: "monthly"},
            setup: func(repo *MockSubscriptionRepository, gw *MockPaymentGateway, uow *MockUnitOfWork) {
                existing := entities.NewSubscription("user-1", "plan-base", 0)
                uow.On("Run", mock.Anything, mock.AnythingOfType("func(context.Context) error")).
                    Run(func(args mock.Arguments) {
                        fn := args.Get(1).(func(context.Context) error)
                        _ = fn(context.Background())
                    }).Return(apperrors.ErrConflict)
                repo.On("FindActiveByUserID", mock.Anything, "user-1").Return(existing, nil)
            },
            wantErr: apperrors.ErrConflict,
        },
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            repo := new(MockSubscriptionRepository)
            gw := new(MockPaymentGateway)
            uow := new(MockUnitOfWork)
            logger := zerolog.Nop()

            tt.setup(repo, gw, uow)

            uc := NewCreateSubscriptionUseCase(repo, gw, uow, nil, &logger)
            _, err := uc.Execute(context.Background(), tt.cmd)

            if tt.wantErr != nil {
                require.Error(t, err)
                assert.True(t, errors.Is(err, tt.wantErr))
            } else {
                require.NoError(t, err)
            }
            repo.AssertExpectations(t)
            gw.AssertExpectations(t)
        })
    }
}
```

Table-driven é **padrão** em Go. Um `t.Run` por caso. Nome descritivo no `name`.

## Property-Based Testing (regras críticas)

Usar `pgregory.net/rapid` para invariantes que table-driven não cobre exaustivamente.

```go
// Regra: fração ideal de todas as unidades soma exatamente 1.000000
func TestIdealFractionSumsToOne(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        unitCount := rapid.IntRange(1, 500).Draw(t, "unit_count")
        fractions := generateFractionsWithSum(t, unitCount)

        total := decimal.Zero
        for _, f := range fractions {
            total = total.Add(f)
        }

        assert.True(t, total.Equal(decimal.NewFromInt(1)),
            "frações devem somar 1.000000, got %s", total)
    })
}

// Regra: ABAC nunca concede ação admin a role não-admin
func TestABACNeverGrantsUnauthorized(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        role := rapid.SampledFrom([]string{"resident", "marketing", "local_company"}).Draw(t, "role")
        action := rapid.SampledFrom(adminOnlyActions).Draw(t, "action")

        result := abacEngine.Evaluate(ABACContext{Role: UserRole(role), Action: action})

        assert.False(t, result.Granted, "role %s não deve acessar %s", role, action)
    })
}
```

### Regras críticas que **devem** ter PBT

- Fração ideal de assembleia (soma = 1, sem drift de float)
- Quotas de Connect Me (reset anual, nunca negativo, nunca > cota do plano)
- ABAC engine (role × action × plan — sem falso positivo, sem bypass)
- Trial expiration (nunca acesso a feature gated após `trial_ends_at`)
- Tenant isolation (toda query inclui `condominium_id` correto)
- Valores monetários BRL (int64 centavos, nunca overflow, nunca negativo em cobrança)

Mínimo 100 casos (default rapid). Regras críticas: 1000.

## Integration — testcontainers-go

```go
//go:build integration

package database_test

func setupPostgres(t *testing.T) *pgxpool.Pool {
    t.Helper()
    ctx := t.Context()

    container, err := postgres.Run(ctx,
        "postgres:18-alpine",
        postgres.WithDatabase("master_sindico_test"),
        postgres.WithUsername("postgres"),
        postgres.WithPassword("postgres"),
        testcontainers.WithReuseByName("ms-pg-integration-test"),
    )
    require.NoError(t, err)
    t.Cleanup(func() { _ = container.Terminate(ctx) })

    connStr, err := container.ConnectionString(ctx, "sslmode=disable")
    require.NoError(t, err)

    pool, err := pgxpool.New(ctx, connStr)
    require.NoError(t, err)
    t.Cleanup(pool.Close)

    require.NoError(t, migrations.Up(ctx, pool))
    return pool
}

func TestSubscriptionRepository_Integration(t *testing.T) {
    pool := setupPostgres(t)
    defer truncateAll(t, pool)

    repo := NewSubscriptionRepository(pool)

    t.Run("saves and retrieves subscription", func(t *testing.T) {
        ctx := t.Context()
        sub := entities.NewSubscription("user-1", "plan-plus", 15)

        require.NoError(t, repo.Save(ctx, sub))

        found, err := repo.FindByID(ctx, sub.ID())
        require.NoError(t, err)
        assert.Equal(t, sub.ID(), found.ID())
        assert.Equal(t, entities.SubscriptionStatusTrialing, found.Status())
    })
}

func truncateAll(t *testing.T, pool *pgxpool.Pool) {
    t.Helper()
    _, err := pool.Exec(t.Context(), `TRUNCATE subscriptions, sessions, users, plans CASCADE`)
    require.NoError(t, err)
}
```

### Boas práticas

- **Container reuse** via `WithReuseByName(...)` — 5-10× mais rápido que recriar
- **TRUNCATE CASCADE** entre testes em vez de recriar container — mantém schema + índices
- `t.Context()` (Go 1.24+) em vez de `context.Background()` — cancelamento automático
- Migrations rodadas uma vez na criação
- Tag `integration` (`//go:build integration`) separa de unit

## HTTP Contract test

```go
func TestSubscriptionHandler_Create(t *testing.T) {
    useCase := new(MockCreateSubscriptionUseCase)
    handler := NewSubscriptionHandler(useCase, nil)

    router := gin.New()
    router.POST("/subscriptions", func(c *gin.Context) {
        handler.Create(server.GinContextAdapter(c))
    })

    useCase.On("Execute", mock.Anything, mock.MatchedBy(func(cmd CreateSubscriptionCommand) bool {
        return cmd.PlanID == "plan-plus"
    })).Return(&SubscriptionDTO{ID: "sub-1"}, nil)

    body := `{"plan_id":"plan-plus","interval":"monthly"}`
    req := httptest.NewRequest(http.MethodPost, "/subscriptions", strings.NewReader(body))
    req.Header.Set("Content-Type", "application/json")
    w := httptest.NewRecorder()

    router.ServeHTTP(w, req)

    assert.Equal(t, http.StatusCreated, w.Code)
    useCase.AssertExpectations(t)
}
```

Contract test valida: status, response body, headers. **Não** duplica lógica de negócio (isso é no teste do use case).

## Regras duras de teste

- Testes independentes — sem ordem, sem estado compartilhado mutável
- `time.Sleep` em teste: **proibido** — use `require.Eventually` ou channels
- Testes não logam exceto em `t.Log` (polui CI verbose)
- Banco de teste: container dedicado ou schema isolado por package
- Flaky tests: consertar ou deletar. "Flaky é melhor que nada" é **falso** — destrói confiança no CI
- `-race` sempre no CI — race detection é 100× mais barato em dev

## Executar

```bash
# Unit + race detection
go test ./... -count=1 -race

# Integração (tag)
go test ./... -count=1 -tags=integration

# Módulo específico
go test ./internal/modules/billing/... -count=1 -v

# Coverage
go test ./... -count=1 -coverprofile=coverage.out
go tool cover -html=coverage.out

# Coverage gate
go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml

# Benchmark (raro, só em suspeita de gargalo)
go test ./... -bench=. -benchmem -count=3 -run=^$
```

## Pipeline CI (exemplo completo)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    services:
      docker:
        options: "--privileged"
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }
      - run: go build ./...
      - run: go vet ./...
      - run: go test -race -count=1 -coverprofile=coverage.out ./...
      - run: go test -tags=integration -count=1 ./...
      - run: go install github.com/vladopajic/go-test-coverage@latest
      - run: go-test-coverage --profile=coverage.out --threshold-file=.coverage.yml
      - run: go install github.com/securego/gosec/v2/cmd/gosec@latest
      - run: gosec -severity high ./...
      - run: go install golang.org/x/vuln/cmd/govulncheck@latest
      - run: govulncheck ./...
```

Qualquer falha = PR bloqueado. Sem override manual sem justificativa em `.kiro/STATE.md`.
