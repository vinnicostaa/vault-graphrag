---
title: Observability — OpenTelemetry + Grafana + Sentry + SLI/SLO
type: guide
tags: [architecture, observability, otel, grafana, sentry, slo, master-sindico, v2]
source: 13-research/google-arch §1 (SRE book) + §4 (Dapper) + 13-research/netflix §6 (Atlas/Mantis)
created: 2026-04-23
updated: 2026-04-23
aliases: [obs, telemetry, slo]
---

# Observability — Master Síndico

Stack canônica **OpenTelemetry + Grafana + Sentry**. SLI/SLO formais por BC, error budget policy, postmortem blameless.

> Herança direta: [[../13-research/google-arch/_destilado]] §1 (SRE framework) + §4 (Dapper → OTel), [[../13-research/netflix/_destilado]] §6 (dimensional tagging + burn rate alerting).

---

## 1. Stack

| Camada | Ferramenta |
|---|---|
| **SDK instrumentação** | OpenTelemetry Go (otelgin, otelpgx, otelhttp, otelredis) |
| **Logs backend** | Grafana Loki (multi-tenant nativo) |
| **Metrics backend** | Prometheus (M1 single) → Grafana Mimir (M3+) |
| **Traces backend** | Grafana Tempo |
| **Error tracking** | Sentry (Go + SolidJS + Flutter) |
| **Dashboards + alerting** | Grafana |
| **Status page** | BetterStack ou statuspage.io |

Decisão: [[adr/0020-opentelemetry-grafana-sentry]].

---

## 2. Telemetria — os 3 pilares

### 2.1 Logs — zerolog structured

```go
log.Info().
    Str("request_id", reqID).
    Str("tenant_id", tenantID.String()).
    Str("user_id", userID.String()).  // user_id opaco, sem PII
    Str("endpoint", "POST /api/v1/assemblies/:id/vote").
    Int("status", 201).
    Dur("duration", dur).
    Msg("vote cast")
```

### 2.1.1 Campos canônicos (todo log)

- `request_id` — ULID por request.
- `tenant_id` — ULID do tenant atual.
- `user_id` — ULID opaco; **nunca** CPF, email, nome.
- `trace_id`, `span_id` — propagação OTel.
- `bc` — bounded context (`identity`, `billing`, ...).
- `endpoint`, `method`, `status`, `duration`.

### 2.1.2 PII proibido em logs (regra dura)

- CPF, CNPJ, email, telefone, tokens, JWT, password.
- Quando precisa logar identificador: usar `Masked()` (`***.***.***-XX`).
- Linter CI: regex grep em `*.go` bloqueia padrões suspeitos.

Herança: [[../CLAUDE]] §3.10.

### 2.2 Métricas — Prometheus

#### RED metrics (requests)

- **Rate**: `http_requests_total{endpoint,status_class,method,bc}` contador.
- **Errors**: `http_requests_total{status_class="5xx"}` derivado.
- **Duration**: `http_request_duration_seconds{endpoint,method,bc}` histograma (p50/p95/p99).

#### USE metrics (recursos)

- **Utilization**: `postgres_connections_active`, `redis_memory_used_bytes`.
- **Saturation**: `goroutines_total`, `heap_alloc_bytes`.
- **Errors**: `postgres_errors_total`, `redis_errors_total`, `provider_errors_total{provider}`.

#### Business metrics

- `assemblies_started_total{tenant_id_anon}` (opcional — cardinalidade controlada).
- `rfps_opened_total{category}`.
- `videos_moderated_total{decision}`.
- `subscription_created_total{plan}`.

### 2.2.1 Cardinalidade — regra dura

**Labels com alta cardinalidade** (user_id, request_id, session_id) **proibidos** em métricas. Explode série.

**Labels permitidos**:
- `tenant_id` quando contagem é baixa (M1: dezenas); migrar para `tenant_tier` se crescer.
- `endpoint`, `method`, `status_class` (2xx/4xx/5xx), `bc`, `provider`.

Herança: [[../13-research/google-arch/_destilado]] §4.2 (Monarch).

### 2.3 Traces — OpenTelemetry

#### Spans canônicos

- `http.request` — envolvendo cada request HTTP.
- `domain.<aggregate>.<operation>` — ex: `domain.assembly.cast_vote`.
- `db.query` — auto via otelpgx; atributo `db.statement` (sem valores PII).
- `provider.<name>.<op>` — ex: `provider.stripe.create_subscription`, `provider.mux.create_direct_upload`.
- `saga.<type>.<step>` — passos de saga.
- `outbox.dispatch` — worker dispatch.

#### Atributos mínimos (todos spans)

- `tenant_id`
- `user_id` (opaco)
- `bc`
- `saga_id` se aplicável
- **Proibido**: PII bruto.

### 2.3.1 Sampling

- **Head-based** em produção: 1% de requests com trace completo; 100% de requests 5xx; 100% de requests com `X-Debug-Trace` header (dev/ops).
- **Tail-based** considerado M3+ (custo operacional alto).

Herança: [[../13-research/google-arch/_destilado]] §4.1 (Dapper principles).

### 2.4 Errors — Sentry

- Captura exceções não tratadas + panics recovered.
- Scrub PII antes de envio (lista: CPF, CNPJ, email, telefone, token, password).
- Release tracking via `SENTRY_RELEASE=<git_sha>`.
- Performance monitoring desabilitado (usa OTel + Tempo em vez).

---

## 3. SLI / SLO por bounded context

Decisão: [[adr/0025-sli-slo-error-budget]]. Baseline conservador **99.5%** em M1, upgrade para 99.9% após 2 ciclos estáveis (4 semanas).

### 3.1 Matriz M1

| BC | SLI-1 (latência p95) | SLI-2 (availability) | SLI-3 (correção negócio) |
|---|---|---|---|
| **identity** | `/auth/login` < 500ms | 99.5% 2xx/3xx | 0 autorizações incorretas (cross-tenant) |
| **billing** | `/billing/subscribe` < 1s | 99.9% webhooks Stripe processados | 100% idempotência |
| **institutional** | `POST /assemblies` < 300ms | 99.5% | 100% atas emitidas ≤ 24h |
| **commercial** | `GET /companies` < 500ms | 99.5% | 100% reputação recalculada diariamente |
| **content** | `POST /videos/upload-url` < 1s | 99.5% | 100% vídeos moderados antes do publish |
| **assembly** | WebSocket lag < 400ms | 99.9% durante assembleia ativa | 0% votos perdidos |
| **cross-domain** | Outbox dispatch p95 < 5s | 99.5% eventos dispatched | DLQ < 0.1% |

### 3.2 Windows

- **Rolling 4 weeks** (integral weeks, herança SRE workbook).
- Reset **não automático** — burn budget forçar decisão humana.

### 3.3 RPO/RTO por BC (Fase 9 — G-015 consolidado, ADR-0035)

Paralelo à matriz SLO; cobre **perda de dados** e **tempo de recuperação** pós-incidente (corruption / delete / ransomware). PITR via WAL archiving contínuo (pgBackRest em S3/R2) garante RPO base de 5 min. Backup imutável offline (Object Lock 90d) cobre ransomware.

| BC | RPO (max data loss) | RTO (restore-to-live) | Comentário |
|---|---|---|---|
| **identity** | 5 min | 15 min | Login quebrado = usuário fora; priorização máxima |
| **billing** | 5 min | 30 min | Perder evento Stripe = reconciliação manual custosa |
| **assembly** | 5 min | 30 min | Assembleia em curso; aceita re-sync pós-restore |
| **institutional** | 5 min | 1 h | Timeline/comunicados |
| **content (vídeos)** | 1 h | 4 h | Upload pode ser retomado pelo usuário |
| **commercial** | 5 min | 1 h | Connect Me / Banco Talentos |
| **cross-domain (audit)** | 0 min | 15 min | Audit log **nunca pode perder** — streaming contínuo via outbox |
| **global (plans)** | 0 min | 5 min | Catálogo; mudança rara via admin |

**Validação**: DR drill mensal (1ª terça do mês) restaura backup em staging-dr e mede RPO observado (último WAL timestamp) + RTO observado (tempo total). Se observado > target em 2 drills consecutivos → Sev2 + revisão ADR.

**Runbook**: [[../06-execution/dr-drill]]; [[../06-execution/playbooks/post-restore-reconcile]].

---

## 4. Error budget policy

Decisão: [[adr/0025-sli-slo-error-budget]].

### 4.1 Fórmula

- `error_budget = 1 - SLO`
- 99.5% SLO → 0.5% budget → **~3.6h de downtime/mês permitido**.

### 4.2 Trigger

Budget exaurido em janela de 4 semanas de um BC:

- **Consequência**: "halt all changes and releases other than P0 issues or security fixes until the service is back within its SLO" ([[../13-research/google-arch/_destilado]] §1.2).
- Exceção: outages de infra externa (Stripe, Zitadel, LiveKit, AWS) não contam; categorização + postmortem.

### 4.3 Escalation

- Incidente consumindo >20% do budget trimestral → **postmortem obrigatório**.
- Findings viram A-### em [[../AUDIT]]; remediação priorizada acima de features.

---

## 5. Alerting — burn rate multi-window

Herança [[../13-research/netflix/_destilado]] §6 + SRE workbook. Alertar em **burn rate**, não em threshold absoluto.

### 5.1 Alerts canônicos por BC

| Window curta | Burn rate | Severity |
|---|---|---|
| 1h | > 14.4× | page on-call |
| 6h | > 6× | ticket |
| 24h | > 1× | ticket |
| 72h | > 0.5× | review quinzenal |

### 5.2 Regra: **todo alert tem runbook**

- Runbook em `12-docs/runbooks/<alert_name>.md`.
- Alert sem runbook é banido (vira alert fatigue).

### 5.3 Alerts operacionais (não-SLO)

- DLQ size > 10 em qualquer stream/outbox.
- Circuit breaker aberto > 1min.
- Postgres connection pool saturation > 80%.
- Redis memory > 80%.

---

## 6. Postmortem blameless

Herança: [[../13-research/google-arch/_destilado]] §1.3 (SRE book).

### 6.1 Triggers (obrigam postmortem)

- User-visible downtime/degradação > 5 min.
- Perda de dados de qualquer tipo.
- Intervenção manual (rollback, reroute) pra resolver.
- Resolução > 1h.
- Falha de monitoramento que exigiu descoberta manual.
- >20% budget trimestral consumido em 1 incidente.

### 6.2 Template

```markdown
# Postmortem — <slug> — 2026-MM-DD

## Status
blameless

## Impact
- Duration: <start_ts> → <end_ts>
- Affected users: ~N
- SLO budget consumed: X% trimestral

## Timeline (UTC)
- HH:MM — evento
- HH:MM — detecção
- HH:MM — mitigação
- HH:MM — resolução

## Root cause
<causa técnica sem culpabilização>

## What went well
- …

## What went wrong
- …

## Action items
- [ ] A-### <item> (@owner, deadline)

## Links
- trace: <grafana link>
- sentry: <url>
- slack incident: <url>
```

Pasta: `11-audit/postmortems/YYYY-MM-DD-<slug>.md`.

---

## 7. Dashboards Grafana canônicos (M1)

1. **Overview** — RED + USE globais.
2. **Per-BC dashboard** (7 dashboards) — SLO burn rate, endpoint breakdown, error rate, provider latency.
3. **Database** — connection pool, query duration, replication lag.
4. **Providers** — Stripe / Mux / LiveKit / Zitadel / SES: rate, error, CB state, retry count.
5. **Business** — assemblies ativas, RFPs por categoria, subscriptions ativas por plano.
6. **Outbox + DLQ** — lag, throughput, failures.

Cada dashboard tem **owner** e **URL shareable** colada em PRs/ADRs.

---

## 8. Trace propagation — X-Request-ID

- Middleware `RequestID` gera ULID quando ausente e propaga.
- Header `X-Request-ID` devolvido em toda response.
- Propagado para logs, métricas, traces.
- Propagado no metadata de events para cross-BC trace.

---

## 9. Logs — retenção e privacidade

| Nível | Retenção | Notas |
|---|---|---|
| `debug` | 7d | só dev/staging |
| `info` | 30d | padrão |
| `warn` | 90d | |
| `error` | 180d | |
| `audit` (lgpd_logs) | **5 anos** | WORM, S3 Object Lock em M2+ |

---

## 10. CI gates relacionados

- `gosec -severity=high` 0 findings.
- `govulncheck` 0 CVEs reachable.
- Linter PII (grep regex) 0 matches em log statements.
- Linter cardinalidade (métricas com labels high-cardinality) 0 matches.

---

## 11. Roadmap maturidade

| Fase | O que entra |
|---|---|
| **M1 (2026-05)** | OTel SDK, Prometheus single, Grafana Cloud free, Sentry, SLO 99.5% por BC, dashboards base, burn rate alerts, postmortem template |
| **M2 (2026-Q3)** | Mimir/Thanos se volume, tail-based sampling avaliação, multi-region traces, DLQ alertas auto-ticketing |
| **M3 (2027)** | SLO 99.9% upgrade, on-call formal (3+ devs), chaos latency injection staging, continuous risk scoring |
| **M4+** | SOC 2 / ISO 27001 auditoria observability, machine learning anomaly detection |

---

## 12. ⚠️ Pendências

- **Grafana Cloud free vs self-host** — M1 free tier suficiente; revisar em M2 se volume ultrapassa limites.
- **Sentry open-source self-host** — avaliar em M3+ se custo SaaS escalar.
- **Lista de scrub Sentry** — validar com dual-check que nada de PII sai em stacktrace (nomes de variáveis, contexto).

---

## 13. Vizinhos

- [[adr/0020-opentelemetry-grafana-sentry]]
- [[adr/0025-sli-slo-error-budget]]
- [[event-driven]] — trace propagation em eventos
- [[c4-containers]] — container obs
- [[../13-research/google-arch/_destilado]] §1, §4
- [[../13-research/netflix/_destilado]] §6
