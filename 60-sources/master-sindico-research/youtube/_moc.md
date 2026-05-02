---
title: MOC — 13-research/youtube
type: moc
tags: [moc, master-sindico, v2, research,youtube]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/youtube

Ingestion, transcoding pipeline, CDN, recommendation, Content ID.

> Pasta da v2 (remontagem do Master Síndico). Paridade com scaffold canônico em `vault/40-templates/project-scaffold/`. Stub criado na Fase 0 (2026-04-23). Conteúdo entra nas Fases 3/4 conforme plano em `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`.

## Arquivos

- [[_destilado]] — síntese aplicada ao master-sindico (8 fontes consolidadas, decisões propostas, antipadrões)
- [[source-01-deep-nn-recommendations]] — Covington et al. 2016, two-stage recsys (Google Research)
- [[source-02-content-id]] — fingerprinting de copyright YouTube (doc oficial support.google.com)
- [[source-03-media-cdn]] — Google Cloud Media CDN / YouTube edge network (cloud.google.com)
- [[source-04-vitess-mysql]] — sharding MySQL em escala YouTube (vitess.io + CNCF)
- [[source-05-mux-hls-best-practices]] — player, codecs, ABR ladder, segment, DRM (mux.com eng blog)
- [[source-06-rekognition-moderation]] — AWS Rekognition vs OpenAI Moderation vs Hive, pricing 2026
- [[source-07-ingestion-pipeline]] — upload → transcode → storage (bytebytego + VCU paper Google)
- [[source-08-live-streaming]] — RTMP/RTMPS/HLS/DASH ingest protocols (developers.google.com)

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
