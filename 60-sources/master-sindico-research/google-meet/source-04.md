---
title: "Source 04 — Escalabilidade: LiveKit Distributed Mesh + Jitsi Octo"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, jitsi, escalabilidade, mesh, octo]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial+engineering-blog
source_org: "LiveKit, Jitsi"
source_urls:
  - "https://livekit.com/blog/scaling-webrtc-with-distributed-mesh/"
  - "https://jitsi.guide/blog/jitsi-scaling-performance-octo-jvb-clustering/"
  - "https://github.com/jitsi/jitsi-videobridge"
  - "https://jitsi.github.io/handbook/docs/architecture/"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 04 — Escalabilidade SFU: LiveKit Mesh + Jitsi Octo

## Resumo

Como dois grandes SFUs open-source (LiveKit e Jitsi) escalam pra dezenas de milhares de participantes: LiveKit com "distributed mesh" multi-home no Cloud, Jitsi com protocolo Octo cascateando bridges por região.

## Pontos-chave LiveKit

- **Cloud multi-home**: sessão única pode ser roteada por múltiplos servidores em DCs distintos; estado compartilhado via pub-sub (não DB).
- **Relay server-to-server**: protocolo custom FlatBuffers sobre RTP; relay trata o outro nó como se fosse um participante.
- **Capacidade alvo**: "media server within 100ms of anyone in the world" via múltiplos PoPs.
- **ICE restart <1s**: migração transparente de participante entre servidores em falhas, sem reconnect visível.
- **Estado leve**: ~100 bytes por participante no servidor → escala lateral fácil.
- **Cloud vs OSS**: Cloud tem mesh; OSS single-home. Mesma API, troca transparente no SDK.

## Pontos-chave Jitsi

- **JVB (Videobridge)**: um nó aguenta 50–100 participantes (hardware dependente); 50–75 streams ativos simultâneos.
- **Octo**: protocolo de cascade entre JVBs regionais — um stream por região em vez de N, bandwidth reduction enorme.
- Deploy K8s real documentado chegou a 10k participantes num webinar com HPA sobre JVB.
- **jicofo** (focus) orquestra sessões; **Prosody** (XMPP) sinaliza; **Jibri** grava/streamma.

## Aplicação ao Master Síndico

- **M1/M2**: LiveKit Cloud single-region SP já resolve até ~1k participantes por sala confortavelmente.
- **M3 (multi-região BR + LATAM)**: Cloud mesh cobre sem reconfig; custo linear em participant-minutes.
- **M4 (broadcast 10k+ de assembleia pública)**: arquitetura certa é SFU pra ativos (~50) + HLS CDN pra passivos (thousands), não cascade SFU pura. Egress Track Composite → HLS cumpre essa função.
- **Self-host só se**: (a) exigência de soberania de dados, ou (b) volume > 3TB/mês de egress consistente justifica economia vs Cloud.

## Citações relevantes

> "LiveKit targets 'a media server within 100ms of anyone in the world' through multiple points of presence (PoPs)."

> "One JVB can typically deal with about 50-100 participants depending on hardware capacity. [...] With Octo [...] one stream per region only is transmitted."

## URL(s)

- https://livekit.com/blog/scaling-webrtc-with-distributed-mesh/
- https://jitsi.guide/blog/jitsi-scaling-performance-octo-jvb-clustering/
- https://github.com/jitsi/jitsi-videobridge
- https://jitsi.github.io/handbook/docs/architecture/
