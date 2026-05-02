---
title: Code Calisthenics aplicado — Master Síndico (Go)
type: note
tags:
  - principle-applied
  - code-calisthenics
  - master-sindico
  - quality
project: master-sindico
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - code-calisthenics-master-sindico
  - code-calisthenics-go-ms
---

# Code Calisthenics aplicado — Master Síndico v2 (Go)

As 9 regras de Jeff Bay adaptadas para Go 1.26 do Master Síndico, com exceções pragmáticas para o stack (DTOs, sqlc generated, ProtoBuf-style). Especialização do canônico [[../../../10-knowledge/principles/code-calisthenics]] com exemplos do código real.

> Cada regra **CAL-###** previne pelo menos um antipattern do catálogo [[antipatterns]] (AP-001..AP-026). Não duplica o knowledge global — linka.

---

## Sumário

Code Calisthenics são 9 regras "exageradas" que disciplinam o código a ser pequeno, expressivo e composto. Em Go elas combinam bem com idiomas naturais (early return, structural typing, composição). No MS o foco crítico é **CAL-002 (sem else)** e **CAL-003 (wrap primitives)** — ambos previnem AP-006 (primitive obsession), AP-002 (god object) e AP-011 (if/else hell). O código real do backend já tem `0 } else {` (validado em `grep -rn '} else {' internal/`) e VOs para CPF/CNPJ/Money/FracaoIdeal.

---

## Regras concretas

### CAL-001 — Um nível de indentação por método

**Regra**: dentro de um método, não aninhar `if`/`for`/`switch` em mais de 1 nível. Extrair método privado quando passar disso.

**Por quê**: previne **AP-001** (god method) e **AP-011** (if/else hell). Cada nível extra é um "branch" que o leitor precisa segurar na cabeça.

**✅ Exemplo Go (correto)**:

```go
// internal/modules/billing/application/use_cases/checkout_subscription_saga.go
func (uc *CheckoutSubscriptionSaga) Execute(ctx context.Context, cmd CheckoutCommand) (*SubscriptionDTO, error) {
    if err := cmd.Validate(); err != nil {
        return nil, err
    }
    plan, err := uc.planRepo.FindByID(ctx, cmd.PlanID)
    if err != nil {
        return nil, fmt.Errorf("find plan: %w", err)
    }
    customer, err := uc.gateway.CreateCustomer(ctx, mappers.ToCreateCustomerReq(cmd))
    if err != nil {
        return nil, fmt.Errorf("create customer: %w", err)
    }
    sub, err := uc.gateway.CreateSubscription(ctx, mappers.ToCreateSubReq(plan, customer, cmd))
    if err != nil {
        return nil, fmt.Errorf("create subscription: %w", err)
    }
    return uc.persist(ctx, plan, customer, sub, cmd)  // método privado
}

func (uc *CheckoutSubscriptionSaga) persist(...) (*SubscriptionDTO, error) {
    var dto *SubscriptionDTO
    err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        local, err := entities.NewSubscription(...)
        if err != nil { return err }
        local.LinkStripe(customer, sub, plan.PriceID())
        if err := uc.subscriptionRepo.Save(ctx, local); err != nil {
            return uc.compensate(ctx, sub.ID, err)  // método privado
        }
        dto = mappers.SubscriptionToDTO(local)
        return nil
    })
    return dto, err
}
```

**❌ Anti-exemplo**:

```go
func (uc *MegaUC) Execute(ctx context.Context, cmd Cmd) error {
    for _, item := range cmd.Items {
        if item.Active {
            if item.Quantity > 0 {
                for _, rule := range item.Rules {
                    if rule.Apply {
                        // 4 níveis de indentação — leitor desistiu
                    }
                }
            }
        }
    }
}
```

**Detecção**:
- `revive` rule `nested-structs` ou `cognitive-complexity` ≤ 10.
- gofmt + code review: function com 4+ níveis de indentação = blocker.

---

### CAL-002 — Não use `else` (early return)

**Regra**: zero blocos `else` no código de produção. Refatore com early return / guard clauses / dispatch via map.

**Por quê**: validado no código MS (legado tinha vários `else`; F4 da [[../11-audit/AUDIT]] eliminou todos). Early return é mais legível, ajuda detecção de fluxo invertido.

**✅ Exemplo Go (early return)**:

```go
// identity/application/use_cases/sync_user_use_case.go (real)
func (uc *SyncUserUseCase) Execute(ctx context.Context, cmd SyncUserCommand) (*UserDTO, error) {
    if err := cmd.Validate(); err != nil {
        return nil, err
    }
    user, err := uc.repo.FindByZitadelID(ctx, cmd.ZitadelID)
    if err != nil && !errors.Is(err, apperrors.ErrNotFound) {
        return nil, err
    }
    if user == nil {
        user, err = entities.NewUser(cmd.ZitadelID, cmd.Email, cmd.Role)
        if err != nil { return nil, err }
    }
    user.UpdateLastSeen(uc.clock.Now())
    if err := uc.repo.Save(ctx, user); err != nil {
        return nil, fmt.Errorf("save user: %w", err)
    }
    return mappers.UserToDTO(user), nil
}
```

**✅ Strategy via dispatcher (em vez de if/elseif)**:

```go
type membershipStrategy func(ctx context.Context, cmd Cmd) error

var strategies = map[string]membershipStrategy{
    "titular":   registerTitularStrategy,
    "dependent": registerDependentStrategy,
}

func (uc *RegisterMembershipUseCase) Execute(ctx context.Context, cmd Cmd) error {
    fn, ok := strategies[cmd.RoleType]
    if !ok {
        return apperrors.ErrValidation.WithMessage("role_type inválido")
    }
    return fn(ctx, cmd)
}
```

**✅ Default + override (em vez de if/else)**:

```go
status := entities.SubscriptionStatusActive
if trialEndsAt != nil && trialEndsAt.After(now) {
    status = entities.SubscriptionStatusTrialing
}
```

**❌ Anti-exemplo**:

```go
if user != nil {
    user.UpdateLastSeen(now)
} else {
    user = entities.NewUser(...)
}
// ou pior: if/else if/else if encadeado (AP-011)
```

**Quando aceitável**: `switch ... default` (não é `else` semântico). `else` em arquivo gerado (sqlc) — não editar.

**Detecção**:

```bash
# CI gate (replica check do legado)
if grep -rn '} else {' internal/ --include='*.go' --exclude='*_test.go'; then
    echo "FAIL: else block encontrado"
    exit 1
fi
```

---

### CAL-003 — Wrap primitives em domain types (VO)

**Regra**: parâmetros e campos que carregam **conceito de domínio** (CPF, CNPJ, email, dinheiro, fração ideal, ULID, código de cupom, fingerprint de device) **não** podem ser `string`/`int`/`float64` cru. Encapsule em VO imutável.

**Por quê**: previne **AP-006** (primitive obsession) + **AP-007** (PII em log — VO tem `Masked()`). Métodos no VO documentam invariantes (ex: `CPF.IsValid()`). Type system pega bug em compile time (não posso passar `email` onde espera `cpf`).

**✅ Exemplo Go (correto)**:

```go
// internal/modules/identity/domain/value_objects/cpf.go
type CPF struct{ digits string }

var (
    ErrInvalidCPFLength     = errors.New("CPF deve ter 11 dígitos")
    ErrInvalidCPFCheckDigit = errors.New("dígito verificador do CPF inválido")
)

func NewCPF(raw string) (CPF, error) {
    cleaned := stripNonDigits(raw)
    if len(cleaned) != 11 {
        return CPF{}, ErrInvalidCPFLength
    }
    if !validateCPFCheckDigits(cleaned) {
        return CPF{}, ErrInvalidCPFCheckDigit
    }
    return CPF{digits: cleaned}, nil
}

func (c CPF) Digits() string { return c.digits }
func (c CPF) Masked() string { return "***.***.***-" + c.digits[len(c.digits)-2:] }

// pkg/money/money.go
type Money struct{ cents int64 }

func NewMoneyBRL(cents int64) Money            { return Money{cents} }
func (m Money) Cents() int64                   { return m.cents }
func (m Money) Add(o Money) Money              { return Money{m.cents + o.cents} }
func (m Money) String() string                 { return fmt.Sprintf("R$ %.2f", float64(m.cents)/100) }

// shared/types/ids.go (ULID typed)
type CondominiumID struct{ ulid.ULID }

func NewCondominiumID() CondominiumID                  { return CondominiumID{ulid.Make()} }
func ParseCondominiumID(s string) (CondominiumID, error) { ... }
```

**❌ Anti-exemplo (AP-006)**:

```go
// uso em handler — compile passa, mas troca de args silenciosa
func (uc *RegisterUC) Execute(ctx context.Context, cpf, email, name string) error
// chamada errada: Execute(ctx, email, cpf, name) — bug compila e roda
```

**Detecção**:
- code review: parâmetros `string` em funções de domínio que representam CPF/CNPJ/Email/UUID = blocker.
- grep linhas tipo `cpf string` em `domain/` (heurística).

---

### CAL-004 — First-class collections (coleção tipada com métodos)

**Regra**: quando uma coleção carrega **invariantes** ou **comportamento próprio** (todas as votações de uma assembleia, todos os memberships de um usuário, todas as rules de uma action ABAC), encapsule em type nomeado com métodos.

**Por quê**: preserva invariantes de coleção (Σ FracaoIdeal ≤ 100%, voto único por user) no tipo, não no código que itera.

**✅ Exemplo Go**:

```go
// internal/modules/assembly/domain/entities/votes.go
type Votes []Vote

func (vs Votes) AlreadyVoted(voterID ULID, itemID ULID) bool {
    for _, v := range vs {
        if v.VoterID() == voterID && v.AgendaItemID() == itemID {
            return true
        }
    }
    return false
}

func (vs Votes) WeightFor(itemID ULID, opt enums.VoteOption) FracaoIdeal {
    total := FracaoIdealZero()
    for _, v := range vs {
        if v.AgendaItemID() == itemID && v.Option() == opt {
            total = total.Add(v.Weight())
        }
    }
    return total
}

// uso em aggregate
func (a *Assembly) CastVote(voter VoterInfo, itemID ULID, opt enums.VoteOption) error {
    if a.votes.AlreadyVoted(voter.UserID, itemID) {
        return ErrDuplicateVote  // INV-071
    }
    // ...
    a.votes = append(a.votes, NewVote(voter, itemID, opt))
    return nil
}
```

**✅ Outro exemplo — Memberships com invariante de fração ideal**:

```go
type Memberships []Membership

func (m Memberships) TotalFracaoIdeal() FracaoIdeal { /* Σ */ }
func (m Memberships) FracaoIdealOK() bool           { return m.TotalFracaoIdeal().Cmp(FracaoIdealHundred()) <= 0 }
```

**❌ Anti-exemplo**:

```go
// código espalhado sem coleção tipada
votes := []Vote{...}
for _, v := range votes {
    if v.VoterID == ... && v.AgendaItemID == ... { /* lógica replicada */ }
}
// invariante "voto único" duplicado em N call sites
```

**Detecção**: code review; quando `[]X` aparece com lógica replicada em 2+ lugares = sinal de criar `type Xs []X` com métodos.

---

### CAL-005 — One dot per line (Lei de Demeter)

**Regra**: evite chains profundas (`a.B().C().D().E()`). Receba o objeto que precisa, não o objeto que tem o objeto.

**Por quê**: cada `.` é acoplamento. Mudar a chain quebra o caller. Exceção: builder pattern, fluent assertions em tests, sqlc generated chains.

**✅ Exemplo Go**:

```go
// errado: caller conhece detalhe interno
func ChargeUserCard(user *entities.User, amount Money) error {
    return processPayment(user.Account().Card().Token(), amount) // 3 dots
}

// correto: passe só o que precisa OR método "tell don't ask"
func ChargeUserCard(token CardToken, amount Money) error {
    return processPayment(token, amount)
}

// chamada
ChargeUserCard(user.PrimaryCardToken(), amount)  // user expõe método semântico
```

**Quando aceitável**:

```go
// Test fluent assertions — OK
require.Equal(t, expected, got)
assert.Equal(t, "ok", got.Result.Status)

// sqlc generated query chain — OK (não editamos)
row, err := q.GetCondominium(ctx, id)

// Builder fluent — OK
opts := stripe.CreateSubReq{}.WithCustomer(c).WithPlan(p).WithTrial(15)
```

**❌ Anti-exemplo**:

```go
fee := order.Customer().Subscription().CurrentPlan().Tier().FeeStructure().BaseFee()
// 6 dots — qualquer mudança em qualquer step quebra
```

**Detecção**: code review; linter custom (planejado).

---

### CAL-006 — Não abreviar (mas com bom senso Go)

**Regra**: nomes descritivos. Escreva `condominium`, não `cond`. Escreva `subscription`, não `sub`. Escreva `userRepository`, não `usrSvc`. **Exceção idiomática Go**: `ctx`, `err`, `i`/`j` em loops, receivers 1-2 letras, `r/w` em readers/writers (stdlib pattern).

**Por quê**: previne **AP-014** (logging confuso) e geral incompreensão. Dia 1 numa code review ninguém sabe o que é `usrSvc` (é UserService? UserSyncer?).

**✅ Exemplo Go**:

```go
// receivers idiomáticos (1-2 letras)
func (a *Assembly) Start() error
func (uc *CastVoteUseCase) Execute(ctx context.Context, cmd CastVoteCommand) error
func (r *VoteRepository) Save(ctx context.Context, v *entities.Vote) error

// vars locais — descritivas em scope longo
activeMembership, err := uc.memberRepo.GetActive(ctx, tenantID, voterID)
fracaoIdeal := activeMembership.FracaoIdeal()

// vars locais — curtas em scope trivial
for i, v := range votes { ... }
```

**❌ Anti-exemplo**:

```go
func (s *SubsSrv) DoIt(c context.Context, x XReq) (*YDTO, error) {
    cnt := 0
    for _, m := range x.Mbrs {
        if m.IsAct {
            cnt++
            tmp := proc(m)
            // ...
        }
    }
}
```

**Detecção**: code review; `revive` rule `var-naming`; convenção PR template.

---

### CAL-007 — Manter entidades pequenas

**Regra (Go-flavor)**:
- VO: 1-3 campos, ~30 linhas (`CPF`, `Money`, `FracaoIdeal`).
- Aggregate: ≤ 200 linhas, ≤ 12 campos. Métodos públicos ≤ 10.
- Use case (struct + Execute): ≤ 150 linhas total.
- Pacote: ≤ 10 arquivos `.go` (`identity/domain/entities/` deve ter ≤ 10 entities).
- Função: ver CC-001/CC-002 (≤ 30 / ≤ 50 / ≤ 20 conforme camada).

**Por quê**: previne **AP-001** (Subscription 450L), **AP-002** (Invoice 550L). Pacote pequeno = grep rápido, code review focado.

**✅ Exemplo Go** (`Subscription` com ~210 linhas, ~10 campos, 12 métodos): aceitável com comentário no topo + revisão.

**Exemplo não-aceitável**: `Subscription` 450L misturando state + Stripe + PDF + email = decompor.

**Quando partir aggregate**:
1. Métodos agrupam-se em 2+ "temas" claros (state machine vs cálculo de quota).
2. 3+ deps do construtor de tipos heterogêneos (Stripe + PDF + Email).
3. Mudança em feature A toca 5 métodos não relacionados.

**Detecção**:

```bash
# Aggregates > 200 linhas (sem comentários/blanks)
for f in internal/modules/*/domain/entities/*.go; do
    lines=$(grep -cE '^\s*\S' "$f")
    if [ "$lines" -gt 200 ]; then
        echo "WARN: $f tem $lines linhas (limite 200)"
    fi
done

# Pacotes com > 10 arquivos
for d in internal/modules/*/domain/entities; do
    n=$(find "$d" -maxdepth 1 -name '*.go' -not -name '*_test.go' | wc -l)
    [ "$n" -gt 10 ] && echo "WARN: $d tem $n entities"
done
```

---

### CAL-008 — Pouco campos por struct (relaxado para Go)

**Regra original (Bay)**: ≤ 2 campos por struct. **Adaptação Go pragmática**:
- VO: 1-3 campos.
- Aggregate: 5-12 campos (DDD natural).
- Use case: ≤ 7 deps no construtor (3-5 ideal).
- DTO HTTP: até 20 campos é aceitável (forma da API).
- Config struct: até 30 campos (aglutina nested config).

**Por quê**: previne **AP-002** (god struct). Aggregate com 30 campos = sinal forte de extrair sub-aggregate ou VO.

**Quando dividir**:
- Aggregate com > 12 campos → extrair sub-aggregate ou VO composto (`Address`, `BillingCycle`).
- Use case com > 7 deps → extrair Saga, dividir em commands menores, ou puxar para domain service.

**✅ Exemplo Go**:

```go
// VO com 1 campo
type CNPJ struct { digits string }
type Money struct { cents int64 }
type ULID struct { ulid.ULID }

// VO composto — Address tem 5 campos mas é coeso
type Address struct {
    Street, Number, Complement, City, State string
    ZipCode CEP
}

// Aggregate com 9 campos (OK)
type Condominium struct {
    id, publicID, name string
    condoType          enums.CondominiumType
    address            value_objects.Address
    totalUnits         int
    blocksOrTowers     int
    createdAt, updatedAt time.Time
}

// Use case com 5 deps (OK)
type CastVoteUseCase struct {
    voteRepo    repositories.IVoteRepository
    sessionRepo repositories.IVoteSessionRepository
    clock       contracts.Clock
    unitOfWork  contracts.IUnitOfWork
    log         logger.Logger
}
```

**❌ Anti-exemplo**:

```go
// God struct (AP-002)
type Subscription struct {
    ID, CustomerID, PlanID, Status, StripeID                 string
    TrialEndsAt, ActivatedAt, CanceledAt, NextBillingAt      *time.Time
    AmountCents, DiscountCents, TaxCents, FeeCents           int64
    PdfPath, InvoicePath, ReceiptPath                         string
    EmailSent, SmsSent, PushSent                              bool
    Quota, QuotaUsed, QuotaLimit                              int64
    // ... 30 campos misturando 5 conceitos
}
```

**Detecção**: code review; `wc -l` aggregate > 200 = warn de campo demais.

---

### CAL-009 — Sem getters/setters genéricos; métodos comportamentais

**Regra (Go-flavor)**:
- Campos privados (`lowercase`); getters explícitos só quando há consumer real (`ID()`, `Name()`, `Status()`).
- **Zero `Set*Field()` exposto**. Mutação é via método com semântica de domínio (`MarkCanceled(reason)`, `Start()`, `CastVote(...)`).
- DTOs/Request structs do HTTP **podem** ter campos exportados — eles são adapters de contrato externo (não domínio).

**Por quê**: previne mutação descontrolada (CA-010). Métodos comportamentais documentam invariantes.

**✅ Exemplo Go (correto)**:

```go
// domain entity — sem setters, getters semânticos
type Subscription struct {
    id     string
    status SubscriptionStatus
    // ...
}

func (s *Subscription) ID() string             { return s.id }
func (s *Subscription) Status() SubscriptionStatus { return s.status }

// mutação com semântica
func (s *Subscription) MarkCanceled(reason string) error { ... }
func (s *Subscription) MarkPastDue() error               { ... }

// DTO HTTP — campos exportados (OK, é adapter de contrato externo)
type CreateSubscriptionRequest struct {
    PlanID    string `json:"plan_id"`
    CouponID  string `json:"coupon_id,omitempty"`
}
```

**❌ Anti-exemplo**:

```go
type Subscription struct {
    ID, Status string  // exportado e mutável
}

func (s *Subscription) SetStatus(v string)  { s.status = v }   // ❌ sem invariante
func (s *Subscription) SetCanceledAt(t time.Time) { s.canceledAt = t }  // ❌
// uso: sub.SetStatus("canceled") — pulou MarkCanceled, sem validação, sem evento
```

**Detecção**: grep CI `func \(\w+ \*\w+\) Set[A-Z]\w+\(` em `domain/entities/` → fail.

---

## Checklist de PR (cole no template)

```markdown
## Code Calisthenics (CAL)

- [ ] CAL-001: nenhum método com > 1 nível de indentação aninhada
- [ ] CAL-002: zero `} else {` no código de produção (CI grep verde)
- [ ] CAL-003: parâmetros/campos de domínio são VO, não primitivos crus
- [ ] CAL-004: coleção com invariante/comportamento próprio é type nomeado (Votes, Memberships)
- [ ] CAL-005: nenhuma chain com > 2 dots fora de tests/sqlc generated
- [ ] CAL-006: zero abreviações fora do canon Go (ctx, err, i/j, r/w, receivers 1-2 letras)
- [ ] CAL-007: aggregates ≤ 200 linhas, pacotes domain ≤ 10 arquivos
- [ ] CAL-008: aggregates ≤ 12 campos; use cases ≤ 7 deps
- [ ] CAL-009: zero `Set*Field()` em domain/entities/; mutação só via método de negócio
```

---

## Exceções aceitas (com justificativa)

| Exceção | Justificativa |
|---|---|
| `ctx`, `err`, `i`/`j` em loops, receivers 1-2 letras, `r/w` em readers/writers | Idioma Go canônico ([effective-go](https://go.dev/doc/effective_go#methods)). CAL-006 não se aplica. |
| `else` em `switch ... default` | Não é `else` semântico; switch já encapsula dispatch. |
| Chains profundas em `*_test.go` (`require.Equal(t, exp, got.Result.Data)`) | Test assertions priorizam legibilidade do erro. |
| Chains em `infrastructure/database/generated/` (sqlc) | Código gerado; **não editar**. |
| DTO HTTP / Request struct / Config struct com campos exportados | Adapter de contrato externo (JSON/env); CAL-009 não aplicável. |
| `Subscription` (210 linhas) | Limite suave 200; comentário no topo + ADR documentam justificativa. |
| Aggregate com > 12 campos quando representa entidade essencialmente complexa (ex: `Assembly` com items, votes, presidente, edital, ata) | Complexidade essencial; refatorar via VO composto (`AgendaItems`, `Votes`) para reduzir contagem efetiva. |
| `Address` VO com 5-6 campos | VO composto coeso; troca por struct atomic seria pior (string concat fragile). |

---

## Mapa CAL → AP-### prevenido

| Regra | AP-### prevenido |
|---|---|
| CAL-001 (1-level indent) | AP-001, AP-011 |
| CAL-002 (sem else) | AP-011 |
| CAL-003 (wrap primitives) | AP-006, AP-007 (PII via VO.Masked) |
| CAL-004 (first-class collections) | AP-002 (lógica de coleção espalhada) |
| CAL-005 (Demeter) | acoplamento excessivo (AP universal) |
| CAL-006 (não abreviar) | AP-014 (logging confuso) |
| CAL-007 (entidades pequenas) | AP-001, AP-002 |
| CAL-008 (poucos campos) | AP-002 (god struct) |
| CAL-009 (sem set/get genéricos) | AP-009 (anemic domain), mutação descontrolada |

---

## Vizinhos

- [[../../../10-knowledge/principles/code-calisthenics]] — referência canônica atemporal
- [[../../../10-knowledge/principles/_moc]]
- [[../../../20-stacks/go/_moc]]
- [[CLEAN_CODE]] — naming + funções pequenas (CC-001..CC-020)
- [[CLEAN_ARCH]] — camadas + regra de dependência (CA-001..CA-017)
- [[SOLID]] — 5 princípios Go-flavor
- [[PATTERNS]] — Strategy via map (CAL-002 + OCP)
- [[antipatterns]] — catálogo AP-001..AP-026
- [[../03-subprojects/backend/patterns]] §8 (sem else, no código real)
- [[../CLAUDE]] §5 — proibições
