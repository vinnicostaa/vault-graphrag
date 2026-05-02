---
inclusion: always
---

# Transaction Patterns — ACID e Saga

Duas ferramentas para consistência. **ACID local** (UnitOfWork + pgx tx) resolve 80% dos casos. **Saga** resolve os 20% que cruzam bounded contexts e providers externos. Usar a ferramenta errada é a origem de race conditions, cobranças duplicadas, estado inconsistente.

## Quando usar cada um (regra de ouro)

| Cenário | Padrão | Motivo |
|---|---|---|
| Operação atinge **1 banco** (Postgres local) | **UnitOfWork / ACID** | pgx tx garante atomicidade real |
| Operação cruza **banco + Redis** | UnitOfWork + eventual consistency | Redis é cache, não source of truth |
| Operação cruza **bounded contexts no mesmo DB** | UnitOfWork se dentro de uma transação curta; **Saga** se houver chamada externa ou operação longa |
| Operação cruza **banco + provider externo** (Stripe, Mux, SES) | **Saga** | Provider externo não participa de pgx tx |
| Operação **longa** (> 5s) com múltiplos passos | **Saga** | Manter tx aberta longa causa lock + vacuum issues |
| Operação **eventual** (notificação, email, push) | Domain event + handler async | Não precisa de atomicidade; falha não rollback |

## ACID via UnitOfWork (pattern padrão)

Todas as operações que mexem em mais de uma tabela dentro do mesmo banco usam `IUnitOfWork.Run()`. Já documentado em `steering/database.md` — resumo aqui para completude.

### Propriedades garantidas

- **A**tomicity: callback retorna erro → rollback automático
- **C**onsistency: CHECK constraints + FKs + domain invariants (em código)
- **I**solation: `pgx.TxOptions` — default `ReadCommitted` (suficiente para 95% dos casos)
- **D**urability: Postgres fsync + WAL

### Exemplo canônico

```go
func (uc *CreateSubscriptionUseCase) Execute(ctx context.Context, cmd CreateSubscriptionCommand) (*SubscriptionDTO, error) {
    var output *SubscriptionDTO

    err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
        // Passo 1: idempotency check
        existing, err := uc.subscriptionRepository.FindActiveByUserID(ctx, cmd.UserID)
        if err != nil && !errors.Is(err, apperrors.ErrNotFound) {
            return fmt.Errorf("FindActiveByUserID: %w", err)
        }
        if existing != nil {
            return apperrors.ErrConflict.WithMessage("user já tem subscription ativa")
        }

        // Passo 2: criar entidade
        subscription := entities.NewSubscription(cmd.UserID, cmd.PlanID, trialDays)

        // Passo 3: persistir
        if err := uc.subscriptionRepository.Save(ctx, subscription); err != nil {
            return fmt.Errorf("Save: %w", err)
        }

        // Passo 4: audit trail (append-only, mesma tx)
        if err := uc.auditRepository.Log(ctx, auditEntry); err != nil {
            return fmt.Errorf("audit Log: %w", err)
        }

        output = mappers.ToSubscriptionDTO(subscription)
        return nil
    })

    if err != nil {
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    return output, nil
}
```

Se qualquer passo falha: **toda a tx é revertida**. Estado do banco nunca fica inconsistente.

### Limitações do UnitOfWork

**Não cobre**:
- Chamadas a providers externos (Stripe, Mux, SES, Twilio, FCM) — eles estão fora do banco
- Operações que cruzam bancos (hoje um só, mas pode mudar)
- Operações longas (uma tx aberta por 30s bloqueia vacuum, locks, etc)
- Operações entre bounded contexts quando um deles dispara domain event para outro sistema

Para esses casos: **Saga**.

## Saga Pattern

Saga decompõe uma transação de negócio em **passos locais**, cada um atômico, com **compensação** definida para cada falha.

### Dois estilos

1. **Choreography** — cada passo emite evento; o próximo passo é listener do evento. Acoplamento mínimo, difícil de debugar.
2. **Orchestration** — um **orchestrator** (use case ou dedicated service) chama cada passo e coordena compensação. Centralizado, mais claro, preferido para operações críticas (billing, checkout).

**Master Síndico usa orchestration por default.** Choreography apenas para reactive flows (ex: `SubscriptionActivated` → invalida cache + envia email de boas-vindas — ambos idempotentes, não precisam de compensação).

### Anatomia de uma Saga orquestrada

```go
// Exemplo: checkout completo — criar customer Stripe + subscription + atualizar plan_tier + enviar email

type SubscriptionCheckoutSaga struct {
    paymentGateway        IPaymentGateway
    subscriptionRepo      ISubscriptionRepository
    userRepo              IUserRepository
    emailProvider         IEmailProvider
    unitOfWork            IUnitOfWork
    logger                *zerolog.Logger
}

type checkoutState struct {
    stripeCustomerID     string
    stripeSubscriptionID string
    subscriptionID       string
    emailSent            bool
}

func (s *SubscriptionCheckoutSaga) Execute(ctx context.Context, cmd CheckoutCommand) (*SubscriptionDTO, error) {
    state := &checkoutState{}

    // Passo 1: criar customer no Stripe (idempotent via idempotency key)
    customer, err := s.paymentGateway.CreateCustomer(ctx, CreateCustomerInput{
        UserID: cmd.UserID,
        Email:  cmd.Email,
        IdempotencyKey: fmt.Sprintf("customer:%s", cmd.UserID),
    })
    if err != nil {
        return nil, fmt.Errorf("CreateCustomer: %w", err)
    }
    state.stripeCustomerID = customer.ID

    // Passo 2: criar subscription no Stripe
    stripeSub, err := s.paymentGateway.CreateSubscription(ctx, CreateSubscriptionInput{
        CustomerID:     customer.ID,
        PriceID:        cmd.PriceID,
        IdempotencyKey: fmt.Sprintf("subscription:%s:%s", cmd.UserID, cmd.PriceID),
    })
    if err != nil {
        // Compensação: se CreateCustomer era novo, podemos deletá-lo; se existia, não tocar
        // Aqui: Stripe customer é idempotent, podemos deixar — não causa problema
        return nil, fmt.Errorf("CreateSubscription: %w", err)
    }
    state.stripeSubscriptionID = stripeSub.ID

    // Passo 3: persistir localmente em transação atômica (ACID local)
    err = s.unitOfWork.Run(ctx, func(ctx context.Context) error {
        sub := entities.NewSubscription(cmd.UserID, cmd.PlanID, stripeSub.TrialEnd)
        sub.LinkStripe(state.stripeCustomerID, state.stripeSubscriptionID)

        if err := s.subscriptionRepo.Save(ctx, sub); err != nil {
            return fmt.Errorf("Save subscription: %w", err)
        }

        // Atualiza plan_tier do user (cache ABAC)
        if err := s.userRepo.UpdatePlanTier(ctx, cmd.UserID, cmd.PlanTier); err != nil {
            return fmt.Errorf("UpdatePlanTier: %w", err)
        }

        state.subscriptionID = sub.ID()
        return nil
    })
    if err != nil {
        // Compensação: cancelar subscription no Stripe que acabamos de criar
        if cancelErr := s.paymentGateway.CancelSubscription(ctx, state.stripeSubscriptionID); cancelErr != nil {
            s.logger.Error().
                Err(cancelErr).
                Str("stripe_subscription_id", state.stripeSubscriptionID).
                Msg("compensação falhou — subscription órfã no Stripe; reconciliar manualmente")
        }
        return nil, fmt.Errorf("persistir subscription: %w", err)
    }

    // Passo 4: enviar email de boas-vindas (fire-and-forget se falha)
    if err := s.emailProvider.SendWelcome(ctx, cmd.UserID, cmd.Email); err != nil {
        // NÃO compensa os passos 1-3 — email é nice-to-have, não bloqueador
        s.logger.Warn().Err(err).Str("user_id", cmd.UserID).Msg("email de boas-vindas falhou")
    }

    return &SubscriptionDTO{ID: state.subscriptionID}, nil
}
```

### Princípios de Saga bem-desenhada

1. **Cada passo é idempotent**: retry com mesma input produz mesmo resultado. Stripe: `idempotency_key`. Banco: `INSERT ... ON CONFLICT DO UPDATE`. Email: enviar com `message_id` determinístico.

2. **Compensação é explícita**: para cada passo que pode ser desfeito, definir a operação inversa. `CreateSubscription` → `CancelSubscription`. `ChargeCard` → `Refund`. Alguns passos **não têm** compensação (email já enviado) — nesses, aceitar e documentar.

3. **Ordem importa**: passos que **não podem ser desfeitos** vêm **depois** de passos que podem. Email de boas-vindas é último (se falha, subscription existe; se email falha depois de sucesso, ok).

4. **Log tudo**: cada passo gera log estruturado com `saga_id` (correlação). Compensação falhada é alerta crítico — há estado órfão em sistema externo.

5. **Estado intermediário persistido** (em Sagas longas): salvar cada passo concluído em tabela `saga_state` com status. Permite retomar após crash.

### Exemplo de Saga simples sem compensação explícita (reactive)

Quando todos os passos são **naturalmente idempotent** e **recuperáveis por retry**, Saga pode ser choreography via domain events:

```
1. User.UpdatePlan -> emite SubscriptionActivated event
2. Listener A (audit): registra event em audit_trail (idempotent — chave = event.ID)
3. Listener B (cache): invalida cache Redis (idempotent — delete é idempotent)
4. Listener C (email): envia welcome email (idempotent — message_id baseado em event.ID)
```

Se listener B falha: retry até sucesso (Redis não é source of truth; cache miss não bloqueia). Sem compensação necessária.

## Quando escolher Saga vs UnitOfWork

Decisão rápida:

```
Operação toca apenas Postgres?
├── Sim → UnitOfWork (ACID local)
└── Não → toca provider externo / outro sistema?
    ├── Sim → Saga orchestrated (com compensação)
    └── Não (toca Redis/cache) → UnitOfWork + eventual consistency (Redis é cache)

Operação é longa (> 5s esperados)?
├── Sim → Saga com state persistido (permite retomada)
└── Não → UnitOfWork

Falha no meio pode deixar estado inconsistente em sistema externo?
├── Sim → Saga com compensação explícita + alerta em falha de compensação
└── Não → UnitOfWork
```

## Domain Events + Eventual Consistency

Para operações que **não precisam** de consistência forte, emitir domain event e deixar handler processar async.

### Quando

- Notificações (email, push, SMS)
- Invalidação de cache
- Sync para sistema de busca (OpenSearch)
- Atualização de métricas / agregados não-críticos
- Webhooks de saída (notificar sistema parceiro)

### Princípios

- **Handler é idempotent** — event pode chegar 2× (retry, restart)
- **Falha de handler não bloqueia fluxo principal** — log + retry com backoff
- **Event ID é único e estável** — idempotency key natural

## Isolation Levels (Postgres)

`pgx.TxOptions{IsoLevel: pgx.XX}`:

| Level | Quando |
|---|---|
| `ReadCommitted` (default) | 95% dos casos. Sem dirty reads. |
| `RepeatableRead` | Relatórios longos onde **não pode** ter non-repeatable read (ex: relatório de assembly). |
| `Serializable` | Raro. Operações onde **anomalias de write skew** são inaceitáveis (ex: quotas concorrentes). |

Preferência: `ReadCommitted` + **lock explícito** onde preciso (`SELECT ... FOR UPDATE`) em vez de subir isolation level — menor contenção.

## Locks Explícitos

Quando dois operadores podem tentar ação concorrente no mesmo recurso:

```go
err := uc.unitOfWork.Run(ctx, func(ctx context.Context) error {
    // SELECT ... FOR UPDATE bloqueia linha para concorrência
    quota, err := uc.quotaRepository.FindByUserIDForUpdate(ctx, cmd.UserID)
    if err != nil {
        return err
    }

    if !quota.HasRemaining() {
        return apperrors.ErrBusiness.WithMessage("quota esgotada")
    }

    quota.Consume()
    return uc.quotaRepository.Save(ctx, quota)
})
```

Query sqlc correspondente:

```sql
-- name: FindQuotaByUserIDForUpdate :one
SELECT * FROM feature_usages
WHERE user_id = $1 AND feature_key = $2 AND period = $3
FOR UPDATE;
```

**Regra**: só usar `FOR UPDATE` em transação. Fora de tx, lock é liberado imediatamente (sem efeito).

### Advisory Locks (para casos sem linha específica)

Quando não há linha para lockar mas precisa de mutex lógico (ex: "apenas uma instância roda esse job por vez"):

```sql
-- Pega lock exclusivo com chave numérica
SELECT pg_try_advisory_lock(12345);

-- No fim:
SELECT pg_advisory_unlock(12345);
```

Útil para migrations (goose já faz), jobs singleton, inicialização.

## Exemplos reais do Master Síndico

### Caso 1 — UnitOfWork ✅
- **1.0.12 UpsertUser** (sync Zitadel → DB): única tx, INSERT ... ON CONFLICT + audit log. UnitOfWork.

### Caso 2 — Saga ✅
- **1.3/1.4 CheckoutSubscription** (criar Stripe customer + subscription + atualizar plan_tier + email): Saga orchestrated com compensação. Exemplo completo acima.

### Caso 3 — Domain event (eventual) ✅
- **Timeline published** → invalida cache do PD + envia push para moradores: domain event `TimelineEntryPublished`, handlers async, idempotent.

### Caso 4 — Lock explícito ✅
- **4.1 Connect Me** (verifica quota + incrementa atomicamente): `SELECT ... FOR UPDATE` dentro de UnitOfWork. Evita race condition que o legacy tinha.

## Referências

- Chris Richardson. *Microservices Patterns* (2018) — capítulo 4 Saga
- Clemens Vasters. [Sagas (Microsoft blog)](https://learn.microsoft.com/en-us/azure/architecture/reference-architectures/saga/saga)
- Postgres: [Transaction Isolation Docs](https://www.postgresql.org/docs/current/transaction-iso.html)
- pgx: `BeginTxFunc` com `pgx.TxOptions`
