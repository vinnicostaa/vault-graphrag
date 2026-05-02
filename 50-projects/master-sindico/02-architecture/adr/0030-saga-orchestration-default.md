---
title: ADR-0030 — Saga orchestration como default, choreography como edge case justificado
type: adr
tags: [adr, saga, orchestration, choreography, uow, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
aliases: [saga-orchestration-default, orchestration-over-choreography]
---

# ADR-0030 — Saga orchestration default, choreography edge case

## Status

accepted — 2026-04-23 (refina [[0015-uow-intra-saga-inter]])

## Contexto

[[0015-uow-intra-saga-inter]] (accepted) define "Saga inter-módulo / provider externo" mas **não declara explicitamente** que **orchestration** é o modelo default — menciona choreography apenas como alternativa rejeitada.

Findings relacionados:

- [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-015]] (🟠 high): "Cross-BC event consumer sem revalidar tenant" — consumer assíncrono (choreography) perde contexto do user publisher, validação tenant fica órfã.
- [[../../11-audit/dual-check-log/2026-04-23-decisoes-arquiteturais]] (dual-check arch) registrou ressalva sobre ausência de explicitação.
- Research [[../../13-research/uber-engineering/_destilado]] + [[../../13-research/netflix/_destilado]] convergem: **orchestration** (saga coordinator persistido) é o padrão de ambos em fluxos multi-step com compensação; choreography só em eventos broadcast sem gate de sucesso.

**Riscos da ausência de default explícito**:

- Desenvolvedor pode implementar `ContractSaga` como choreography (`deal.signed_complete` → institutional listener cria entry), sem coordenador → se listener falha, não há retry / compensação centralizada.
- Debug distribuído vira pesadelo (não há "estado da saga" para consultar).
- Auditoria LGPD (Art. 37 — registro de tratamento) fica fragmentada entre listeners.

## Decisão

**Orchestration é o modelo default** para toda saga em Master Síndico. **Choreography só é permitido** em cenários específicos listados abaixo, com **justificativa explícita** no ADR de feature.

### Orchestration (default)

- Existe um **coordinator** (módulo `application/sagas/<name>_saga.go`) que:
  1. **Persiste estado** em `shared.saga_instances` antes do 1º step (INV-095).
  2. **Invoca** cada step via port (UoW para intra-BC; RPC/HTTP para provider externo).
  3. **Recebe callback** (webhook ou event) e avança estado.
  4. **Dispara compensação** em reverso em caso de falha em step N.
  5. **Timeout** explícito por step (`Compensating_Manual` se webhook provider não vem).

Sagas canônicas M1/M3 (confirmadas do [[0015-uow-intra-saga-inter]]):

- `ContractSaga` — RFP → Proposal → Contract → Milestones (commercial + institutional + billing).
- `VideoPublishSaga` — upload Mux → transcode → moderate → publish → timeline (content + cross-domain).
- `AssemblySaga` — schedule → CreateRoom Livekit → Egress → record → Publish Minutes (assembly + Livekit + content).
- `TrialExpirySaga` — trial expira → update billing + downgrade features + email (billing + identity + content).
- `AccountDeletionSaga` (**novo M1**) — request → pre-cancel Stripe → aguardar `customer.subscription.deleted` → pseudonimizar → hard-delete (identity + billing + cross-domain) — cobre [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]].

### Choreography (edge case — só permitido nestes cenários)

1. **Broadcast analítico sem gate**: evento `video.viewed` → consumers atualizam counters em paralelo; falha de um não bloqueia outros.
2. **Cache invalidation**: `subscription.updated` → consumer invalida ABAC cache (XD-006). Se falha, cache TTL natural 5 min elimina stale.
3. **Notification fan-out**: `timeline.entry_created` → listeners emitem FCM + email; falha parcial é aceitável.

**Checklist para permitir choreography** (todos os 5 devem ser true):

- [ ] Evento é **broadcast** (N consumers independentes).
- [ ] **Nenhum consumer** precisa bloquear o fluxo do publisher.
- [ ] Falha de consumer **não exige compensação** no publisher.
- [ ] Não há **ordem causal** entre consumers (commutativos).
- [ ] **Stale tolerance** documentada (TTL, retry count, DLQ).

### Consumer validation (cobre A-DC-PEN-015)

Mesmo em choreography, **todo consumer** valida:

1. `event.tenant_id` contra dados do próprio evento (ex: `deal.tenant_id == condominium.owner_id`).
2. `event.version` compatível com schema atual (rejeita eventos de versão futura desconhecida).
3. **HMAC do outbox** assinado pelo publisher — detecta tampering em eventos NATS intra-cluster.
4. Idempotência: `UNIQUE(event_id)` em `processed_events` por consumer.

## Consequências

### Positivas

- Fecha A-DC-PEN-015 (consumer tenant revalidation).
- Explicita a política — reduz risco de choreography ad-hoc em fluxos críticos.
- `saga_instances` dá observabilidade end-to-end (dashboard Grafana possível).
- Compensação centralizada vs espalhada em N listeners.
- Simplifica LGPD audit (1 coordinator = 1 `lgpd_log` trail por saga run).

### Negativas

- Coordinator é single point of failure se não for resiliente — mitigado por estado persistido + worker restart retoma.
- Temporal.io (alternativa considerada em [[0015-uow-intra-saga-inter]]) continua como reavaliação M3+.

## Regras

1. **Toda saga nova** tem nome `XxxSaga` em `application/sagas/` + schema em `saga_instances`.
2. **`saga_instance.state`** máquina de estados explícita — documentar em [[../../01-domain/state-machines]].
3. **Choreography exige ADR de feature** com checklist acima preenchido.
4. **Consumer sempre valida tenant + HMAC + idempotency** mesmo em choreography.
5. **Timeout por step** obrigatório (default 30 min; provider externo 2h).
6. **Dashboard** Grafana `saga_instances` com métricas `pending / completed / failed / compensating`.

## Alternativas consideradas

1. **Choreography default** (event-driven puro) — rejeitado: debug distribuído + tenant validation órfã + LGPD fragmentado.
2. **Temporal.io / Cadence** — excelente fit, mas custo adicional em M1 (self-host ou Temporal Cloud). Reavaliar M3+ se volume de sagas > 20 tipos.
3. **Process Manager style (NServiceBus)** — conceito equivalente a orchestration; padronizamos vocabulário "saga coordinator".
4. **Híbrido sem regra explícita** — ambiguidade vira débito técnico; ADR fecha a porta.

## Referências

- Finding: [[../../11-audit/audit-log/2026-04-23-pentest-specs#A-DC-PEN-015]] (🟠 high, consumer revalidation).
- Finding: [[../../11-audit/dual-check-log/2026-04-23-security-gaps#A-DC-SEC-004]] (LGPD race Stripe — AccountDeletionSaga cobre).
- [[0015-uow-intra-saga-inter]] — ADR refinada pelo update 2026-04-23.
- [[../patterns]] §7 (Saga) — atualizar para refletir orchestration default.
- [[../event-driven]] §10 — outbox + saga coordinator.
- [[../../01-domain/invariants#INV-095]] — outbox pattern (eventos publicados em mesma tx).
- [[../../01-domain/state-machines]] — máquinas de estado das sagas.
- Chris Richardson "Microservices Patterns" Ch 4 (Saga pattern): https://microservices.io/patterns/data/saga.html
- Uber "Distributed Transactions": https://www.uber.com/blog/sagas-java-microservices/
- Netflix "Orchestrating Distributed Workflows" (Conductor): https://netflixtechblog.com/netflix-conductor-a-microservices-orchestrator-2e8d4771bf40
