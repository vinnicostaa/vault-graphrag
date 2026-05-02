---
title: Antipatterns do Legacy — Não Repetir
version: 1.0
status: stable
type: reference
tags: [legacy, antipatterns, lessons-learned]
---

# Antipatterns do full-stack legacy — o que NÃO repetir no backend Go

O `full-stack/` (TypeScript + Fastify + Drizzle) foi uma tentativa anterior do Master Síndico que falhou por razões específicas, documentadas aqui. O backend Go **não repete nenhum desses padrões** — e código que violar vai ser bloqueado em code review ou por hook/CI.

Fonte: auditoria do código `full-stack/apps/api/src/`, `contextos/ANALISE_COMPLETA.md` e `plans/architecture-refined.md`.

---

## A1 — ABAC desativado silenciosamente

### O que aconteceu
`PermissionService` estava implementado no legacy mas **comentado no bootstrap**. Resultado: o sistema rodou semanas sem enforcement real de permissões. Todos os usuários tinham acesso implícito a tudo que passava pelo middleware auth.

### Por que aconteceu
- Durante debug, alguém comentou para testar fluxo end-to-end
- Sem teste de integração validando que ABAC **nega** request não autorizado
- Sem alerta no startup ("ABAC inicializado com 0 regras")

### Fix obrigatório no Go
1. **Falhar no startup** (`main.go`) se matriz ABAC vazia ou inválida — `config.Validate()` retorna erro, aplicação não sobe
2. **Teste de integração** `TestABAC_FailsStartupIfMatrixEmpty` verifica que a API não aceita requests sem ABAC configurado
3. **Log estruturado em startup**: `"ABAC engine inicializado | rules: 47 | roles: 6 | actions: 123"`
4. Em `.kiro/steering/security.md`: regra dura "ABAC engine **nunca** é opcional"

---

## A2 — Vazamento de domínio entre bounded contexts

### O que aconteceu
`UpdateProfileUseCase` em `profile/` importava `UserRole` de `user/`:

```typescript
// ❌ Acoplamento direto entre bounded contexts
import { UserRole } from '../user/domain/types';
// ...
switch (user.role as UserRole) { ... }
```

Cada mudança em `UserRole` quebrava 5 módulos. Testes unitários acoplados. Refactor impossível sem efeito dominó.

### Fix obrigatório no Go
- Cada bounded context define **seus próprios tipos** de enum no `domain/enums/`
- Cruzar contexto via `shared/types/` (ex: `AuthUser` vive em `shared/types/`, não em `modules/identity/`)
- Ou via **domain events** (`IDomainEvent`) — contexto A emite, contexto B subscribe
- Regra em `steering/go-patterns.md`: **nenhum `modules/X/` importa `modules/Y/`** — verificação via `go list` + CI gate

```go
// ✅ Contexto A define seu enum
// modules/institutional/domain/enums/profile_type.go
type ProfileType string

// ✅ Contexto B define o seu, mesmo se conceitualmente similar
// modules/identity/domain/enums/user_role.go
type UserRole string

// ✅ Se precisar compartilhar: shared/types/
// shared/types/auth_user.go
type AuthUser struct {
    ID       string
    Role     string   // string primitivo, parse local em cada contexto
    PlanTier string
    // ...
}
```

---

## A3 — If/else hell em Update use cases

### O que aconteceu
`UpdateProfileUseCase` tinha switch de 5 cases (Resident, Syndic, Enterprise, LocalCompany, Marketing) — cada adição de tipo = editar classe gigante. Violação direta de OCP (SOLID).

```typescript
// ❌ 150 linhas de switch com lógica divergente por case
async execute(cmd: UpdateProfileCommand) {
  switch (cmd.profileType) {
    case 'resident':
      this.authorize(...);
      await this.residentRepo.update(...);
      await this.validateCPF(...);
      // ...
    case 'syndic':
      this.authorize(...);  // duplicado
      await this.syndicRepo.update(...);
      // ...
    // ... 5 cases
  }
}
```

### Fix obrigatório no Go: Strategy Pattern

```go
// ✅ Interface polimórfica
type ProfileUpdater interface {
    Update(ctx context.Context, cmd UpdateProfileCommand) (*ProfileDTO, error)
}

// ✅ Implementação por tipo — arquivo separado
// modules/institutional/application/profile_updaters/resident_profile_updater.go
type ResidentProfileUpdater struct {
    repo IResidentProfileRepository
    uow  contracts.IUnitOfWork
}

func (u *ResidentProfileUpdater) Update(ctx context.Context, cmd UpdateProfileCommand) (*ProfileDTO, error) {
    // lógica específica de morador
}

// modules/institutional/application/profile_updaters/syndic_profile_updater.go
// ... implementação independente

// ✅ Factory retorna implementação correta
func NewProfileUpdater(profileType ProfileType, deps Deps) (ProfileUpdater, error) {
    switch profileType {
    case ProfileTypeResident:  return NewResidentProfileUpdater(deps), nil
    case ProfileTypeSyndic:    return NewSyndicProfileUpdater(deps), nil
    // ...
    }
    return nil, apperrors.ErrValidation.WithMessage("unknown profile type")
}

// Use case fica ENXUTO
func (uc *UpdateProfileUseCase) Execute(ctx context.Context, cmd UpdateProfileCommand) (*ProfileDTO, error) {
    updater, err := NewProfileUpdater(cmd.ProfileType, uc.deps)
    if err != nil { return nil, err }
    return updater.Update(ctx, cmd)
}
```

Adicionar novo tipo = novo arquivo. **Zero** modificação no use case existente (OCP respeitado).

---

## A4 — Duplicação de `authorize()` em múltiplos métodos

### O que aconteceu
`this.authorize(userID, action)` repetido em 4 métodos do `UpdateProfileUseCase` — mesma lógica, com variações pequenas. Impossível manter consistência ao longo do tempo.

### Fix obrigatório no Go
Autorização **no middleware**, nunca no use case. Use case confia que quem chegou até ele já passou por ABAC.

```go
// ✅ Middleware RequirePermission no registro da rota
profileRoutes.POST("/update",
    middleware.RequirePermission("profile.update", "profile"),
    handler.UpdateProfile,
)

// Use case sem authorize()
func (uc *UpdateProfileUseCase) Execute(ctx context.Context, cmd UpdateProfileCommand) error {
    // A essa altura ABAC já validou — foco em regra de negócio
}
```

Exceções (autorização dentro do use case): quando depende de **dados do recurso** (ex: "só pode editar profile do próprio tenant") — aí o use case faz o check de ownership **após** carregar o recurso. Ainda assim, check único no começo, não replicado em 4 métodos.

---

## A5 — Ordem inconsistente do fluxo de autorização

### O que aconteceu
`updateResident` chamava `authorize()` **antes** de carregar o recurso; `updateSyndic` carregava e depois chamava. Comportamento imprevisível para o cliente (response varia por tipo).

### Fix obrigatório no Go: padrão rígido de 5 passos

```
1. FETCH         — carrega o recurso (erro = NotFound)
2. VALIDATE      — valida invariantes do recurso
3. AUTHORIZE     — confirma que este user pode agir neste recurso (ownership check)
4. MUTATE        — aplica mudança (método de domínio)
5. SAVE + EVENTS — persiste + emite domain events
```

Código-padrão documentado em `steering/sdd-workflow.md`. Violação em code review.

---

## A6 — `PermissionService` sem log de negação

### O que aconteceu
Quando ABAC negava, retornava `false` sem contexto. Usuário via "acesso negado" genérico. Suporte não conseguia debugar. Logs não tinham razão da negação.

### Fix obrigatório no Go

```go
type ABACResult struct {
    Granted bool
    Reason  string   // "role_not_allowed" | "plan_below_required" | "tenant_mismatch" | "resource_locked"
    Details map[string]any
}

// Sempre logar denial com contexto
if !result.Granted {
    logger.Info().
        Str("user_id", authUser.ID).
        Str("tenant_id", authUser.TenantID).
        Str("role", string(authUser.Role)).
        Str("plan_tier", string(authUser.PlanTier)).
        Str("action", action).
        Str("reason", result.Reason).
        Any("details", result.Details).
        Msg("access denied by ABAC")
    return apperrors.ErrForbidden.WithMessage("INSUFFICIENT_PERMISSION: " + result.Reason)
}
```

Response ao cliente: apenas categoria (`INSUFFICIENT_PERMISSION`), **sem revelar detalhes** internos da política.

---

## A7 — Falta de domain events (update não invalida cache)

### O que aconteceu
Quando `UpdateProfile` terminava, o cache Redis de dados do usuário **não era invalidado** — próximo request ainda via dados velhos por até 5 minutos. Bugs intermitentes difíceis de reproduzir.

### Fix obrigatório no Go
- `IDomainEventPublisher` em `core/contracts/` **desde o Sprint 2** (não esperar problema aparecer)
- Toda mutação relevante emite evento: `ProfileUpdated`, `SubscriptionActivated`, `TimelineEntryPublished`
- Listeners invalidam cache, notificam sistemas externos, reindexam busca, **idempotent** para aceitar retry
- Publicação dentro da mesma tx via outbox pattern (tabela `domain_events` commit-ada junto)

```go
// Use case
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    // ... update
    return uc.eventBus.Publish(ctx, ProfileUpdated{
        ProfileID: profile.ID(),
        UserID:    userID,
        ChangedAt: time.Now(),
    })
})

// Listener idempotent
func (l *CacheInvalidationListener) Handle(ctx context.Context, e ProfileUpdated) error {
    return l.cache.Delete(ctx, fmt.Sprintf("profile:%s", e.ProfileID))  // idempotent
}
```

Ver `steering/transaction-patterns.md` seção "Domain Events + Eventual Consistency".

---

## A8 — Race condition em quotas

### O que aconteceu
`incrementUsage()` fazia `read → modify → write` sem lock. Dois requests concorrentes do Connect Me passavam ambos por `if usage < limit` e ambos incrementavam, excedendo a quota. Usuários reportaram conseguir enviar N+1 requests.

### Fix obrigatório no Go

```go
// ✅ Pessimistic locking via SELECT FOR UPDATE dentro de transação
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    quota, err := uc.quotaRepository.FindByUserIDForUpdate(ctx, userID, feature, period)
    if err != nil { return err }

    if !quota.HasRemaining() {
        return apperrors.ErrBusiness.WithMessage("quota esgotada")
    }

    quota.Consume()
    return uc.quotaRepository.Save(ctx, quota)
})
```

Query sqlc:
```sql
-- name: FindQuotaByUserIDForUpdate :one
SELECT * FROM feature_usages
WHERE user_id = $1 AND feature_key = $2 AND period = $3
FOR UPDATE;
```

Alternativa: **advisory lock** do Postgres se não há linha dedicada (raro).

**Nunca**: atomic increment no Redis sem source of truth no Postgres — cache perdido = quota perdida = consistência inconsistente.

---

## A9 — Validação apenas na aplicação (DB aceitava lixo)

### O que aconteceu
Aplicação validava `email` formato — mas DB aceitava `''` vazio ou `NULL`. Job de sync gravou diretamente sem passar pela validação. Uma semana depois, 30 usuários com email inválido no banco, bug em relatórios, desalinhamento com Zitadel.

### Fix obrigatório no Go + PostgreSQL

**Validação em 3 camadas, redundante por design**:

1. **Handler** (validator.Struct + DTO validation) — primeira barreira
2. **Value Object** (auto-validado na factory) — invariante do domínio
3. **Postgres CHECK constraint** — última linha de defesa

```sql
-- ✅ CHECK constraint na migration
CREATE TABLE users (
    id UUID PRIMARY KEY,
    email TEXT NOT NULL CHECK (email ~ '^[^@\s]+@[^@\s]+\.[^@\s]+$'),
    status user_status NOT NULL DEFAULT 'pending_verification',
    -- ...
    CHECK (status IN ('pending_verification', 'active', 'suspended', 'deleted'))
);
```

Regra em `steering/database.md`: **toda migration com enum tem CHECK constraint**; **toda coluna crítica tem NOT NULL + CHECK** quando aplicável.

---

## A10 — Logging inconsistente

### O que aconteceu
Uns logs tinham `user_id`, outros só `request_id`, outros nada. Debug em produção era arqueologia — precisava de 3 ferramentas pra correlacionar.

### Fix obrigatório no Go
Middleware `Logger` injeta contexto estruturado. Todos os logs de uma request herdam:

```go
// shared/middleware/logger.go
logger := zerolog.New(output).With().
    Str("request_id", requestID).
    Str("method", c.Request.Method).
    Str("path", c.Request.URL.Path).
    Logger()

// Após Authenticate middleware:
if authUser := ctx.AuthUser(); authUser != nil {
    logger = logger.With().
        Str("user_id", authUser.ID).
        Str("tenant_id", authUser.TenantID).
        Str("role", string(authUser.Role)).
        Logger()
}

ctx = logger.WithContext(ctx)
// Daqui pra frente: zerolog.Ctx(ctx).Info()... herda tudo
```

Regra em `steering/security.md`: log obrigatório por camada (handler, use case, repository) com campos `user_id`, `tenant_id`, `request_id` quando disponíveis.

---

## A11 — Drizzle v1-beta instável

### O que aconteceu
ORM em beta quebrou múltiplas vezes durante desenvolvimento — breaking changes entre releases, schema sync travava, erros obscuros em SQL gerado. Time perdia horas por semana debugando tooling em vez de implementar.

### Fix — já aplicado no Go
- **sqlc v1.30** (estável, deterministic, code gen) — queries são texto SQL puro, gera Go tipado
- **goose v3** (estável) — migrations versionadas
- **pgx/v5** (maduro, alta performance) — driver nativo Go

Ver `steering/database.md`.

---

## A12 — Mistura User / Profile / Subscription

### O que aconteceu
No legacy, `User` continha campos de profile (cpf, birth_date) e de billing (plan, subscription_status). Single table de ~40 colunas. Cada feature nova adicionava campo. Entidade virou God Object.

### Fix — já aplicado no Go
**Separação rígida por bounded context**:

- `identity.User` — auth, email, role, status (dados vindos do Zitadel + BeyondCorp tracking)
- `institutional.Profile*` — CPF, CNPJ, dados profissionais, governance markers (dados de domínio)
- `billing.Subscription` — Stripe IDs, status, period, trial_ends_at (dados comerciais)

Relação entre agregados via **ID**, não via referência direta:

```go
type Subscription struct {
    id     string
    userID string    // ← referência por ID, não por ponteiro
    planID string
    // ...
}
```

Ver `steering/code-calisthenics.md` seção "Mantenha entidades pequenas" + regra #7.

---

## Resumo executivo

| # | Armadilha | Causa raiz | Fix Go |
|---|---|---|---|
| A1 | ABAC desativado | Sem startup check + sem teste de integração | `config.Validate()` falha; teste `TestABAC_FailsStartupIfEmpty` |
| A2 | Vazamento de domínio | Import direto entre bounded contexts | Cada BC tem seus tipos; cruzar via `shared/types/` ou eventos |
| A3 | If/else hell em Update | Switch de N cases em use case | Strategy pattern: interface + impl por arquivo |
| A4 | `authorize()` duplicado | Autorização dentro de use case | `RequirePermission` middleware na rota |
| A5 | Ordem inconsistente de fluxo | Sem padrão | `Fetch → Validate → Authorize → Mutate → Save` rígido |
| A6 | Denial sem contexto | `bool` de retorno | `ABACResult{Granted, Reason, Details}` + log estruturado |
| A7 | Update não invalida cache | Sem domain events | `IDomainEventPublisher` + outbox + listeners idempotent |
| A8 | Race em quota | `read→modify→write` sem lock | `SELECT ... FOR UPDATE` em tx |
| A9 | Validação só na app | Sem CHECK no DB | 3 camadas: handler + VO + CHECK constraint |
| A10 | Log inconsistente | Sem contexto estruturado | Middleware Logger injeta `user_id`, `tenant_id`, `request_id` em todo ctx |
| A11 | Drizzle beta | Tooling instável | pgx/v5 + sqlc v1.30 + goose v3 estáveis |
| A12 | User God Object | Single table com tudo | Separação rígida por BC, relação por ID |

Code review e `/go-review` skill **checam** cada um desses itens. Violação = refactor antes de merge.

## Referências
- Full-stack legacy: `../../../full-stack/contextos/ANALISE_COMPLETA.md`
- Plano original de refatoração: `../../../full-stack/plans/architecture-refined.md`
- Steering correspondente: `.kiro/steering/go-patterns.md`, `.kiro/steering/code-calisthenics.md`, `.kiro/steering/transaction-patterns.md`, `.kiro/steering/security.md`
