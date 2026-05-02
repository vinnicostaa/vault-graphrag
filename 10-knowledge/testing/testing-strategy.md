---
title: Testing Strategy — TDD, pirâmide, thresholds
type: concept
tags: [testing, tdd, coverage, property-based, integration]
source: .kiro/steering/testing.md
created: 2026-04-22
aliases: [Testing strategy, Estratégia de testes]
---

# Testing Strategy — TDD, pirâmide, thresholds

## Filosofia: TDD por default

**Testes vêm antes da implementação.** Ciclo **Red → Green → Refactor**. Não é "depois quando sobrar tempo" — metodologia obrigatória.

```
1. RED       — escreve teste que falha (descreve comportamento desejado)
2. GREEN     — implementa o mínimo pra teste passar (sem over-engineer)
3. REFACTOR  — melhora código mantendo testes verdes
```

### Exceções aceitas (raras, documentadas em STATE)

- **Spike exploratório** pra validar viabilidade de SDK externo antes de contrato estável. Spike **sempre** descartado; implementação final nasce do zero com testes primeiro.
- **Refatoração puramente estética** (rename, move) sem mudança de comportamento — testes existentes cobrem.
- **Migration SQL** — o teste é a migration rodar up/down repetidamente em integração.

### Por que TDD, não "testes depois"

- **Força design testável**: acoplamento e dependências implícitas aparecem no primeiro teste. Use case difícil de testar revela falha de design.
- **Specs executáveis**: teste é documentação viva da intenção.
- **Refactor seguro**: suite verde torna refatoração mecânica em vez de arriscada.
- **Regressão zero**: bug que pegou uma vez tem teste daí em diante.

Legacy que **não** fazia TDD produz bugs silenciosos (PermissionService comentado, race condition em quotas, vazamento de domínio). Ver [[../methodology/legacy-as-reference]].

## Pirâmide de testes (ordem de valor)

```
           /\
          /E2\          Sprint final — fluxos completos (persona X faz jornada Y)
         /----\
        / INT  \        testcontainers: DB + cache reais
       /--------\
      /   UNIT   \      tabela-driven + PBT para regras críticas
     /------------\
    /   CONTRACT   \    testes de contrato HTTP (httptest)
   /________________\
```

- **Unit** — base. Rápidos (<100ms cada). Mocks pra deps externas. Cobrem lógica de domínio e use cases.
- **Contract (HTTP)** — valida request/response do handler. **Não duplica** regra de negócio (isso é no teste do use case).
- **Integration** — banco real via testcontainers. Cobrem repository + migration + codegen juntos.
- **E2E** — fluxos críticos end-to-end no sprint final. Playwright MCP no frontend, HTTP client no backend.

## Thresholds por camada (gate duro em CI)

| Camada | Coverage mínimo | Racional |
|---|---|---|
| `modules/*/domain/` | **95%** | Regras de negócio — testa todos os caminhos |
| `modules/*/application/` | **90%** | Use cases — testa commands, queries, erros |
| `modules/*/infrastructure/database/` | **85%** | Repositories — happy + erros de driver |
| `modules/*/infrastructure/http/` | **75%** | Handlers — contratos HTTP |
| `modules/*/infrastructure/providers/` | **85%** | Providers externos — mapeamento de erros SDK |
| `shared/middleware/` | **90%** | Segurança — todas as branches |
| `shared/providers/*` | **90%** | Infra compartilhada |
| `core/` | **95%** | Contracts e errors — invariantes |
| `pkg/**` | **90%** | Helpers (logger, utils, money) |
| **Global mínimo** | **85%** | Sanidade |

Threshold não atingido → PR **bloqueado**. Exceção documentada em STATE com prazo de pagamento.

Arquivo `.coverage.yml` na raiz define limites; excluir gerados (`generated/`, `mocks/`) e `cmd/**`.

```bash
<test> -count=1 -coverprofile=coverage.out ./...
<coverage-tool> --profile=coverage.out --threshold-file=.coverage.yml
```

## Padrões obrigatórios

### Red → Green → Refactor sempre

Teste primeiro. Implementação depois. Se não dá pra escrever teste antes, sinal de falha de design — revisar antes de codar.

### Arrange / Act / Assert explícito

```
// Arrange: setup de mocks, dados de entrada
// Act: chamada ao sistema sob teste
// Assert: verifica comportamento esperado
```

### Table-driven (padrão pra múltiplos casos)

Em Go:

```go
tests := []struct {
    name    string
    input   InputType
    setup   func(mocks ...)
    wantErr error
}{
    {name: "happy path", ...},
    {name: "fails when X", ...},
    {name: "fails when Y", ...},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        // setup + act + assert
    })
}
```

Um `t.Run` por caso. Nome descritivo no `name`.

### Mocks em helper compartilhado

Nunca redeclarar mock no topo de cada teste — vai em `<path>/mocks/` ou `test/helpers/`.

### Testes independentes

Sem ordem, sem estado compartilhado mutável. Cada teste monta e limpa seu próprio estado.

### Zero `any`/`interface{}` em teste

Use generics das libs (`testing-library`, `rapid`). Typo em teste fere o valor.

## Property-Based Testing (regras críticas)

PBT exerce invariantes que table-driven não cobre exaustivamente. Ferramentas: `pgregory.net/rapid` (Go), `fast-check` (TS), `hypothesis` (Python), `proptest` (Rust).

### Regras que **devem** ter PBT

- **Valores monetários** — inteiro, nunca overflow, nunca negativo em cobrança
- **Proporção com soma fixa** (fração ideal, percentuais) — soma exata, sem drift
- **Quotas** — reset, nunca negativo, nunca > limite do plano
- **ABAC engine** — role × action × plan, sem falso positivo, sem bypass
- **Trial expiration** — nunca acesso a feature gated após `trial_ends_at`
- **Tenant isolation** — toda query inclui `tenant_id` correto

### Exemplo (Go + rapid)

```go
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

func TestABACNeverGrantsUnauthorized(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        role   := rapid.SampledFrom([]string{"resident", "marketing", "local_company"}).Draw(t, "role")
        action := rapid.SampledFrom(adminOnlyActions).Draw(t, "action")

        result := abacEngine.Evaluate(ABACContext{Role: UserRole(role), Action: action})

        assert.False(t, result.Granted, "role %s não deve acessar %s", role, action)
    })
}
```

Mínimo 100 casos (default rapid). Regras críticas: 1000.

## Integration — testcontainers

Teste com banco **real**, não mock. Essencial pra validar repository + migration + codegen juntos.

### Padrão (Go + testcontainers-go)

```go
//go:build integration

func setupPostgres(t *testing.T) *pgxpool.Pool {
    t.Helper()
    ctx := t.Context()

    container, err := postgres.Run(ctx,
        "postgres:18-alpine",
        postgres.WithDatabase("app_test"),
        postgres.WithUsername("postgres"),
        postgres.WithPassword("postgres"),
        testcontainers.WithReuseByName("integration-test-pg"),
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

func truncateAll(t *testing.T, pool *pgxpool.Pool) {
    t.Helper()
    _, err := pool.Exec(t.Context(), `TRUNCATE subscriptions, sessions, users CASCADE`)
    require.NoError(t, err)
}
```

### Boas práticas

- **Container reuse** via `WithReuseByName(...)` — 5-10× mais rápido que recriar
- **TRUNCATE CASCADE** entre testes em vez de recriar container — mantém schema + índices
- `t.Context()` (Go 1.24+) em vez de `context.Background()` — cancelamento automático
- Migrations rodadas **uma vez** na criação do container
- **Build tag** separada (`//go:build integration`) — unit não depende de Docker

## HTTP Contract test

Valida contrato de API **sem** duplicar regra de negócio (isso fica no teste do use case).

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

Valida: status, response body, headers. **Não** lógica.

## Regras duras

- **Testes independentes** — sem ordem, sem estado compartilhado mutável
- **`time.Sleep` em teste: proibido** — use `require.Eventually` ou channels
- **Testes não logam** exceto em `t.Log` (polui CI verbose)
- **Banco de teste**: container dedicado ou schema isolado por package
- **Flaky tests**: consertar ou deletar. "Flaky é melhor que nada" é **falso** — destrói confiança no CI
- **`-race` sempre** no CI — race detection é 100× mais barato em dev

## Pipeline CI (esqueleto)

```yaml
jobs:
  test:
    steps:
      - uses: actions/checkout@v4
      - <setup-lang>
      - run: <build> ./...
      - run: <lint> ./...
      - run: <test> -race -count=1 -coverprofile=coverage.out ./...
      - run: <test> -tags=integration -count=1 ./...
      - run: <coverage-tool> --profile=coverage.out --threshold-file=.coverage.yml
      - run: <security-scanner> -severity high ./...
      - run: <vuln-check> ./...
```

Qualquer falha = PR bloqueado. Sem override manual sem justificativa em STATE.

## Links

- [[_moc]]
- [[../methodology/sdd-workflow]] — `<verify>` declara testes no plano antes do código
- [[../principles/do-dont-checklist]] — TDD obrigatório
- [[../../30-agents/mcp-integration]] — Playwright MCP em E2E
- [[../antipatterns/_moc]] — testes **não** cobrem antipatterns isolados; cobrem invariantes
