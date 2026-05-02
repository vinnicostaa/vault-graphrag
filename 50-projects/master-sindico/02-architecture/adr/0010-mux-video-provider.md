---
title: ADR-0010 — Mux como video provider (terceirizar 100% pipeline)
type: adr
tags: [adr, video, mux, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0010 — Mux como video provider

## Status (atualizado 2026-04-27)

⚠️ **Atualização D-140 (2026-04-27)**: Mux é a **implementação inicial**, encapsulada como adapter de `IVideoProvider`. Cliente decidiu adicionar **Cloudflare Stream** como segundo adapter (alinhado D-138 Cloudflare-where-possible). Domínio NÃO conhece Mux SDK. Refactor obrigatório M1: port `IVideoProvider` domain-typed; `MuxProvider` em `infrastructure/providers/mux/` implementando interface; `CloudflareStreamProvider` em `infrastructure/providers/cloudflare_stream/` será adicionado M2. Ver [[../../STATE#D-140]].

### Status original

accepted — 2026-04-23 (herdado de legado)

## Contexto

Pipeline de vídeo é caro e complexo: upload resumable, transcode ABR, HLS manifest, DRM, signed URLs, CDN global, QoE monitoring. YouTube gastou bilhões operando este stack ([[../../13-research/youtube/_destilado]] §1). Master Síndico não tem budget para operar CDN própria nem pipeline de transcodificação.

## Decisão

**Mux** entrega 100% do pipeline até HLS (Mux-as-YouTube). Master Síndico só implementa: botão de upload client (Mux Upload tus.io), endpoint server-side que cria signed Direct Upload URL, webhook handler para `video.asset.ready`.

### Detalhes

- **Upload**: Mux Direct Upload URL (gerada server-side via SDK); cliente usa `@mux/upchunk` (tus.io resumable).
- **Transcode ABR**: 360p / 480p / 720p / 1080p (default Mux; 240p desnecessário em banda BR; 4K injustificado em condominial).
- **Codecs**: H.264 AVC baseline; H.265 opt-in quando volume justificar.
- **Segment / keyframe**: 6s segment / 2s keyframe (default Mux).
- **Playback**: Mux Player (`<mux-player>`); **não** custom hls.js.
- **Signed URLs JWT** — TTL 5-15 min; renovação client-side. Carrega `tenant_id + user_id` para audit.
- **DRM**: **NÃO** em M1. Signed URLs + lock 90d bastam.
- **Captions auto**: habilitar (acessibilidade + base para moderação OpenAI Moderation).
- **Mux Data (QoE)**: habilitar desde dia 1 — startup time, rebuffer ratio, bitrate.

### Lock 90 dias (diferenciador)

Abstract `Video` aggregate domain carrega VO `LockPeriod(90d)`. Moderação **pré-publish** obrigatória; pós-publish é tarde demais ([[../../13-research/youtube/_destilado]] §8).

## Consequências

### Positivas

- Zero operação CDN/transcoding.
- Mux Data QoE out-of-the-box.
- Signed URL JWT com `tenant_id` reforça tenant isolation.
- Custo estimado M1-M2: $100-500/mês (volume baixo).

### Negativas

- Vendor lock parcial — `IVideoProvider` port minimiza mas webhook schema Mux leaks se não filtrado.
- Custo escalonado: ~$0.05/min stored + transcode. Se volume explodir (M3+ com milhões de minutos), reavaliar Bunny.net ou Cloudflare Stream.
- Feature proprietária (Mux Data) só roda com Mux Player; trocar player perde QoE.

## Alternativas consideradas

1. **Bunny.net Stream** — CDN barata, ABR; QoE mais limitado.
2. **Cloudflare Stream** — $1/1000min; bom para M3+ se custo Mux escalar.
3. **AWS MediaConvert + CloudFront** — montar pipeline próprio; opex explode.
4. **YouTube Unlisted** — violaria LGPD (vídeos condominiais em servidor público US) + sem lock 90d.

## Referências

- [[../../13-research/youtube/_destilado]] TL;DR + §1 + §8
- [[../c4-components]] §3.5 content BC
- [[0024-moderation-cascade]] — moderação pré-publish
