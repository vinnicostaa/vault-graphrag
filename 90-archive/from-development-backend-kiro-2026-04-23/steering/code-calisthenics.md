---
inclusion: always
---

# Code Calisthenics — adaptado ao Go idiomático

9 regras de Jeff Bay (*ThoughtWorks Anthology*, 2008) adaptadas à realidade do Go. **Regras são exercício de disciplina, não religião** — aplicar 100% das regras 100% do tempo produz código rígido. Aplicar **90% das regras 100% do tempo** produz código excepcional.

O full-stack legacy falhou em várias dessas regras (switch de 5 cases em `UpdateProfileUseCase`, `this.authorize()` duplicado 4×, classes com 15+ campos). O Go backend não repete.

## As 9 regras

### 1. Um nível de indentação por método

```go
// ❌ 3 níveis — difícil de ler, testar, refatorar
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

// ✅ Extrair, early return, guard clause
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

**Como aplicar em Go**: early returns, helpers pequenos, guard clauses. Em teste, se o handler tem mais de 2 níveis de indent, ele está fazendo demais.

### 2. Não use `else`

```go
// ❌ else esconde intenção e aumenta indentação
if user.HasPermission() {
    return result, nil
} else {
    return nil, apperrors.ErrForbidden
}

// ✅ Guard clause + caminho feliz no nível 0
if !user.HasPermission() {
    return nil, apperrors.ErrForbidden
}
return result, nil
```

**Exceções aceitas**: `switch` com `default` (é a variação estruturada do else). Blocos `else if` longos são cheiro de **Strategy pattern** — extraia.

### 3. Envolva primitivos e strings (Value Objects)

```go
// ❌ Tipo primitivo circulando sem validação
type User struct {
    email    string  // pode ser "", "invalido", etc
    cpf      string  // pode ser "123", "000.000.000-00", etc
    phone    string
}

// ✅ Value objects auto-validados
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

func (e EmailVO) Value() string { return e.value }
func (e EmailVO) Domain() string { _, d, _ := strings.Cut(e.value, "@"); return d }
```

**Quando aplicar**: todo dado que **tem regras** (formato, range, invariantes) é value object. `UserID string` é aceitável se é apenas identificador opaco; `Email string` solto é cheiro.

**Exceção pragmática**: DTOs de request/response HTTP usam primitives — o parse para value object acontece no handler (uma vez, no boundary).

### 4. First-class collections

```go
// ❌ Slice + método solto — invariante não garantida
type Assembly struct {
    votes []Vote
}

// Cliente pode fazer assembly.votes = append(assembly.votes, invalidVote)
// ou assembly.votes[0] = fraudulento — sem invariante

// ✅ Wrap em tipo próprio
type VoteCollection struct {
    votes []Vote
}

func NewVoteCollection() *VoteCollection { return &VoteCollection{} }

func (vc *VoteCollection) Add(vote Vote) error {
    if vc.hasVoterVoted(vote.VoterID) {
        return apperrors.ErrConflict.WithMessage("voter já votou")
    }
    vc.votes = append(vc.votes, vote)
    return nil
}

func (vc *VoteCollection) Count() int { return len(vc.votes) }

func (vc *VoteCollection) Winner() (OptionID, bool) { /* ... */ }

func (vc *VoteCollection) hasVoterVoted(id VoterID) bool {
    for _, v := range vc.votes {
        if v.VoterID == id {
            return true
        }
    }
    return false
}
```

**Quando aplicar**: slice/map que tem invariantes (único por voter, soma fixa, ordenação específica) → wrap. Slice de DTOs sem invariantes → primitivo ok.

### 5. Um ponto por linha (Law of Demeter)

```go
// ❌ Cascata viola Demeter — mudança em Address quebra User
city := user.Profile().Address().City().Name()

// ✅ Tell, don't ask — pergunte ao objeto, não navegue
city := user.CityName()

// Ou, se Address é entidade separada (aggregate root diferente):
addr, err := addressRepo.FindByUserID(ctx, user.ID())
city := addr.CityName()
```

**Exceção aceita**: builders fluentes (`strings.Builder`), queries SQL fluentes (`sq.Select(...).From(...).Where(...)`) — são design pattern explícito, não vazamento de encapsulamento.

**Em Go**: método fluente com receiver pointer que muta estado e retorna `*T` para encadear é aceitável. Cascata **sobre entidades diferentes** (Demeter) não é.

### 6. Não abrevie

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

**Exceções idiomáticas de Go**:
- `ctx context.Context` — convenção forte da linguagem
- `err` — idiomático
- `i`, `j` em loops curtos (< 5 linhas)
- `r *http.Request`, `w http.ResponseWriter` — convenção da stdlib
- Receivers de método: uma ou duas letras (`s *Subscription`) — idiomático Go

**Regra prática**: se o contexto permite escolha, escolha o nome completo. **Nunca** invente abreviação nova ("usr", "sub", "cfg" quando "subscription" cabe).

### 7. Mantenha entidades pequenas

- **Funções**: máximo ~50 linhas. Acima: extrair helpers.
- **Structs/entidades**: máximo ~10 campos. Acima: cheiro de **God Object** — dividir em agregados.
- **Arquivos**: máximo ~500 linhas. Acima: provavelmente é múltiplas coisas no mesmo arquivo.

Exemplo — quando dividir:

```go
// ❌ Entidade gigante — 20 campos, 30 métodos
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

// ✅ Agregados separados — cada um com sua raiz
type Condominium struct {          // aggregate root: dados identitários
    id       ID
    name     string
    address  Address
    syndicID UserID
}

type Timeline struct {             // aggregate root separado
    condominiumID ID
    entries       TimelineEntryCollection
}

type MasterPlan struct {           // aggregate root separado
    condominiumID ID
    actions       ActionCollection
}
// ...
```

Cada agregado tem seu repository. Relação entre agregados via `ID`, não via referência direta (evita cascata do #5).

### 8. Máximo 2 campos por classe (adaptado — Go idiomático)

Regra original é **muito** dura para Go. Adaptação:

- **Value Objects**: 1-2 campos — ok. Mais de isso: compor via embed ou dividir.
- **Entidades**: 5-10 campos — ok. Mais: revisar se são múltiplos agregados.
- **Use Cases**: dependências via construtor — tipicamente 3-5 (repository, use case colaborador, unit of work, logger, event bus). Mais de 7: cheiro — use case fazendo demais.

Se um use case tem 10 dependências, ele provavelmente está orquestrando o que deveria ser Saga ou múltiplos use cases compostos.

### 9. Sem getters/setters/propriedades (adaptado)

Regra original: "tell, don't ask". Em Go:

- **Setters (`SetX`)**: proibidos em entidades. Mutação via **métodos de domínio** que expressam intenção (`Activate()`, `Cancel(reason)`, `MarkAsPaid(invoice)`) — não `SetStatus("active")`.
- **Getters (`GetX`)**: evite o prefixo `Get` — vá direto ao campo: `user.Email()`, `user.Status()`. Idiomático Go.
- Getters **estão ok** em entidades (precisa expor estado para mappers → DTO), mas use com consciência — se o chamador está usando 5 getters para decidir algo, a decisão provavelmente é método da entidade.

```go
// ❌ Setter expõe estado mutável
sub.SetStatus(SubscriptionStatusActive)
sub.SetCanceledAt(time.Now())

// ✅ Método de domínio expressa intenção + garante invariantes
if err := sub.Cancel(cancelReason); err != nil {
    return err
}
// Dentro: sub.status = canceled, sub.canceledAt = now, domain event publicado
```

## Testando se as regras estão sendo aplicadas

No `/go-review` skill + code review manual:

- [ ] Algum método com > 1 nível de indentação?
- [ ] Algum `else` ou `else if` fora de switch?
- [ ] Algum `string`/`int` circulando onde um VO resolveria?
- [ ] Algum slice com invariantes sem wrap em tipo próprio?
- [ ] Algum `a.b().c().d()` que viola Demeter?
- [ ] Alguma variável abreviada sem convenção idiomática?
- [ ] Alguma entidade com > 10 campos?
- [ ] Algum use case com > 7 dependências?
- [ ] Algum setter em entidade?

Qualquer "sim" = refactor antes de merge.

## Por que essas regras valem

Não é estética. É **manutenibilidade comprovada**:

- **Cada regra reduz uma classe de bug**. Indent profundo esconde lógica; else esconde guards; strings soltas escondem validação; getters demais indicam feature envy.
- **Código que segue calisthenics é fácil de testar** — função pequena, dependências explícitas, invariantes garantidas.
- **É exercício**. Nem todo código final precisa seguir 100% — mas treinar a escrita sob essas restrições produz instinto de design melhor. Com o tempo, você escreve assim naturalmente.

## Referências

- Jeff Bay. *Object Calisthenics* in *ThoughtWorks Anthology* (2008)
- William Durand. [Object Calisthenics (tradução para PHP/OOP moderna)](https://williamdurand.fr/2013/06/03/object-calisthenics/)
- Sandi Metz. *Practical Object-Oriented Design in Ruby* — as 4 regras ("5 linhas por método", etc)
