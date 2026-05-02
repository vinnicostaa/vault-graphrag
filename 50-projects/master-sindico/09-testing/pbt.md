---
title: Property-based testing (PBT) — Master Síndico
type: guide
tags: [testing, pbt, rapid, invariants, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 01-domain/invariants.md + 90-ingested
created: 2026-04-23
updated: 2026-04-23
aliases: [pbt-master-sindico, property-based-tests]
---

# Property-based testing (PBT) — Master Síndico v2

PBT com **`pgregory.net/rapid@v1.2.0`** em invariantes críticos. Ao invés de casos example-based, geramos entrada aleatória satisfazendo precondição e verificamos propriedade. Rapid aplica **shrinking** automático em falha para isolar mínimo caso.

> Herda [[test-strategy]] §2 (unit). Invariantes canônicos em [[../01-domain/invariants]]. Obrigatório para: INV-016 (quota), INV-021 (fração ideal), INV-086 (ABAC matrix), INV-071 (vote TOCTOU), INV-043 (evaluation TOCTOU).

---

## 1. Quando usar PBT

Usar PBT **obrigatoriamente** quando:

- **Muitas combinações de input** (mais que table-driven comporta).
- **Propriedade matemática**: `sum, avg, bounds, conservation, monotonicity`.
- **Mutações preservam propriedade**: `Add(x); Remove(x)` deve voltar ao estado original.
- **Race/TOCTOU concorrente**: PBT gera ordens variadas de execução.

**NÃO usar** quando:
- Caso single-ou-dois óbvio (table-driven basta).
- Input não tem estrutura combinatória (ex: parse de JSON fixo).
- Propriedade é expressa melhor via `require.Equal` direto.

---

## 2. Setup rapid

```go
import (
    "testing"
    "pgregory.net/rapid"
)

func TestFracaoIdeal_SumLeq100(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        // ... gerar fractions, verificar propriedade
    })
}
```

Rodar: `go test -run=TestFracaoIdeal_SumLeq100 -rapid.checks=1000` — 1000 casos (default 100).

---

## 3. Invariantes obrigatórios com PBT

### INV-021, INV-022 — Σ FracaoIdeal ≤ 100% (100 casos)

```go
// domain/value_objects/fracao_ideal_pbt_test.go
func TestUnits_AddAndRemove_neverExceeds100(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        units := &Units{}
        fracs := rapid.SliceOfN(
            rapid.Float64Range(0.01, 100.0),
            1, 200,
        ).Draw(t, "fracs")

        totalAdded := 0.0
        for _, f := range fracs {
            fv, err := NewFracaoIdeal(f)
            if err != nil { continue } // out of (0,100]
            if totalAdded+f > 100.0 {
                // invariante: deve rejeitar
                err := units.Append(Unit{f: fv})
                require.Error(t, err, "expected rejection when sum would exceed 100")
                continue
            }
            require.NoError(t, units.Append(Unit{f: fv}))
            totalAdded += f
        }

        // propriedade final
        require.True(t, units.SumFraction().BasisPoints() <= 10000,
            "Σ fração ideal deve permanecer ≤ 100%")
    })
}
```

**Shrinking**: em falha, rapid reduz a lista até o menor exemplo que viola — típicamente 2 fracs cuja soma > 100.

---

### INV-016, BR-014 — Quota Consume/Reset (300 casos)

```go
// billing/domain/quota_pbt_test.go
func TestQuota_ConsumeNeverNegative_ResetGoesToLimit(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        limit := rapid.IntRange(1, 1000).Draw(t, "limit")
        q := NewQuota(limit, /*unlimited=*/false)

        ops := rapid.SliceOf(
            rapid.OneOf(
                rapid.Custom(func(t *rapid.T) op {
                    n := rapid.IntRange(1, 100).Draw(t, "n")
                    return op{kind: "consume", n: n}
                }),
                rapid.Custom(func(t *rapid.T) op { return op{kind: "reset"} }),
            ),
        ).Draw(t, "ops")

        for _, o := range ops {
            switch o.kind {
            case "consume":
                before := q.Used()
                err := q.Consume(o.n)
                if err == nil {
                    require.True(t, q.Used() == before+o.n)
                    require.True(t, q.Used() <= q.Limit())
                } else {
                    require.ErrorIs(t, err, ErrQuotaExceeded)
                    require.Equal(t, before, q.Used()) // inalterado em erro
                }
            case "reset":
                q.Reset()
                require.Equal(t, 0, q.Used())
            }
        }

        require.True(t, q.Used() >= 0, "used nunca negativo")
        require.True(t, q.Used() <= q.Limit(), "used ≤ limit")
    })
}
```

---

### INV-086, BR-059 — ABAC engine (400 casos, ~22ms)

```go
// cross-domain/abac/engine_pbt_test.go
func TestABAC_OrderedEvaluation(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        user := User{
            ID:      ULIDNew(),
            Role:    rapid.SampledFrom([]Role{RoleSindico, RoleMorador, RoleEmpresa, RoleAdmin}).Draw(t, "role"),
            IsAdmin: rapid.Bool().Draw(t, "isAdmin"),
        }
        action := rapid.SampledFrom(actionCatalog).Draw(t, "action")
        tenant := rapid.SampledFrom(tenants).Draw(t, "tenant")

        engine := NewABACEngine(defaultConfig)
        decision := engine.Evaluate(EvalRequest{User: user, Action: action, Tenant: tenant})

        // Propriedade 1: admin_bypass pula resto
        if user.IsAdmin {
            require.True(t, decision.Allowed)
            require.Equal(t, "admin_bypass", decision.Reason)
            return
        }

        // Propriedade 2: action não configurada → deny
        if !defaultConfig.ActionConfigured(action) {
            require.False(t, decision.Allowed)
            require.Equal(t, "action_not_configured", decision.Reason)
            return
        }

        // Propriedade 3: role permitida vs proibida
        if !defaultConfig.RoleAllowed(user.Role, action) {
            require.False(t, decision.Allowed)
            require.Equal(t, "role_not_allowed", decision.Reason)
            return
        }

        // Propriedade 4: tenant não bate → deny
        if !user.Tenants.Contains(tenant) {
            require.False(t, decision.Allowed)
            require.Equal(t, "tenant_mismatch", decision.Reason)
            return
        }

        require.True(t, decision.Allowed)
    })
}
```

**Nota**: 400 casos em ~22ms (benchmark). CI roda `-rapid.checks=500` nessa suite.

---

### INV-071, INV-042, INV-043 — TOCTOU (vote/evaluation único)

PBT + goroutines concorrentes (integration, não unit, pelo DB real):

```go
func TestCastVote_TOCTOU_enforcesUnique(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        concurrency := rapid.IntRange(2, 50).Draw(t, "concurrency")
        db := testutil.PgPool(t)
        testutil.TruncateAll(t, db)
        seedAssembly(db, assemblyID, agendaItemID, voterID)

        uc := NewCastVoteUseCase(NewAssemblyRepo(db), NewVoteRepo(db), realClock)

        var wg sync.WaitGroup
        results := make(chan error, concurrency)
        for i := 0; i < concurrency; i++ {
            wg.Add(1)
            go func() {
                defer wg.Done()
                _, err := uc.Execute(ctx, CastVoteCommand{
                    AssemblyID: assemblyID, AgendaItemID: agendaItemID,
                    VoterID: voterID, Option: OptionYes,
                })
                results <- err
            }()
        }
        wg.Wait()
        close(results)

        success, conflict := 0, 0
        for err := range results {
            switch {
            case err == nil: success++
            case errors.Is(err, apperrors.ErrConflict): conflict++
            }
        }

        // propriedade TOCTOU: exatamente 1 sucesso, resto conflict
        require.Equal(t, 1, success, "exatamente 1 voto aceito")
        require.Equal(t, concurrency-1, conflict, "todos os outros → ErrConflict")
    })
}
```

---

### Pagination (monotonicity + idempotent page iteration)

```go
func TestPagination_MonotonicAndNoDupes(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        total := rapid.IntRange(0, 1000).Draw(t, "total")
        limit := rapid.IntRange(1, 50).Draw(t, "limit")

        seen := map[ULID]bool{}
        var cursor *ULID
        lastKey := ULID{}
        for {
            page, next := listPage(total, limit, cursor)
            for _, item := range page {
                require.False(t, seen[item.ID], "nenhum item duplicado")
                require.True(t, item.SortKey >= lastKey, "ordem monotônica")
                seen[item.ID] = true
                lastKey = item.SortKey
            }
            if next == nil { break }
            cursor = next
        }
        require.Equal(t, total, len(seen), "todos os itens visitados")
    })
}
```

---

### Money / Currency (associativity + int64 only — INV-014)

```go
func TestMoney_AddAssociative(t *testing.T) {
    rapid.Check(t, func(t *rapid.T) {
        a := NewMoneyBRL(rapid.Int64Range(-1_000_000, 1_000_000).Draw(t, "a"))
        b := NewMoneyBRL(rapid.Int64Range(-1_000_000, 1_000_000).Draw(t, "b"))
        c := NewMoneyBRL(rapid.Int64Range(-1_000_000, 1_000_000).Draw(t, "c"))
        require.Equal(t, a.Add(b).Add(c), a.Add(b.Add(c)))
    })
}
```

---

## 4. Contagem de casos por propriedade

| Invariante | Casos rapid | Tempo alvo |
|---|---|---|
| INV-021 FracaoIdeal Σ ≤ 100% | 100 | < 10ms |
| INV-016 Quota | 300 | < 50ms |
| INV-086 ABAC matrix | 400 | < 30ms |
| INV-071 Vote TOCTOU | 50 rounds × 20 goroutines | < 2s |
| INV-043 Evaluation TOCTOU | 50 × 10 | < 1s |
| Pagination | 200 | < 20ms |
| Money associativity | 500 | < 10ms |
| CPF módulo-11 | 1000 | < 15ms |

Soma PBT total em CI unit: ~3s. Tolerável.

---

## 5. Shrinking

Quando rapid detecta falha, ele **reduz** a entrada ao mínimo caso. Exemplo:

```
--- FAIL: TestUnits_AddAndRemove_neverExceeds100 (0.02s)
    fracao_ideal_pbt_test.go:23: [rapid] failed after 42 tests, shrunk to:
        fracs = [50.0, 50.01]
```

Esse é o caso mínimo que viola. Fix: `Append` deve rejeitar antes de exceder 100.

**Para reproduzir falha específica**:

```bash
go test -run=TestX -rapid.seed=<seed> -rapid.verbose
```

---

## 6. Custom generators

Para tipos do domínio, criar generators reutilizáveis:

```go
// testutil/gen.go
func GenEmailVO() *rapid.Generator[EmailVO] {
    return rapid.Custom(func(t *rapid.T) EmailVO {
        local := rapid.StringMatching(`[a-z0-9]{1,20}`).Draw(t, "local")
        domain := rapid.SampledFrom([]string{"test.com", "ms-test.br", "x.io"}).Draw(t, "domain")
        email, _ := NewEmail(local + "@" + domain)
        return email
    })
}

func GenCPFVO() *rapid.Generator[CPFVO] {
    return rapid.Custom(func(t *rapid.T) CPFVO {
        // Gera 9 dígitos aleatórios + 2 dígitos verificadores (módulo-11)
        ...
    })
}
```

---

## 7. Anti-patterns PBT

| Sintoma | Fix |
|---|---|
| Propriedade tautológica (`x == x`) | Testar propriedade **não trivial** (ordem, sum, conservation) |
| Generator sem distribuição | Usar `rapid.Bool`/`SampledFrom`/`OneOf` pra diversidade |
| PBT em tudo | Só em invariantes + props matemáticas |
| Shrinking ignorado | Sempre investigar menor exemplo que falhou |
| Concorrência sem synchronization | PBT + goroutines exige `sync.WaitGroup` + channel |
| PBT em integration que demora > 5s | Reduzir casos ou mover pra load test |

---

## 8. Checklist review PBT

- [ ] Propriedade é **não trivial**?
- [ ] Gerador cobre casos de fronteira (zero, limite, negativo)?
- [ ] Número de casos apropriado?
- [ ] Shrinking produz caso mínimo legível em falha?
- [ ] Integra com invariante documentado em [[../01-domain/invariants]]?
- [ ] Custom generators reutilizados via `testutil/`?

---

## 9. Links

- [[_moc]]
- [[test-strategy]]
- [[unit]]
- [[integration]]
- [[../01-domain/invariants]]
- [[../01-domain/business-rules]]
- [[../07-quality/PATTERNS]]
- https://pkg.go.dev/pgregory.net/rapid
