---
title: Source — TikTok architecture overview (panorama engineering)
type: source
tags: [research, tiktok, architecture, overview, pulsar, redis, cdn, edge]
source: https://asyncthinking.com/p/tiktok-architecture-secrets
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-architecture-overview]
---

# Source — Panorama de arquitetura TikTok (análise agregada)

## Metadados

- **Async Thinking (2026)**: https://asyncthinking.com/p/tiktok-architecture-secrets
- **Medium (Rameshwar Singh, 2026)**: https://medium.com/@rameshwar.blog/video-processing-post-promotion-in-tiktok-77961062ae89
- **The New Stack — What makes TikTok's algorithms so effective**: https://thenewstack.io/what-makes-tiktoks-algorithms-so-effective/
- **Autoridade**: média. São análises secundárias/agregadoras; números são **estimativas de engenharia reversa**, não publicação oficial. Tratar como direcionais, não ground truth.

## Uso deste source

Este arquivo reúne números e afirmações de análises agregadoras pra dar **visão panorâmica** da infra TikTok. Onde houver conflito com fontes primárias ([[source-01-monolith-recsys]], [[source-02-bytegraph]], [[source-04-video-delivery]]), as **primárias prevalecem**.

## Escala agregada (estimativas)

- **350 horas** de vídeo uploaded/min (~**16.000 vídeos/min**).
- **1.2 bilhão** de usuários.
- **~34M vídeos/dia**.
- **Anual CDN cost**: ~US$ 1.3 bilhão (offset por receita de ads).

## Video pipeline

- Chunking: segments de **1-5 MB** via MPEG-DASH.
- Parallel processing em **200+ global PoPs**.
- **Edge GPU transcoding** em real-time.
- Estratégia de availability: **720p first** (imediato), full resolutions em background.
- Hot cache: **NVMe SSDs por ~48h**.
- Cold storage: "HDFS++" proprietário da ByteDance.

## CDN

- **8.000+ edge servers** em ISP hubs cacheiam top 0.1% viral content.
- **QUIC** reduz buffering em 75% vs TCP.
- **H.266/VVC** codec em produção (50% menos bandwidth que AV1 — **afirmação a verificar**).

## Recommendation (For You Page)

- **Sub-50ms** latency de recommendation desde app launch.
- Training infra: **10.000+ A100 GPUs** centralizados + **federated learning on-device**.
- Feedback loop: preference updates em **~90 segundos**.
- Features extracted: swipe velocity, scroll patterns, visual content (CLIP-like), audio, transcript.
- Pre-compute de 3 variantes de feed por usuário.

## Real-time infra

- **Pulsar** (Apache Pulsar) como backbone principal de message queue. Supostamente substitui Kafka em sistemas novos.
- **Kafka** mantido em alguns sistemas legados.
- **RedisCell** pra atomic counters (likes, views).
- **CRDTs** (Conflict-free replicated data types) resolvem colisões de comment em ~11ms.
- **30+ AI models** scan comments **durante composição** (proactive moderation).

## Disaster recovery

- **Daily intentional cluster crashes** (chaos engineering) — ~9.7s recovery average.
- WebAssembly-based UI como fallback em 22s deployment.
- Geographic traffic rerouting.
- Influencer prioritization durante outages (preservar receita alto-valor).

## Afirmações a tratar com cautela

- Números específicos (8.000 servers, 10.000 A100s, 11ms CRDT) são **estimativas de analistas externos**, não disclosed oficialmente.
- **H.266/VVC em produção** — plausível mas não confirmado em fonte primária ByteDance.
- **HDFS++ proprietário** — nome não aparece em papers públicos; pode ser designação interna.

## Aplicabilidade ao master-sindico

### Valor deste source: ordens de grandeza

O que importa aqui não é copiar números, mas entender **escala relativa**:

- ByteDance: bilhões de QPS, milhares de GPUs, PBs/dia.
- Master-sindico M1: centenas de QPS, CPU-only workloads, GBs/dia.
- **Razão ~10^5-10^6**. Toda conclusão "TikTok usa X" precisa ser filtrada por "precisamos disso na nossa escala?".

### Padrões conceituais aproveitáveis

- **Chaos engineering**: introduzir Litmus / Chaos Mesh / simple pod kills em ambiente staging. Valoroso já em M1.
- **Pulsar como message queue**: consideração legítima vs Kafka/NATS. Pulsar tem geo-replication nativa útil se multi-região (Brasil-centro-sul + nordeste). Mas **NATS JetStream é mais leve** e provavelmente suficiente pra M1-M3.
- **CRDTs**: sobrecomplicado pra master-sindico. Postgres transactional isolation + optimistic locking cobrem.
- **Pre-compute de variantes de feed**: interessante pra Connect Me — pré-computar top-20 empresas recomendadas por condomínio, 1x/dia, cache em Redis. Reduz latência de listagem.

### Chaos engineering recomendado

M1 já pode adotar:

- Kill aleatório de 1 pod por dia em staging.
- Network partition injeção entre DB e app 1x/semana.
- Simulação de expiração de token em meio de request.
- **Runbook de recovery** pra cada cenário.

## Links

- [[source-01-monolith-recsys]]
- [[source-02-bytegraph]]
- [[source-04-video-delivery]]
- [[source-03-tiktok-live]]
- [[_destilado]]
- [[_moc]]
