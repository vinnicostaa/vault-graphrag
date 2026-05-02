---
title: Powered by AI — Instagram's Explore recommender system
type: source
tags: [research, instagram, explore, ig2vec, embeddings, ann, session-based, source]
source: https://instagram-engineering.com/powered-by-ai-instagrams-explore-recommender-system-7ca901d2a882
created: 2026-04-23
updated: 2026-04-23
aliases: [source-07-ig2vec]
---

# source-07 — Powered by AI: Instagram's Explore recommender system

- **URL**: https://instagram-engineering.com/powered-by-ai-instagrams-explore-recommender-system-7ca901d2a882
- **Publicado**: ~2019 (post original Instagram Engineering; data exata não extraída nesta sessão, fetch falhou por TLS)
- **Autor**: Ivan Medvedev (Instagram)
- **Canal**: Instagram Engineering (Medium)
- **Tipo**: post técnico Instagram Engineering
- **Ressalva**: fetch direto da URL falhou por cert TLS intermediário; pontos-chave vieram de múltiplos posts derivados + citações no próprio `source-01`. Conceitos centrais são consistentes entre fontes.

## Sinopse

Post seminal descrevendo como o Explore usa embeddings de contas ("ig2vec") e K-NN pra fazer session-based recommendation.

## Pontos-chave

1. **ig2vec** — framework tipo word2vec. Trata sequência de IDs de conta que um usuário interage como "palavras numa frase". Aprende embedding de conta baseado em co-interação.
2. **Account embeddings** — cada conta vira vetor N-dim no espaço. Contas tematicamente similares ficam próximas.
3. **Session-based** — prediz próximas contas com que o usuário provavelmente interagirá naquela sessão.
4. **K-NN / ANN** — busca aproximada do vizinho mais próximo em tempo real sobre os embeddings.
5. **Candidate generation heurístico + ML** (mesmo ponto que `source-01`).

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §1.

- **Session-based recommendation** no Master Síndico é relevante **pra Connect Me** (durante uma sessão de busca, ranqueamento pode aprender com cliques da sessão) — mas apenas quando houver dados suficientes (M3+).
- **ig2vec-like embeddings** de empresas do Connect Me (baseado em co-contratação ou categorias similares) é aplicável em M4+ se o volume justificar.
- **K-NN / ANN service** (ex: pgvector no Postgres, Qdrant, Pinecone) vira opção quando partirmos pra recommendation ML.

## Confiança / datas

- Conceitos (ig2vec, session-based) — **alta confiança** (citados oficialmente em `source-01` 2023, que é Meta Engineering).
- Data exata do post 2019 — **média confiança** (não reconfirmada nesta sessão).
