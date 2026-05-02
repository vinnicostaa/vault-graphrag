---
title: Integration tests — Master Síndico
type: guide
tags: [testing, integration, testcontainers, postgres, redis, multi-tenant, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 90-ingested
created: 2026-04-23
updated: 2026-04-23
aliases: [integration-tests-master-sindico]
---

# Integration tests — Master Síndico v2

Integration tests com **testcontainers-go** (Postgres 18 real, Redis 7 real, opcionalmente NATS JetStream), build tag `//go:build integration`, fixtures via factory, setup/teardown canônicos, 100+ cross-tenant tests sistemáticos.

> Herda [[test-strategy]] §3 e os thresholds em [[coverage-thresholds]]. Exemplos de patterns integrados em [[../07-quality/PATTERNS]].

---

## 1. Setup canônico

### Build tag

```go
//go:build integration

package userrepo_test
```

Rodar isolado:

```bash
go test -tags=integration -count=1 ./...
```

Nunca `// +build` (legacy). Sempre `//go:build`.

### Helper reusable (`internal/shared/testutil/pg.go`)

```go
//go:build integration

package testutil

import (
    "context"
    "database/sql"
    "sync"
    "testing"

    "github.com/stretchr/testify/require"
    "github.com/testcontainers/testcontainers-go/modules/postgres"
)

var (
    once       sync.Once
    pgContainer *postgres.PostgresContainer
    dsn        string
)

func PgPool(t *testing.T) *sql.DB {
    once.Do(func() {
        ctx := context.Background()
        c, err := postgres.Run(ctx,
            "postgres:18-alpine",
            postgres.WithDatabase("test"),
            postgres.WithUsername("test"),
            postgres.WithPassword("test"),
        )
        require.NoError(t, err)
        pgContainer = c
        dsn, _ = c.ConnectionString(ctx, "sslmode=disable")
        runMigrations(dsn) // embed.FS via goose
    })
    db, err := sql.Open("pgx", dsn)
    require.NoError(t, err)
    return db
}

func TruncateAll(t *testing.T, db *sql.DB) {
    _, err := db.Exec(`TRUNCATE TABLE users, sessions, ... RESTART IDENTITY CASCADE`)
    require.NoError(t, err)
}
```

### Pattern em `*_integration_test.go`

```go
//go:build integration

package userrepo_test

func TestMain(m *testing.M) {
    os.Exit(m.Run()) // container sobe lazy no primeiro uso
}

func TestUserRepo_when_saveExistingCPF_should_returnConflict(t *testing.T) {
    db := testutil.PgPool(t)
    defer testutil.TruncateAll(t, db)

    repo := NewUserRepo(db)
    u1, _ := domain.NewUser(cpf, email1, RoleSindico)
    u2, _ := domain.NewUser(cpf, email2, RoleSindico) // mesmo CPF

    require.NoError(t, repo.Save(context.Background(), u1))
    err := repo.Save(context.Background(), u2)

    require.ErrorIs(t, err, repository.ErrConflict)
}
```

---

## 2. O que cobrir em integration

### Repositories
- Happy path (insert, update, select, delete soft).
- UNIQUE constraint violation → `repository.ErrConflict`.
- FK violation → `repository.ErrDependency`.
- Pagination (cursor + limit + stable order).

### Migrations
- `up` + `down` idempotentes.
- `goose up` roda em container fresco sem erro.

### UoW (transações)
- Tx commit → state persistido.
- Tx rollback → state zerado.
- Nested tx rejeitado (ou flattened conforme política).
- Outbox insertado na mesma tx (INV-095).

### Cross-tenant isolation (≥ 100 testes sistemáticos)

Pra cada repo de tabela com `tenant_id`:
- Tenant X cria N records.
- Tenant Y consulta → deve ver **zero** do X.
- Tenant Y tenta update/delete por ID do X → `ErrNotFound` (não `ErrForbidden`, pra evitar enumeration).

Template reusable:

```go
func TestCrossTenantIsolation_voteRepo(t *testing.T) {
    db := testutil.PgPool(t)
    defer testutil.TruncateAll(t, db)

    repo := NewVoteRepo(db)
    tenantA, tenantB := ULIDNew(), ULIDNew()

    // Tenant A insere
    voteA, _ := domain.NewVote(tenantA, ...)
    require.NoError(t, repo.Save(ctx, voteA))

    // Tenant B consulta
    got, err := repo.Get(ctx, tenantB, voteA.ID())

    require.ErrorIs(t, err, repository.ErrNotFound)
    require.Nil(t, got)
}
```

Gerar permutas em loop (tenant A,B,C × acao GET/UPDATE/DELETE × recursos existing/not-existing).

### Idempotency (webhook)

```go
func TestStripeWebhook_when_sameEventIdTwice_should_processOnce(t *testing.T) {
    app := bootstrapTestApp(t)
    body := fixturePayload("stripe_invoice_paid.json")
    sig := signHMAC(body, secret)

    // 1ª entrega
    r1 := app.Post("/webhooks/stripe", body, headers{"Stripe-Signature": sig})
    require.Equal(t, 200, r1.Code)

    // 2ª entrega com mesmo event.id
    r2 := app.Post("/webhooks/stripe", body, headers{"Stripe-Signature": sig})
    require.Equal(t, 200, r2.Code)

    // Verifica que idem ledger registrou apenas 1 processamento
    count, _ := app.DB.QueryRow("SELECT count(*) FROM webhook_events WHERE event_id=$1", "evt_123").Scan()
    require.Equal(t, 1, count)
}
```

### Rate limit + cleanup (AP-006 fix)

```go
func TestRateLimiter_when_highChurn_should_cleanupInactiveEntries(t *testing.T) {
    clock := FakeClock{At: time.Now()}
    rl := NewRateLimiter(clock, CleanupEvery(1*time.Minute), InactiveTTL(10*time.Minute))

    for i := 0; i < 10_000; i++ {
        rl.Allow(fmt.Sprintf("ip-%d", i))
    }
    clock.Advance(11 * time.Minute)
    rl.RunCleanup()

    require.Equal(t, 0, rl.Size())
}
```

### TOCTOU / race (INV-071, AP-026)

```go
func TestCastVote_when_concurrent_should_enforceUniqueConstraint(t *testing.T) {
    db := testutil.PgPool(t)
    defer testutil.TruncateAll(t, db)

    seedAssembly(db, assemblyID)
    ctx := context.Background()
    uc := NewCastVoteUseCase(NewAssemblyRepo(db), NewVoteRepo(db), realClock)

    const N = 100
    var wg sync.WaitGroup
    errs := make(chan error, N)

    for i := 0; i < N; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            _, err := uc.Execute(ctx, CastVoteCommand{
                VoterID: voterID, AgendaItemID: agendaID, Option: OptionYes,
            })
            errs <- err
        }()
    }
    wg.Wait()
    close(errs)

    var success, conflict int
    for err := range errs {
        switch {
        case err == nil:
            success++
        case errors.Is(err, apperrors.ErrConflict):
            conflict++
        default:
            t.Fatalf("unexpected: %v", err)
        }
    }
    require.Equal(t, 1, success)
    require.Equal(t, N-1, conflict)
}
```

---

## 3. Cross-BC integration (eventos)

Testa que evento publicado em BC A → consumer em BC B reage corretamente dentro de tx/outbox.

```go
func TestClosedDealSigned_should_appendTimelineEntry(t *testing.T) {
    // INV-041: ClosedDeal → TimelineEntry auto
    app := bootstrapTestApp(t)
    signClosedDeal(app, dealID)

    // aguarda consumer processar (polling curto)
    require.Eventually(t, func() bool {
        var count int
        app.DB.QueryRow("SELECT count(*) FROM timeline_entries WHERE ref_id=$1", dealID).Scan(&count)
        return count == 1
    }, 5*time.Second, 100*time.Millisecond)
}
```

---

## 4. HTTP integration (httptest)

Handlers completos com middleware stack + DB real, sem subir server externo.

```go
func TestPostVote_e2eHandler_when_validInput_should_return201(t *testing.T) {
    app := bootstrapTestApp(t)
    token := issueToken(t, userID, []string{"sindico"})
    body := `{"option":"yes"}`

    req := httptest.NewRequest("POST",
        fmt.Sprintf("/api/v1/assemblies/%s/vote/%s", assemblyID, agendaID), strings.NewReader(body))
    req.Header.Set("Authorization", "Bearer "+token)
    req.Header.Set("Content-Type", "application/json")
    rec := httptest.NewRecorder()

    app.Router.ServeHTTP(rec, req)

    require.Equal(t, 201, rec.Code)
}
```

---

## 5. Fixtures e factories

Factories em `internal/modules/<bc>/testutil/`:

```go
func NewAssemblyFactory() AssemblyFactory { ... }
func (f AssemblyFactory) WithPresident(u ULID) AssemblyFactory { ... }
func (f AssemblyFactory) Persist(t *testing.T, db *sql.DB) *Assembly { ... }
```

Fixtures JSON para webhook payloads: `internal/modules/<bc>/testdata/*.json` (não-versionado se contiver dados reais; usar dados sintéticos).

---

## 6. Isolation + parallelism

- `TruncateAll()` no setup de cada teste — **não** `defer` que roda depois (setup garante estado limpo).
- `t.Parallel()` permitido em testes que usam **pools de banco separados por teste** (schema-per-test); senão, sem paralelismo.
- Timeout por teste com `context.WithTimeout(ctx, 30*time.Second)` — teste integration que passa 30s = bug.

---

## 7. CI — service containers

```yaml
jobs:
  integration:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:18-alpine
        env: { POSTGRES_PASSWORD: test }
        ports: ['5432:5432']
        options: >-
          --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5
      redis:
        image: redis:7-alpine
        ports: ['6379:6379']
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with: { go-version: '1.26' }
      - run: go test -tags=integration -count=1 -timeout=10m ./...
```

Alternativa local: `testcontainers-go` starta containers on-demand (preferido pra dev).

---

## 8. Anti-patterns

| Sintoma | Fix |
|---|---|
| Teste sem `//go:build integration` mexendo em DB | Adicionar build tag ou virar unit |
| Container compartilhado sem truncate | `TruncateAll()` obrigatório no setup |
| Seed global via `TestMain` | Factory por teste |
| Tempo real (`time.Sleep`) esperando consumer | `require.Eventually(...)` com timeout explícito |
| Teste depende de ordem | `-shuffle=on` deve passar |
| Sem teste cross-tenant por repo de tabela scoped | Blocker — 100+ casos obrigatórios |
| Integration test que não usa tx/pool real | Virar unit |

---

## 9. Checklist do review

- [ ] `//go:build integration`?
- [ ] `TruncateAll()` no setup?
- [ ] Cross-tenant isolation coberto?
- [ ] UNIQUE/FK/CHECK violations convertidos pra `ErrConflict`/`ErrNotFound`?
- [ ] TOCTOU testado com goroutines + `sync.WaitGroup`?
- [ ] Idempotency testada (webhook, commands retry-safe)?
- [ ] Timeout por teste explícito?
- [ ] Factory em vez de fixture estática?
- [ ] CI roda em pipeline isolado com services PG/Redis?

---

## 10. Links

- [[_moc]]
- [[test-strategy]]
- [[unit]]
- [[pbt]]
- [[e2e]]
- [[coverage-thresholds]]
- [[../07-quality/PATTERNS]]
- [[../07-quality/antipatterns]]
- [[../01-domain/invariants]]
- [[../02-architecture/topology-multitenant]]
