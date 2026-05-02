---
title: OP-020 — Erro engolido silenciosamente
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - observability
  - error-handling
id: OP-020
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Swallowed error
  - Silent catch
---

# OP-020 — Erro engolido silenciosamente

**Severidade**: High (debug impossível em prod)
**Categoria**: Observabilidade

## Sintoma

`_ = fn()`, `catch {}` vazio, `return []` quando dep falhou, `err ?? []`. Erro acontece, sistema segue como se nada fosse; usuário vê "nada encontrado" em vez de erro claro; logs não registram. Debug em produção vira adivinhação.

## Por que é problema

- **Invisibilidade**: não há sinal de que algo deu errado; só reclamação do cliente.
- **Propagação de estado corrompido**: vazio silencioso pode ser interpretado como "não existe" por consumers → deleções, contadores zerados.
- **Postmortem impossível**: sem log do erro original, causa raiz é especulativa.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — erro descartado
items, _ := repo.ListAll(ctx)
if len(items) == 0 {
    return 0   // pode ser lista vazia genuína OU falha de DB — indistinguível
}
```

```ts
// ❌ ERRADO — catch vazio
try {
  await sendEmail(user)
} catch {}
```

## Fix preferido — propagar erro tipado; se mesmo assim decide seguir, logar com contexto

```go
items, err := repo.ListAll(ctx)
if err != nil {
    return 0, fmt.Errorf("listAll falhou: %w", err)
}
if len(items) == 0 {
    return 0, nil   // agora sabemos: lista vazia genuína
}
```

Quando o design **precisa** seguir apesar do erro (fail-open), logar com severidade apropriada:

```go
if err := s.emailer.Send(ctx, email); err != nil {
    s.logger.Error("email falhou (não crítico, seguindo)",
        "to", email.ToMasked(),
        "err", err,
        "request_id", ctx.RequestID())
    s.metrics.Inc("email.send.failed")
}
```

## Alternativa

- **Result/Either**: força o caller a tratar explicitamente.
- **Errors as values** (Go, Rust): erro é dado, não exceção; impossível "ignorar acidentalmente" quando linter está em posição adequada.
- **Sentinel errors** com `errors.Is/errors.As` — caller distingue tipos.

## Quando tolerar

Nunca ignorar sem log. Ignorar + logar (fail-open) é trade-off aceitável em fluxo de auxílio (ex.: telemetria best-effort). Explicitar razão no comentário + cobrir com métrica.

## Relações

- **Patterns**: Railway-Oriented Programming, Either/Result types
- **OPs relacionados**: [[op-021-admin-sem-audit]] (mesma classe — invisibilidade), [[op-022-log-com-pii]] (cuidado ao logar)
- **Constituição**: CLAUDE §8.12 (No try/catch defensivo)

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-020, §3.13 Error Handling
- Go Proverbs — [Errors are values](https://go.dev/blog/errors-are-values) (consultada 2026-04-24)
- Rust — [Error handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../principles/clean-code]]
- [[../observability/structured-logging]]
- [[../principles/do-dont-checklist]]
