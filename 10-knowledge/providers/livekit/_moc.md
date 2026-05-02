---
title: MOC — Livekit (WebRTC realtime)
type: moc
tags: [moc, provider, livekit, webrtc, realtime, audio, video]
category: realtime
mcp: null
sdk-official-go: livekit-server-sdk-go (github.com/livekit/server-sdk-go/v2)
sdk-official-node: livekit-server-sdk
sdk-official-flutter: livekit_client
doc-oficial: https://docs.livekit.io
doc-consulted: 2026-04-23
created: 2026-04-22
updated: 2026-04-23
---

# Livekit

**Categoria**: Realtime WebRTC · rooms · participants · audio/video/chat · recording · egress (broadcast pra RTMP/HLS).

**MCP disponível**: ❌ — usar WebFetch em https://docs.livekit.io ou Context7 `resolve-library-id "livekit"`.

**SDKs oficiais**:
- Go: `github.com/livekit/server-sdk-go/v2` (token gen + REST)
- Node: `livekit-server-sdk`
- Client SDKs: Web, iOS, Android, Flutter, React Native, Unity

**Doc oficial**: https://docs.livekit.io — consultada em 2026-04-23.

**Self-hosted vs Cloud**: cloud = `<project>.livekit.cloud`; self-host via Docker/K8s (SFU em Go).

**Modelo de preço**: cloud = pay-per-minute participante · self-hosted = infra only.

---

## Quando usar Livekit

1. **Videoconferência peer-to-peer com muitos participantes** (SFU architecture — não mesh) — alternative: Daily, Agora, 100ms
2. **Voice rooms** (Clubhouse-like, Discord-like)
3. **Interviews with recording** (egress pra S3)
4. **Broadcast live WebRTC → HLS** (egress target)
5. **Self-hosted** necessário por compliance (Livekit OSS é solid)

## Quando NÃO usar Livekit

- **Live streaming broadcast grande escala (1→N sem interação)** → Mux ou IVS (Livekit SFU não escala pra 10k+ espectadores passivos)
- **Mensagens síncronas texto apenas** → WebSocket direto ou Pusher
- **Signaling custom já implementado** → Pion (lib Go direto) + SFU próprio

---

## Sub-docs do Livekit

- [[overview]] — rooms, participants, tracks, SFU vs MCU
- [[integration-guide]] — token server (Go) + client SDK + room orchestration
- [[patterns]] — token TTL curto, room cleanup, egress pra recording, reconnect com token fresh
- [[antipatterns]] — ex: token longa duração, trust no client pra room permissions
- [[usage-in-projects]] — quais projetos

## Glossário interno do Livekit

- **Room** — sala virtual; tem até N participants; criada on-demand ao primeiro participant join.
- **Participant** — usuário connectado; tem tracks (audio/video/data).
- **Track** — fluxo individual de mídia (ex: 1 camera + 1 microphone = 2 tracks).
- **Access Token (JWT)** — gerado no server com API key; define room + participant identity + permissions (can_publish, can_subscribe, can_update_metadata). TTL curto (15min típico).
- **Egress** — exporta tracks pra S3/RTMP/HLS (recording + broadcast).
- **Ingress** — importa RTMP/WHIP externo pra dentro do Livekit.

## Fontes brutas (pendente destilar)

> **Estado**: destilação pendente. Conteúdo direto da doc oficial. Master Síndico usa pra assembleia ao vivo ([[../../../50-projects/master-sindico/01-domain/bounded-contexts|ver bounded context Assembly]]).

## Vizinhos no grafo

- [[../_moc]]
- [[../mux/_moc]] — broadcast-scale concorrente (Livekit = realtime peer-to-peer; Mux = broadcast 1→N)
- [[../../security/_moc]] — JWT tokens com TTL curto
