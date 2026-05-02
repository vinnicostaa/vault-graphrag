---
title: MOC — 13-research/instagram
type: moc
tags: [moc, master-sindico, v2, research, instagram, meta]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/instagram

Feed ranking, Stories infra, TAO, multi-master replication, moderação de conteúdo, DM/E2EE, anti-abuse, GraphQL, Haystack.

> Pasta da v2 (remontagem do Master Síndico). Paridade com scaffold canônico em `vault/40-templates/project-scaffold/`. Stub criado na Fase 0 (2026-04-23). Primeira leva de destilado escrita em 2026-04-23 (11 arquivos: 1 destilado + 10 sources + 1 extra Haystack).

## Arquivos

- [[_destilado]] — síntese PT-BR aplicável ao Master Síndico (ranking, TAO patterns, moderação cronograma, write-through vs write-behind, Connect Me não é chat, GraphQL não agora, multi-region prep)
- [[_aplicacoes-master-sindico]] — cruzamento nominativo com feeds sociais Master Síndico v2 (Timeline, Vizinhança, Banco de Talentos, Connect Me, Conteúdo); trade-off strong/eventual consistency; pull-vs-push fanout; moderação M1 cupons; top-5 M1 2026-05-08
- [[source-01]] — Scaling Instagram Explore Recommendations (Meta Eng, 2023-08-09) — funil 4 estágios, two-tower NN, MTML, value model
- [[source-02]] — Journey to 1000 models (Meta Eng, 2025-05-21) — Model Registry, launch automation, stability metric
- [[source-03]] — Instagram Stories ephemeral (fontes derivadas) — 24h lifecycle, CDN agressivo, prefetch; **NÃO adotar**
- [[source-04]] — TAO: Power of the Graph (Meta Eng, 2013-06-25 + USENIX ATC 2013) — objects/associations, write-through, eventual consistency
- [[source-05]] — Akkio data placement (Meta Eng, 2018-10-08) — μshards, multi-region locality, −40%/−50%/−50%
- [[source-06]] — GraphQL: A Data Query Language (Meta Eng, 2015-09-14) — origem 2012, Lee Byron, REST pain points
- [[source-07]] — Powered by AI: Instagram's Explore (Instagram Eng ~2019) — ig2vec, account embeddings, session-based K-NN
- [[source-08]] — Instagram content moderation (Meta docs + análise) — CNN + NLP + Rosetta OCR, confidence threshold, feedback loop
- [[source-09]] — Messenger E2EE default (Meta Eng, 2023-12-06) — Signal + Labyrinth, contra-exemplo pro Connect Me
- [[source-10]] — Fighting Spam with Haskell / Sigma (Meta Eng, 2015-06-26) — regra engine, Haxl, 1M req/s, anti-abuse
- [[source-11]] — Haystack photo storage (Meta Eng, 2009-04-30 + OSDI 2010) — needle aggregation, in-memory index, por que usar S3

## Temas → decisões sugeridas

| Tema | Decisão | ADR sugerida |
|---|---|---|
| Ranking Connect Me | determinístico ponderado M1, ML só M3+ | ADR-0001 |
| Cache pattern | write-through ABAC / write-behind feed | ADR-0002 |
| Connect Me = chat? | **não** — state machine estruturada | ADR-0003 |
| Moderação | manual M1, filtros M2, API externa M3, ML próprio M4+ | ADR-0004 |
| API style | REST + OpenAPI 3.1 (sem GraphQL) | ADR-0005 |
| Multi-region | BR-only M1-M2; sharding por tenant prepara futuro | ADR-0006 |
| Storage de mídia | S3/MinIO commodity (sem Haystack próprio) | ADR-0007 |

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../linkedin/_moc]] — marketplace reputacional (complementar pra Connect Me)
- [[../tiktok/_moc]] — feed de vídeos curtos (complementar pro feed do condomínio)
- [[../beyond-corp/_moc]] — identity-centric / ABAC
