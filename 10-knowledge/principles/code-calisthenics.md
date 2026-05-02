---
title: Code Calisthenics — 9 regras
type: concept
tags: [principles, code-quality, refactoring]
source: .kiro/steering/code-calisthenics.md
created: 2026-04-22
aliases: [Object Calisthenics, 9 regras Jeff Bay]
---

# Code Calisthenics — 9 regras

9 regras de Jeff Bay (*ThoughtWorks Anthology*, 2008). **Exercício de disciplina, não religião** — aplicar 100% das regras 100% do tempo produz código rígido. Aplicar **90% das regras 100% do tempo** produz código excepcional.

Exemplos abaixo em Go. Equivalentes cross-linguagem em `20-stacks/<lang>/code-calisthenics.md` *(a criar conforme necessidade)*.

## 1. Um nível de indentação por método

Método com 3 níveis de if/for aninhados é **difícil** de ler, testar, refatorar. Resolver com **early returns** + helpers pequenos + guard clauses.

```go
// ❌ 3 níveis
func (uc *UseCase) Execute(ctx context.Context, cmd Cmd) error {
    if user != nil {
        if user.IsActive() {
            for _, sub := range user.Subscriptions {
                if sub.IsExpired() {
                    return errExpired
                }
            }
        }
    }
    return nil
}

// ✅ Extrair + early return
func (uc *UseCase) Execute(ctx context.Context, cmd Cmd) error {
    if user == nil {
        return apperrors.ErrNotFound
    }
    if !user.IsActive() {
        return apperrors.ErrBusiness.WithMessage("user inativo")
    }
    if hasExpiredSubscription(user) {
        return errExpired
    }
    return nil
}

func hasExpiredSubscription(u *User) bool {
    for _, sub := range u.Subscriptions() {
        if sub.IsExpired() {
            return true
        }
    }
    return false
}
```

**Checagem**: handler com > 2 níveis de indent = faz demais.

## 2. Não use `else`

`else` esconde intenção e aumenta indentação. Guard clause + caminho feliz no nível 0 é mais claro.

```go
// ❌
if user.HasPermission() {
    return result, nil
} else {
    return nil, apperrors.ErrForbidden
}

// ✅
if !user.HasPermission() {
    return nil, apperrors.ErrForbidden
}
return result, nil
```

**Exceções aceitas**: `switch` com `default` (variação estruturada). `else if` encadeado é cheiro de **Strategy pattern** — extraia.

## 3. Envolva primitivos e strings (Value Objects)

Primitivo com invariante (formato, range, regra de negócio) circulando sem validação = bugs esperando acontecer.

```go
// ❌
type User struct {
    email string  // pode ser "", "invalido", etc.
    cpf   string
    phone string
}

// ✅
type User struct {
    email EmailVO
    cpf   CPFVO
    phone PhoneVO
}

type EmailVO struct{ value string }

func NewEmail(raw string) (EmailVO, error) {
    if !isValidEmail(raw) {
        return EmailVO{}, apperrors.ErrValidation.WithMessage("email inválido")
    }
    return EmailVO{value: strings.ToLower(strings.TrimSpace(raw))}, nil
}

func (e EmailVO) Value() string  { return e.value }
func (e EmailVO) Domain() string { _, d, _ := strings.Cut(e.value, "@"); return d }
```

**Quando aplicar**: todo dado **com regras** vira VO. `UserID string` é OK se é identificador opaco; `Email string` solto é cheiro.

**Exceção pragmática**: DTOs de request/response HTTP usam primitivos — parse pra VO acontece no handler (1 vez, no boundary).

## 4. First-class collections

Slice com invariantes (único por voter, soma fixa, ordenação) → wrap em tipo próprio.

```go
// ❌ Cliente pode fazer assembly.votes = append(..., invalidVote)
type Assembly struct {
    votes []Vote
}

// ✅ Invariantes garantidas
type VoteCollection struct {
    votes []Vote
}

func (vc *VoteCollection) Add(vote Vote) error {
    if vc.hasVoterVoted(vote.VoterID) {
        return apperrors.ErrConflict.WithMessage("voter já votou")
    }
    vc.votes = append(vc.votes, vote)
    return nil
}

func (vc *VoteCollection) Count() int { return len(vc.votes) }
func (vc *VoteCollection) Winner() (OptionID, bool) { /* ... */ }
```

**Quando aplicar**: slice com invariantes → wrap. Slice de DTO sem invariante → primitivo OK.

## 5. Um ponto por linha (Law of Demeter)

Cascata entre entidades diferentes é acoplamento oculto — mudança em uma quebra as outras.

```go
// ❌
city := user.Profile().Address().City().Name()

// ✅ Tell, don't ask
city := user.CityName()

// Ou se Address é agregado separado:
addr, err := addressRepo.FindByUserID(ctx, user.ID())
city := addr.CityName()
```

**Exceções aceitas**:
- **Builders fluentes** (`strings.Builder`) — design pattern explícito
- **Query builders** (`sq.Select(...).From(...).Where(...)`)
- **Método fluente em uma única entidade** com receiver mutável retornando `*T` pra encadear

Cascata entre **entidades diferentes** viola Demeter. Cascata na mesma entidade (API fluente) não.

## 6. Não abrevie

```go
// ❌
uow := NewUnitOfWork(pool)
condo := repo.FindByID(ctx, id)
repo := NewUserRepository(pool)
q := "processing"

// ✅
unitOfWork := NewUnitOfWork(pool)
condominium := repository.FindByID(ctx, id)
userRepository := NewUserRepository(pool)
queue := "processing"
```

**Exceções idiomáticas de cada linguagem**:
- Go: `ctx context.Context`, `err`, `i`/`j` em loops curtos (<5 linhas), `r *http.Request`, `w http.ResponseWriter`, receivers de 1-2 letras (`s *Subscription`)
- TypeScript: `e` em catch/event, `i` em loop curto, `ctx` em contexto explícito
- Rust: `i`/`j` em loops, receivers idiomáticos

**Regra prática**: se contexto permite escolha, escolha **nome completo**. **Nunca** invente abreviação nova (`usr`, `sub`, `cfg` quando `subscription` cabe).

## 7. Mantenha entidades pequenas

- **Funções**: máximo ~50 linhas. Acima: extrair helpers.
- **Structs/entidades**: máximo ~10 campos. Acima: cheiro de **God Object** — dividir agregados.
- **Arquivos**: máximo ~500 linhas. Acima: provavelmente múltiplas coisas no mesmo arquivo.

```go
// ❌ 20 campos, 30 métodos
type Condominium struct {
    id            ID
    name          string
    address       Address
    syndicID      UserID
    units         []Unit
    timeline      []TimelineEntry
    masterPlan    MasterPlan
    subscriptions []Subscription
    assemblies    []Assembly
    validations   []Validation
    // ...
}

// ✅ Agregados separados, cada um com raiz
type Condominium struct {
    id       ID
    name     string
    address  Address
    syndicID UserID
}

type Timeline struct {
    condominiumID ID
    entries       TimelineEntryCollection
}

type MasterPlan struct {
    condominiumID ID
    actions       ActionCollection
}
```

Cada agregado com seu repository. Relação entre agregados via **ID**, não referência direta (evita cascata da regra #5).

## 8. Máximo N campos por classe (adaptado ao idiomático)

Regra original (2 campos) é **muito** dura pra linguagens modernas. Adaptação:

- **Value Objects**: 1–2 campos — OK. Mais: compor via embed ou dividir.
- **Entidades**: 5–10 campos — OK. Mais: múltiplos agregados.
- **Use Cases**: dependências via construtor — tipicamente **3–5** (repository, colaborador, unit of work, logger, event bus). Mais de **7**: cheiro — use case fazendo demais.

Use case com 10 deps está orquestrando o que deveria ser Saga ou use cases compostos.

## 9. Sem getters/setters/propriedades

**Tell, don't ask.** Adaptação:

- **Setters (`SetX`)**: **proibidos** em entidades. Mutação via **métodos de domínio** que expressam intenção (`Activate()`, `Cancel(reason)`, `MarkAsPaid(invoice)`).
- **Getters (`GetX`)**: evitar prefixo `Get` — em Go, ir direto ao campo (`user.Email()`). Em outras linguagens, seguir convenção da comunidade mas sem proliferação.
- Getters **estão OK** em entidades (mappers → DTO precisam), mas **com consciência**. Se chamador usa 5 getters pra decidir algo, a decisão é **método da entidade**.

```go
// ❌
sub.SetStatus(SubscriptionStatusActive)
sub.SetCanceledAt(time.Now())

// ✅ Intenção + invariante
if err := sub.Cancel(cancelReason); err != nil {
    return err
}
// Dentro: sub.status = canceled, sub.canceledAt = now, domain event publicado
```

## Checklist de review

- [ ] Algum método com > 1 nível de indentação?
- [ ] Algum `else` ou `else if` fora de switch?
- [ ] Algum primitivo circulando onde um VO resolveria?
- [ ] Algum slice com invariantes sem wrap em tipo próprio?
- [ ] Alguma cascata `a.b().c().d()` que viola Demeter?
- [ ] Alguma variável abreviada sem convenção idiomática?
- [ ] Alguma entidade com > 10 campos?
- [ ] Algum use case com > 7 dependências?
- [ ] Algum setter em entidade?

Qualquer "sim" = refactor antes de merge.

## Por que essas regras valem

Não é estética. É **manutenibilidade comprovada**:

- **Cada regra reduz uma classe de bug.** Indent profundo esconde lógica; else esconde guards; strings soltas escondem validação; getters demais indicam feature envy.
- **Código que segue calisthenics é fácil de testar** — função pequena, dependências explícitas, invariantes garantidas.
- **É exercício.** Nem todo código final precisa seguir 100% — mas treinar a escrita sob essas restrições produz instinto de design melhor. Com o tempo, você escreve assim naturalmente.

## Referências

- Jeff Bay. *Object Calisthenics* in *ThoughtWorks Anthology* (2008)
- William Durand. [Object Calisthenics (tradução OOP moderna)](https://williamdurand.fr/2013/06/03/object-calisthenics/)
- Sandi Metz. *Practical Object-Oriented Design in Ruby* — as 4 regras ("5 linhas por método" etc.)

## Links

- [[_moc]] — princípios
- [[do-dont-checklist]] — consolidado que aplica estas regras
- [[../antipatterns/_moc]] — catálogo clássico (ex.: AP-002 God Object)
- [[../runtime-antipatterns/_moc]] — catálogo operacional (ex.: OP-013 Switch replicado)
- [[../patterns/strategy]] — refactor para regra #2
- [[../patterns/value-object]] *(a criar)* — refactor para regra #3
