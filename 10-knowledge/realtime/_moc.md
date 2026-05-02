---
title: MOC — Realtime (WebSocket, SSE, Webhooks, WebRTC)
type: moc
tags:
  - moc
  - realtime
  - websocket
  - webhooks
  - webrtc
  - sse
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Realtime MOC
  - Realtime Communication
---

# Realtime Communication

Cluster de knowledge sobre protocolos e padrões de comunicação em tempo real: bidirectional streams, server push, event callbacks, peer-to-peer. Conteúdo atemporal — implementação concreta em providers ([[../providers/livekit/_moc|Livekit]], [[../providers/cloudflare/durable-objects|CF DOs]], [[../providers/stripe/_moc|Stripe]]).

## 🧭 Decision tree — qual protocolo?

```
Preciso comunicação em tempo real. O fluxo é:
│
├─ Server → Client **apenas** (one-way push)?
│   │
│   ├─ Eventos async para B2B/integração (N segundos-minutos acceptable)?
│   │   → [[webhooks]]  (HTTP POST outbound, server-to-server)
│   │
│   └─ Stream contínuo para browser do user (sub-segundo)?
│       → [[server-sent-events]]  (HTTP long-lived, auto-reconnect)
│
├─ Bidirectional (client ↔ server)?
│   │
│   ├─ Frequente (jogos, chat, colaboração, LLM agent)?
│   │   → [[websocket]]  (RFC 6455, frames binários, subprotocols)
│   │     + [[websocket-scaling]] para patterns produção
│   │
│   └─ Esporádico (polling simple)?
│       → HTTP REST normal (com cache + If-None-Match)
│
├─ Peer-to-peer (latência sub-200ms crítica)?
│   │
│   ├─ Mídia (áudio, vídeo, screen share)?
│   │   → [[webrtc]]  (ICE, STUN, TURN, SRTP, signaling via WS)
│   │
│   └─ Arbitrary data (game state, file transfer)?
│       → [[webrtc]] DataChannel (UDP-like ou TCP-like, SCTP-based)
│
└─ Multi-peer (N > 10 em sala)?
    → [[webrtc]] com SFU (server recebe + forward) — ver [[../providers/livekit/_moc|Livekit]]
    OU [[websocket]] + pub/sub broker (se não é mídia)
```

## 📚 Notas no cluster

| Nota | Foco | Quando usar |
|---|---|---|
| [[websocket]] | Protocolo RFC 6455 (handshake, frames, close codes, subprotocols) | Bidirectional user-server |
| [[webhooks]] | HTTP POST outbound com HMAC + idempotência | Eventos server-to-server (Stripe, GitHub, Mux) |
| [[webrtc]] | P2P media + data; ICE/STUN/TURN, SDP, DTLS | Mídia real-time ou P2P data |
| [[server-sent-events]] | Server → browser, HTTP long-lived | LLM streaming, live dashboards, notifications |
| [[websocket-scaling]] | Sticky session, pub/sub, heartbeat, reconnect | Operar WS em produção multi-server |

## 📊 Comparação resumida

| Dimensão | [[websocket\|WebSocket]] | [[server-sent-events\|SSE]] | [[webhooks\|Webhook]] | [[webrtc\|WebRTC]] |
|---|---|---|---|---|
| **Direção** | Bidirectional | Server → Client | Provider → Consumer | Peer ↔ Peer |
| **Connection** | Long-lived TCP | Long-lived HTTP | 1 HTTP por evento | Long-lived UDP/TCP |
| **Cliente típico** | Browser, app | Browser (EventSource) | Server público | Browser, mobile, desktop |
| **Serverless-friendly?** | Só com providers edge (CF DOs, Lambda WS via API Gateway) | Sim (stream response) | Sim (stateless) | Com SFU — não puro P2P em server |
| **Binary payload** | Sim (frames binary) | Não (só text — base64 se precisa) | Raw HTTP body | Sim (SRTP binário) |
| **Auto-reconnect browser** | Não — manual com backoff | Sim (built-in EventSource) | N/A | Manual (perfect negotiation) |
| **Escala N conexões em 1 server** | 500k+ Go / 5M Erlang | 100k-500k | Stateless — unlimited | Limitado (mesh N²); use SFU |
| **Auth típica** | Cookie httpOnly ou subprotocol | Cookie httpOnly | HMAC signature per event | JWT no signaling server |
| **Defesa de replay** | Por aplicação | Não aplicável | HMAC + timestamp window | Não aplicável (media ephemeral) |
| **Casos canônicos** | Chat, games, collaborative editing, LLM agents | LLM tokens streaming, logs, notifications | Payment events, CI/CD triggers | Video calls, screen share |

## 🔑 Integração entre os quatro

- **WebRTC usa WebSocket** (normalmente) para signaling (troca SDP/ICE).
- **Webhook pode disparar SSE/WebSocket broadcast** — recebe evento de Stripe → empurra update para UI users conectados via WS.
- **SSE + Webhook**: stream de build logs (SSE para UI) + webhook GitHub notifica que build começou.
- **WebSocket + SSE**: mesmo app pode usar ambos por caso — SSE para logs streaming (one-way); WS para terminal interativo (bidirectional).

## 🛡️ Segurança — referências cruzadas

| Tópico | Nota(s) relevantes |
|---|---|
| Webhook HMAC | [[../security/security-principles#Webhook-Signature]], [[../runtime-antipatterns/op-023-webhook-sem-hmac\|OP-023]] |
| Webhook idempotência | [[../runtime-antipatterns/op-003-webhook-sem-idempotencia\|OP-003]] |
| WebSocket auth + origin check | [[websocket\|websocket.md §auth]], [[websocket-scaling]] |
| Rate limit (todos) | [[../runtime-antipatterns/op-024-rate-limit-ausente\|OP-024]] |
| TLS obrigatório | [[../security/security-principles#Princípios-inegociáveis]] |
| LLM streaming token leak | [[../runtime-antipatterns/op-022-log-com-pii\|OP-022]] (tokens em log é vazamento) |

## 🏗️ Providers que implementam

| Provider | Realtime feature |
|---|---|
| [[../providers/cloudflare/durable-objects]] | WebSocket server-side com state por chave + Hibernation API |
| [[../providers/cloudflare/workers]] | WebSocket + SSE em edge |
| [[../providers/cloudflare/_moc|Cloudflare Queues (lazy — D-vault-020)]] *(a destilar)* | Queue para async webhook processing |
| [[../providers/livekit/_moc]] | WebRTC SFU managed + client SDKs |
| [[../providers/stripe/_moc]] | Webhook reference implementation (HMAC Stripe-Signature) |
| [[../providers/mux/_moc]] | Webhook para VOD/live events |
| Ably, Pusher, Ably | WebSocket-as-a-service (managed) |
| Supabase Realtime | WebSocket + Postgres changes |
| Amazon API Gateway WebSocket | Managed WS (Lambda-integrated) |
| Cloudflare Calls | TURN + orchestration para WebRTC |

## 🎯 Leitura recomendada por caso

### Implementando webhook consumer (ex.: Stripe, GitHub)
1. [[webhooks]]
2. [[../security/security-principles#Webhook-Signature]]
3. [[../runtime-antipatterns/op-003-webhook-sem-idempotencia|OP-003]], [[../runtime-antipatterns/op-023-webhook-sem-hmac|OP-023]]
4. [[../providers/stripe/_moc]] (exemplo concreto)

### Implementando chat / app colaborativo
1. [[websocket]]
2. [[websocket-scaling]]
3. [[../providers/cloudflare/durable-objects]] (pattern alternativo serverless)
4. Considerar [[../security/session-management]] para auth

### Implementando streaming LLM no frontend
1. [[server-sent-events]]
2. [[websocket]] (se precisa interrupt/tool-call bidirectional)
3. Referência: OpenAI/Anthropic stream APIs

### Implementando chamada áudio/vídeo
1. [[webrtc]]
2. [[../providers/livekit/_moc]] (SFU managed — production-ready)
3. [[websocket]] (signaling)

## 🔗 Vizinhos no grafo

- [[../_moc]] — knowledge raiz
- [[../providers/_moc]] — implementações concretas
- [[../security/_moc]] — HMAC, rate limit, TLS
- [[../runtime-antipatterns/_moc]] — OP-003, OP-022, OP-023, OP-024
- [[../../00-meta/STATE]] — D-vault-023 (formalização do cluster)

## 📖 Fontes-chave do cluster

- [websocket.org — guias completos](https://websocket.org/) — referência comunitária excelente (consultada 2026-04-24)
- [RFC 6455 — WebSocket Protocol](https://datatracker.ietf.org/doc/rfc6455/) (consultada 2026-04-24)
- [W3C WebRTC 1.0](https://www.w3.org/TR/webrtc/) (consultada 2026-04-24)
- [HTML Living Standard — Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html) (consultada 2026-04-24)
- [WebRTC for the Curious](https://webrtcforthecurious.com/) (consultada 2026-04-24)
