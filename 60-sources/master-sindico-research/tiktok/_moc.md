---
title: MOC — 13-research/tiktok
type: moc
tags: [moc, master-sindico, v2, research, tiktok]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/tiktok

Research sobre arquitetura TikTok/ByteDance aplicável ao master-sindico. Cobre For You recsys (Monolith), ByteGraph, scheduler Gödel, vídeo short-form, TikTok LIVE, moderação multilíngue e debate DSA/transparência. Populado 2026-04-23 na Fase 3.

## Arquivos

- [[_destilado]] — síntese executiva: o que aplicar, o que é overkill, tabela-resumo por área.
- [[_aplicacoes-master-sindico]] — Top-5 aplicações no MS v2 (recsys BT M3+, Postgres vs ByteGraph, Livekit vs TikTok LIVE, cascade moderation, transparência DSA/PL 2.630) + anti-catálogo.
- [[source-01-monolith-recsys]] — paper Monolith (arXiv 2209.07663): collisionless embedding + online training.
- [[source-02-bytegraph]] — paper VLDB 2022: graph DB ByteDance, 10^10 vértices, 3 workloads.
- [[source-03-tiktok-live]] — pilha TikTok LIVE: RTMP ingest + LL-HLS/WebRTC delivery; comparação com LiveKit.
- [[source-04-video-delivery]] — arXiv 2410.17073: CDN+PCDN híbrido, chunking, codecs; paridade Mux.
- [[source-05-content-moderation]] — MLLM cascade 2-stage TikTok; aplicação BR-PT com LGPD Art. 20.
- [[source-06-godel-scheduler]] — CNCF 2024: scheduler unificado online/offline (substituto funcional de "ByteTask").
- [[source-07-transparency-dsa]] — TAC + DTC + violação preliminar DSA fev/2026; implicações LGPD/PL 2.630.
- [[source-08-architecture-overview]] — panorama agregado (Pulsar, Redis, chaos engineering, QUIC); números são estimativas.

## Destinos na migração (Fase 6)

- [[_destilado]] → `02-architecture/research-inspirations.md` (seção TikTok).
- Princípios de transparência → `08-security/beyond-corp-references.md` + política de decisões automatizadas.
- Specs de vídeo → `03-subprojects/banco-talentos/video-spec.md` e `03-subprojects/conteudo-videos/ingestion-spec.md`.
- Padrões de moderação → `03-subprojects/banco-talentos/moderation-pipeline.md`.

## Vizinhos

- [[../_moc]] — MOC raiz de 13-research
- [[../instagram/_moc]] — comparar padrões de social feed
- [[../youtube/_moc]] — comparar vídeo long-form
- [[../netflix/_moc]] — comparar streaming + recsys
- [[../../CLAUDE]] — contrato do agente
- [[../../STATE]] — decisões vivas
