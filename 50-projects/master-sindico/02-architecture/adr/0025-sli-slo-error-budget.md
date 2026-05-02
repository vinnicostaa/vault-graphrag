---
title: ADR-0025 — SLI / SLO 99.5% M1 + error budget policy
type: adr
tags: [adr, observability, slo, sre, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0025 — SLI / SLO 99.5% M1 + error budget policy + postmortem blameless

## Status

accepted — 2026-04-23

## Contexto

Sem SLO formal não há critério objetivo para "funcionando" vs "quebrado". SRE book ([[../../13-research/google-arch/_destilado]] §1) define SLI/SLO/SLA canônico + error budget policy + postmortem blameless como fundação operacional.

## Decisão

**Adotar framework SRE desde M1** com 3 SLIs por BC, window rolling 4 semanas, target 99.5% conservador inicial.

### SLIs por bounded context (M1)

Tabela em [[../observability]] §3.1. Resumo:

| BC | Latência p95 | Availability | Correção negócio |
|---|---|---|---|
| identity | `/auth/login` < 500ms | 99.5% | 0 cross-tenant auth |
| billing | `/subscribe` < 1s | 99.9% webhooks | 100% idempotência |
| institutional | `POST /assemblies` < 300ms | 99.5% | 100% atas ≤ 24h |
| commercial | `GET /companies` < 500ms | 99.5% | 100% reputação diária |
| content | `POST /videos/upload-url` < 1s | 99.5% | 100% moderado pré-publish |
| assembly | WebSocket lag < 400ms | 99.9% em live | 0 votos perdidos |
| cross-domain | outbox dispatch p95 < 5s | 99.5% | DLQ < 0.1% |

### Error budget policy

- `budget = 1 - SLO` → 99.5% = 0.5% ≈ 3.6h/mês downtime permitido por BC.
- **Rolling 4-week window** (integral weeks — herança SRE workbook).
- **Trigger budget exaurido**: "halt all changes and releases other than P0 issues or security fixes until the service is back within its SLO".
- **Exceções**: outages provider externo (Stripe, Zitadel, LiveKit, AWS) não contam; categorização + postmortem.

### Postmortem triggers

Obrigatório postmortem blameless em `11-audit/postmortems/YYYY-MM-DD-<slug>.md` quando:

- User-visible downtime > 5 min.
- Perda de dado (imediato + A-### urgente).
- Intervenção manual (rollback, reroute) pra resolver.
- Resolução > 1h.
- Falha monitoramento (descoberta manual).
- >20% budget trimestral consumido em 1 incidente.

Template em [[../observability]] §6.2.

### Alerting: multi-window burn rate

Inspirado [[../../13-research/google-arch/_destilado]] §1.2 e [[../../13-research/netflix/_destilado]] §6.

| Window curta | Burn rate | Severity |
|---|---|---|
| 1h | > 14.4× | page on-call |
| 6h | > 6× | ticket |
| 24h | > 1× | ticket |
| 72h | > 0.5× | review quinzenal |

**Todo alert tem runbook** em `12-docs/runbooks/`. Alert sem runbook é banido.

## Roadmap

| Fase | SLO | Error budget action |
|---|---|---|
| M1 (2026-05) | 99.5% baseline | Halt automático manual (decisão humana) |
| M2 (2026-Q3) | 99.5% confirmado; alguns endpoints 99.9% | Halt semi-auto via feature flags |
| M3 (2027) | 99.9% upgrade + SLA contratual enterprise | Halt auto via CI gate |
| M4+ | 99.95% em endpoints críticos | Chaos engineering + SLO-driven canary |

## Consequências

### Positivas

- Critério objetivo substitui "tá lento", "deu erro".
- Baseline de 99.5% evita over-promise em M1.
- Postmortem sistemático = aprendizado acumulado.
- Pipeline de maturidade claro.

### Negativas

- Métricas/alertas bem calibrados exigem tuning inicial (2-4 semanas de observação).
- Budget exaurido travando release pode criar conflito produto × eng — policy escrita mitiga.

## Alternativas consideradas

1. **SLA contratual direto sem SLO interno** — liga primeira para cliente em caso de falha.
2. **SLO só p95 latência** — ignora disponibilidade e correção negócio.
3. **99.9% M1** — over-promise; budget muito apertado para time pequeno.
4. **Sem SLO (só threshold de alerta)** — alert fatigue + sem crítica.

## Referências

- [[../observability]] §3-6 (canônico)
- [[../../13-research/google-arch/_destilado]] §1 (SRE, error budget, postmortem)
- [[../../13-research/netflix/_destilado]] §6 (burn rate alerting)
- [[0020-opentelemetry-grafana-sentry]]
- Google SRE Book + Workbook
