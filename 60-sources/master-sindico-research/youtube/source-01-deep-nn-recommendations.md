---
title: "Source 01 — Deep Neural Networks for YouTube Recommendations (Covington et al., 2016)"
type: source
tags: [research, youtube, recsys, ml, deep-learning, connect-me]
source: https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/
created: 2026-04-23
updated: 2026-04-23
aliases: [youtube-recsys-2016]
---

# Source 01 — Deep Neural Networks for YouTube Recommendations

## Metadados

- **Autores**: Paul Covington, Jay Adams, Emre Sargin (Google)
- **Ano**: 2016
- **Publicação**: Proceedings of the 10th ACM Conference on Recommender Systems (RecSys '16)
- **URL canônica**: https://research.google/pubs/deep-neural-networks-for-youtube-recommendations/
- **PDF**: https://research.google.com/pubs/archive/45530.pdf

## Resumo da arquitetura

Sistema em **duas etapas** (two-stage funnel clássico de IR):

1. **Candidate Generation (Retrieval)**
   - Rede neural profunda que mapeia milhões de vídeos → algumas centenas de candidatos relevantes por usuário.
   - Entrada: embeddings de watch history, search tokens, demographics, contextuais (age, geolocation).
   - Saída: top-N via approximate nearest neighbor (ANN) sobre embeddings.

2. **Ranking**
   - Outra DNN que reordena os candidatos usando features mais ricas (impressão, dispositivo, tempo desde último watch, idade do vídeo).
   - Otimiza proxy de engajamento (expected watch time) — não CTR puro (evita clickbait).

## Desafios-chave declarados pelos autores

- **Scale**: bilhões de usuários, bilhões de vídeos → modelos distribuídos.
- **Freshness**: catálogo muda o tempo todo; modelo precisa generalizar pra vídeos novos.
- **Noise**: user behavior é ruidoso; metadata do vídeo é pobre/inconsistente.

## Insights práticos

- **Example age feature**: adicionar "idade do vídeo no momento do training example" como feature resolve bias de recência.
- **Negative sampling**: tratar recomendação como classificação multi-classe extrema com sampled softmax.
- **Watch time como alvo** (weighted logistic regression) em vez de click — combate clickbait.
- **Feature engineering manual ainda importa**, mesmo com deep learning (frequência, tempo desde, etc.).

## Relevância para Master Síndico

- **Connect Me** (marketplace de empresas/prestadores) é um problema de recomendação:
  - Pool pequeno (empresas cadastradas por região), mas com matching rico (categoria, histórico, rating, distância, preço).
  - Não precisa DNN — o funil two-stage serve como **modelo mental**: retrieval (filtro determinístico por região+categoria) + ranking (score heurístico/regras).
  - MVP: ranking = score composto (rating × recência × distância × disponibilidade). ML só quando tiver volume suficiente de interações pra treinar.
- **Conteúdo/Vídeos**: recomendação de vídeo pode copiar o padrão — mas o volume do master-sindico não justifica DNN; regras heurísticas + BM25/embeddings via OpenSearch resolvem.
- **Lição principal**: otimize pro **proxy certo** (ex: "contatou empresa" > "clicou no card"), evite vanity metrics.

## O que NÃO aplicar

- Complexidade de DNN com bilhões de parâmetros — massive overkill pra nosso volume.
- Embeddings aprendidos fim-a-fim — pode começar com features estruturadas.

## Links internos

- [[_destilado]]
- [[../../02-architecture/patterns]] (futuro)
- [[../_moc]]
