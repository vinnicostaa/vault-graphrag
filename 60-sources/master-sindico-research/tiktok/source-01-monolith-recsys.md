---
title: Source — Monolith (ByteDance recsys real-time, arXiv 2209.07663)
type: source
tags: [research, tiktok, bytedance, recsys, online-training, embedding, collisionless, tensorflow]
source: https://arxiv.org/abs/2209.07663
created: 2026-04-23
updated: 2026-04-23
aliases: [monolith-paper]
---

# Source — Monolith: Real-Time Recommendation System With Collisionless Embedding Table

## Metadados

- **URL**: https://arxiv.org/abs/2209.07663
- **PDF**: https://arxiv.org/pdf/2209.07663
- **Venue**: ORSUM@ACM RecSys 2022 (5th Workshop on Online Recommender Systems and User Modeling)
- **Autores**: Zhuoran Liu, Leqi Zou, Xuan Zou, Caihua Wang, Biao Zhang, Da Tang, Bolin Zhu, Yijie Zhu, Peng Wu, Ke Wang, Youlong Cheng (ByteDance)
- **Código**: https://github.com/bytedance/monolith (Apache 2.0)
- **Autoridade**: alta — paper peer-reviewed + repo oficial ByteDance + deployed em BytePlus Recommend.

## Problema que resolve

Frameworks genéricos (TensorFlow, PyTorch) são desenhados pra **dense features + batch training**. Sistemas de recomendação têm **sparse features dinâmicas** (IDs de usuário, vídeo, etc. — novos a cada segundo) e precisam aprender feedback em **tempo real** (trend que surge/morre em horas).

## Contribuições principais

1. **Collisionless embedding table** — usa **cuckoo hashing** pra garantir que cada feature ID tenha embedding único. Hash collisions degradam qualidade do modelo; cuckoo hashing resolve sem custo de memória proibitivo.
2. **Otimizações de memória**: embeddings expiráveis (features antigas somem) + frequency filtering (IDs com poucas ocorrências não ganham embedding).
3. **Arquitetura online-training production-ready** com alta tolerância a falhas. Paper demonstra que **trade-off reliability ↔ freshness** é aceitável nesse domínio.

## Arquitetura

- **Worker-ParameterServer (PS)** em TensorFlow distribuído. Workers computam gradientes; PS armazena parâmetros. Paralelização horizontal.
- **Pipeline de dados**:
  - User interactions → **Kafka queues** → "joiner" component
  - Joiner faz dois trabalhos: (a) matching interaction ↔ user/item embedding, (b) **negative sampling** (desequilíbrio positivo/negativo).
  - Dados processados vão simultaneamente pra (1) training workers (online) e (2) database (batch futuro).
- **Dois modelos distintos coexistindo**:
  - **Serving model**: live, alta disponibilidade, atende inferência.
  - **Training model**: continuamente atualizado com novas interações.
- **Update propagation**:
  - **Sparse features (embeddings)**: atualizadas a cada minuto. Só os IDs que mudaram são sincronizados — **não substitui tudo**.
  - **Dense features (pesos do DNN)**: atualizados semanalmente/mensalmente.

## Dual-mode training

- **Batch**: processa milhões de linhas, roda em dias. Usado pra re-treino periódico.
- **Online**: atualiza só embeddings afetados por interação recente. Roda em minutos. Handle concept drift.

## Concept drift handling

- Dynamic hash tables que **removem automaticamente embeddings antigos** após TTL.
- Embeddings de features que não aparecem mais somem → memória livre pra novos IDs.

## Números / escala

- Paper não publica números absolutos. Usa benchmarks proprietários da BytePlus.
- Monolith roda em produção no **BytePlus Recommend** (plataforma B2B de recsys da ByteDance).

## Aplicabilidade ao master-sindico

- **Conceitual** aplicável ao Connect Me (ranking empresas) e feed de descoberta de vídeos.
- **Implementação overkill**: não temos volume pra justificar parameter server, Kafka, Flink, cuckoo hashing.
- **Padrão reaproveitável**: **separar serving model de training model** + atualizar só deltas. Aplicável a qualquer ranking/scoring no master-sindico.

## Fontes complementares

- [Aaron Abraham — TikTok Monolith System](https://www.aaronabraham.ca/technical-writing/tiktok-monolith-system) — análise didática do paper.
- [Haneul Kim — Paper review](https://haneulkim.medium.com/paper-review-monolith-tiktoks-real-time-recommender-system-72b90bece653) — resumo com diagramas.
- [MarkTechPost — Monolith summary](https://www.marktechpost.com/2022/11/14/computer-science-researchers-at-bytedance-developed-monolith-a-collisionless-optimised-embedding-table-for-deep-learning-based-real-time-recommendations-in-a-memory-efficient-way/).
- [Hacker News discussion](https://news.ycombinator.com/item?id=35573624) — debate de praticantes.

## Links

- [[_destilado#1. For You Page (Monolith) — recsys real-time]]
- [[_moc]]
