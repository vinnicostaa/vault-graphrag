---
title: ADR-0002 — Gin abstraído via contracts.HTTPRouter
type: adr
tags: [adr, architecture, http, gin, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0002 — Gin abstraído via contracts.HTTPRouter

## Status

accepted — 2026-04-23

## Contexto

Gin é o router HTTP de M1 (performance, ergonomia, maturidade). Mas:

- Handlers testáveis devem independer de `*gin.Context`.
- Futuro reavaliar (Fiber / Echo / net/http 1.22+) sem reescrever handlers.
- Middleware chain genérica sem acoplar ao Gin.

## Decisão

Abstrair Gin via interface `contracts.HTTPRouter` + adapter `GinAdapter`. Handlers recebem `contracts.Context` que wrapa `*gin.Context`. Middleware implementa `contracts.MiddlewareFunc`.

```go
// internal/core/contracts/http_router.go
type HTTPRouter interface {
    Handle(method, path string, h HandlerFunc, mws ...MiddlewareFunc)
    Group(prefix string) HTTPRouter
    Run(addr string) error
}

type Context interface {
    Request() *http.Request
    ResponseWriter() http.ResponseWriter
    BindJSON(dst any) error
    JSON(status int, body any)
    Param(name string) string
    Query(name string) string
    User() *UserClaims
    TenantID() ULID
}

type HandlerFunc func(Context) error
type MiddlewareFunc func(HandlerFunc) HandlerFunc
```

## Consequências

### Positivas

- Handlers testáveis via fake `Context` (sem HTTP servidor em unit tests).
- Troca de router sem mexer em 100+ handlers.
- Middleware composta genericamente.

### Negativas

- Indireção extra (overhead ~zero, mas cognitive).
- Alguns recursos Gin avançados (streaming, SSE) podem exigir escape hatch na interface.

## Alternativas consideradas

- **Usar `*gin.Context` direto** — rápido, mas locks-in.
- **net/http 1.22+ stdlib router** — ainda falta ergonomia de Gin (middleware, params).
- **Chi** — boa alternativa minimalista; mas equipe já fluente em Gin.

## Referências

- [[../clean-arch-mapping]] §4 (Infrastructure → http)
- [[../c4-components]] §2
- Código legado: [[../../90-ingested/from-vault-50-projects-master-sindico/02-architecture/system-overview]]
