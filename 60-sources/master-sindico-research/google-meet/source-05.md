---
title: "Source 05 — Zoom Architecture (MMR, UDP/TCP/SSL fallback) + Daily.co"
type: research-source
tags: [research, master-sindico, v2, webrtc, zoom, daily, arquitetura, comparacao]
created: 2026-04-23
updated: 2026-04-23
source_type: engineering-blog
source_org: "Cometchat, Daily.co"
source_urls:
  - "https://www.cometchat.com/blog/zoom-video-technology-architecture"
  - "https://www.cs.princeton.edu/~ravian/publications/zoom_imc22.pdf"
  - "https://www.daily.co/"
  - "https://www.daily.co/blog/you-dont-need-a-webrtc-server-for-your-voice-agents/"
acessado_em: 2026-04-23
confiabilidade: média-alta (Zoom proprietário; Princeton IMC22 é paper acadêmico com medições)
---

# Source 05 — Zoom + Daily.co (para comparação)

## Resumo

Zoom é proprietário mas muito analisado; serve benchmark de patterns. Daily.co é competidor direto de LiveKit com infra global e DX parecida, útil pra validar nossa escolha.

## Pontos-chave Zoom

- **Transport fallback em cascata**: UDP → TCP → SSL/443. Aguenta redes corporativas hostis.
- **Arquitetura**: MMR (Multimedia Router) = SFU proprietário. Meetings agrupados em "Meeting Zones" por DC, duplicadas.
- **P2P em 2 participantes**; SFU em 3+.
- **QoS proprietário client-side**: detecta packet loss, latency, jitter, CPU e ajusta bitrate/resolução dinamicamente. FEC (Forward Error Correction) mitiga perda.
- **Paper Princeton IMC'22**: medições empíricas mostram Zoom extremamente resiliente em redes degradadas graças a FEC agressivo.

## Pontos-chave Daily.co

- Focus em DX developer: SDK + infra global com 75+ PoPs.
- **Dual mode**: WebRTC tradicional com servidor + "serverless" pra voice agents (peer direto quando faz sentido).
- E2EE + HIPAA BAA + GDPR out-of-the-box.
- Proposta de valor muito similar a LiveKit Cloud; escolha entre os dois é mais de preço/timbre do que técnica.

## Aplicação ao Master Síndico

- **Fallback UDP→TCP→TLS:443**: LiveKit já faz (TURN/TLS). Não há gap funcional vs Zoom nesse aspecto.
- **QoS LiveKit**: o SDK também mede e ajusta, mas não tão maduro quanto Zoom em 2026 — monitorar em produção.
- **Daily.co**: alternativa de vendor lock-in se LiveKit Cloud subir preço. API é distinta → migração custa; manter como "plano C" documentado.
- Zoom não é alternativa pro nosso caso (proprietário, sem self-host, preço e licenciamento inadequados pra SaaS B2B condominial BR).

## Citações relevantes

> "Zoom dynamically decides which protocol to use depending on how many participants are connected. [...] When a client connects to the server, it uses UDP. If that doesn't work, it tries via TCP, and if that fails, it tries via SSL."

> "For group meetings Zoom uses a Selective Forwarding Unit (SFU) architecture."

## URL(s)

- https://www.cometchat.com/blog/zoom-video-technology-architecture
- https://www.cs.princeton.edu/~ravian/publications/zoom_imc22.pdf
- https://www.daily.co/
- https://www.daily.co/blog/you-dont-need-a-webrtc-server-for-your-voice-agents/
