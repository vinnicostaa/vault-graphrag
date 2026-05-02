---
title: Railway — Patterns
type: note
tags: [provider, railway, patterns, deploy, graceful-shutdown]
source: https://docs.railway.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [Railway Patterns]
---

# Railway — Patterns

Padrões atemporais para operar aplicações no Railway de forma confiável.

## 1. Healthcheck obrigatório

Railway usa healthcheck HTTP para decidir promover/revertir deployment. Endpoint `/health` retornando 200 em < 5s.

```go
// Go — minimal healthcheck
mux.HandleFunc("/health", func(w http.ResponseWriter, r *http.Request) {
    if err := db.PingContext(r.Context()); err != nil {
        http.Error(w, "db down", http.StatusServiceUnavailable)
        return
    }
    w.WriteHeader(http.StatusOK)
})
```

Recomendações:
- **Liveness**: apenas "processo vivo" (sempre 200).
- **Readiness**: checa dependências (DB, cache). Pode retornar 503 para Railway não rotear tráfego.

Configurável em Settings → Deploy → Healthcheck Path + Timeout.

## 2. Graceful shutdown

Railway envia **SIGTERM 10s antes** do kill forçado. App deve:

1. Parar de aceitar novas conexões.
2. Drenar in-flight requests (timeout configurável).
3. Fechar pools (DB, Redis).
4. Exit 0.

```go
// Go — padrão via signal.NotifyContext
ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
defer stop()

srv := &http.Server{Addr: ":" + port, Handler: mux}
go func() { _ = srv.ListenAndServe() }()

<-ctx.Done()                       // aguarda SIGTERM
shutdownCtx, cancel := context.WithTimeout(context.Background(), 8*time.Second)
defer cancel()
_ = srv.Shutdown(shutdownCtx)      // drena
_ = db.Close()
```

## 3. Log aggregation (structured logs)

Railway captura stdout/stderr nativamente, mas parse mínimo. Para downstream (Grafana Loki, Datadog, Better Stack):

- Emit **JSON structured logs** (zerolog/pino/slog).
- Campos padrão: `level`, `ts`, `msg`, `trace_id`, `user_id` (mascarado), `service`.
- Webhook Integration → Logs: Railway envia cada linha para endpoint HTTPS (tipicamente forwarder → Loki/Datadog).

```go
// zerolog JSON structured
logger := zerolog.New(os.Stdout).With().Timestamp().Str("service", "api").Logger()
logger.Info().Str("user_id", util.Masked(uid)).Msg("login success")
```

## 4. Migrations como pré-deploy step

**Evite** rodar migrations no startup da app — race entre múltiplas instâncias (2+ pods subindo em paralelo tentam `ALTER TABLE` simultâneo).

**Preferido**:
- Settings → Deploy → Pre-deploy command.
- Ou job separado `railway run migrate` antes de `railway up`.
- Advisory lock do Postgres (goose já usa).

## 5. Private networking para reduzir egress billing

Services do mesmo project → `<service>.railway.internal:<port>` (IPv6 privado). Traffic zero-cost.

```bash
# em vez de:  API_URL=https://api.example.com
# use:        API_URL=http://api.railway.internal:3000
```

Limitações: só IPv6 (sua lib HTTP precisa suportar — `net.DefaultResolver` Go OK; alguns clientes Node antigos não).

## 6. Env vars: scoped > shared quando não há reason contrário

- Variables service-scoped ficam isoladas entre services.
- Shared variables (project-wide) são úteis só quando **múltiplos services** consomem (ex.: `SENTRY_DSN` usado por api + worker).
- Preferir scoped para reduzir blast radius de mudança.

## 7. Preview deploy por PR (gate natural de CI)

- Settings → Environments → Preview Environments: ativar.
- Cada PR gera URL temporária.
- Review manual + smoke test antes de merge.

## 8. Cron jobs idempotentes

Cron services em Railway são processos disparados por schedule. Invariante crítica: **cada execução é idempotente**.

- Use `advisory_lock` do Postgres se há risco de sobreposição.
- Se job demora > 1h, prefira worker permanente com fila (Redis/NATS) em vez de cron.

## 9. Monitoring + alerting

- Logs + metrics nativos no dashboard.
- Integração Sentry (error tracking) obrigatória para produção.
- Webhook Integration → Deployment Failed → Slack/Discord.
- Custom alertas: exportar metrics via OpenTelemetry para Grafana Cloud / Datadog.

## 10. Secrets via Railway Variables, nunca em git

- Dashboard → Variables → marcar como "sealed" (mascara no dashboard).
- Nunca commit de `.env` real — só `.env.example`.
- Rotate secrets via dashboard (invalida keys velhas no próximo deploy).

## Links

- [[_moc]]
- [[../_moc]]
- [[antipatterns]]
- [[integration-guide]]
- [[../../observability/structured-logging]]
