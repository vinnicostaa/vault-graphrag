---
title: source-04 — Engineering the next generation of LinkedIn's Feed (LLM ranking)
type: source
tags: [research, linkedin, feed, ranking, llm, transformer, gpu]
source: https://www.linkedin.com/blog/engineering/feed/engineering-the-next-generation-of-linkedins-feed
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-feed-llm, generative-recommender]
---

# source-04 — LinkedIn Feed: LLM-based ranking

**URL**: https://www.linkedin.com/blog/engineering/feed/engineering-the-next-generation-of-linkedins-feed
**Complementares**: [LiRank paper arXiv](https://arxiv.org/html/2402.06859v1), [ByteByteGo LinkedIn Feed](https://blog.bytebytego.com/p/how-linkedin-feed-uses-llms-to-serve), [PPC Land summary](https://ppc.land/linkedin-rebuilds-its-feed-from-scratch-with-llms-and-gpu-powered-ranking/)
**Data**: 2025 (retrabalho profundo do feed)

## Por que essa fonte importa

Estado-da-arte de ranking ML em produção (2025-2026). Relevante pra calibrar **o quanto NÃO precisamos de ML** em M1/M2 e entender em que ponto ML faz sentido.

## Pontos extraídos

### Retrieval (geração de candidatos)
- Substituiu arquitetura multi-source por **pipeline único** com LLM embeddings.
- **Dual encoder**: LLM compartilhado processa prompts de member e de item; comparação por cosine similarity.
- Prompt library converte dados estruturados (metadata do post, autor, engagement counts, perfil do member) em texto processável.
- **Percentile bucket encoding** pra features numéricas — valores embrulhados em tokens especiais pra modelo aprender ordinalidade.
- Training: **InfoNCE loss** + 2 hard negatives por member → +3.6% recall.

### Ranking (Generative Recommender)
- Trata histórico de interações como **sequência**, processa 1000+ eventos.
- Transformer com causal attention (só atende posições anteriores).
- Interleaved post↔action pairs passam por múltiplas transformer layers.
- **Late fusion** combina output do transformer com context features (device, perfil embeddings, counts, affinity).
- Prediction head: **Multi-gate Mixture-of-Experts** com DCNv2 experts compartilhados, gates task-specific pra passive (click/skip/dwell) e active (like/comment/share) engagement.

### Infra de training
- Custom C++ data loader (elimina overhead Python).
- Custom CUDA kernels pra multi-label AUC.
- Fused optimizer.
- Checkpoint evaluation paralelizado.

### Infra de serving
- **Arquitetura CPU/GPU disaggregated** — feature processing em CPU, inference em GPU.
- **Shared context batching** — representação do histórico do user computada 1x, todos candidatos scored em paralelo.
- **GRMIS** — Flash Attention custom, 2× speedup.
- PyTorch-native serving, zero-copy, columnar layouts.
- Latência sub-second end-to-end, thousands ops/s.

### Serving pipeline online
1. Prompt generation nearline — captura updates reais em **minutos**.
2. Embedding generation — LLM inference em GPU cluster com batching configurável.
3. GPU-accelerated k-NN index → sub-50ms retrieval de milhões de posts.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#5-reputação-bronzediamante--determinística-linkedin-é-ml-heavy]].

- **NADA disso em M1/M2**. Escala (1.3B users) e investimento (GPU cluster, custom CUDA) não justificam em nenhum cenário razoável do master-sindico nos próximos 5 anos.
- **O que SIM aproveitar conceitualmente**: LinkedIn separa **retrieval** de **ranking**. OpenSearch é nosso retrieval; ranking deve ser **camada separada** e (ao contrário deles) **determinística**.
- **Features interpretáveis** sobrevivem ao ML: mesmo com LLM, LinkedIn indexa "identity" (profile structural) + "activity". Nossa reputação usa mesmas famílias — ganhamos explicabilidade que eles perderam.
- **Multi-gate MoE pra múltiplos objetivos**: no ranking do Connect Me (M3+?), se quisermos otimizar simultaneamente "empresa tecnicamente qualificada" + "bem avaliada" + "próxima" + "preço justo", pensar em score composto com pesos por objetivo (não single linear).

## Por que NÃO replicar

- **Custo**: GPU cluster dedicado + engenheiros de ML pra manter. Financeiramente inviável em M1/M2.
- **Dados**: LLM ranking precisa de bilhões de interações pra calibrar. Nosso volume não produz sinal.
- **Explicabilidade**: LGPD art. 20 exige explicar decisões automatizadas — modelo caixa-preta quebra.
- **Drift**: modelo 2025-2026 é estado-da-arte efêmero. Investimento descarta em 2 anos.

## Quando releer

- Se master-sindico ultrapassar 1M usuários ativos + precisar de feed de posts (sub-produto Conteúdo/Vídeos escalou).
- Como referência comparativa em ADR de ranking — sempre que surgir pressão pra "ML ranking", revisitar.

## Vizinhos

- [[_destilado]]
- [[source-03]] — Search federation (retrieval layer)
- [[source-09]] — PYMK (outro sistema de ranking ML)
