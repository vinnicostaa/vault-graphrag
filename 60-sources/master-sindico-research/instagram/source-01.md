---
title: Scaling the Instagram Explore recommendations system
type: source
tags: [research, instagram, meta, recommendation, ranking, ml, two-tower, mtml, source]
source: https://engineering.fb.com/2023/08/09/ml-applications/scaling-instagram-explore-recommendations-system/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-01-instagram-explore]
---

# source-01 — Scaling the Instagram Explore recommendations system

- **URL**: https://engineering.fb.com/2023/08/09/ml-applications/scaling-instagram-explore-recommendations-system/
- **Publicado**: 2023-08-09
- **Autor/Canal**: Engineering at Meta (blog oficial)
- **Tipo**: post técnico de engenharia

## Sinopse

Descreve a arquitetura de 4 estágios do sistema de recomendação do Instagram Explore, que serve centenas de milhões de visitantes diários e escolhe centenas de itens de um pool de bilhões.

## Pontos-chave

1. **Funil multi-estágio** — Retrieval → First-stage ranker → Second-stage ranker → Final reranker.
2. **Retrieval** combina heurísticas (trending) + ML (embeddings, two-tower NN) + sources real-time + sources pré-computadas (offline).
3. **Two Towers NN** — duas redes neurais independentes (usuário, item). Permite pré-computar embeddings do item offline; embedding do usuário é gerado on-the-fly com features frescas. ANN (approximate nearest neighbor) service faz lookup.
4. **First-stage ranker** — lightweight, cacheable, produz milhares de candidatos.
5. **Second-stage ranker** — MTML (Multi-Task Multi-Label) neural net, prediz simultaneamente probabilidade de múltiplos eventos (click, like, see-less, comment, save).
6. **Value model linear** combina predições: `EV = W_click·P(click) + W_like·P(like) − W_see_less·P(see_less) + …`. Pesos são tunáveis.
7. **Sinais de aversão** (ex: "see less", "not interested") entram como labels negativos — não só engajamento positivo.
8. **Filtering rule-based** remove conteúdo sexual, reportado, baixa qualidade antes de entrar em training/inference.
9. **Caching + pré-computação** é o que torna viável servir em latência aceitável.

## Aplicabilidade ao Master Síndico

Ver `_destilado.md` §1. Resumo:

- Funil completo **não** se aplica (escala errada).
- **Separação retrieval vs ranking** (pattern arquitetural) se aplica sim — tanto no feed do condomínio quanto no Connect Me.
- **Sinais de aversão** (reportar, cancelar contrato, avaliação ruim) devem ser coletados desde M1.
- **Value model linear ponderado** é aplicável ao ranking Connect Me determinístico.

## Citações

> "In most large-scale recommender systems, the retrieval stage consists of multiple candidates' retrieval sources, with the main purpose of a source being to select hundreds of relevant items from a media pool of billions of items."

> "Ranking in a high load system is usually divided into multiple stages that gradually reduce the number of candidates from a few thousand to few hundred."

## Confiança / datas

- Data confirmada via fetch direto (2023-08-09) — **alta confiança**.
- Fonte primária Meta Engineering.
