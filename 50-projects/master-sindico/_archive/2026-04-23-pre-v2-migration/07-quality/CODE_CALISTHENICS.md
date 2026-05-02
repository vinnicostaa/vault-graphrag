---
title: CODE_CALISTHENICS — Aplicação ao Master Síndico
type: project
tags: [master-sindico, code-calisthenics, quality]
project: master-sindico
created: 2026-04-22
---

# CODE_CALISTHENICS — Master Síndico

9 regras de Jeff Bay, adaptadas pra Go, TypeScript e Dart neste projeto. Exceções **documentadas**; violação crônica vira ticket.

Universal em [[../../../10-knowledge/principles/code-calisthenics]].

---

## As 9 regras

### 1. Um nível de indent por método

**Meta**: método lê como prosa linear.

- Go: ≤ 3 níveis (incluindo `for`/`if` externos). Guard clauses eliminam aninhamento.
- TS: mesmo.
- Dart: mesmo.

### 2. Sem bloco `else`

**Inviolável neste projeto.** Ver [[CLEAN_CODE#sem-else-regra-do-projeto]].

Só early return / guard / strategy / table-driven.

### 3. Envolver primitivos e strings em Value Objects

Já aplicado em `pkg/money` (int64 centavos) e value_objects de cada módulo:

- `Email`, `CPF`, `CNPJ`, `Phone`, `Password` — em `identity/domain/value_objects/`
- `Money` — em `pkg/money/` (lib compartilhada)
- `FracaoIdeal` — em `institutional/domain/value_objects/` (percentual 0-100 com soma = 100)
- `ReputacaoTier` — em `commercial/domain/value_objects/` (Bronze|Prata|Ouro|Platina|Diamante)

**Regra**: se o `string` ou `int` tem regra de negócio, vira VO.

### 4. Coleções de primeira classe

**Meta**: listas com regras de negócio viram tipo próprio, não `[]T` solto.

Exemplos aplicáveis:
- `AgendaItems` (com ordem, estado, SetFractionIdeal invariante)
- `Votes` (com contagem, percentuais)
- `TimelineEntries` (imutável; append-only)
- `Memberships` (com regras de ativação/desativação)

Impl típica:

```go
type AgendaItems struct {
    items []AgendaItem
}
func (a *AgendaItems) Append(item AgendaItem) error { ... }
func (a AgendaItems) SumFraction() Percentage { ... }
func (a AgendaItems) MustSumTo100() error { ... }
```

### 5. Um ponto por linha

**Meta**: evitar `a.b.c.d()` (Law of Demeter).

Permitido quando cadeia fluente intencional (builder, query): `db.Where().Order().Limit()`.

Proibido quando vaza abstração: `user.Condominium().Subscription().Plan().Features()`.

### 6. Não abreviar

Aplicado. Ver [[CLEAN_CODE#naming]].

### 7. Manter entidades pequenas

- Entidade ≤ 200 linhas (Go)
- Classe ≤ 150 linhas (TS/Dart)
- Método ≤ 40 linhas

Legacy TS tinha `Subscription` 450 linhas, `Invoice` 550 — **proibido** neste projeto. Ver AP-014 e [[../legacy-reference#o-que-não-copiar-antipatterns-identificados]].

### 8. Classes com no máximo 2 variáveis de instância

**Adaptação necessária pra Go/DDD**: agregados legitimamente têm mais (condomínio tem id, nome, endereço, plano, unidades, memberships, timeline…). O espírito é: **se precisa de muitos campos, pode indicar que há agregado menor escondido** (ex: `Address` como VO, `Subscription` como agregado separado).

Regra pragmática: campos `> 7` → revisar; provavelmente há VO/agregado oculto.

### 9. Sem getters/setters (sem lógica)

**Meta**: entidades expõem comportamento, não dados.

```go
// RUIM
func (u *User) GetEmail() string { return u.email }
func (u *User) SetEmail(e string) { u.email = e }

// BOM
func (u User) Email() Email { return u.email }  // exposição read-only via VO
func (u *User) ChangeEmail(newEmail Email) error {
    if u.isLocked { return ErrUserLocked }
    u.email = newEmail
    return nil
}
```

Entidade = **o que você pode fazer**, não o que você tem.

---

## Exceções documentadas

Exceções ao Calisthenics precisam de comment + justificativa. Ex:

```go
// Calisthenics exception: 8 campos.
// Condominium é agregado-raiz; Address, Plan, Timeline já são agregados separados.
// Reduzir mais fragmentaria sem ganho real.
type Condominium struct {
    id            CondominiumID
    name          string
    cnpj          CNPJ
    address       Address
    subscriptionID billing.SubscriptionID
    unitsCount    int
    createdAt     time.Time
    updatedAt     time.Time
}
```

---

## Detectores (tooling)

Go:
- `revive` / `golangci-lint` — alguns checks disponíveis
- Custom grep rules em pre-commit pra `else` / `_ = `
- **Manual** em code review: tamanho de entidade, VO, coleções

TS:
- `eslint-plugin-sonarjs` tem várias regras (`cognitive-complexity`, `max-lines-per-function`)
- `eslint --max-lines-per-function` configurado

Dart:
- `dart analyze` + `custom_lint`

---

## Aplicação por módulo

| Módulo | Estado |
|---|---|
| identity | ✅ Value objects robustos (Email, CPF, Password); entidades < 200 linhas |
| billing | ✅ Subscription enxuto; use cases separados por comando |
| institutional | ✅ Timeline entries imutáveis; AgendaItems coleção; FracaoIdeal VO |
| commercial | ✅ ReputacaoTier VO; coleções de Proposta, Contrato |
| content | ✅ Video enxuto; Mux provider isolado |
| assembly | ✅ Agenda/Votes coleções; entity Assembly < 200 linhas |

---

## Checklist do review

- [ ] Indent ≤ 3?
- [ ] Sem `else`?
- [ ] Primitivo com regra → VO?
- [ ] Coleção com regra → tipo próprio?
- [ ] Law of Demeter respeitada?
- [ ] Sem abreviação?
- [ ] Entidade < 200 linhas?
- [ ] Sem getter/setter vazio?

---

## Links

- [[_moc]]
- [[CLEAN_CODE]]
- [[CLEAN_ARCH]]
- [[PATTERNS]]
- [[../../../10-knowledge/principles/code-calisthenics]]
