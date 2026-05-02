---
title: "Source 08 — YouTube Live ingestion protocols (RTMP/RTMPS/HLS/DASH)"
type: source
tags: [research, youtube, live-streaming, rtmp, rtmps, hls, dash, assembleias]
source: https://developers.google.com/youtube/v3/live/guides/ingestion-protocol-comparison
created: 2026-04-23
updated: 2026-04-23
aliases: [youtube-live-protocols]
---

# Source 08 — YouTube Live ingestion protocols

## Metadados

- **Fonte oficial**: https://developers.google.com/youtube/v3/live/guides/ingestion-protocol-comparison

## Tabela oficial YouTube — protocolos suportados

| Protocolo | Encrypted | Codecs | Latência | Recomendado pra |
|-----------|-----------|--------|----------|-----------------|
| **RTMP**  | Não | H.264 | Normal / Low / Ultra-low | Legacy; ainda comum em OBS |
| **RTMPS** | Sim (TLS) | H.264 | Normal / Low / Ultra-low | Default recomendado |
| **HLS**   | Sim (TLS) | H.264, H.265 | Higher (segment-based) | 4K / HDR |
| **DASH**  | Sim (TLS) | H.264, VP9 | Higher | 4K streaming premium |

## Latency tiers YouTube

- **Normal**: 30-60s delay
- **Low-latency**: 5-15s
- **Ultra-low-latency**: 2-5s (sacrifica buffering, reconnect)

## Tradeoffs explícitos

- **RTMPS** vence pra real-time (menor latência end-to-end).
- **HLS/DASH** vence pra qualidade (codecs melhores, adaptive) mas pagam em latência (segments de 2-6s).
- **Low-Latency HLS (LHLS)**: compromise — partial segments ~200ms reduzem latência pra 2-4s mantendo HLS-compat.

## Relevância para Master Síndico — sub-produto Assembleias Live

### Requisitos Assembleias Live (legado herdado)

- Síndico + condôminos em call (realtime bidirecional) — estilo Google Meet/Zoom.
- Gravação da assembleia pra ata posterior.
- Lista de participantes + votação + chat.

**Isto NÃO é "YouTube Live" puro** — é mais um **produto conferencing + broadcast**. Portanto:

### Arquitetura aplicável

Opção A — **Daily.co / LiveKit / 100ms** (WebRTC SFU-as-a-service):
- Latência <500ms (bidirecional real-time).
- Gravação em MP4/HLS exportada.
- SDK React/React Native pronto.
- Este é o match correto pra assembleia.

Opção B — **Mux Live** (RTMP ingest → HLS delivery):
- One-to-many broadcast (síndico fala, condôminos assistem).
- Latência 6-18s (HLS) ou 2-4s (LL-HLS).
- Gravação automática → VoD asset.
- Serve só se assembleia for **monológica** (síndico apresentando, Q&A via chat text).

### Proposta

- **Assembleia participativa (voto)**: **WebRTC SFU** (LiveKit self-hosted ou Daily managed). Latência real-time é requisito pra votação ordenada e debate.
- **Broadcast de apresentação institucional (não-interativa)**: Mux Live basta.
- **Gravação**: toda assembleia gera **VoD asset pós-call** → aplica lock 90d (decisão D-### pendente).

### Protocolos suportados que importam pra nós

- **WebRTC** (navegador → SFU) — pra participantes ao vivo.
- **RTMPS → Mux** (se usar Mux Live pra broadcast) — encrypted, H.264, suportado por OBS.
- **HLS** (entrega aos viewers + VoD final) — padrão já escolhido no sub-produto Conteúdo/Vídeos.

### Lição YouTube aplicável

1. **Separar ingest de delivery**: protocolo de entrada (RTMPS) ≠ protocolo de saída (HLS). Isto permite trocar um sem afetar outro.
2. **Tiers de latência são trade-off explícito** — documentar no produto qual modo escolhido por cenário. Sindico escolhe "debate ao vivo" (real-time) vs "apresentação" (low-latency).
3. **Codec limitation awareness**: RTMP só aceita H.264. Se precisar H.265/VP9, precisa HLS/DASH ingest (ou transcode server-side). Nosso caso: H.264 basta.

## O que NÃO aplicar

- Múltiplos ingestion protocols simultâneos — 1 bem escolhido >> 3 mal implementados.
- DASH ingest — Mux Live não suporta; HLS/RTMPS bastam.

## Links internos

- [[source-05-mux-hls-best-practices]]
- [[source-07-ingestion-pipeline]]
- [[_destilado]]
- [[../_moc]]
