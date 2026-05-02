---
title: Unit tests — Master Síndico
type: guide
tags: [testing, unit, go, table-driven, master-sindico, v2]
source: vault/10-knowledge/testing/testing-strategy.md + 90-ingested
created: 2026-04-23
updated: 2026-04-23
aliases: [unit-tests-master-sindico]
---

# Unit tests — Master Síndico v2

Unit tests em Go: **table-driven obrigatório**, stubs hand-rolled (sem mockery/mocktail), `stretchr/testify/require`, arrange-act-assert, nomenclatura `TestX_when_Y_should_Z`. Cobertura por camada segue thresholds em [[coverage-thresholds]].

> Sem DB real, sem HTTP real, sem SDK real. Unit = puramente em-memória, determinístico, < 100ms cada.

---

## 1. Anatomia de um unit test

### Pattern canônico table-driven

```go
// domain/value_objects/fracao_ideal_test.go
package valueobjects

import (
    "testing"

    "github.com/stretchr/testify/require"
    apperrors "github.com/mastersindico/backend/core/errors"
)

func TestNewFracaoIdeal_when_inputValid_should_returnVO(t *testing.T) {
    tests := []struct {
        name    string
        input   float64
        wantBps int64
        wantErr error
    }{
        {"zero rejeitado", 0, 0, apperrors.ErrValidation},
        {"negativo rejeitado", -0.1, 0, apperrors.ErrValidation},
        {"acima de 100 rejeitado", 100.01, 0, apperrors.ErrValidation},
        {"minimo valido", 0.01, 1, nil},
        {"percentual intermediario", 33.33, 3333, nil},
        {"limite superior 100", 100, 10000, nil},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            // Arrange + Act
            got, err := NewFracaoIdeal(tt.input)

            // Assert
            if tt.wantErr != nil {
                require.ErrorIs(t, err, tt.wantErr)
                return
            }
            require.NoError(t, err)
            require.Equal(t, tt.wantBps, got.BasisPoints())
        })
    }
}
```

### Nomenclatura

`Test<Subject>_when_<Context>_should_<Outcome>`:

- `TestCastVoteUseCase_when_voterAlreadyVoted_should_returnErrConflict`
- `TestFracaoIdeal_when_sumExceeds100_should_rejectAppend`
- `TestPublishAnnouncementCommand_when_userNotSindico_should_returnForbidden`

---

## 2. Stubs hand-rolled (sem mockery)

**Sem** `mockery`, `mocktail`, `gomock` — geram complexidade sem ganho. Hand-rolled é ≤ 20 linhas e explícito.

```go
// domain/entities/user.go
type IUserRepo interface {
    FindByID(ctx context.Context, id ULID) (*User, error)
    Save(ctx context.Context, u *User) error
}

// domain/entities/user_test.go
type userRepoStub struct {
    findByID func(ctx context.Context, id ULID) (*User, error)
    save     func(ctx context.Context, u *User) error
}

func (s *userRepoStub) FindByID(ctx context.Context, id ULID) (*User, error) {
    return s.findByID(ctx, id)
}
func (s *userRepoStub) Save(ctx context.Context, u *User) error { return s.save(ctx, u) }

func TestRegisterUser_when_emailTaken_should_returnConflict(t *testing.T) {
    repo := &userRepoStub{
        findByID: func(_ context.Context, id ULID) (*User, error) { return nil, repository.ErrNotFound },
        save:     func(_ context.Context, _ *User) error { return repository.ErrConflict },
    }
    uc := NewRegisterUserUseCase(repo, FakeClock{})

    _, err := uc.Execute(context.Background(), RegisterUserCommand{...})

    require.ErrorIs(t, err, apperrors.ErrConflict)
}
```

Se o stub precisa de lógica (mais de 1 caminho), cria struct concreta com campos públicos de configuração. Nada de spy elaborado.

---

## 3. Arrange-Act-Assert (AAA)

Estrutura inviolável:

```go
func TestX_when_Y_should_Z(t *testing.T) {
    // Arrange — fixture, stubs, inputs
    clock := FakeClock{at: time.Date(2026, 5, 8, 12, 0, 0, 0, time.UTC)}
    repo := &stubs.VoteRepo{}
    uc := NewCastVoteUseCase(repo, clock)
    cmd := CastVoteCommand{...}

    // Act — única chamada ao SUT (Subject Under Test)
    result, err := uc.Execute(context.Background(), cmd)

    // Assert — expectativas
    require.NoError(t, err)
    require.Equal(t, expected, result)
}
```

---

## 4. Clock fake (obrigatório — domain nunca usa `time.Now()` direto)

```go
// core/contracts/clock.go
type Clock interface{ Now() time.Time }

// core/testutil/clock.go
type FakeClock struct{ At time.Time }
func (f FakeClock) Now() time.Time { return f.At }
func (f *FakeClock) Advance(d time.Duration) { f.At = f.At.Add(d) }
```

Use em qualquer teste que envolva `time`: lock 90d, quota reset, session expire, trial expiration.

---

## 5. Cobertura por camada

| Camada | Foco do unit | Threshold |
|---|---|---|
| `domain/entities/` | factory validation, state transitions, domain methods | 95% |
| `domain/value_objects/` | `New()` validation, operators, equality | 95% |
| `domain/services/` | cálculo puro (QuorumCalculator, ReputationCalculator) | 95% |
| `domain/specifications/` | spec composition (And/Or/Not) | 95% |
| `application/use_cases/` | orquestração (com stubs de repo/provider) | 90% |
| `application/event_handlers/` | side-effect handlers | 90% |
| `core/` + `pkg/` | utilities (ULID, Money, Masked, validators) | 95% |
| `infrastructure/http/middleware/` | guard, ABAC branches | 90% (complementa integration) |

---

## 6. Exemplos por camada

### Value Object

```go
func TestNewCPF_when_inputVariations_should_validate(t *testing.T) {
    tests := []struct {
        name    string
        in      string
        wantErr error
    }{
        {"vazio", "", apperrors.ErrValidation},
        {"com mascara", "123.456.789-09", nil}, // módulo-11 válido
        {"só dígitos", "12345678909", nil},
        {"dígitos repetidos", "11111111111", apperrors.ErrValidation},
        {"mod11 inválido", "12345678900", apperrors.ErrValidation},
    }
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            _, err := NewCPF(tt.in)
            if tt.wantErr != nil {
                require.ErrorIs(t, err, tt.wantErr)
                return
            }
            require.NoError(t, err)
        })
    }
}
```

### Aggregate (state transition INV-058 content moderation)

```go
func TestVideo_when_publishBeforeApproved_should_returnConflict(t *testing.T) {
    v, _ := NewVideo(...)
    require.Equal(t, VideoStateModeration, v.State())

    err := v.Publish(time.Now())

    require.ErrorIs(t, err, apperrors.ErrConflict)
}

func TestVideo_when_transitionModerationApprovedPublished_should_setLockedUntil90d(t *testing.T) {
    clock := FakeClock{At: time.Date(2026, 5, 8, 0, 0, 0, 0, time.UTC)}
    v, _ := NewVideo(...)
    require.NoError(t, v.Approve(clock.Now()))
    require.NoError(t, v.Publish(clock.Now()))

    require.True(t, v.LockedUntil().Equal(clock.Now().AddDate(0, 0, 90)))
}
```

### Use Case (com stubs)

```go
func TestCastVoteUseCase_when_presidentSindico_should_returnForbidden(t *testing.T) {
    // INV-081
    assembly := buildAssemblyFactory().WithPresident(user.ID).Build()
    asmRepo := &stubs.AssemblyRepo{GetFn: func(...) { return assembly, nil }}
    uc := NewCastVoteUseCase(asmRepo, nil, FakeClock{})

    _, err := uc.Execute(ctx, CastVoteCommand{VoterID: user.ID, AssemblyID: assembly.ID(), ...})

    require.ErrorIs(t, err, apperrors.ErrForbidden)
}
```

### Middleware

```go
func TestABACMiddleware_when_actionNotConfigured_should_deny(t *testing.T) {
    // INV-087 default-deny
    engine := NewABACEngine(abacConfig{actions: map[string]Action{}}) // vazio
    ctx := newFakeCtx(WithUser(User{Role: RoleSindico}), WithAction("video:upload"))

    decision := engine.Evaluate(ctx)

    require.False(t, decision.Allowed)
    require.Equal(t, "action_not_configured", decision.Reason)
}
```

---

## 7. Benchmarks (hot paths)

Benchmarks em `*_bench_test.go` pra:

- `sessions.QueryByUser`
- `abac.Evaluate`
- `mappers.UserRowToEntity`
- `ReputationCalculator.Compute`

```go
func BenchmarkReputationCalculator_Compute(b *testing.B) {
    calc := NewReputationCalculator()
    company := buildCompanyFactory().WithDeals(100).Build()
    b.ResetTimer()
    for i := 0; i < b.N; i++ {
        _ = calc.Compute(company)
    }
}
```

Rodar: `go test -bench=. -benchmem ./...`.

---

## 8. Anti-patterns em unit tests

| Sintoma | Fix |
|---|---|
| `time.Now()` dentro do SUT | Injeta `Clock`; teste usa `FakeClock` |
| Dependência de ordem (`TestA → TestB`) | Cada teste isolado; `-shuffle=on` passa |
| Uso de DB/HTTP/SDK real | Não é unit — vira integration |
| Mock com `mockery`/mocktail | Stub hand-rolled ≤ 20 linhas |
| Fixtures globais stale | Factory por teste |
| 1 test function com 15 scenarios sem table | Table-driven |
| `assert.Equal` em erro (comparação string) | `require.ErrorIs(t, err, sentinel)` |

---

## 9. Checklist do review

- [ ] Table-driven ou justificado?
- [ ] Nome `TestX_when_Y_should_Z`?
- [ ] AAA explícito?
- [ ] `FakeClock` se envolve tempo?
- [ ] `require.ErrorIs` (não compara string)?
- [ ] Stub hand-rolled, sem mockery?
- [ ] Cada invariante crítico tocado tem unit test?
- [ ] Benchmark pra hot path?
- [ ] `-race -shuffle=on` passa?

---

## 10. Links

- [[_moc]]
- [[test-strategy]]
- [[pbt]]
- [[integration]]
- [[coverage-thresholds]]
- [[../07-quality/CLEAN_CODE]]
- [[../07-quality/CODE_CALISTHENICS]]
- [[../01-domain/invariants]]
