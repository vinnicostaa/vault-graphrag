---
title: "Source 02 — LiveKit SFU Internals — docs.livekit.io"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, sfu, arquitetura]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "LiveKit"
source_urls:
  - "https://docs.livekit.io/reference/internals/livekit-sfu/"
  - "https://sheerbit.com/livekit-architecture-deep-dive-sfu-media-routing-and-scaling/"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 02 — LiveKit SFU Internals

## Resumo

Documentação oficial LiveKit explicando o SFU: router de mídia em Go sobre Pion, que nunca decodifica/re-encoda (só forward RTP), com adaptação automática de bitrate por subscriber. Confirma a escolha arquitetural do Master Síndico.

## Pontos-chave

- **Stack**: Go + Pion WebRTC. SFU opera no nível de pacote RTP; CPU proporcional ao volume de streams, não à complexidade do vídeo.
- **Forward transparente**: cada peer publica uma vez; servidor replica seletivamente pros subscribers interessados (respeitando subscrição e banda downstream).
- **Escala single-node**: "hundreds of simultaneous video streams" em hardware moderno (benchmark LiveKit).
- **Escala multi-node**: horizontal, config idêntica, roteamento P2P entre nós via Redis (garante clientes da mesma sala caírem no mesmo nó).
- **Adaptação client/server-side**: mede banda downstream do subscriber e ajusta track automaticamente (res/bitrate), sem intervenção app.

## Aplicação ao Master Síndico

- **Dimensionamento confortável pra M1/M2**: 50–200 participantes numa assembleia cabe folgadamente num nó único LiveKit (Cloud ou self-host).
- **Zero transcode** = custo previsível: CPU só cresce com participantes ativos que publicam, não com qualidade.
- **Adaptação automática** = moradores em 3G/4G não degradam a experiência dos outros; próprio SFU gere step-down de resolução.
- Self-host M3+: precisaremos Redis (já usamos no stack para BullMQ).

## Citações relevantes

> "LiveKit's SFU also contains smarts on both the server and client [...] to automatically (and invisibly) measure a subscriber's downstream bandwidth and adjust track parameters (e.g. resolution or bitrate) accordingly."

> "The SFU is horizontally-scalable: you can run it on one or one hundred nodes with an identical configuration."

## URL(s)

- https://docs.livekit.io/reference/internals/livekit-sfu/
- https://sheerbit.com/livekit-architecture-deep-dive-sfu-media-routing-and-scaling/
