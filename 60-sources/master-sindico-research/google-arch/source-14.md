---
title: Source 14 — Google Cloud Pub/Sub message ordering
type: source
tags: [research, google, pubsub, messaging, ordering, at-least-once, idempotency]
source: https://docs.cloud.google.com/pubsub/docs/ordering
created: 2026-04-23
updated: 2026-04-23
aliases: [pubsub-ordering]
---

# Source 14 — Google Cloud Pub/Sub: Order messages + delivery semantics

- **URL primária**: https://docs.cloud.google.com/pubsub/docs/ordering
- **Relacionado**: https://docs.cloud.google.com/pubsub/docs/exactly-once-delivery, https://docs.cloud.google.com/pubsub/docs/subscription-overview
- **Data fetch**: 2026-04-23

## Trechos rastreáveis (WebFetch 2026-04-23)

### Ordering
- "With message ordering, the subscriber client receives the published messages in the same order".
- Ordering key = metadata string (até 1 KB).
- **Throughput**: "The publish throughput on each ordering key is limited to 1 MBps".
- **Ordem só dentro da mesma chave**: "messages with different ordering keys have no guaranteed delivery sequence".

### At-least-once (default)
- "By default, Pub/Sub offers at-least-once delivery with no ordering guarantees on all subscription types".
- "The message is guaranteed to be delivered at least one time, but it might be sent more than once".
- "Accommodating more-than-once delivery requires your subscriber to be idempotent when processing messages".

### Trade-offs ordered delivery
- "Compared with unordered delivery, ordered delivery decreases publish availability and increases end-to-end message delivery latency".
- Redelivery: "redelivery of all subsequent messages for that key" (até acknowledgments de msgs anteriores).
- **Hot key**: "A hot key occurs when a backlog builds on an individual ordering key because the number of messages produced per second exceeds" capacity do subscriber.

### Exactly-once
- Pub/Sub suporta exactly-once delivery opcional (source: docs/exactly-once-delivery) como feature configurável — aumenta latência.

## Aplicabilidade ao Master Síndico

### Stack atual / M1
- Redis Streams + outbox pg (não Pub/Sub GCP literal).
- **Padrões Pub/Sub a adotar desde M1 independente da implementação**:

1. **Idempotência é obrigatória sempre** — "the subscriber to be idempotent" (source). Toda msg tem `idempotency_key`; handler upsert em `processed_events(idempotency_key PK)`.
2. **Ordering key quando semântica exige**:
   - `assembleia.abrir → voto → encerrar` → key = `assembleia_id`.
   - `notificacao.broadcast` → sem key (ordem não importa).
3. **Trade-off registrado**: ordem custa throughput por key (1 MBps no Pub/Sub; no Redis Streams similar). ADR deve listar tópicos ordenados.
4. **Hot key avoidance**: não usar `tenant_id` como ordering key pra eventos frequentes — spread em sub-keys (`tenant_id:assembleia_id`).
5. **DLQ**: após N retries → move pra dead-letter + A-### em AUDIT automático.

### Quando migrar pra Pub/Sub real (GCP)?
- Somente em M3+ se volume/distribuição exigir. Redis Streams aguenta 100k+ msg/s por tópico com latência sub-ms — suficiente até dezenas de milhares de usuários.

## Links relacionados

- [[source-11]] Postmortem (DLQ + incidentes de msg).
- Destilado: [[_destilado#5 Messaging]].
