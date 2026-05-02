[FFV Academy](../index.html)/Sistemas Distribuídos

🧭

Blog

# Sistemas Distribuídos

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

9artigos

755XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/cap-pacelc/index.html)

01

## ⚖️ CAP e PACELC: o teorema que define toda arquitetura distribuída

Partition tolerance, availability, consistency — e por que você sempre escolhe entre 2. PACELC estende para latência. Decisões reais: Dynamo vs Spanner, Cassandra vs HBase.

⏱ 16 min·+80 XP

→[](../aprenda/consistency-models/index.html)

02

## 🔄 Modelos de Consistência: strong, eventual, causal, read-your-writes

Linearizability, sequential consistency, causal, eventual. Session guarantees (read-your-writes, monotonic reads, bounded staleness). Qual usar em cada cenário.

⏱ 17 min·+85 XP

→[](../aprenda/consensus-raft/index.html)

03

## 🗳️ Consensus e Raft: como nós discordam e chegam a acordo

FLP impossibility, intuição do Paxos, Raft explicado (leader election, log replication, safety). Por que etcd, Consul, K8s, CockroachDB usam Raft.

⏱ 18 min·+90 XP

→[](../aprenda/idempotencia-retries/index.html)

04

## 🔁 Idempotência e Retries: o antídoto pra rede que quebra

Idempotency keys, exponential backoff com jitter, circuit breaker, deadline propagation, dedup via Redis/Postgres, at-least-once vs exactly-once.

⏱ 15 min·+75 XP

→[](../aprenda/sagas-2pc/index.html)

05

## 🪢 Sagas vs 2PC: transações distribuídas sem perder o sono

Por que 2PC trava tudo, saga orchestration vs choreography, compensating actions, outbox pattern, idempotência nas compensações.

⏱ 17 min·+85 XP

→[](../aprenda/event-sourcing-cqrs/index.html)

06

## 📜 Event Sourcing e CQRS: quando eventos são a fonte da verdade

Event log como source of truth, projections, CQRS (command vs query), event store (Kafka, EventStoreDB), vantagens, armadilhas, quando evitar.

⏱ 17 min·+85 XP

→[](../aprenda/postgres-mvcc-isolation/index.html)

07

## 🐘 Postgres Profundo: MVCC, Isolation Levels e Locks

Como MVCC funciona por dentro, read committed vs repeatable read vs serializable, SELECT FOR UPDATE, advisory locks, VACUUM e bloat.

⏱ 17 min·+85 XP

→[](../aprenda/rate-limiting-distribuido/index.html)

08

## 🚦 Rate Limiting Distribuído: token bucket, sliding window, Redis

Token bucket, leaky bucket, sliding window log, sliding window counter, implementação com Redis + Lua script atômico, distributed rate limit em produção.

⏱ 15 min·+75 XP

→[](../aprenda/capstone-sistemas-distribuidos-saga/index.html)

09

## 🏁 Capstone: saga distribuída ponta a ponta

Projeto: e-commerce com 3 serviços (orders, payments, inventory). Saga orchestrator em Step Functions/Temporal. Compensations em cada fail. Outbox pattern pra eventos. Idempotency em retry. Tracing distribuído.

⏱ 20 min·+95 XP

→

[← Voltar à home](../index.html)
