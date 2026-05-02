---
title: OP-026 — Endpoint de lista sem paginação default
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - performance
  - api-design
id: OP-026
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Unbounded list endpoint
  - No default pagination
---

# OP-026 — Endpoint de lista sem paginação default

**Severidade**: Medium (sobe conforme tabela cresce)
**Categoria**: Performance / API Design

## Sintoma

`GET /orders` sem `limit` default retorna **tudo** quando cliente não envia parâmetro. Em dev (50 rows) funciona; em produção (500k rows) derruba DB + cliente + rede. Mesmo pattern: `GET /users`, `GET /tenants/:id/members`.

## Por que é problema

- **OOM cliente/servidor**: serialização de 500k rows estoura memória.
- **Latência linear**: cliente espera N segundos; requests upstream (reverse proxy, load balancer) timeout.
- **DB sofre**: full scan + serialização saturam CPU/IO.
- **Custo de rede**: payload de MB quando bastavam KB.

## Exemplo (anti-pattern)

```go
// ❌ ERRADO — sem default, cliente pode omitir
func (h *OrderHandler) List(ctx gin.Context) {
    limit := parseInt(ctx.Query("limit"), 0)      // 0 = sem limite
    offset := parseInt(ctx.Query("offset"), 0)
    orders, _ := h.repo.List(ctx, limit, offset)
    ctx.JSON(200, orders)
}
```

## Fix preferido — default + teto máximo + paginação estável

```go
const (
    DefaultPageSize = 20
    MaxPageSize     = 100
)

func (h *OrderHandler) List(ctx gin.Context) {
    limit := parseIntDefault(ctx.Query("limit"), DefaultPageSize)
    if limit > MaxPageSize { limit = MaxPageSize }
    if limit < 1           { limit = DefaultPageSize }

    cursor := ctx.Query("cursor")   // cursor-based preferido a offset em prod
    orders, nextCursor, err := h.repo.ListByCursor(ctx, cursor, limit)
    if err != nil { ctx.SetError(err); return }

    ctx.JSON(200, map[string]any{
        "data":        orders,
        "next_cursor": nextCursor,
        "limit":       limit,
    })
}
```

## Paginação: cursor vs offset

| Aspecto | Offset | Cursor (keyset) |
|---|---|---|
| Sintaxe | `LIMIT 20 OFFSET 400` | `WHERE id > $cursor ORDER BY id LIMIT 20` |
| Performance em tabela grande | Degrada linearmente | Constante (se índice certo) |
| Deep pages (>10k) | Patológico | Ok |
| Resultado consistente em mutation | Sujeito a skip/duplicate | Estável |
| Facilidade de uso | Simples | Exige cursor opaco |

**Regra**: cursor em produção; offset em dashboards admin com baixa concorrência e dataset pequeno.

## Complementos

- **Total count separado**: `HEAD /orders` ou campo `?include=total` explícito; count é caro, não rodar em toda page.
- **Expand/sparse fields**: `?fields=id,status` reduz payload.
- **Rate limit proporcional**: endpoint de lista cara merece limit menor.

## Alternativa

- **Streaming** (Server-Sent Events, NDJSON) quando cliente precisa de volume genuíno (export CSV) — ainda com chunking.
- **Pagination HATEOAS** com links `next`/`prev` no response.

## Quando tolerar

Endpoint interno que retorna lista **garantidamente pequena** (`WHERE id IN (<10 UUIDs>)`) — default é irrelevante. Mesmo aí, teto previne bug futuro (lista cresce).

## Relações

- **Patterns**: Pagination (Offset, Cursor), CQRS (read model)
- **OPs relacionados**: [[op-025-coluna-sem-indice]] (paginação eficiente exige índice), [[op-012-fan-out-desnecessario]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-026
- Use The Index, Luke! — [Paging Through Results](https://use-the-index-luke.com/sql/partial-results/fetch-next-page) (consultada 2026-04-24)
- GitHub API — [Pagination using cursors](https://docs.github.com/en/rest/using-the-rest-api/using-pagination-in-the-rest-api) (consultada 2026-04-24)
- Slack API — [Cursor-based pagination](https://api.slack.com/docs/pagination) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[op-025-coluna-sem-indice]]
- [[../database/database-patterns]]
- [[../patterns/cqrs]]
