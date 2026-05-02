---
title: Test Strategy — Master Síndico
type: project
tags: [master-sindico, testing, strategy, tdd, pbt]
project: master-sindico
created: 2026-04-22
---

# Test Strategy — Master Síndico

Pirâmide + PBT para regras críticas + integration com testcontainers + E2E para fluxos críticos + load/security.

---

## Pirâmide de Testes (alvo)

```
                    10%  E2E (Playwright)
                  20%  Integration (testcontainers-go)
                70%  Unit (go test + PBT)
```

Inversão (muitos E2E, poucos unit) = red flag; sinal de que unit não cobre cenários.

---

## Thresholds de cobertura

Ver [[../CLAUDE#8-coverage-thresholds-coverageyml]]. Resumo:

| Camada | Mínimo |
|---|---|
| `domain/` | 95% |
| `application/` | 90% |
| `infrastructure/database/` | 85% |
| `infrastructure/http/` | 75% |
| `shared/middleware/` | 90% |
| `core/`, `pkg/` | 95% |
| **Global** | ≥ 85% |

Gate de CI: `go tool cover -func=coverage.out` + awk/jq parser bloqueia merge.

---

## Tipos de teste (aplicação)

### 1. Unit (Go)

- `*_test.go` ao lado do arquivo
- Table-driven:
  ```go
  tests := []struct {
      name     string
      input    Input
      want     Output
      wantErr  error
  }{
      {"happy_path", ..., ..., nil},
      {"error_invalid_cpf", ..., ..., ErrValidation},
      ...
  }
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) {
          got, err := sut.Do(ctx, tt.input)
          assert.ErrorIs(t, err, tt.wantErr)
          assert.Equal(t, tt.want, got)
      })
  }
  ```
- **Sem mocktail / mockery** — stubs hand-rolled simples (ex: `type userRepoStub struct{ fn func(...) ... }`)
- **Sem DB real** em unit

### 2. Property-Based (Go)

Biblioteca: `pgregory.net/rapid` v1.2.0.

**Regras críticas** (obrigatório PBT):
- Fração ideal: soma ≤ 100% em qualquer sequência de Add/Remove
- Quotas billing: `Consume` nunca negativo; `Unlimited` nunca decrementa; `ZeroLimit` sempre falha
- ABAC matrix: admin_bypass > role_check > tenant_check (ordem)
- Money BRL: op sempre em int64 centavos; no float32/64 em cálculo
- Tenant isolation: cross-tenant 403 em qualquer combinação user+tenant+resource

Exemplos ativos:
- `agenda_item_pbt_test.go` — fração ideal (F6, 100 casos)
- `feature_usage_pbt_test.go` — quotas (F31, 300 casos ~10ms)
- `abac_engine_pbt_test.go` — ABAC (F32, 400 casos ~22ms)

### 3. Integration (testcontainers-go)

- Postgres 18 real via testcontainer
- Tag build: `// +build integration` (atualizar pra `//go:build integration`)
- Helper: `internal/shared/testutil/pg.go` — start container reusable + `TruncateAll()` entre testes
- Lifecycle: `TestMain(m)` sobe container; tests chamam `TruncateAll` no setup

Exemplos ativos (8 total):
- condominium, user, subscription, feature_usage, timeline, supplier_vote, session (1-device), announcement

Rodar: `go test -tags=integration -count=1 ./...`

### 4. E2E (Playwright, web)

- `web/tests/e2e/` (a criar no Sprint 11+)
- Fluxos críticos apenas:
  - Cadastro síndico → contrata plano → onboard condomínio
  - Login morador → vê home → acessa Connect Me
  - Upload vídeo empresa → moderação → publicado
  - Assembleia: entrar → votar → ver ata

### 5. Mobile E2E

- `flutter integration_test` + Appium se necessário
- Fluxos:
  - Login + refresh automático
  - Push notification recebido abre tela correta
  - Offline → online sync

### 6. Load (k6)

- Scripts em `tests/load/`
- Cenários (a criar em Sprint 11+):
  - **Auth storm**: 1000 req/s em `/auth/login` — rate limit protege
  - **Assembleia concorrente**: 10k users em mesma assembleia ao vivo
  - **Connect Me burst**: 100 req/s em criação de Connect Me request

### 7. Security (SAST, DAST, SCA)

- SAST: `gosec -severity=high` + Semgrep rules custom
- DAST: OWASP ZAP baseline em staging (manual por enquanto; automatizar em Sprint 12)
- SCA: `govulncheck` + `osv-scanner` — ver [[cve-process]]

### 8. Contract (OpenAPI)

- `spectral lint 02-architecture/openapi/*.yaml`
- `schemathesis` or `dredd` contra API em staging (Sprint 11+)

### 9. Chaos (futuro)

- Sprint 14+
- Gremlin ou toxic-proxy local
- Cenários: DB latência, Redis down, Livekit 503, Stripe 500

---

## Factories vs Fixtures

**Factories (recomendado)**:
```go
func NewUserFactory() UserFactory {
    return UserFactory{
        id:    uuid7.New(),
        email: faker.Email(),
        cpf:   faker.CPF(),
    }
}
func (f UserFactory) WithRole(r string) UserFactory { ... }
func (f UserFactory) Build() *User { ... }
```

**Fixtures estáticas proibidas** (quebram ao mínimo refactor).

---

## Dados em testes

**Proibido**:
- Dados reais de usuários (LGPD)
- Tokens reais
- Keys reais

**Permitido**:
- Faker (gofaker, @faker-js/faker)
- Seed determinístico pra reproduzir flakes
- CPF válido algorítmicamente gerado (mas não de pessoa real)

---

## Test isolation

- Cada teste: estado isolado (TruncateAll em integration; fresh stubs em unit)
- **Sem shared state** entre testes
- **Sem ordem dependência**: `go test -shuffle=on` deve passar
- **Race detection** obrigatório: `-race`

---

## Flaky test policy

1. Teste flaky detectado → **quarentena** com `t.Skip("flaky, ticket T-###")`
2. Ticket aberto com dump + repro
3. SLA de 1 sprint pra resolver
4. **Nunca** `go test -count=100 || true` como "solução"

---

## Mutation testing (piloto)

- `go-mutesting` em `domain/` de identity (Sprint 11 piloto)
- Meta inicial: kill rate ≥ 60%
- Expandir a outros módulos se valor comprovado

---

## Performance testing em unit

Benchmarks (`*_bench_test.go`) pra hot paths:
- Sessions query
- ABAC decision
- Mapper hot path (row → entity)

Rodar: `go test -bench=. -benchmem ./...`

---

## Ferramentas instaladas

| Tool | Instalado em | Finalidade |
|---|---|---|
| `go test` | stdlib | Unit + integration |
| `pgregory.net/rapid@v1.2.0` | go.mod | PBT |
| `testcontainers-go` | go.mod | Integration DB real |
| `/home/vinni/go/bin/gosec` | manual | SAST |
| `/home/vinni/go/bin/govulncheck` | manual | CVE |
| Playwright (web) | npm (Sprint 11+) | E2E web |
| Appium / integration_test (mobile) | pub (Sprint 11+) | E2E mobile |
| k6 | brew/apt | Load |

---

## CI pipeline de teste

```yaml
name: CI
on: [push, pull_request]
jobs:
  backend-tests:
    steps:
      - checkout
      - setup-go: 1.26
      - go mod download
      - go build ./...
      - go vet ./...
      - go test -race -count=1 -coverprofile=coverage.out ./...
      - coverage-check: threshold=85
      - gosec -severity=high -exit-status ./...
      - govulncheck ./...
  backend-integration:
    needs: backend-tests
    services: postgres:18, redis:7
    steps:
      - go test -tags=integration -count=1 ./...
  web-tests:
    steps:
      - bun install
      - bun run typecheck
      - bun run test
      - bun run build
  mobile-tests:
    steps:
      - flutter test
      - flutter analyze
```

---

## Links

- [[_moc]]
- [[coverage-thresholds]]
- [[unit-tests]]
- [[property-based-tests]]
- [[integration-tests]]
- [[e2e-tests]]
- [[load-tests]]
- [[security-tests]]
- [[../07-quality/PATTERNS]]
- [[../08-security/cve-process]]
- [[../06-execution/STEPS]]
- [[../../../10-knowledge/testing/testing-strategy]]
