---
title: CLEAN_CODE — Aplicação ao Master Síndico
type: project
tags: [master-sindico, clean-code, readability]
project: master-sindico
created: 2026-04-22
---

# CLEAN_CODE — Master Síndico

Princípios de Clean Code aplicados por stack. Base universal em [[../../../10-knowledge/principles/do-dont-checklist]].

---

## Naming

- **Nomes revelam intenção.** `GetUserByID` > `Fetch`; `QuorumCalculator` > `Calc`.
- **Ubiquitous language** é lei. Prefira `Assembleia`, `Sindico`, `Convencao`, `FracaoIdeal` (mesmo em código Go — preservam semântica jurídica).
- **Sem abreviações**. `ctx` é ok (stdlib); `idx` também. **Não** `asm` pra assembleia, `sbscr` pra subscription.
- **Booleans usam prefixos**: `isActive`, `hasPermission`, `canPublish`, `shouldRefresh`.
- **Funções são verbos**: `CreateUnit`, `ApproveCompany`, `PublishVideo`, `ClosAssembly`.
- **Tipos são substantivos**: `Condominium`, `VoteSession`, `ReputationScore`.
- **Erros** têm sentinela tipada: `apperrors.ErrValidation.WithMessage("starts_at obrigatório")`.

**Proibidos**:
- `data`, `info`, `obj`, `tmp`, `temp`, `val`, `x`, `foo` (exceto em testes table-driven internos)
- `process()`, `handle()`, `do()` — sempre substantivo
- Sufixos redundantes: `UserUser`, `UnitEntity`

---

## Funções

- **Uma responsabilidade**. Se o nome precisa "e" (`createAndSendEmail`) → split.
- **Tamanho alvo ≤ 40 linhas** em Go; se maior, extrair helper.
- **Argumentos ≤ 3**. Mais que isso → struct de entrada (`CreateUnitCommand`).
- **Sem boolean arg** que vira switch no corpo → extrair 2 funções.
- **Retorno sempre explícito** — sem named returns exceto em deferreds complexos.
- **Erro é último retorno**: `(T, error)`.

---

## Comentários

- **O porquê, não o quê.** Nome e tipo já dizem o quê.
- **Sem comments-as-doc** que duplicam assinatura: `// GetUserByID retorna user by id` é **ruim**.
- **Ok**: `// A-022 fix: rota pública pra Mux não pode estar em /api/v1 (Authenticate bloqueia)`.
- **TODO / FIXME** só com owner + data (`// TODO(@dev, 2026-05): promover pra DT-###`).
- **Sem código comentado** — git é a memória.

---

## Error handling (Go)

```go
// BOM — propagar com contexto
user, err := userRepo.FindByID(ctx, id)
if err != nil {
    return nil, fmt.Errorf("find user %s: %w", id, err)
}

// BOM — sentinela tipada quando faz parte do contrato
if errors.Is(err, repository.ErrNotFound) {
    return apperrors.ErrNotFound.WithMessage("usuário não encontrado")
}

// BOM — compensação explícita em saga
if err := repo.Create(ctx, video); err != nil {
    if cErr := provider.Cancel(ctx, uploadID); cErr != nil {
        logger.Error().Err(cErr).Str("original_err", err.Error()).Msg("saga compensation failed")
    }
    return err
}

// RUIM — silenciar erro
_ = repo.Save(ctx, entity)

// RUIM — swallow + log vazio
if err != nil {
    logger.Error(err)
    return nil // ⚠️ dado está divergente — nunca faça
}
```

**Regra estratégica para try/catch (panic/recover em Go)**:
- **Padrão**: deixar erro propagar. Se caller não sabe lidar, **não** capture.
- **Capturar** só quando:
  - Há **ação útil** (compensação, log com contexto rico, transformação de erro)
  - É **camada de fronteira** (handler → HTTP status, middleware global)
  - É **goroutine** que **deve** sobreviver (e você loga o panic)

---

## Sem if/else hell (Go)

### RUIM

```go
if user.IsActive {
    if user.HasRole("sindico") {
        if condo.IsSubscribed() {
            if unit.FracaoIdeal() > 0 {
                // lógica...
            } else {
                return errors.New("fração inválida")
            }
        } else {
            return errors.New("sem subscription")
        }
    } else {
        return errors.New("não é síndico")
    }
} else {
    return errors.New("user inativo")
}
```

### BOM (guard clauses + early return)

```go
if !user.IsActive {
    return apperrors.ErrForbidden.WithMessage("usuário inativo")
}
if !user.HasRole("sindico") {
    return apperrors.ErrForbidden.WithMessage("não é síndico neste condomínio")
}
if !condo.IsSubscribed() {
    return apperrors.ErrPaymentRequired.WithMessage("condomínio sem plano ativo")
}
if unit.FracaoIdeal() <= 0 {
    return apperrors.ErrValidation.WithMessage("fração ideal inválida")
}
// lógica...
```

### Ainda melhor (extrair validações)

```go
func (uc *X) validatePreconditions(user User, condo Condominium, unit Unit) error {
    for _, check := range []func() error{
        func() error { return ensureActive(user) },
        func() error { return ensureRoleSindico(user, condo.ID) },
        func() error { return ensureSubscription(condo) },
        func() error { return ensureFracaoIdeal(unit) },
    } {
        if err := check(); err != nil {
            return err
        }
    }
    return nil
}
```

---

## Sem `else` (regra do projeto)

**NUNCA**:

```go
if x {
    // a
} else {
    // b
}
```

**SEMPRE**:

```go
if x {
    // a
    return
}
// b
```

Exceção: testes table-driven internos onde o `else` é óbvio e não agrega complexidade — ainda assim, refatorar.

---

## Legibilidade

- **Declarações próximas do uso**: não declarar var 40 linhas antes.
- **Função sobe em nível de abstração**: primeiro o "o que", depois o "como".
- **Consistência > criatividade**: se o módulo usa `ctx context.Context` primeiro, **todos** usam.
- **Sem magic numbers**: `const TokenTTL = 5 * time.Minute`.
- **Nomes sem negações** quando possível: `isActive` > `isNotDisabled`.

---

## Go-specific

- `any` é proibido onde concreto serve (só em generics quando precisa)
- `interface{}` legado → migrar pra `any` no `gofmt` + evitar
- Structs expostos: campos com comment só se WHY não é óbvio
- **Pointers em structs** com cuidado: se valor é barato (< 24 bytes), passa por valor
- **Receiver** sempre consistente: método com pointer se algum método é pointer
- **Tests** em `*_test.go` ao lado; table-driven com `tests := []struct {...}` pattern
- **Use `//go:build integration`** (não `// +build`)

---

## TS-specific (web)

- `strict: true` sempre; `noImplicitAny`
- **Narrowing** com type guards, não type casts
- **Discriminated unions** pra estados: `type State = { kind: "loading" } | { kind: "error"; err: string } | { kind: "data"; data: T }`
- **Zod/valibot** pra validação de input externo (API response)
- **Nenhum `any`** — se precisa, comment explicando + `// eslint-disable-next-line @typescript-eslint/no-explicit-any`

---

## Dart-specific (mobile)

- **Sound null-safety** sempre (`?` + `!` disciplinado)
- **Freezed** para models imutáveis + union types
- **const constructors** sempre que possível (menos rebuilds)
- **final** para variáveis locais que não mudam
- **avoid `dynamic`** — use genéricos

---

## Checklist do review

- [ ] Nomes revelam intenção (não precisei ler o corpo pra entender)?
- [ ] Funções < 40 linhas ou com justificativa?
- [ ] Sem `else`, sem aninhamento ≥ 3 níveis?
- [ ] Sem `_ = fn()` em I/O?
- [ ] Comments só onde WHY não é óbvio?
- [ ] Sem código comentado?
- [ ] Error wrapping preserva contexto?
- [ ] Tipos explícitos no retorno?
- [ ] Teste cobre happy + error paths?
- [ ] Nome da função é verbo + substantivo?

---

## Links

- [[_moc]]
- [[CLEAN_ARCH]]
- [[CODE_CALISTHENICS]]
- [[PATTERNS]]
- [[../../../10-knowledge/principles/code-calisthenics]]
- [[../../../10-knowledge/principles/do-dont-checklist]]
- [[../../../10-knowledge/antipatterns/_moc]]
