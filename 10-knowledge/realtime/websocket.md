---
title: WebSocket — bidirectional protocol (RFC 6455)
type: concept
tags:
  - realtime
  - websocket
  - network
  - protocol
  - rfc6455
  - bidirectional
doc-oficial: https://datatracker.ietf.org/doc/rfc6455/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - WebSocket
  - WS
  - RFC 6455
---

# WebSocket — RFC 6455

**O que é**: protocolo bidirecional persistente sobre TCP que começa como HTTP Upgrade e depois opera com **frames binários** independentes de HTTP. RFC 6455 (2011). Resolve o caso onde servidor precisa **enviar ao cliente** sem polling — jogos, chat, notificações, live dashboards, streaming de tokens LLM, colaboração real-time.

> **Mental model**: "TCP socket com handshake HTTP-compatible para atravessar proxies/firewalls". Depois do handshake, é socket full-duplex — **ambos os lados podem enviar quando quiserem**.

## Quando usar WebSocket (vs alternativas — ver [[_moc]] decision tree)

1. **Bidirectional de baixa latência** (chat, jogos, collaborative editing).
2. **Server push frequente** com state no cliente (live trading, scoreboards).
3. **LLM token streaming** com possibilidade de interrupção cliente→servidor.
4. **Signaling para [[webrtc|WebRTC]]** — negociação SDP/ICE antes da conexão P2P.

Se server→client **apenas** (one-way push), **[[server-sent-events|SSE]]** é mais simples e barato (HTTP/2 friendly, auto-reconnect nativo).

Se o cliente polla para atualizar (forecast: ≤ 1 update/min), **HTTP long-poll ou webhook** basta.

Se mídia P2P (voz/vídeo), **[[webrtc]]**.

## Handshake (HTTP Upgrade)

Cliente envia HTTP GET com headers de upgrade:

```http
GET /chat HTTP/1.1
Host: app.example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat.v2, chat.v1
Origin: https://app.example.com
```

Servidor responde `101 Switching Protocols` computando `Sec-WebSocket-Accept` = base64(SHA1(`Sec-WebSocket-Key` + `258EAFA5-E914-47DA-95CA-C5AB0DC85B11`)):

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: chat.v2
```

A partir daí, **não é mais HTTP** — TCP com frames WebSocket.

### Subprotocols (`Sec-WebSocket-Protocol`)

Cliente oferece lista em ordem de preferência; servidor escolhe **um** (ou nenhum). Use para versionar API (`chat.v2` vs `chat.v1`) ou declarar tipo (`mqtt`, `graphql-ws`). Sem subprotocol = "free-form JSON", o que é frágil para evolução.

## Frames

Formato binário (RFC 6455 §5):

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| opcode|M| Payload len |    Extended payload length    |
|I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
|N|V|V|V|       |S|             |                               |
| |1|2|3|       |K|             |                               |
+-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
|     Extended payload length continued, if payload len == 127  |
+-------------------------------+-------------------------------+
| Masking-key (4 bytes, client→server only)                     |
+-------------------------------+-------------------------------+
|                     Payload Data (unmasked if server→client)  |
+---------------------------------------------------------------+
```

### Opcodes

| Opcode | Tipo | Uso |
|---|---|---|
| `0x0` | Continuation | Fragmentation |
| `0x1` | Text (UTF-8) | JSON, strings |
| `0x2` | Binary | Protobuf, msgpack, MP4, etc. |
| `0x8` | Close | Fim ordenado |
| `0x9` | Ping | Heartbeat |
| `0xA` | Pong | Resposta a ping |

### Masking (client→server obrigatório)

Cada frame **client→server** tem 32-bit random mask XOR'd sobre payload. **Defesa contra cache poisoning** em proxies HTTP intermediários (herança de ataques pré-WebSocket). Server→client **não** mascara.

### Fragmentation

Mensagem grande pode vir em múltiplos frames: primeiro com `FIN=0` + opcode real, meio(s) com `FIN=0` + opcode=continuation, último com `FIN=1` + continuation. Até hoje pouco usado (libs concatenam); limite prático por mensagem é do application.

## Close codes (RFC 6455 §7.4)

| Code | Razão |
|---|---|
| **1000** | Normal closure |
| **1001** | Going away (server shutdown, browser navigate) |
| **1002** | Protocol error |
| **1003** | Unsupported data (servidor recebeu binary esperando text) |
| **1006** | Abnormal closure — **não é sent on wire**; é reportado pela lib quando não houve close frame |
| **1007** | Invalid payload (UTF-8 malformado) |
| **1008** | Policy violation (generic) |
| **1009** | Message too big |
| **1010** | Extension required |
| **1011** | Server internal error |
| **1012** | Service restart |
| **1013** | Try again later (overload) |
| **4000-4999** | Application-specific (sua escolha) |

**Regra**: use 4000+ para codes semânticos do seu domínio (`4001 = auth expired`, `4002 = banned user`, etc.). Cliente faz reconnect com backoff diferente por code.

## Ping/Pong (heartbeat)

- Servidor envia `Ping` (opcode 0x9) a cada N segundos (30-60s típico).
- Cliente **deve** responder `Pong` (opcode 0xA) com mesmo payload (echo).
- Browser envia pong **automaticamente** — JS não vê nem gerencia.
- Se server não recebe pong em timeout → assume conexão morta, fecha + reconnect.

**Obrigatório em prod** — previne "zombie connections" que TCP sozinho não detecta (NAT/firewall expira sessão sem enviar FIN).

## Client API (browser)

```js
const ws = new WebSocket('wss://app.example.com/chat', ['chat.v2'])

ws.addEventListener('open', () => {
  ws.send(JSON.stringify({ type: 'hello', room: 'room-42' }))
})

ws.addEventListener('message', (event) => {
  const msg = JSON.parse(event.data)
  // event.data pode ser string (text frame) ou Blob/ArrayBuffer (binary)
})

ws.addEventListener('error', (err) => {
  console.error('ws error', err)   // não tem detalhe por segurança (mesma origem XSS)
})

ws.addEventListener('close', (event) => {
  console.log('closed', event.code, event.reason, event.wasClean)
  // reconnect com backoff se wasClean=false
})

ws.close(1000, 'bye')
```

**Regras importantes**:
- `ws.send()` em state < OPEN → throws.
- `ws.bufferedAmount` indica bytes não enviados; use para backpressure.
- Browser **não expõe** ping/pong — é transparente.

## Server (Node, `ws` lib)

```ts
import { WebSocketServer } from 'ws'
import { createServer } from 'http'

const httpServer = createServer()
const wss = new WebSocketServer({ server: httpServer, path: '/chat' })

wss.on('connection', (ws, req) => {
  // Auth on upgrade — ver [[websocket-scaling|websocket-scaling]] §auth
  const user = authenticate(req)
  if (!user) { ws.close(4001, 'unauthorized'); return }

  ws.on('message', (data, isBinary) => {
    const msg = JSON.parse(data.toString())
    // process
  })

  // Heartbeat: ping a cada 30s; assume morta se nenhum pong em 60s
  ws.isAlive = true
  ws.on('pong', () => { ws.isAlive = true })
})

const interval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) return ws.terminate()
    ws.isAlive = false
    ws.ping()
  })
}, 30_000)

httpServer.listen(3000)
```

## Go (`gorilla/websocket` ou `coder/websocket`)

```go
import "github.com/coder/websocket"   // v1+ recomendado (ex-nhooyr.io/websocket)

func handler(w http.ResponseWriter, r *http.Request) {
    c, err := websocket.Accept(w, r, &websocket.AcceptOptions{
        Subprotocols: []string{"chat.v2"},
        OriginPatterns: []string{"app.example.com"},
    })
    if err != nil { return }
    defer c.CloseNow()

    ctx, cancel := context.WithTimeout(r.Context(), time.Hour)
    defer cancel()

    for {
        _, data, err := c.Read(ctx)
        if err != nil { return }
        // process
        c.Write(ctx, websocket.MessageText, []byte(`{"ack":true}`))
    }
}
```

Libs Go modernas (`coder/websocket`) fazem ping/pong automático + context-aware.

## Compression (`permessage-deflate`, RFC 7692)

Extensão negociada no handshake (`Sec-WebSocket-Extensions`). Reduz payload via DEFLATE per-message.

- **Pros**: bandwidth menor, especialmente JSON repetitivo.
- **Cons**: CPU extra; memória 50KB-1MB por conexão (context); vulnerabilidade BREACH-like em alguns cenários.
- **Regra**: ligar em produção se payloads > 1KB e text (JSON); desligar se binário já comprimido ou CPU limitado.

## Limits práticos

| Limite | Valor típico |
|---|---|
| Max frame size | 2^63 bytes (teórico); libs capam em 64KB-4MB |
| Max message size | config — default 1MB em maioria das libs |
| Connections per port | ~65k em teoria (ports source); na prática limit é file descriptors (`ulimit -n`) |
| Memory per connection | 2-10 KB idle (server); mais com compression |
| Idle timeout (LB, CF) | 60s-120s default; **configurar** para > heartbeat interval |

## Anti-patterns

1. **Sem heartbeat** — zombie connections acumulam; user "online" que não está. Ping/pong 30s é default.
2. **Reconnect sem backoff exponencial** — server cai → milhões de reconnects instantâneos = thundering herd. Backoff com jitter.
3. **Auth via query string** (`ws://...?token=xxx`) — token em log de proxy/server. Ver [[websocket-scaling]] §auth.
4. **Sem limite de tamanho de mensagem** — atacante envia 1GB frame; server OOM. `maxPayload` ou equivalente obrigatório.
5. **Sem origin check** no handshake — CSWSH (Cross-Site WebSocket Hijacking). Validar `Origin` header server-side.
6. **Broadcast em fan-out ingênuo** — 10k conexões + 1 mensagem por cliente = 100M ops/s. Use [[websocket-scaling|pub/sub broker]].
7. **Session state em memória do server único** — crash = perda. Externalize em Redis/DB.
8. **Frame texto sempre** quando binário seria menor — MessagePack, Protobuf, CBOR são 30-60% menores que JSON.
9. **Ignorar close code** — cliente reconecta em `1008 policy violation` (user banido) infinitamente. Respeitar semântica.
10. **`wss://` não usado em produção** — texto em claro em rede hostil. TLS sempre.

## Como grandes empresas fazem

- **Slack**: WebSocket para mensagens realtime; binary encoding (Flatbuffers antigamente, agora mix JSON/binary); geograficamente distribuído; fallback long-poll para mobile com bateria baixa.
- **Discord**: WebSocket para gateway; Erlang/Elixir (OTP) para conexões paralelas massivas; shards por guild.
- **WhatsApp Web**: WebSocket over XMPP-like custom protocol.
- **Trading (Binance, Coinbase)**: WebSocket para market data push; subprotocols versionados; heartbeat 3s (mais agressivo).
- **OpenAI / Anthropic**: WebSocket para realtime LLM streaming (tokens + tool calls interleaved).
- **Figma**: WebSocket para multi-user cursor/selection sync; CRDTs over WS.
- **GitHub (Copilot chat, Codespaces)**: WebSocket para terminal/editor sync.
- **Cloudflare**: [[../providers/cloudflare/durable-objects|Durable Objects]] oferece WebSocket native com Hibernation API; ~milhões de conexões por DO.
- **AWS**: API Gateway WebSocket API (managed), ALB proxy; Lambda não é ideal (stateless).

## Relações

- **Cluster realtime**: [[_moc]] decision tree
- **Alternativas**: [[server-sent-events]] (one-way), [[webhooks]] (callback out-of-band), [[webrtc]] (media P2P)
- **Escala**: [[websocket-scaling]] (este arquivo é protocol; scaling é operação)
- **Server-side providers**: [[../providers/cloudflare/durable-objects|CF Durable Objects]] (WebSocket + state por chave), [[../providers/livekit/_moc|Livekit]] (WebRTC com WS signaling)
- **Segurança**: [[../security/security-principles]] (TLS, auth), [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]] (rate limit em upgrade endpoint)
- **Pattern WebSocket room stateful**: [[../providers/cloudflare/patterns]] §4 (DO + WebSocket)

## Fontes

- [RFC 6455 — The WebSocket Protocol](https://datatracker.ietf.org/doc/rfc6455/) (consultada 2026-04-24)
- [RFC 7692 — Compression Extensions (permessage-deflate)](https://datatracker.ietf.org/doc/rfc7692/) (consultada 2026-04-24)
- [MDN — WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API) (consultada 2026-04-24)
- [websocket.org — protocol guide](https://websocket.org/guides/websocket-protocol) (consultada 2026-04-24)
- [ws (Node lib)](https://github.com/websockets/ws) (consultada 2026-04-24)
- [coder/websocket (Go, ex-nhooyr.io/websocket)](https://github.com/coder/websocket) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[websocket-scaling]]
- [[server-sent-events]]
- [[webhooks]]
- [[webrtc]]
- [[../providers/cloudflare/durable-objects]]
- [[../security/security-principles]]
