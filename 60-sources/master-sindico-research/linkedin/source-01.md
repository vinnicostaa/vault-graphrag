---
title: "source-01 — The Log: What every software engineer should know (Jay Kreps)"
type: source
tags: [research, linkedin, kafka, log, event-sourcing, unified-log, jay-kreps]
source: https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
created: 2026-04-23
updated: 2026-04-23
aliases: [the-log, unified-log, kreps-log]
---

# source-01 — The Log: real-time data's unifying abstraction

**URL**: https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying
**Autor**: Jay Kreps (LinkedIn, depois Confluent)
**Data original**: 2013-12-16
**Fonte primária**: sim (LinkedIn Engineering)

## Por que essa fonte importa

Texto fundador do conceito "log unificado" que deu origem ao Kafka. Justifica **quando** adotar event bus central vs integrações pontuais. Referência obrigatória pra decisão D-LKD-001 (NATS JetStream condicional).

## Pontos extraídos

### Conceito de log
- "Append-only, totally-ordered sequence of records ordered by time."
- Não é o log de texto de aplicação; é a estrutura de dados.

### Problema M×N
- Dezenas de sistemas especializados (Search, Social Graph, Voldemort, Espresso, Hadoop) × conectar cada um a cada outro = O(N²) pipelines custom.
- Cada sistema novo amplifica.
- Solução: log central, cada sistema conecta **uma vez** ao log (como consumer ou producer).

### Kafka nasce assim
- "Combinando mensageria com o conceito de log de databases/distributed systems."
- Pipeline central pra: (i) activity data, (ii) data deployment out of Hadoop, (iii) monitoring, (iv) change capture de databases.

### Decisões técnicas justificadas no texto
- **Partitioning** — ordem total só por partição, **não global**. Trade-off aceito porque "a interação com o log vem de centenas/milhares de processos distintos".
- **Batching agressivo** — cliente→servidor, disco, replicação, consumidor. Limite: "taxa suportada por disco ou rede".
- **Log compaction** — mantém só latest por chave; não é infinito.

### Três usos do log segundo Kreps
1. **Data integration** — entre sistemas de storage/processing.
2. **Stream processing** — derivações contínuas, não batch.
3. **System building** — log como base pra construir sistema distribuído (ex: sourcing, replicação).

### State Machine Replication
- "Dois processos determinísticos idênticos, mesma entrada na mesma ordem → mesma saída e mesmo estado."
- Justifica log como source of truth pra consistência distribuída sem precisar de Paxos/Raft em cada ponto.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#1-kafka--log-unificado--nats-jetstream-quando-necessário]]. Resumo:

- M×N do LinkedIn = centenas de sistemas. M×N master-sindico M1 = 1×1 (AssemblyEvent → NotificationModule). **Outbox sobre Postgres resolve**.
- Condições pra promover a NATS JetStream (≥ 3 produtores × ≥ 3 consumidores, ou replay histórico, ou latência de polling > p95 5s) são **regra-de-ouro extraída deste texto**, não convenção arbitrária.

## Quando releer

- Quando escrevermos ADR de event bus.
- Quando AssemblyModule precisar emitir pro que for **segundo consumidor novo** (aproximar threshold).
- Quando surgir pressão pra "CQRS + event sourcing" em algum agregado — releitura evita confundir log-integração com log-como-fonte-de-verdade-de-agregado.

## Vizinhos

- [[_destilado]]
- [[source-02]] — Espresso (outro sistema que virou consumer do log)
- [[source-10]] — Endorsements infra usa Kafka + Samza
