---
title: "Source 06 — LiveKit Data Channels (chat, reactions, RPC, voting)"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, data-channel, votacao, chat]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "LiveKit"
source_urls:
  - "https://docs.livekit.io/transport/data/"
  - "https://docs.livekit.io/home/client/data/text-streams/"
  - "https://medium.com/@rajabalasuguna/how-livekit-handle-realtime-data-24ccaf717e1a"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 06 — LiveKit Data Channels

## Resumo

LiveKit expõe 6 APIs de dados sobre WebRTC DataChannel: Text Streams, Byte Streams, RPC, Data Packets, State Sync (reliable) + Data Tracks (lossy). Decide comportamento do canal de chat, fila de fala, e UI de votação em tempo real.

## Pontos-chave

### Entrega garantida (reliable)

- **Text Streams**: chunking automático + topic-based routing. Ideal pra chat e respostas LLM streaming.
- **Byte Streams**: arquivos/binário com progress tracking.
- **RPC**: chamada remota com retorno de outro participante (ex: "pedi pra si, síndico validou"). Útil pra fluxo procuração/proxy.
- **Data Packets**: primitiva baixa pra casos avançados.
- **State Synchronization**: replica `attributes` e `metadata` entre participantes (evento `ParticipantAttributesChanged`).

### Lossy (real-time > entrega)

- **Data Tracks**: prioriza tempo real; frames atrasados são descartados. Pra telemetria/gaming, não pra nosso caso.

### Limitações

- **Sem persistência**: tudo é só tempo real; histórico precisa ser implementado por fora (nossa DB).
- Mensagens só alcançam participantes online na sala naquele momento.
- Topics permitem multiplexar canais (ex: `chat`, `votacao`, `fila-fala`) no mesmo room.

## Aplicação ao Master Síndico

### Votação em tempo real

- **UI "votos ao vivo"** (barra subindo em tela) → Text Stream reliable topic `votacao:<id_pauta>` com payload `{"voto":"sim","peso":0.043}`.
- **Contagem oficial** → API REST nossa persistindo em Postgres. Voto só "conta" quando 200 OK + hash no ledger.
- Redundância: receber via data channel + confirmar via API REST próprio cliente. Se divergir, API é fonte da verdade.

### Chat da assembleia

- Text Stream reliable topic `chat`. Persistir no nosso BD pra compor ata (manifestações escritas são legalmente relevantes).

### Reactions (palminhas, emoji)

- Text Stream reliable ou Data Packet topic `reaction` — baixa criticidade, não persistir.

### Fila de fala

- Ver source-07 — combina `attributes` (State Sync) + RPC.

## Citações relevantes

> "LiveKit's data channels are built on WebRTC's DataChannel and allow low-latency messaging for chat, events, and metadata between participants, routed via the LiveKit server."

> "LiveKit does not include long-term persistence for text streams [...] message history must be implemented using a database or other persistence layer."

## URL(s)

- https://docs.livekit.io/transport/data/
- https://docs.livekit.io/home/client/data/text-streams/
- https://medium.com/@rajabalasuguna/how-livekit-handle-realtime-data-24ccaf717e1a
