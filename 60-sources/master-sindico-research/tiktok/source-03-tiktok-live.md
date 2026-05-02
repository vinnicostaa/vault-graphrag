---
title: Source — TikTok LIVE architecture (RTMP + LL-HLS + WebRTC)
type: source
tags: [research, tiktok, live-streaming, webrtc, ll-hls, rtmp, architecture]
source: https://dev.to/conquerym/tiktok-live-streaming-feature-technical-details-and-architecture-1n0j
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-live]
---

# Source — TikTok LIVE: arquitetura e protocolos

## Metadados

- **Dev.to (análise arquitetural)**: https://dev.to/conquerym/tiktok-live-streaming-feature-technical-details-and-architecture-1n0j
- **Mux — low-latency guide**: https://www.mux.com/articles/low-latency-video-streaming-a-complete-guide-with-definitions-examples-and-more
- **Cloudinary — LL-HLS vs CMAF vs WebRTC**: https://cloudinary.com/guides/live-streaming-video/low-latency-hls-ll-hls-cmaf-and-webrtc-which-is-best
- **Medium (bugfree.ai) — system design**: https://medium.com/@bugfreeai/tiktok-software-engineer-interview-design-a-scalable-live-streaming-platform-26e060cedcc3
- **Autoridade**: Mux + Cloudinary são fontes autorais (CDN/streaming); dev.to e Medium são análises secundárias — úteis para padrões, não pra números exatos de TikTok.

## Pilha técnica (consensus das fontes)

### Ingestão
- **RTMP** (Real-Time Messaging Protocol) — padrão da indústria pra ingest de streamer.
- **SRT** (Secure Reliable Transport) pode ser usado em casos profissionais.
- Stream entra no **origin server** que faz transcoding single-pass.

### Delivery (duas camadas simultâneas)
- **LL-HLS (Low-Latency HLS)** — **2-5s de latência**. Distribuição massiva via CDN. Chunks pequenos (sub-segments CMAF) permitem menor latência que HLS tradicional (~30s).
- **WebRTC** — **200-500ms de latência**. Usado pra interatividade real-time (co-host, call-in, bidding em live shopping).
- **Hybrid delivery**: WebRTC pros interativos, LL-HLS pra passive viewers em escala.

### Infra de distribuição
- **Origin → regional nodes → edge servers** (tree distribution).
- Edge servers cacheiam segments **30-60s**.
- CDNs próprias + comerciais (Akamai/Cloudflare mix).

### Canais paralelos
- **WebSocket** pra chat e gifts (comunicação real-time leve).
- **Message queue (Pulsar ou Kafka)** pra eventos de like/gift/follow durante live.
- **Redis** pra contadores atômicos (viewers count, gift totals).

## Latência: tabela comparativa

| Protocolo | Latência típica | Uso |
|---|---|---|
| WebRTC | 200-500ms | Interativo 1↔1 ou conferência |
| LL-HLS | 2-5s | Broadcast 1→muitos, interativo leve |
| HLS tradicional | 15-30s | Broadcast 1→muitos, não-interativo |
| DASH | 5-20s | Similar a HLS |
| CMAF + chunked transfer | 2-4s | Otimização de HLS/DASH |

## Aplicabilidade ao master-sindico

### Assembleias virtuais (M3) — já decidido: LiveKit/WebRTC

- **Diferenças de caso de uso**:
  - TikTok LIVE = broadcast 1 → N (milhões).
  - Assembleia = conversacional N ↔ N (10-500 participantes, todos podem falar).
- **Requisitos diferentes**:
  - Latência obrigatória < 400ms (quórum, voto ao vivo, deliberação).
  - Participantes têm audio + vídeo (não só viewers).
  - Precisa **multi-party SFU**, não CDN tree.
- **LiveKit é a escolha certa**. Nenhum aprendizado de TikTok LIVE muda isso.

### Futuro: broadcast de síndico → moradores (M4+?)

- Se master-sindico oferecer **transmissão de evento do condomínio** (ex: festa, obra, comunicado institucional) com 1 streamer → 500-5000 viewers passivos:
  - **Mux Live** (já temos contrato Mux) entrega LL-HLS + CDN. Custo linear no bandwidth.
  - **Ingestão**: RTMP do síndico (via OBS ou app mobile) → Mux → HLS no player do morador.
  - Não faz sentido montar infra própria — Mux cobre.

### Padrões conceituais aproveitáveis

- **Separar canal de vídeo do canal de eventos** (chat/gifts via WebSocket + Redis; não empacotar no stream). Diretamente aplicável a assembleias (voto e chat em canais separados do audio/vídeo).
- **Edge cache de segments 30-60s**: se master-sindico fizer replay de assembleias gravadas, Mux/CDN comum cobre.
- **Hybrid delivery** (WebRTC + LL-HLS): possível híbrido em assembleias — participantes ativos via WebRTC, observadores passivos via LL-HLS. Mas complexidade extra; postergar.

## Fontes complementares

- [Design Gurus — TikTok system design Q&A](https://www.designgurus.io/answers/detail/tiktok-system-design-interview-questions-key-concepts-and-architecture-insights) — contexto de perguntas de entrevista; útil pra padrões.
- [Mux — RTMP streaming guide](https://www.mux.com/articles/rtmp-streaming-protocol).
- [Get Stream — protocol comparison](https://getstream.io/blog/protocol-comparison/).
- [Red5 — 6 approaches to low latency](https://www.red5.net/blog/6-approaches-to-low-latency-video-streaming-which-is-most-effective/).

## Caveat de autoridade

Fontes secundárias (dev.to, Medium, Design Gurus) reconstroem arquitetura por análise externa — não publicação oficial TikTok. Consenso nos padrões (RTMP + LL-HLS + WebRTC) é confiável porque são padrões da indústria de streaming; **os números específicos de TikTok (ex: tamanho exato de chunk, PoPs) devem ser lidos como estimativas**, não ground truth.

## Links

- [[_destilado#5. Live streaming global (TikTok LIVE)]]
- [[_moc]]
