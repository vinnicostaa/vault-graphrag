---
title: Clean Code aplicado — Master Síndico (Go)
type: note
tags:
  - principle-applied
  - clean-code
  - master-sindico
  - quality
project: master-sindico
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - clean-code-master-sindico
  - clean-code-go-ms
---

# Clean Code aplicado — Master Síndico v2 (Go)

Regras concretas de Clean Code para o backend Go 1.26 do Master Síndico (Gin abstraído + pgx/v5 + sqlc + zerolog + Redis 7 + Stripe + Mux + Livekit + Zitadel). Especialização do canônico atemporal [[../../../10-knowledge/principles/clean-code]] com exemplos do **stack real** e detecção via golangci-lint, gosec e grep CI.

> Cada regra **CC-###** previne pelo menos um antipattern do catálogo [[antipatterns]] (AP-001..AP-026) ou protege uma invariante de [[../01-domain/invariants]] (INV-###). Não duplica o knowledge global — linka.

---

## Sumário

Clean Code aqui é o que diferencia código que outro dev (ou agente Claude em modo autônomo) consegue **ler, modificar e testar em < 5min** de código que vai gerar um AP-### novo no [[antipatterns]]. Em legado TS havia funções de 450 linhas (AP-001), `Subscription` virou God Object com Stripe importado no domínio. A reescrita Go corta isso pela raiz com regras numéricas e linters obrigatórios.

---

## Regras concretas

### CC-001 — Função em `domain/` ≤ 30 linhas

**Regra**: nenhuma função/método dentro de `internal/modules/*/domain/` excede 30 linhas (sem contar comments e blank lines).

**Por quê**: previne **AP-001** (God Object — `Subscription` 450L do legado) e **AP-002** (`Invoice` 550L). Função grande no domínio = responsabilidade misturada = invariante implícita = bug.

**✅ Exemplo Go (correto)**:

```go
// internal/modules/assembly/domain/entities/assembly.go
func (a *Assembly) Start() error {
    if a.status == enums.AssemblyStatusEncerrada { return ErrAssemblyAlreadyEnded }
    if a.status != enums.AssemblyStatusPublicada { return ErrAssemblyNotPublished }
    if !a.IsEditalPublished() { return ErrAssemblyEditalRequired }
    now := time.Now()
    a.status = enums.AssemblyStatusEmAndamento
    a.startedAt = &now
    a.updatedAt = now
    return nil
}
```

**❌ Anti-exemplo (AP-001)**:

```go
// God method — 80+ linhas misturando validação, mutação, side-effect, log
func (s *Subscription) Activate(ctx context.Context, plan Plan, customer stripe.Customer, ...) error {
    // 80+ linhas com if/else aninhado, chamadas Stripe, logs, mutação direta...
}
```

**Detecção**:
- `gocyclo -over 8 ./internal/modules/*/domain/...` (também limita complexidade)
- golangci-lint config: `funlen: { lines: 30, statements: 25 }` aplicado a `domain/`
- Code review: PR template tem item "função em domain ≤ 30 linhas?"

---

### CC-002 — Função em `application/use_cases/` ≤ 50 linhas

**Regra**: handler de use case (`Execute(ctx, cmd)`) ≤ 50 linhas; passos auxiliares são métodos privados.

**Por quê**: previne **AP-010** (use case com lógica espalhada) e **AP-012** (autorização duplicada). Um use case longo = sinal de Saga não decomposta ou query misturada com command (CQRS quebrado).

**✅ Exemplo Go**:

```go
// internal/modules/commercial/application/use_cases/cast_vote_use_case.go
func (uc *CastVoteUseCase) Execute(ctx context.Context, cmd CastVoteCommand) (*VoteDTO, error) {
    if err := cmd.Validate(); err != nil {
        return nil, err
    }
    var dto *VoteDTO
    err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        sess, err := uc.voteRepo.FindSessionByIDForUpdate(ctx, cmd.SessionID)
        if err != nil { return err }
        if sess.Status.IsTerminal() {
            return apperrors.ErrConflict.WithMessage("sessão encerrada")
        }
        vote, err := entities.NewVote(cmd.SessionID, cmd.VoterID, cmd.Option)
        if err != nil { return err }
        if err := uc.voteRepo.CastVote(ctx, vote); err != nil { return err }
        dto = mappers.VoteToDTO(vote)
        return uc.voteRepo.IncrementVotesFor(ctx, sess.ID)
    })
    return dto, err
}
```

**❌ Anti-exemplo**: use case de 120 linhas que faz query + autorização + mutação + 3 calls externas sem Saga.

**Detecção**: `funlen` no golangci-lint para `application/use_cases/` com `lines: 50`. PR review: handler com mais de 1 nível de aninhamento dentro do `uow.Run` é sinal de extrair método privado.

---

### CC-003 — Handler HTTP ≤ 20 linhas

**Regra**: função em `infrastructure/http/routes/*.go` ≤ 20 linhas. Handler decodifica, chama use case, serializa.

**Por quê**: previne **AP-010** (regra de negócio no handler). Handler é **plumbing**, não lógica.

**✅ Exemplo Go**:

```go
// internal/modules/commercial/infrastructure/http/routes/vote_handler.go
func (h *VoteHandler) Cast(c contracts.Context) {
    var dto CastVoteRequest
    if err := c.DecodeJSON(&dto); err != nil {
        c.SetError(apperrors.ErrValidation.WithMessage(err.Error()))
        return
    }
    cmd := usecases.CastVoteCommand{
        SessionID: dto.SessionID,
        VoterID:   authUserFrom(c).UserID,
        Option:    dto.Option,
    }
    out, err := h.castVoteUC.Execute(c.RequestContext(), cmd)
    if err != nil { c.SetError(err); return }
    c.JSON(201, out)
}
```

**❌ Anti-exemplo (AP-010)**:

```go
func (h *VoteHandler) Cast(c contracts.Context) {
    // ... 60 linhas com query, autorização inline, cálculo de quórum, log...
}
```

**Detecção**: golangci-lint `funlen` com regra específica para `infrastructure/http/`; code review.

---

### CC-004 — Cyclomatic complexity ≤ 8

**Regra**: nenhuma função tem complexidade ciclomática maior que 8 (gocyclo).

**Por quê**: previne **AP-011** (if/else hell — `UpdateProfileUseCase` legado com 5 cases). Complexidade alta = strategy/dispatch faltando.

**✅ Exemplo (correto — Strategy via map)**:

```go
// internal/modules/billing/domain/services/trial_policy.go
type TrialPolicy interface {
    TrialDays(persona enums.Persona) int
}

var trialRegistry = map[enums.Persona]TrialPolicy{
    enums.PersonaSindico:  SindicoTrial{},
    enums.PersonaEmpresa:  EmpresaTrial{},
    enums.PersonaParceiro: ParceiroTrial{},
    enums.PersonaMorador:  NoTrialPolicy{},
}

func TrialDaysFor(p enums.Persona) int {
    policy, ok := trialRegistry[p]
    if !ok { return 0 }
    return policy.TrialDays(p)
}
```

**❌ Anti-exemplo (AP-011)**:

```go
func TrialDaysFor(p Persona) int {
    if p == PersonaSindico { return 15 }
    if p == PersonaEmpresa { return 7 }
    if p == PersonaParceiro { return 7 }
    if p == PersonaAgencia { return 14 }
    if p == PersonaMorador { return 0 }
    return 0
}
```

**Detecção**:
- `gocyclo -over 8 ./...` (gate CI)
- golangci-lint `gocognit: { min-complexity: 10 }`
- `revive: { rules: [{ name: cognitive-complexity, arguments: [10] }] }`

---

### CC-005 — Erro sempre encapsulado com `fmt.Errorf("...: %w", err)`

**Regra**: ao retornar erro de outra camada, encapsular com contexto. Domain retorna sentinels exportados (`ErrXxx`); infra/application encapsulam com `%w`.

**Por quê**: previne **AP-014** (logging inconsistente). Sem `%w`, perde-se a stack semântica e `errors.Is` quebra.

**✅ Exemplo Go**:

```go
// infrastructure/database/vote_repository_impl.go
func (r *VoteRepository) CastVote(ctx context.Context, v *entities.Vote) error {
    db := database.DBFromContext(ctx, r.pool)
    q := generated.New(db)
    _, err := q.InsertVote(ctx, generated.InsertVoteParams{...})
    if err != nil {
        if isUniqueViolation(err) {
            return apperrors.ErrConflict.WithMessage("votante já registrou voto") // INV-071
        }
        return fmt.Errorf("vote_repo.CastVote: %w", err)
    }
    return nil
}
```

**❌ Anti-exemplo**:

```go
if err != nil { return errors.New("erro ao salvar vote") } // perde causa
if err != nil { return err }                                // sem contexto, debug impossível
```

**Detecção**:
- golangci-lint `wrapcheck: { ignoreSigs: ["fmt.Errorf"] }`
- `errcheck` ativado para todas as packages
- `errorlint: { errorf: true, errorf-multi: true, asserts: true }`

---

### CC-006 — Proibido `_ = fn()` em I/O

**Regra**: descartar erro de função que faz I/O (DB, HTTP, Redis, NATS, SDK externo) é **blocker absoluto**.

**Por quê**: **AP-019** — finding histórico A-001..A-008. Causou estado divergente entre DB e SearchEngine no legado por meses.

**✅ Exemplo Go (correto — Outbox)**:

```go
// command publica via outbox, não fire-and-forget
if err := h.outboxRepo.Enqueue(ctx, events.SubscriptionActivatedEvent{...}); err != nil {
    return fmt.Errorf("outbox enqueue: %w", err)
}
```

**❌ Anti-exemplo (AP-019)**:

```go
_ = h.searchIndexer.Index(ctx, user)  // PROIBIDO
go func() { h.notifier.Send(ctx, msg) }() // fire-and-forget sem outbox + sem log
```

**Quando aceitável (com justificativa explícita)**:

```go
defer func() {
    if cErr := tx.Rollback(ctx); cErr != nil && !errors.Is(cErr, pgx.ErrTxClosed) {
        log.Warn().Err(cErr).Msg("rollback failed (already committed?)")
    }
}()
```

**Detecção**:
- grep CI: `^\s*_\s*=\s*\w+\.(Index|Send|Publish|Save|Insert|Update|Delete|Enqueue|Notify|Commit)` → fail
- `errcheck` no golangci-lint com `check-blank: true`

---

### CC-007 — Sentinel errors em `domain/`, AppError em `application/`

**Regra**: domain exporta sentinels (`var ErrXxx = errors.New(...)`); application traduz para AppError padronizado (`apperrors.ErrConflict`, `apperrors.ErrValidation`); infra retorna sentinel ou wraps com `%w`.

**Por quê**: permite `errors.Is(err, domain.ErrXxx)` em testes e fluxo; HTTP serializa AppError uniformemente (CC-014).

**✅ Exemplo Go**:

```go
// internal/modules/billing/domain/entities/feature_usage.go
var ErrQuotaExhausted = errors.New("quota exhausted for current period")

// internal/modules/billing/application/use_cases/consume_quota_use_case.go
if err := usage.Consume(limit); err != nil {
    if errors.Is(err, entities.ErrQuotaExhausted) {
        return nil, apperrors.ErrQuotaExhausted.WithMessage("limite mensal atingido")
    }
    return nil, fmt.Errorf("consume: %w", err)
}
```

**❌ Anti-exemplo**: `return errors.New("quota cheia")` em domínio — sem `errors.Is`, sem AppError, log ambíguo.

**Detecção**: code review; grep `errors.New(` em `*_use_case.go` e `*_repository_impl.go` → suspeito; em `domain/entities/*.go` deve ser apenas em `var Err... = errors.New(...)`.

---

### CC-008 — Naming idiomático Go

**Regra**:
- Package: `lowercase`, singular, curto (`identity`, `billing`, `assembly`, **não** `identities`/`bills`).
- Interface: termina em `-er` se 1 método (`Validator`); prefixo `I` aceito quando múltiplos métodos coordenam contrato (`IPaymentGateway`, `IUnitOfWork`) — convenção MS herdada (ver [[../03-subprojects/backend/architecture]] §3).
- Função/método exportado: `PascalCase` em inglês; godoc obrigatório quando público.
- Variável local: scope curto = nome curto (`u`, `c`, `i`); scope longo = descritivo (`condominium`, `activeMembership`).
- Sentinel error: prefixo `Err` (`ErrQuotaExhausted`, `ErrAssemblyEditalRequired`).
- Constante: `MaxLoginAttempts`, `DefaultPageSize` — sem `_`/`SCREAMING`.

**Por quê**: idioma Go reduz fricção entre desenvolvedores e Claude autônomo.

**✅ Exemplo Go**:

```go
package assembly  // OK: lowercase singular

type IAssemblyRepository interface { /* convenção MS */ }
type Voter interface { Vote() error }       // -er sufixo

const MaxAgendaItemsPerAssembly = 50        // const exportada
var ErrAssemblyEditalRequired = errors.New("edital obrigatório")
```

**❌ Anti-exemplo**:

```go
package Assemblies                          // package PascalCase + plural
type assembly_repo interface { ... }        // snake_case + minúsculo exportado
var ASSEMBLY_NOT_FOUND = errors.New(...)    // SCREAMING
```

**Detecção**: `revive: { rules: [{ name: var-naming }, { name: package-comments }, { name: error-naming }] }`; golangci-lint `stylecheck` com defaults.

---

### CC-009 — Variáveis proibidas: `data`, `info`, `obj`, `temp`, `value`

**Regra**: nomes inexpressivos só em escopo trivial (loop counter, helper de 3 linhas). Em fluxo de negócio, sempre nome semântico.

**Por quê**: `temp.Save()` numa code review é blocker. Custo cognitivo de reler o tipo todo cresce O(N).

**✅ Exemplo**:

```go
activeMembership, err := uc.memberRepo.GetActive(ctx, tenantID, voterID)
fracaoIdeal := activeMembership.FracaoIdeal()
```

**❌ Anti-exemplo**:

```go
data, _ := uc.memberRepo.GetActive(ctx, tenantID, voterID)
val := data.FracaoIdeal()
temp := val.Multiply(...)
```

**Detecção**: grep CI em `*.go` não-teste com regex `\b(data|info|obj|temp|value|item|result)\s*[:,]?=` em scope > 5 linhas → flag em PR (warn, não fail).

---

### CC-010 — Sem números mágicos: `const` nomeada

**Regra**: literal numérico/string só dentro de:
- testes (`require.Equal(t, 42, ...)` OK);
- `0`, `1`, `-1`, `nil`, `""` em scope óbvio;
- regex/SQL inline.

Caso contrário, `const Name = ...`.

**Por quê**: `if attempts > 5` esconde a regra; `if attempts > MaxLoginAttempts` documenta. Previne AP-007 universal (magic constants).

**✅ Exemplo Go**:

```go
// internal/shared/middleware/rate_limiter.go
const (
    DefaultRequestsPerMinute = 60
    AuthRequestsPerMinute    = 20
    DefaultBurst             = 10
    AuthBurst                = 5
    CleanupInterval          = 30 * time.Minute
    InactiveIPTTL            = 2 * time.Hour
)
```

**❌ Anti-exemplo**:

```go
limiter := rate.NewLimiter(rate.Every(time.Second*60/60), 10) // 60? 10? wat
if user.LoginAttempts > 5 { lockAccount() }                   // por que 5?
```

**Detecção**: golangci-lint `gomnd: { settings: { mnd: { ignored-numbers: "0,1,2,10,100" } } }` ativado em `internal/`.

---

### CC-011 — Boolean param trap → option struct ou enum

**Regra**: nunca passe `bool` como parâmetro funcional em função pública. Use enum tipado ou option struct.

**Por quê**: `CreateUser("ana", true)` em call site é ilegível. Bool flips silenciosos = bugs.

**✅ Exemplo Go**:

```go
// enum tipado
type SubscriptionMode string
const (
    ModeTrial   SubscriptionMode = "trial"
    ModePaid    SubscriptionMode = "paid"
    ModePending SubscriptionMode = "pending"
)
func NewSubscription(planID PlanID, mode SubscriptionMode) (*Subscription, error)

// option struct
type CreateOpts struct {
    SkipEmail        bool
    InitialTier      PlanTier
    SeedCondoIDs     []string
}
func CreateUser(ctx context.Context, cmd Cmd, opts CreateOpts) (*User, error)
```

**❌ Anti-exemplo**:

```go
func CreateUser(ctx context.Context, name string, sendEmail bool, isAdmin bool) error
// chamada: CreateUser(ctx, "ana", true, false) — quem lê não sabe o que é cada bool
```

**Detecção**: code review; linter customizado `revive` rule `flag-parameter` ativada.

---

### CC-012 — Comments: WHY > WHAT; godoc em exportado

**Regra**:
- Public function/method/type/const **deve** ter godoc (frase começando com o identifier).
- Comments inline explicam **por quê** (invariante INV-###, ADR-####, AP-### prevenido). **Não** parafraseiam o código.
- Sem dead code comentado: `git history` é o histórico.

**✅ Exemplo Go**:

```go
// IsActive retorna true quando a assinatura é trialing válida (trialEndsAt no futuro)
// ou ativa. Bug Sprint 7.6 #2: trialing com trialEndsAt expirado retornava true.
// Ver INV-017 (trial válido) e BR-001 (politica de trial por persona).
func (s *Subscription) IsActive() bool {
    if s.status == SubscriptionStatusTrialing && !s.IsInTrial() {
        return false
    }
    return s.status.IsActiveStatus()
}
```

**❌ Anti-exemplo**:

```go
// incrementa o contador
counter++
// se status é active retorna true
return s.status == "active"
```

**Detecção**:
- `revive: { rules: [{ name: exported, arguments: [checkPrivateReceivers] }] }`
- `godot` linter ativo (godoc termina com ponto).
- grep `// TODO` / `// FIXME` em CI: alerta se tem mais de 30d sem issue linkada.

---

### CC-013 — Imports: `goimports -local github.com/master-sindico` enforced

**Regra**: três grupos de imports (separados por blank line):
1. stdlib
2. external (`github.com/...`, `gorm.io/...`)
3. internal (`github.com/master-sindico/...`)

Sem dot imports (`.`); sem aliases sem necessidade.

**Por quê**: legibilidade + git diff limpo + impede import cycle por engano.

**✅ Exemplo Go**:

```go
import (
    "context"
    "fmt"
    "time"

    "github.com/jackc/pgx/v5"
    "github.com/rs/zerolog"

    "github.com/master-sindico/backend/internal/core/contracts"
    "github.com/master-sindico/backend/internal/modules/billing/domain/entities"
)
```

**Detecção**: pre-commit hook + CI `goimports -local github.com/master-sindico -d` → fail se diff > 0.

---

### CC-014 — Estrutura de erro HTTP padronizada (AppError)

**Regra**: **toda** resposta de erro HTTP é serializada via `apperrors.AppError`. Handler nunca retorna `c.JSON(500, "msg")` direto. Middleware `ErrorHandler` intercepta `ctx.SetError(err)`.

**Por quê**: contrato uniforme com web/mobile; client (SolidJS, Flutter) tem 1 deserializer.

**✅ Exemplo Go**:

```go
// shape canônico em /core/errors/errors.go
type AppError struct {
    Code    string `json:"code"`     // "VALIDATION", "FORBIDDEN", "NOT_FOUND", "QUOTA_EXHAUSTED"
    Message string `json:"message"`  // pt-BR amigável
    Details any    `json:"details,omitempty"`
}

// uso em use case
return nil, apperrors.ErrValidation.WithMessage("CPF inválido").WithDetails(map[string]string{"field": "cpf"})

// handler delega — nunca toca status direto
out, err := h.uc.Execute(ctx, cmd)
if err != nil { c.SetError(err); return }
c.JSON(201, out)
```

**❌ Anti-exemplo**:

```go
c.JSON(500, gin.H{"error": "erro interno"})  // bypassa AppError; código não-padronizado
```

**Detecção**: grep em `infrastructure/http/routes/` por `c.JSON(4` ou `c.JSON(5` → fail (deve usar `c.SetError`).

---

### CC-015 — Logger estruturado, zero PII, sempre com `ctx`

**Regra**:
- Sempre `logger.Ctx(ctx).Info()...` (preserva trace_id, span_id, request_id, user_id, tenant_id).
- Nunca `fmt.Printf`, `fmt.Println`, `log.Print*`.
- Nunca log de campo PII direto: usar `Masked()` (CPFVO, CNPJVO, EmailVO).
- Erro sempre via `.Err(err)`, não `%v`/`%+v` (preserva stack trace zerolog).

**Por quê**: previne **AP-007** + **AP-014** + **AP-022**. LGPD §14.

**✅ Exemplo Go**:

```go
// pkg/logger/logger.go expõe logger.F() helper
log.Info().
    Ctx(ctx).
    Str("user_id", user.ID()).
    Str("tenant_id", tenantID).
    Str("cnpj", company.CNPJ().Masked()).  // **.***.***/****-**
    Err(err).
    Msg("connect_me request rejected")
```

**❌ Anti-exemplo (AP-007/AP-022)**:

```go
fmt.Printf("user: %+v\n", user)                      // imprime CPF, email, tudo
log.Info().Str("email", user.Email().Value()).Msg(...) // PII em claro
log.Info().Msgf("erro: %v", err)                      // perde stack
```

**Detecção**:
- regex CI: `\b\d{3}\.\d{3}\.\d{3}-\d{2}\b` em `.go` não-teste → fail
- regex CI: `logger.*\.Str\("(email|cpf|cnpj_digits|token|password)"` → fail
- regex CI: `(fmt\.Printf|log\.Print)` em `internal/` → fail
- gosec `G101`/`G102` ativados

---

### CC-016 — `context.Context` é primeiro parâmetro

**Regra**: toda função que faz I/O, transação ou pode demorar **deve** receber `ctx context.Context` como primeiro parâmetro. Nunca armazenar `ctx` em struct.

**Por quê**: cancelamento, timeout, propagação de tracing/logger/auth_user. Sem isso, observability quebra e shutdown vaza goroutines.

**✅ Exemplo**:

```go
func (r *VoteRepository) FindByID(ctx context.Context, id string) (*entities.Vote, error)
```

**❌ Anti-exemplo**:

```go
type Service struct { ctx context.Context }                     // antipattern Go canônico
func (r *VoteRepository) FindByID(id string) (*entities.Vote, error) // sem ctx
```

**Detecção**: golangci-lint `containedctx` ativado; `revive` rule `context-as-argument`.

---

### CC-017 — Time injection via `Clock` interface

**Regra**: domain/application **nunca** chamam `time.Now()` diretamente. Recebem `Clock` no construtor (`type Clock interface { Now() time.Time }`).

**Por quê**: testabilidade (PBT em invariantes temporais como trial expira em 15 dias — INV-017). Permite simular tempo avançando.

**✅ Exemplo Go**:

```go
// core/contracts/clock.go
type Clock interface { Now() time.Time }

// stub em testes
type FakeClock struct{ Time time.Time }
func (f *FakeClock) Now() time.Time { return f.Time }

// uso
type CastVoteUseCase struct {
    clock Clock
    // ...
}
func (uc *CastVoteUseCase) Execute(ctx context.Context, cmd Cmd) (*VoteDTO, error) {
    castAt := uc.clock.Now().UTC()
    // ...
}
```

**❌ Anti-exemplo**: `time.Now()` direto em handler de use case → impossível testar trial expirando.

**Detecção**: grep `time\.Now\(\)` em `application/` e `domain/` → revisão obrigatória; OK em `infrastructure/`.

---

### CC-018 — DTO HTTP separado de DTO use case

**Regra**: 3 shapes distintos:
1. **Request DTO** (`infrastructure/http/dto`) — JSON tags, primitivos, validação `validator`
2. **Command/Query** (`application/use_cases`) — VO, IDs, `time.Time`
3. **Response DTO** (`application/mappers`) — output do use case

Mapper em `application/mappers/` traduz entre cada par.

**Por quê**: previne AP universal "DTO crua vazando para domínio". Frontend muda contract sem refazer use case.

**✅ Exemplo Go**:

```go
// HTTP DTO
type CastVoteRequest struct {
    SessionID string `json:"session_id" validate:"required,ulid"`
    Option    string `json:"option" validate:"required,oneof=sim nao abstencao branco"`
}

// Command
type CastVoteCommand struct {
    SessionID  ULID
    VoterID    ULID  // injetado do auth_user
    Option     enums.VoteOption
}

// Response DTO
type VoteDTO struct {
    ID        string    `json:"id"`
    SessionID string    `json:"session_id"`
    Option    string    `json:"option"`
    CastAt    time.Time `json:"cast_at"`
}
```

**Detecção**: code review; grep por `json:"` em `domain/` ou `application/` → fail (JSON tags pertencem a `infrastructure/http/dto/`).

---

### CC-019 — Receiver pointer consistente

**Regra**: por type, um único receiver kind. Aggregate sempre `*Type` (mutável). VO sempre `Type` (imutável).

**Por quê**: misturar = bugs sutis (cópia parcial, mutex em valor).

**✅ Exemplo Go**:

```go
// Aggregate — mutável
func (a *Assembly) Start() error { ... }
func (a *Assembly) Status() enums.AssemblyStatus { ... }

// VO — imutável
func (m Money) Add(o Money) Money { return Money{m.cents + o.cents} }
func (m Money) Cents() int64 { return m.cents }
```

**❌ Anti-exemplo**:

```go
func (a *Assembly) Start() error { ... }
func (a Assembly) Status() AssemblyStatus { ... }  // inconsistente
```

**Detecção**: golangci-lint `recvcheck` (Go 1.22+).

---

### CC-020 — Unused/dead code = blocker

**Regra**: variável, função ou import não usado **bloqueia** build.

**Por quê**: dead code mascara intenção. Em legado TS havia `helpers/` com 30 funções nunca importadas.

**Detecção**:
- `go vet ./...` (já fail por padrão em unused vars)
- `unused` linter no golangci-lint
- `unparam` para parâmetros nunca usados

---

## Checklist de PR (cole no template)

```markdown
## Clean Code (CC)

- [ ] CC-001: nenhuma função `domain/` > 30 linhas (`gocyclo` ok)
- [ ] CC-002: nenhum `Execute()` de use case > 50 linhas
- [ ] CC-003: nenhum handler HTTP > 20 linhas
- [ ] CC-004: cyclomatic complexity ≤ 8 (`gocyclo -over 8` zero findings)
- [ ] CC-005: erros encapsulados com `fmt.Errorf("...: %w", err)`
- [ ] CC-006: zero `_ = fn()` em I/O (grep CI verde)
- [ ] CC-007: domain usa sentinels; application traduz para AppError
- [ ] CC-008: naming idiomático (package lowercase, ErrXxx, etc.)
- [ ] CC-009: nenhum `data`/`info`/`temp` em scope > 5 linhas
- [ ] CC-010: literais numéricos/strings de negócio em `const` nomeada
- [ ] CC-011: zero parâmetros bool em função pública
- [ ] CC-012: godoc em todo identifier exportado novo
- [ ] CC-013: `goimports -local github.com/master-sindico` zero diff
- [ ] CC-014: handlers usam `c.SetError(err)`; nunca `c.JSON(4xx/5xx)` direto
- [ ] CC-015: zero PII em logs (regex CI verde) + `Ctx(ctx)` em todo log call
- [ ] CC-016: `ctx context.Context` é primeiro param em I/O
- [ ] CC-017: domain/application não chamam `time.Now()` direto (Clock injetado)
- [ ] CC-018: DTO HTTP ≠ Command ≠ Response DTO
- [ ] CC-019: receiver pointer consistente por type
- [ ] CC-020: zero unused/dead code (`go vet` verde)
```

---

## Exceções aceitas (com justificativa)

| Exceção | Justificativa |
|---|---|
| Receivers de 1-2 letras (`r *Repo`, `u *User`) | Idioma Go canônico ([effective-go](https://go.dev/doc/effective_go#methods)) |
| `ctx`, `err`, `ok`, `i`/`j` | Idioma Go; `ctx context.Context` em chamadas é tão denso que abreviar é mais legível |
| DTO HTTP com primitivos (`string`, `int64`) | Validation acontece no handler com `validator`; VO entra no command |
| `_, err := db.Exec(...)` | OK — `_` é o `Result`, não o `error`. Ainda checa `err`. |
| sqlc-generated (`internal/modules/*/infrastructure/database/generated/*.go`) | Código gerado; **não editar**; CC-### não aplicáveis |
| Test fixtures (`*_test.go`, `*_stubs_test.go`) | Magic numbers em assertions são parte do teste |
| Migrations (`migrations/*.sql`) | DDL não é Go |
| Funções em `pkg/safecast/` | Helpers genéricos com 5-10 linhas — função pequena demais para godoc longo |

---

## Vizinhos

- [[../../../10-knowledge/principles/clean-code]] — referência canônica atemporal
- [[../../../10-knowledge/principles/_moc]]
- [[../../../20-stacks/go/_moc]]
- [[PATTERNS]] — patterns aplicados (DDD, CQRS, Saga, etc.)
- [[CLEAN_ARCH]] — camadas e regra de dependência
- [[SOLID]] — princípios SOLID Go-flavor
- [[CODE_CALISTHENICS]] — 9 regras Calisthenics
- [[antipatterns]] — catálogo AP-001..AP-026
- [[../01-domain/invariants]] — INV-### testáveis
- [[../03-subprojects/backend/architecture]] — arquitetura backend real
- [[../03-subprojects/backend/patterns]] — patterns aplicados (código real)
- [[../CLAUDE]] §5 — proibições
