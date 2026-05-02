---
title: Source 19 — Chubby Lock Service for Loosely-Coupled Distributed Systems
type: source
tags: [research, google, chubby, lock-service, paxos, distributed-coordination]
source: https://research.google/pubs/the-chubby-lock-service-for-loosely-coupled-distributed-systems/
created: 2026-04-23
updated: 2026-04-23
aliases: [chubby-paper]
---

# Source 19 — The Chubby Lock Service for Loosely-Coupled Distributed Systems

- **URL primária**: https://research.google/pubs/the-chubby-lock-service-for-loosely-coupled-distributed-systems/
- **PDF**: https://research.google.com/archive/chubby-osdi06.pdf
- **Conferência**: OSDI 2006
- **Autor**: Mike Burrows
- **Data fetch**: 2026-04-23

## Trechos rastreáveis

### Função
- "Coarse-grained locking as well as reliable (though low-volume) storage for distributed systems".
- "Interface much like a distributed file system with advisory locks".
- "Design emphasis is on availability and reliability, as opposed to high performance".

### Implementação
- Serviço composto por **5 réplicas ativas**, uma eleita master.
- Consenso via **Paxos** pra manter réplicas consistentes.
- Namespace = diretórios + arquivos pequenos; cada dir/arquivo pode ser lock; reads/writes atômicos.
- Consistent client-side caching com invalidação por updates.

### Uso interno Google
- "Chubby has become Google's primary internal name service".
- Rendezvous mechanism pra MapReduce.
- GFS e Bigtable usam Chubby pra eleger primary entre réplicas redundantes.
- Inspirou Apache **ZooKeeper**.

## Aplicabilidade ao Master Síndico

### Literal Chubby/Paxos
- **NÃO**. Master Síndico não precisa de consenso cross-machine com latência tolerável > 10ms.

### Cases onde precisamos de coordenação distribuída
- **Leader election** (ex: worker de cron sindico não rodar duplicado em múltiplos pods).
- **Mutex distribuído** (ex: processar upload vídeo único por tenant por vez).

### Soluções leves aplicáveis (M1)

| Necessidade | Solução | Por quê |
|---|---|---|
| Advisory lock simples intra-pod | Go `sync.Mutex` | Stdlib |
| Advisory lock cross-pod (mesmo DB) | **Postgres `pg_advisory_lock`** | Já temos; atomic; TTL via session |
| Cron leader election | **Redis Redlock** ou `pg_try_advisory_lock` | Infra já presente |
| Service discovery | **Kubernetes Service + DNS** (M2) | Built-in |
| Config distribuída | **env vars + ExternalSecrets** | Não precisa ZooKeeper |

**Padrão herdado de Chubby sem adotar Chubby**: *"coarse-grained, not fine-grained"*. Lock com duração de segundos/minutos, não de microsegundos. Aplicar no Master: `pg_advisory_lock('job:vídeo:{id}')` com timeout > 30s, não lock em cada row de insert.

## Links relacionados

- [[source-02]] Bigtable (consumidor de Chubby).
- [[source-16]] Omega (Paxos-based store, influência Chubby).
- Destilado: [[_destilado#8 Pub/Sub Research at Google]].
