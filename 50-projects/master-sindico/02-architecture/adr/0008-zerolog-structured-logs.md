---
title: ADR-0008 — zerolog para logs estruturados
type: adr
tags: [adr, observability, logs, zerolog, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0008 — zerolog para logs estruturados

## Status

accepted — 2026-04-23 (herdado de legado)

## Contexto

Logs canônicos precisam: zero-alloc em hot path, JSON estruturado, níveis, context propagation (tenant_id/request_id/trace_id), PII masking enforcement.

## Decisão

**rs/zerolog** como lib de logging. Wrapper em `pkg/logger/` exporta `Logger` com métodos tenant-aware e PII scrub. Output JSON em stdout capturado por Loki.

```go
// pkg/logger/logger.go
type Logger interface {
    With(ctx context.Context) Logger    // injeta request_id, tenant_id, trace_id
    Debug() *zerolog.Event
    Info() *zerolog.Event
    Warn() *zerolog.Event
    Error() *zerolog.Event
}

// uso
log.With(ctx).Info().
    Str("endpoint", "POST /api/v1/assemblies/:id/vote").
    Int("status", 201).
    Dur("duration", dur).
    Msg("vote cast")
```

### Regras

1. **Formato JSON em prod, console colored em dev**.
2. **Zero PII** — CPF/CNPJ/email/token masked via `Masked()`; linter CI bloqueia.
3. **Níveis**: debug (dev), info (default), warn (comportamento anormal recuperável), error (falha que requer atenção).
4. **Contexto propagado** via middleware — toda hora que cruzar camada (handler → use case → repo), `log.With(ctx)` rehidrata campos.

## Consequências

### Positivas

- Zero-alloc em hot path (benchmark oficial zerolog 7× mais rápido que logrus).
- JSON structured direto pro Loki sem parser.
- API fluente e legível.

### Negativas

- Não é `log/slog` stdlib 1.21+ — mas quando estabilizar, port é direto (API similar).
- Opção global `zerolog.TimeFieldFormat` é package-level (cuidado em testes).

## Alternativas consideradas

1. **logrus** — mais popular mas significativamente mais lento.
2. **zap (Uber)** — performance similar; API menos ergonômica.
3. **log/slog (stdlib 1.21+)** — candidato futuro; em 2026-04 ainda menos maduro que zerolog para este use case.
4. **go-kit/log** — bom para alinhar com go-kit; overkill.

## Referências

- [[../observability]] §2.1
- [[../../CLAUDE]] §3.10 (no-PII-logs)
- Linter CI: `12-docs/ci-rules/pii-masking.md` (a escrever)
