---
title: Mux — overview
type: note
tags: [provider, mux, video, overview, streaming]
source: https://docs.mux.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Mux — overview

Plataforma de vídeo managed: absorve a complexidade de encoding, storage, CDN e analytics, e entrega um fluxo `upload → assinatura de URL → player` com poucas chamadas HTTP. Construída sobre HLS + CMAF; não expõe buckets nem transcoders diretamente.

---

## Arquitetura conceitual

Todo conteúdo no Mux vive em um grafo de três nós encadeados:

```
Asset  ──►  Playback ID  ──►  Delivery URL
(privado)    (público)         (HLS/MP4)
```

- **Asset** — representação interna do vídeo. Tem `id` (UUID Mux), `status` (`preparing` → `ready` → `errored`), `duration`, `aspect_ratio`, `tracks` (video/audio/text), `passthrough` (string livre que o app define pra correlacionar com o domínio).
- **Playback ID** — identificador público atrelado ao Asset. Um Asset pode ter **N** Playback IDs, cada um com `policy: public | signed`. O ID entra na URL HLS/MP4, mas não é a chave primária do vídeo.
- **Delivery URL** — `https://stream.mux.com/{playback_id}.m3u8` (HLS) ou `.mp4` (MP4 render). Quando `policy=signed`, exige query `?token=<JWT>`.

**Corolário**: nunca usar `playback_id` como chave primária no DB do app. Usar `asset_id` (privado) e manter `playback_ids` como lista filha. Detalhes em [[antipatterns]].

---

## Modos de ingestão

| Modo | Fluxo | Caso típico |
|---|---|---|
| **VOD (upload)** | Cliente envia arquivo → Mux encoda → `asset.ready` via webhook | Videoaula, replay, clipe curto |
| **VOD (URL)** | Mux busca o arquivo via HTTP em URL pública | Migração de legado, import batch |
| **Live Stream** | RTMP ingest em `global-live.mux.com` → playback HLS gerado durante transmissão | Webinar, assembleia, transmissão ao vivo |
| **Simulcast** | Mux replica RTMP da live para N destinos externos | Live que vai simultaneamente para YouTube + Twitch |

**Live → VOD**: toda live cria automaticamente um Asset ao encerrar (`live_stream.idle`), permitindo replay sem ingest extra.

---

## Mux Data (analytics)

Produto separado (cobrado à parte) que instrumenta o player e manda telemetria para o backend do Mux. Entrega:

- **QoE metrics**: rebuffer ratio, startup time, exits before video start
- **Heatmap** de engagement segundo a segundo
- **Breakdown** por região, dispositivo, CDN
- **Alertas** em degradação de experiência

Setup: o Mux Player já vem instrumentado; players terceiros (Video.js, Shaka, ExoPlayer) usam SDKs `mux-embed` / `mux-stats-*`.

---

## Mux Player

Player oficial, disponível em:

- **Web** — `@mux/mux-player` (web component `<mux-player>`) e `@mux/mux-player-react`
- **Flutter** — pacote community `mux_video_player` sobre `video_player` do Flutter team
- **iOS/Android** — `@mux/mux-player-ios` / `@mux/mux-player-android`

Já integra assinatura JWT (prop `playback-token`), Mux Data, CMCD e fallback MP4. Suporta `stream-type="on-demand" | "live" | "ll-live"` (low-latency live).

---

## Modelo de preço (resumido)

Três dimensões cobradas independentemente:

| Dimensão | Como cobra | Observação |
|---|---|---|
| **Encoding** | $ / minuto de vídeo processado | Cobrado uma vez por Asset |
| **Storage** | $ / minuto armazenado / mês | Pode reduzir gerando `master access: none` |
| **Delivery** | $ / minuto entregue ao viewer | Depende do resolution tier (SD/HD/FHD) |
| **Live** | $ / minuto de ingest ativo | Segue contando enquanto a live está `active` |

**Sandbox**: chaves `development` vêm com quota grátis mensal, mas o vídeo gera watermark Mux e TTL reduzido. Produção exige toggle do Environment no dashboard.

**Mux Data**: por view, com tiers distintos. Não compartilha budget com o core VOD/Live.

---

## Concorrentes — quando cada um vence

| Concorrente | Vence quando | Perde quando |
|---|---|---|
| **Cloudflare Stream** | vídeos curtos, social, bundle com CDN Cloudflare, preço agressivo | sem analytics profundos, sem simulcast nativo |
| **AWS IVS** | tudo já está na AWS, low-latency (<5s) chat integrado | requer compor MediaConvert + S3 + CloudFront pra VOD |
| **Livepeer** | self-host / edge compute, preço escalável, cripto-friendly | operação mais crua, tooling menos maduro |
| **Vimeo OTT** | quer OTT white-label pronto (assinaturas, paywall) | precisa de API flexível |
| **Bitmovin** | transcoding customizado, DRM avançado | quer opinionated managed |

Mux ganha quando: **live de alta qualidade com analytics first-class + VOD com replay automático + DX de SDK amigável**, aceitando pagar prêmio em delivery cost. Para realtime peer-to-peer (conferência), ver [[../livekit/_moc]].

---

## Operações típicas do dia-a-dia

1. **Criar Asset de VOD** — `POST /video/v1/assets` com `input` (URL) ou `POST /video/v1/uploads` (upload direto)
2. **Criar Live Stream** — `POST /video/v1/live-streams`, anotar `stream_key` (RTMP) e `playback_ids`
3. **Assinar playback** — gerar JWT HS256 com key privada do Mux
4. **Consumir webhook** — processar `video.asset.ready` para marcar disponível no DB
5. **Listar/deletar assets** — `GET|DELETE /video/v1/assets/{id}`

Fluxo detalhado em [[integration-guide]] e patterns de produção em [[patterns]].

---

## Links

- [[_moc]]
- [[../_moc]]
- [[integration-guide]]
- [[patterns]]
- [[antipatterns]]
- [[../livekit/_moc]] — concorrente (realtime WebRTC)
