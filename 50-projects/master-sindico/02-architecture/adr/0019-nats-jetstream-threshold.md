---
title: ADR-0019 — NATS JetStream quando ≥ 3 produtores × 3 consumidores (ou replay / latência)
type: adr
tags: [adr, messaging, nats, outbox, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0019 — NATS JetStream sob critério objetivo

## Status

accepted — 2026-04-23

## Contexto

Adotar broker de mensageria dia 1 é cargo-cult. LinkedIn construiu Kafka em 2010 com 100M+ membros, não em dia 1 ([[../../13-research/linkedin/_destilado]] §1). Outbox pattern em Postgres cobre 99% dos casos de M1 sem custo de infra/operacional adicional.

## Decisão

**Outbox Postgres** é o event bus default em M1. **NATS JetStream** entra somente quando **pelo menos uma** destas condições for satisfeita:

1. **Topologia M×N real**: ≥ 3 módulos produtores **e** ≥ 3 módulos consumidores do mesmo evento.
2. **Replay histórico** obrigatório (auditoria LGPD, reprocessamento de bug).
3. **Latência outbox polling p95 > 5s** (virou gargalo).
4. **Coordenação fina** entre N workers sobrecarrega `pg_advisory_lock`.

Migração prevista M2+ quando critério bater.

### Por que NATS (não Kafka nem Redis Streams)

- **Kafka**: overkill operacional (Zookeeper/KRaft, partições por tópico); escala que não teremos.
- **Redis Streams**: sem replay histórico robusto, sem consumer groups maduros, limit 512MB memory-backed.
- **NATS JetStream**: persistência em disco, consumer groups, ordering key, DLQ, baixo footprint operacional, Go-native client.

## Schema e convenções

### Streams canônicos (quando entrar)

| Stream | Produtor(es) | Consumer(s) | Retention | Ordering |
|---|---|---|---|---|
| `events.identity` | identity | billing, commercial, institutional | 7d | by user_id |
| `events.billing` | billing | identity, content | 30d | by subscription_id |
| `events.institutional` | institutional | content, assembly, commercial, notifications | 30d | by condominium_id |
| `events.commercial` | commercial | institutional, notifications | 30d | by rfp_id |
| `events.content` | content | institutional, commercial | 7d | by video_id |
| `events.assembly` | assembly | institutional, content | 90d (ata) | by assembly_id |
| `events.dlq` | todos (falha) | admin | 30d | — |

### Event schema

Ver [[../event-driven]] §4.

### Migração sem downtime

1. Outbox dispatcher continua rodando.
2. Adicionar segundo dispatcher que publica NATS.
3. Consumer NATS com idempotency key.
4. Switchover: outbox → NATS full; outbox vira source of truth / backup.

## Consequências

### Positivas

- M1 sem overhead de broker.
- Critério objetivo evita "adotar porque é moderno".
- Outbox pattern continua como fonte de verdade — NATS é transporte.
- Replay histórico vem "grátis" com JetStream.

### Negativas

- Migração exige 2-4 semanas (schemas, consumer, dispatcher dual, switchover).
- Adicionar infra (NATS cluster self-host ou Synadia Cloud) em M2+.

## Alternativas consideradas

1. **Kafka em M1** — cargo-cult ([[../../13-research/linkedin/_destilado]] §1 antipadrão).
2. **Redis Streams** — insuficiente para replay histórico.
3. **Google Pub/Sub** — ótimo, mas GCP-lock e master-sindico está em Railway → AWS.
4. **AWS SQS + SNS** — funciona, mas fanout limitado e sem ordering key robusto.
5. **RabbitMQ** — sólido, mas NATS tem footprint menor + go-native excelente.

## Referências

- [[../../13-research/linkedin/_destilado]] §1 ("The Log")
- [[../../13-research/google-arch/_destilado]] §5 (Pub/Sub ordering)
- [[../event-driven]] (canônico)
- [[../patterns]] §14 (Outbox), §11 (Idempotency)
