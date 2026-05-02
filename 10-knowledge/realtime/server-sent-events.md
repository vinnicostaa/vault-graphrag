---
title: Server-Sent Events (SSE) — unidirectional HTTP push
type: concept
tags:
  - realtime
  - sse
  - server-sent-events
  - http
  - push
  - streaming
doc-oficial: https://html.spec.whatwg.org/multipage/server-sent-events.html
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - SSE
  - Server-Sent Events
  - EventSource
---

# Server-Sent Events (SSE)

**O que é**: mecanismo HTML5 para **push server → client** sobre HTTP. Cliente abre conexão HTTP long-lived; server envia eventos formatados em `text/event-stream` contínuo. Spec: HTML Living Standard (WHATWG, 2024+).

> **Mental model**: "HTTP long-poll elegante com reconnect e event IDs built-in". Unidirecional (server → client only); se precisa bidirectional use [[websocket|WebSocket]].

## Quando usar SSE (vs WebSocket)

**SSE é melhor quando**:
- Apenas **server → client** (notifications, live feeds, LLM token streaming, build logs).
- HTTP/2 ou HTTP/3 disponível (múltiplos streams sobre uma conexão TCP).
- Reconnect automático desejado (browser nativo).
- Quer passar por proxies HTTP sem config (WebSocket exige Upgrade support).
- Auth via cookie/Authorization header já funcionar (HTTP standard).

**WebSocket melhor quando**:
- Client precisa enviar msg frequentes (não só request-response).
- Binary payloads compactos (SSE é text-only).
- Subprotocol negotiation (graphql-ws, mqtt, etc.).

**Casos de uso clássicos SSE**:
- OpenAI / Anthropic / Gemini streaming API (respostas LLM token-by-token).
- Server logs / deploy status feeds.
- Real-time dashboards (preço, métricas).
- Notifications push.
- Vercel deployment events.

## Formato do stream

```
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive
X-Accel-Buffering: no                ← Nginx: desabilita buffering

: this is a comment (ignored by client)

event: message
data: {"type":"token","value":"Hello"}

event: message
data: {"type":"token","value":" world"}

id: 42
event: message
data: {"type":"done"}

retry: 5000
```

**Regras**:
- Cada evento separado por `\n\n`.
- Campos: `event:` (tipo, default "message"), `data:` (payload, pode ter múltiplas linhas), `id:` (para resume), `retry:` (ms reconnect).
- Linhas iniciando com `:` são comentários (úteis como heartbeat).
- Encoding **UTF-8 sempre**.
- Sem binary — use base64 se precisa bytes.

## Client API (browser)

```js
const source = new EventSource('/api/stream', {
  withCredentials: true,   // send cookies
})

source.addEventListener('open', () => console.log('connected'))

source.addEventListener('message', (event) => {
  const data = JSON.parse(event.data)
  console.log('msg', event.lastEventId, data)
})

source.addEventListener('custom-event', (event) => {
  // events com `event: custom-event` no server
})

source.addEventListener('error', (event) => {
  if (source.readyState === EventSource.CLOSED) {
    console.log('disconnected permanently')
  } else {
    // Browser faz reconnect automático
  }
})

source.close()
```

**`EventSource` built-in no browser**:
- **Auto-reconnect** em disconnect (default 3s, configurável via `retry:` do server).
- **`lastEventId`** enviado em `Last-Event-ID` header no reconnect → server resume do ponto.
- **Não suporta custom headers em request** (limitação histórica da spec). Workaround: `fetch` + ReadableStream para SSE em casos onde precisa header custom (Authorization com Bearer).

### Fetch API alternative (para header custom)

```js
const resp = await fetch('/api/stream', {
  headers: { Authorization: `Bearer ${token}` }
})
const reader = resp.body.getReader()
const decoder = new TextDecoder()

while (true) {
  const { value, done } = await reader.read()
  if (done) break
  const chunk = decoder.decode(value, { stream: true })
  // parse chunks (handle partial events split across chunks)
}
```

Libs como `@microsoft/fetch-event-source` abstraem parse + reconnect.

## Server (Node + Hono)

```ts
app.get('/api/stream', (c) => {
  return c.streamSSE(async (stream) => {
    let counter = 0
    while (true) {
      await stream.writeSSE({
        event: 'message',
        data: JSON.stringify({ counter, at: Date.now() }),
        id: String(counter++),
      })
      await stream.sleep(1000)
    }
  })
})
```

Hono tem helper `streamSSE`; Express precisa manual:

```ts
app.get('/api/stream', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream')
  res.setHeader('Cache-Control', 'no-cache')
  res.setHeader('Connection', 'keep-alive')
  res.setHeader('X-Accel-Buffering', 'no')
  res.flushHeaders()

  const interval = setInterval(() => {
    res.write(`data: ${JSON.stringify({ ts: Date.now() })}\n\n`)
  }, 1000)

  req.on('close', () => clearInterval(interval))
})
```

## Server (Go)

```go
func streamHandler(w http.ResponseWriter, r *http.Request) {
    flusher, ok := w.(http.Flusher)
    if !ok { http.Error(w, "streaming unsupported", 500); return }

    w.Header().Set("Content-Type", "text/event-stream")
    w.Header().Set("Cache-Control", "no-cache")
    w.Header().Set("Connection", "keep-alive")

    ticker := time.NewTicker(1 * time.Second)
    defer ticker.Stop()

    for {
        select {
        case <-r.Context().Done(): return
        case t := <-ticker.C:
            fmt.Fprintf(w, "data: {\"ts\":%d}\n\n", t.Unix())
            flusher.Flush()
        }
    }
}
```

## Resume com `Last-Event-ID`

Browser reconnect envia último `id` visto:

```
GET /api/stream HTTP/1.1
Last-Event-ID: 42
```

Server pula pra evento 43. Se store de eventos é em memória volátil, usar Redis stream ou Postgres logical replication.

## LLM streaming — padrão OpenAI/Anthropic

```
event: message_start
data: {"type":"message_start","message":{"id":"msg_..."}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":"Hello"}}

event: content_block_delta
data: {"type":"content_block_delta","index":0,"delta":{"type":"text_delta","text":" world"}}

event: message_stop
data: {"type":"message_stop"}
```

Cliente reassembla tokens incrementalmente. Anthropic, OpenAI, Gemini, Mistral usam variações deste formato.

## Heartbeat

Proxies/LB fecham conexões idle. Envie comentário a cada 15-30s:

```ts
// Padrão: comentário SSE (não é evento, cliente ignora)
stream.write(':\n\n')   // em Node
stream.writeSSE({ event: 'ping' })   // ou evento ignorado pelo handler
```

Cloudflare, Nginx, AWS ALB timeouts default 60-120s. Comentário a cada 15s é seguro.

## Nginx config obrigatório

```nginx
location /api/stream {
    proxy_pass http://upstream;
    proxy_http_version 1.1;
    proxy_set_header Connection '';
    proxy_buffering off;                 # CRÍTICO — senão nginx buffers o stream
    proxy_cache off;
    proxy_read_timeout 24h;              # max que você aceita
    chunked_transfer_encoding off;
}
```

`proxy_buffering off` é **obrigatório** — sem isso, cliente só vê eventos quando buffer enche.

## Limits / considerações

- **Max connections per browser por origin**: **6 em HTTP/1.1** (limite do browser). HTTP/2 multiplexa → 100+.
- **CORS**: `Access-Control-Allow-Origin` deve estar presente se cross-origin + `credentials: 'include'`.
- **Auth**: se via cookie, funciona normalmente. Se via Bearer token, usar fetch+ReadableStream (não EventSource).
- **Proxies corporativos**: bloqueiam às vezes; WebSocket tem mesmo problema.
- **Mobile radio**: conexão longa consome bateria; prefira WebSocket com ping controlado.

## Anti-patterns

1. **Esquecer `X-Accel-Buffering: no` / `proxy_buffering off`** — eventos são enviados em batch (nginx espera 4KB).
2. **Não implementar heartbeat** — LB/proxy fecha silenciosamente.
3. **`Content-Type` incorreto** (`text/plain` em vez de `text/event-stream`) — EventSource rejeita.
4. **EventSource em cenário bidirectional** — use WebSocket.
5. **Múltiplas conexões SSE por tab em HTTP/1.1** — atinge limite 6; use HTTP/2 ou compartilhe via SharedWorker.
6. **Sem retry no server** quando crash — cliente reconecta mas perde eventos sem `Last-Event-ID` handling.
7. **Payloads binários via SSE** — é text-only; base64 dobra tamanho. Use WebSocket.
8. **Sem `id:` em eventos críticos** — reconnect não resume.

## Como grandes empresas usam

- **OpenAI Chat Completions API** (`stream: true`) — SSE.
- **Anthropic Messages API** (`stream: true`) — SSE.
- **Google Gemini API** — SSE.
- **Vercel Deployments** — SSE para build logs e status.
- **GitHub Actions logs** — similar (também usa WebSocket em alguns contextos).
- **Supabase Realtime (alternative mode)** — tem SSE além de WebSocket.
- **Shopify GraphQL Subscriptions (alternative transport)** — SSE sobre HTTP/2.
- **Twitter/X** (legacy "streaming API" — agora mix SSE + WebSocket).

## Relações

- **Cluster**: [[_moc]] decision tree realtime
- **Alternativas**: [[websocket]] (bidirectional), [[webhooks]] (server-to-server)
- **Escala**: [[websocket-scaling]] (patterns aplicáveis — HTTP/2, backpressure, broker)
- **Providers**: [[../providers/cloudflare/workers]] (SSE em Workers via ReadableStream)
- **Security**: [[../security/security-principles]] (auth via cookie httpOnly preferido)

## Fontes

- [HTML Living Standard — SSE](https://html.spec.whatwg.org/multipage/server-sent-events.html) (consultada 2026-04-24)
- [MDN — Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events) (consultada 2026-04-24)
- [websocket.org — SSE vs WebSocket](https://websocket.org/guides/websockets-vs-sse) (consultada 2026-04-24)
- [@microsoft/fetch-event-source](https://github.com/Azure/fetch-event-source) (consultada 2026-04-24)
- [Anthropic streaming API](https://docs.anthropic.com/en/api/messages-streaming) (consultada 2026-04-24)
- [OpenAI streaming](https://platform.openai.com/docs/api-reference/streaming) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[websocket]]
- [[webhooks]]
- [[websocket-scaling]]
- [[../providers/cloudflare/workers]]
