---
title: OP-004 — Operação crítica sem retry strategy
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - idempotency
  - resilience
id: OP-004
severity: High
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - No retry strategy
  - Retry eterno
doc-consulted: '2026-04-24'
---

# OP-004 — Operação crítica sem retry strategy

**Severidade**: High
**Categoria**: Idempotência / Resiliência

## Sintoma

Operação crítica falha e o código segue um dos dois caminhos ruins:

1. **Throw e deixa o caller ver** → worker/provider retenta eternamente (loop), queue estoura, ou usuário vê 500 em série.
2. **Swallow** (`catch {} `ou `_ = fn()`) → evento some, compensação não roda, estado fica inconsistente.

Falta classificação retentável vs não-retentável + política de desistência.

## Por que é problema

- **Retry eterno**: custo (N× reprocessamento) + barulho em logs + dependências sofrem cascata.
- **Swallow silencioso**: estado divergente, ninguém sabe que falhou, debug em produção só via cliente reclamando.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — throw tudo, provider retenta eternamente
func (h *WebhookHandler) Handle(ctx context.Context, event Event) error {
    if err := h.processor.Process(ctx, event); err != nil {
        return err  // provider vê 500 → retenta 3 dias
    }
    return nil
}
```

## Fix preferido — classificar + backoff + DLQ

```go
type ErrorKind int
const (
    ErrKindRetryable   ErrorKind = iota  // timeout, DB down, conexão
    ErrKindPermanent                     // validação, business rule, schema
)

func (h *WebhookHandler) Handle(ctx context.Context, event Event) error {
    err := h.processor.Process(ctx, event)
    if err == nil { return nil }

    kind := classifyError(err)
    if kind == ErrKindPermanent {
        h.dlq.Send(ctx, event, err)   // parar retry
        h.logger.Error("permanent failure", "event_id", event.ID, "err", err)
        return nil  // provider para de retentar (retorna 200)
    }
    return err  // retentável — provider retenta com backoff próprio
}
```

Para worker interno (não webhook), política **exponential backoff com teto**:

```
Tentativa 1: imediato
Tentativa 2: 1s
Tentativa 3: 5s
Tentativa 4: 30s
Tentativa 5: 300s
Após N tentativas → DLQ + alerta oncall
```

## Alternativa

- Circuit breaker (Hystrix-style) quando dependência externa mostra instabilidade prolongada.
- Bulkhead — isolar pools de thread/conexão por dependência pra falha não cascatear.

## Quando tolerar

Operação idempotente e barata que não afeta estado (ex: refresh cache periódico — se falhou, próximo ciclo cobre). Explicitar que é *best-effort* + observar métricas.

## Relações

- **Patterns**: [[../patterns/transaction-patterns#saga]] (compensação quando retry falha), circuit breaker, bulkhead
- **OPs relacionados**: [[op-003-webhook-sem-idempotencia]] (retry seguro exige idempotência), [[op-020-erro-engolido]], [[op-006-cache-falha-quebra-fluxo]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-004
- AWS — [Error retries and exponential backoff](https://docs.aws.amazon.com/general/latest/gr/api-retries.html)
- Michael Nygard — *Release It!* (stability patterns)

## Links

- [[_moc]]
- [[../patterns/transaction-patterns#saga]]
- [[../security/security-principles]]
- [[../observability/alerting]]
