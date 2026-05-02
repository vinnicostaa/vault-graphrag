---
title: WebRTC — Real-Time Communication P2P (media + data)
type: concept
tags:
  - realtime
  - webrtc
  - p2p
  - media
  - rtc
  - ice
  - stun
  - turn
  - sdp
doc-oficial: https://www.w3.org/TR/webrtc/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - WebRTC
  - RTC
  - Real-Time Communication
---

# WebRTC — Real-Time Communication

**O que é**: stack W3C + IETF para **comunicação peer-to-peer** em browsers/apps — áudio, vídeo e **data channels** arbitrários com latência sub-200ms. Não é um protocolo; é um conjunto (ICE, STUN, TURN, SDP, DTLS, SRTP, SCTP). Nav browsers modernos suportam nativamente.

> **Mental model**: "browser-to-browser TCP/UDP socket com NAT traversal + criptografia built-in". O servidor ajuda a **estabelecer** a conexão (signaling) mas depois fica de fora — mídia flui P2P.

## Quando usar WebRTC

1. **Chamadas áudio/vídeo** (Zoom-like, Google Meet-like, Apple FaceTime-like).
2. **Live streaming low-latency** (<500ms glass-to-glass).
3. **Voice/video em chat apps** (WhatsApp/Discord calls).
4. **Screen sharing + remote desktop** (Chrome Remote, Parsec).
5. **Cloud gaming** (latência ultra-baixa).
6. **P2P file transfer** (dataChannel — WebTorrent, Wormhole).
7. **Collaborative apps com state sync realtime** (dataChannel ao invés de WebSocket quando peer-to-peer faz sentido).

## Quando **NÃO** usar

- **N > 10 peers em sala** — mesh P2P escala mal (N² connections). Use SFU (Selective Forwarding Unit) — ver §Arquiteturas.
- **Gravação obrigatória** — P2P não passa por servidor; recording precisa SFU/MCU.
- **Tolerância a delay > 500ms** — WebRTC é overkill; use HLS/DASH streaming.
- **Cliente não-browser sem SDK** — WebRTC em servers/CLI exige libs maduras (Pion em Go, aiortc em Python).

## Componentes da stack

```
┌─────────────────────────────────────────────────────┐
│ Aplicação JS/nativa                                 │
├─────────────────────────────────────────────────────┤
│ WebRTC API (RTCPeerConnection, getUserMedia, etc.)  │
├─────────────────────────────────────────────────────┤
│ SDP (Session Description Protocol — RFC 8866)       │  ← "o que vamos transmitir"
│ ICE (RFC 8445) — candidate gathering + nominacão    │  ← "como nos encontramos"
├─────────────────────────────────────────────────────┤
│ STUN (RFC 8489) — descobre IP público NAT           │
│ TURN (RFC 8656) — relay quando P2P falha            │
├─────────────────────────────────────────────────────┤
│ DTLS (TLS sobre UDP) — handshake seguro             │  ← "criptografia"
│ SRTP (RFC 3711) — media criptografada               │
│ SCTP sobre DTLS — data channels                     │
└─────────────────────────────────────────────────────┘
```

## Fluxo end-to-end (simplificado)

```
1. Alice e Bob querem conectar.

2. SIGNALING (out-of-band — você implementa):
   • Alice e Bob falam com um "signaling server" (WebSocket, SSE, etc.)
   • Signaling server NÃO é parte do WebRTC — você escolhe.

3. Alice cria RTCPeerConnection, chama createOffer() → gera SDP.
   SDP descreve: codecs (H.264, VP9, Opus), streams, ICE candidates
   iniciais.

4. Alice envia SDP offer para Bob via signaling server.

5. Bob cria RTCPeerConnection, chama setRemoteDescription(offer),
   chama createAnswer() → gera SDP answer, envia a Alice via signaling.

6. ICE GATHERING (paralelo):
   • Cada peer coleta "candidates" (IPs/portas):
     - HOST (IP local)
     - SRFLX (IP público visto por STUN server)
     - RELAY (endereço TURN server — fallback)
   • Trocam candidates via signaling server (trickle ICE).

7. ICE NEGOTIATION:
   • Peers testam pares (host↔host, srflx↔srflx, etc.).
   • Primeiro par que funciona vira nominated candidate.

8. DTLS HANDSHAKE P2P — deriva chaves SRTP.

9. Mídia flui P2P (ou via TURN se P2P falhou):
   RTP/SRTP packets pela internet sem passar pelo seu servidor.
```

## SDP — Session Description Protocol

Texto declarativo descrevendo sessão:

```
v=0
o=- 461734 2 IN IP4 127.0.0.1
s=-
t=0 0
a=group:BUNDLE 0 1
m=audio 9 UDP/TLS/RTP/SAVPF 111
c=IN IP4 0.0.0.0
a=rtcp:9 IN IP4 0.0.0.0
a=ice-ufrag:abc123
a=ice-pwd:xyz...
a=fingerprint:sha-256 AB:CD:...
a=setup:active
a=mid:0
a=rtpmap:111 opus/48000/2
a=sendrecv
m=video 9 UDP/TLS/RTP/SAVPF 96 97
a=rtpmap:96 VP9/90000
a=rtpmap:97 H264/90000
...
```

**Regra**: você **raramente edita SDP** direto. Libs abstraem; você chama `createOffer()` e `setLocalDescription()`. Mas entender o básico ajuda debug (codec negotiation, BUNDLE policy, simulcast).

## ICE — NAT Traversal

Problema: maioria dos devices está atrás de NAT (home router). IP privado não é roteável.

**Solução ICE**:
1. **STUN server** responde "seu IP público visto por mim é X.X.X.X:PORT".
2. Peer descobre seu endereço público; envia ao peer pelo signaling.
3. Tenta hole-punching: ambos enviam pacotes ao endereço público do outro simultaneamente; routers NAT abrem porta.
4. **TURN server** fallback: se NAT é simétrico (não permite hole-punching, comum em corp/mobile), TURN **relay** packets. Você paga bandwidth do TURN.

### Success rate típico (2024):
- Hole punch direto (STUN only): **~85%** dos pares residenciais.
- Precisa TURN: **~15%** (mobile → mobile, corporate firewalls, symmetric NAT).

### STUN servers públicos (grátis)

```
stun:stun.l.google.com:19302
stun:stun.cloudflare.com:3478
stun:stun1.l.google.com:19302
```

### TURN servers (pagos — bandwidth custoso)

- [Cloudflare Calls TURN](https://developers.cloudflare.com/calls/turn/) — $0.05/GB egress.
- [Twilio STUN/TURN](https://www.twilio.com/stun-turn).
- [Xirsys](https://xirsys.com/).
- **Self-hosted**: coturn (C, open-source, padrão-ouro).

## DataChannel (não-mídia)

```js
const pc = new RTCPeerConnection()
const dc = pc.createDataChannel('game', {
  ordered: true,          // ou false para jogos (UDP-like)
  maxRetransmits: 3,      // ou maxPacketLifeTime
})

dc.addEventListener('open', () => {
  dc.send('hello')
  dc.send(new Uint8Array([1, 2, 3]))
})

dc.addEventListener('message', (event) => {
  // event.data pode ser string, ArrayBuffer, Blob
})
```

**Trade-offs vs WebSocket**:
- **DataChannel**: P2P (sem server no middle), pode ser UDP-like (perda tolerada), latência menor.
- **WebSocket**: client-server, TCP ordered/reliable, servidor escala.

Use DataChannel em: jogos P2P, file transfer, sync state entre peers.

## Arquiteturas (quando N > 2)

### Mesh (N² connections)

Cada par de peers abre PeerConnection. **Escala até ~5 peers**; depois bandwidth + CPU explodem.

### SFU — Selective Forwarding Unit

Server recebe 1 stream de cada peer; **forward** para outros peers. Não re-encoda. Bandwidth client = 1 up + (N-1) down.

**Dominant pattern em produção**: Zoom, Google Meet, Twitch Studio, Jitsi, LiveKit usam SFU.

SFU open-source:
- **mediasoup** (Node + C++).
- **LiveKit** (Go) — ver [[../providers/livekit/_moc|provider dedicado]].
- **Jitsi Videobridge** (Java).
- **Janus** (C).
- **Pion** (Go lib para construir custom).

### MCU — Multipoint Control Unit (legado)

Server recebe, **decodifica, mixa e re-encoda**. Muito caro em CPU. Preferido só em contextos legados (gravação central, PSTN gateway).

## Signaling — você escolhe

WebRTC spec **não define** signaling. Use:
- **WebSocket** (mais comum) — ver [[websocket]]
- **HTTP POST** (simples mas latência)
- **SIP** (telecom legacy)
- **XMPP** (IM legacy)

Mensagens típicas: `offer`, `answer`, `ice-candidate`, `bye`. Biblioteca comum: [Signal Protocol over WS](https://webrtcforthecurious.com/) 1 socket = 1 sala.

## Perfect Negotiation Pattern

Problema clássico: 2 peers tentam enviar offer simultaneamente → glare. Solução:

```js
// peer A polite, peer B impolite (decida por user_id hash)
let makingOffer = false
let ignoreOffer = false
const polite = true   // true para 1 peer, false para outro

pc.addEventListener('negotiationneeded', async () => {
  try {
    makingOffer = true
    await pc.setLocalDescription()   // sem args → offer
    signal.send({ offer: pc.localDescription })
  } finally {
    makingOffer = false
  }
})

signal.onmessage = async ({ offer, answer, candidate }) => {
  if (offer) {
    const offerCollision = makingOffer || pc.signalingState !== 'stable'
    ignoreOffer = !polite && offerCollision
    if (ignoreOffer) return
    await pc.setRemoteDescription(offer)
    await pc.setLocalDescription()
    signal.send({ answer: pc.localDescription })
  } else if (answer) {
    await pc.setRemoteDescription(answer)
  } else if (candidate) {
    try { await pc.addIceCandidate(candidate) }
    catch (e) { if (!ignoreOffer) throw e }
  }
}
```

Padrão oficial em [W3C — Perfect Negotiation](https://w3c.github.io/webrtc-pc/#perfect-negotiation-example).

## Codecs

### Vídeo
- **VP8** — fallback garantido; qualidade OK.
- **VP9** — ~30% melhor que VP8; mais CPU.
- **H.264** — hardware-accelerated quase everywhere; patent fees resolvidos em WebRTC.
- **AV1** — qualidade superior; suporte crescente (Chrome 90+, Firefox 93+); mais CPU.

### Áudio
- **Opus** — default universal. Excelente qualidade 6-510kbps, wide-band.
- **G.711** — PSTN compatibility (telecom).

## Simulcast + SVC

Para N peers com bandwidth variável (alguns no celular 3G, outros fibra):
- **Simulcast**: peer envia MÚLTIPLAS qualidades (ex: 180p, 360p, 720p) em streams separadas; SFU escolhe qual enviar a cada receiver.
- **SVC (Scalable Video Coding)**: 1 stream com layers (base + enhancement); SFU drop layers. Mais moderno (AV1-SVC, VP9-SVC).

## Segurança

### DTLS (mandatório)

Todo WebRTC é **criptografado end-to-end** por design. DTLS (TLS over UDP) nego chaves; SRTP cripta RTP packets. Não tem como desligar. Chave derivada por-sessão; não há shared secret global.

**Importante**: "E2E" significa entre **peers**. Com SFU, mídia é **decriptada no servidor** (SFU precisa enxergar para fazer forwarding). Isto é **E2E até SFU, não E2E entre usuários finais**. Para true E2E com SFU: **Insertable Streams** + E2E encryption (camada extra — Jitsi, LiveKit suportam).

### Identity

WebRTC não autentica peers nativamente. **Você** implementa auth no signaling server (JWT + IdP). Recomendado: emitir **short-lived TURN credentials** assinadas pelo seu backend (HMAC sobre timestamp + user_id) — previne abuse do TURN.

### Anti-patterns de segurança

1. **TURN credentials hardcoded no cliente** — atacante abusa relay. Use ephemeral credentials.
2. **Signaling server sem auth** — atacante conecta qualquer um.
3. **SDP mal validado** — atacante injeta addresses maliciosos. Libs modernas validam.
4. **IP leak via STUN** — em cenários de privacy (VPN users), ICE expõe IP real. Solução: `iceTransportPolicy: 'relay'` força só TURN (perde P2P benefit).
5. **Sem E2E real** em SFU de alto compliance (health, legal) — decriptação em transit no server. Use Insertable Streams.

## Mobile / native

### iOS / Android

Browser WebRTC OK. App nativo usa SDK oficial:
- **Google WebRTC lib** — C++ base; bindings iOS/Android.
- **[[../providers/livekit/_moc|LiveKit Client SDK]]** — iOS/Android/Unity/Unreal.
- **Daily.co SDK**.
- **Twilio Video SDK**.

### React Native

- **react-native-webrtc** — bridge para libs nativas.
- **expo-webrtc** — managed workflow.

## Como grandes empresas usam

- **Google Meet / Duo / Voice / Hangouts** — stack própria; SFU massivo; VP9 + AV1 simulcast; Duo usa DataChannel extensivamente.
- **Zoom** — híbrido: WebRTC em browser, protocolo próprio (não-WebRTC) em app nativo para latência menor. Gateway entre os dois.
- **Microsoft Teams** — WebRTC + Skype legacy protocol bridge.
- **WhatsApp calls** — WebRTC stack custom com E2E real (Signal Protocol layer).
- **Discord voice** — não é WebRTC puro; UDP custom + Opus; usa WebRTC para screen share.
- **Apple FaceTime** — não é WebRTC; protocolo proprietário.
- **Amazon Chime SDK** — WebRTC-based com SFU próprio.
- **Figma multiplayer cursors** — NÃO é WebRTC; é WebSocket (requer server broadcast; P2P não escala para canvas shared).
- **GitHub Codespaces remote desktop** — WebRTC.
- **Parsec / Moonlight (cloud gaming)** — WebRTC-based com custom H.264 pipeline.
- **Livekit, Daily, Twilio Video, Amazon Chime, Dolby.io** — provedores SFU comerciais.

## Padrão canônico — call simples 1-to-1

```js
// Peer A (caller)
const pc = new RTCPeerConnection({
  iceServers: [
    { urls: 'stun:stun.l.google.com:19302' },
    {
      urls: 'turn:turn.example.com:3478',
      username: ephemeralUsername,    // HMAC do backend
      credential: ephemeralPassword,
    },
  ],
})

// 1. Captura mídia
const stream = await navigator.mediaDevices.getUserMedia({ audio: true, video: true })
stream.getTracks().forEach(track => pc.addTrack(track, stream))

// 2. ICE trickle
pc.addEventListener('icecandidate', ({ candidate }) => {
  if (candidate) signal.send({ candidate })
})

// 3. Remote stream
pc.addEventListener('track', (event) => {
  remoteVideo.srcObject = event.streams[0]
})

// 4. Offer → signaling → answer (perfect negotiation pattern)
// ... ver seção acima

// 5. Close
function hangup() {
  pc.getSenders().forEach(s => s.track?.stop())
  pc.close()
  signal.send({ bye: true })
}
```

## Debug

### chrome://webrtc-internals

Página built-in Chrome com todos os stats: bitrate, RTT, packet loss, jitter, ICE state, codec negotiated. **Primeiro lugar para debug**.

### `getStats()`

API programática:
```js
const stats = await pc.getStats()
stats.forEach((report) => {
  console.log(report.type, report)   // 'inbound-rtp', 'outbound-rtp', 'candidate-pair', etc.
})
```

## Reference architectures

- [Livekit reference](https://docs.livekit.io/home/get-started/intro-to-livekit/) — SFU production (consultada 2026-04-24)
- [Cloudflare Calls — TURN relay + orchestration](https://developers.cloudflare.com/calls/) (consultada 2026-04-24)
- [WebRTC for the Curious (book)](https://webrtcforthecurious.com/) — open-source, muito bom para mental model

## Relações

- **Cluster**: [[_moc]]
- **Signaling**: geralmente sobre [[websocket]]
- **Alternativas**: [[websocket]] (para apps que escalam N>10 peers em mesh — use WebSocket + server broadcast), HLS/DASH (streaming sem low-latency)
- **Provider destilado**: [[../providers/livekit/_moc]] (SFU managed)
- **Provider adjacente**: Cloudflare Calls (TURN + orchestration; lazy)
- **Codecs**: não temos notas; CODEC é fora do escopo realtime
- **Security**: E2E em `insertable streams` — tópico avançado

## Fontes

- [W3C WebRTC 1.0 spec](https://www.w3.org/TR/webrtc/) (consultada 2026-04-24)
- [RFC 8445 — ICE](https://datatracker.ietf.org/doc/rfc8445/) (consultada 2026-04-24)
- [RFC 8489 — STUN](https://datatracker.ietf.org/doc/rfc8489/) (consultada 2026-04-24)
- [RFC 8656 — TURN](https://datatracker.ietf.org/doc/rfc8656/) (consultada 2026-04-24)
- [RFC 8866 — SDP](https://datatracker.ietf.org/doc/rfc8866/) (consultada 2026-04-24)
- [RFC 3711 — SRTP](https://datatracker.ietf.org/doc/rfc3711/) (consultada 2026-04-24)
- [MDN — WebRTC API](https://developer.mozilla.org/en-US/docs/Web/API/WebRTC_API) (consultada 2026-04-24)
- [WebRTC for the Curious](https://webrtcforthecurious.com/) (consultada 2026-04-24)
- [webrtchacks.com](https://webrtchacks.com/) (consultada 2026-04-24)
- [LiveKit docs](https://docs.livekit.io) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[websocket]] — signaling comum
- [[../providers/livekit/_moc]] — SFU managed
- [[../security/security-principles]]
- [[webhooks]] — call status events (ex.: LiveKit publica events)
