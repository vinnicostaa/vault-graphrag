---
title: Source — ByteGraph (VLDB 2022, ByteDance graph database)
type: source
tags: [research, tiktok, bytedance, graph-database, vldb, distributed-systems]
source: https://vldb.org/pvldb/vol15/p3306-li.pdf
created: 2026-04-23
updated: 2026-04-23
aliases: [bytegraph-paper]
---

# Source — ByteGraph: A High-Performance Distributed Graph Database in ByteDance

## Metadados

- **URL**: https://vldb.org/pvldb/vol15/p3306-li.pdf
- **ACM**: https://dl.acm.org/doi/abs/10.14778/3554821.3554824
- **Venue**: Proceedings of the VLDB Endowment, Vol 15, No 12 (VLDB 2022)
- **Autores**: Changji Li, Hongzhi Chen, Shuai Zhang et al. (ByteDance)
- **Autoridade**: altíssima — paper peer-reviewed VLDB + em produção.

## Contexto

TikTok, Douyin e Toutiao geram volumes massivos de dados de grafo (usuário-vídeo, usuário-usuário, vídeo-hashtag). Databases existentes tinham gargalos diferentes em workloads mistos. ByteGraph foi desenhado pra cobrir os três tipos de carga simultaneamente.

## Workloads suportados (OLAP + OLTP + OLSP)

- **OLAP** — analítico, queries longas sobre subgrafos grandes.
- **OLTP** — transacional, updates ACID em conjuntos pequenos.
- **OLSP (Online Serving Processing)** — **novo tipo explicitado pelo paper**: queries de latência baixa pra recommendation/risk serving. Volume altíssimo, traversals rasos.

## Arquitetura em 3 camadas

1. **BGE (ByteGraph Engine)** — camada de execução. Parsing, planning, orquestração.
2. **Cache layer in-memory** — mantém grafo quente com política de freshness por aplicação.
3. **BGS (ByteGraph Storage)** — persistência. Usa **edge-trees** pra adjacency lists.

## Inovações técnicas

- **Edge-trees** em vez de adjacency lists planas: permite alta paralelização + baixo uso de memória. Cada vértice tem sua adjacency organizada em árvore, queries podem particionar a busca.
- **Adaptive optimizations** em thread pools e índices (auto-tuning baseado em workload observado).
- **Geographic replication** pra fault tolerance e disponibilidade — replicas em regiões diferentes.
- **Multi-object transactions** — ACID em updates que tocam múltiplos vértices/arestas.

## Escala / números

- **Tens of billions of vertices, trillions of edges** (10^10 vértices, 10^12 arestas).
- Throughput: **millions a 30M+ queries/s** no cluster inteiro.
- **99.99% SLA**, error rate 0.002% em 24h.

## Comparação com alternativas

Paper argumenta que soluções existentes têm gargalos:

- **Cloud databases** (single master) — gargalo no master.
- **Document stores** (MongoDB-style JSON) — overhead de gerenciamento de JSON anidado.
- **In-memory systems** (Redis graph) — escalabilidade limitada.

## Follow-up: BG3

- ByteDance publicou **BG3** em 2024 (ACM SIGMOD 2024) — "cost-effective and I/O efficient graph database". Evolução de ByteGraph focada em redução de custo.
- URL: https://dl.acm.org/doi/10.1145/3626246.3653373

## Aplicabilidade ao master-sindico

**Overkill pra qualquer horizonte próximo.** Nosso grafo (condomínio ↔ morador ↔ síndico ↔ empresa ↔ avaliação) cabe em Postgres com dezenas/centenas de milhões de edges max.

**Lição conceitual aproveitável**:

- **Modelar explicitamente o grafo** no domínio, mesmo persistindo em Postgres. Facilita queries como "empresa mais recomendada entre condomínios similares ao meu".
- **Recursive CTE** em Postgres cobre traversals de 2-3 níveis sem drama.
- **Reavaliar graph DB dedicado** (Neo4j / ArangoDB / Dgraph) apenas se o sub-produto **Vizinhança** escalar pra rede social com milhões de usuários e traversals profundos ("amigos dos amigos").

## Fontes complementares

- [Distributed Systems Blog — ByteGraph for TikTok](https://www.mydistributed.systems/2023/01/bytegraph-graph-database-for-tiktok.html) — resumo didático.
- [Semantic Scholar — ByteGraph](https://www.semanticscholar.org/paper/d1ae4ab5047489c2b010c7ce72262982ad66ad60) — metadata e citações.
- [VLDB Joyn — talk recording](https://vldb.joyn-us.app/talks/bytegraph-a-high-performance-distributed-graph-database-in-bytedance).

## Links

- [[_destilado#4. ByteGraph — graph DB in-house]]
- [[_moc]]
