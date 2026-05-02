---
title: WebSocket scaling — sticky session, pub/sub broker, heartbeat, reconnect
type: pattern
tags:
  - realtime
  - websocket
  - scaling
  - pub-sub
  - load-balancing
  - production
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - WebSocket scaling
  - WebSocket production patterns
---

# WebSocket scaling

**Problema**: [[websocket|WebSocket]] é **stateful** — conexão TCP persistente entre 1 cliente e 1 servidor específico. Isso quebra a abstração "stateless + round-robin" que APIs REST usam. Este documento cobre os padrões para operar WebSocket em produção com N servers e M clients.

> **Regra operacional**: 500k+ conexões por server well-tuned é factível. Limites reais são **file descriptors** (`ulimit -n`), memory (2-10 KB/conn idle), e CPU em ping/pong + message routing.

## Decisões-chave

### 1. Autenticação no handshake

**Problema**: WebSocket upgrade é HTTP GET; JS browser não consegue setar header `Authorization` em `new WebSocket()`. Três padrões:

#### A. Cookie httpOnly (preferido para browser)

Client já logado em `app.example.com` tem cookie de sessão. `new WebSocket('wss://app.example.com/ws')` envia cookie automaticamente. Server valida no handshake (middleware). **Padrão mais seguro e simples para browser**.

#### B. Token em subprotocol

```js
const ws = new WebSocket('wss://...', ['access_token', jwt])
```

Server lê subprotocol, valida, responde escolhendo um subprotocol válido. Não é a intenção original do header, mas é pattern comum (Kubernetes kubectl usa).

#### C. Token em query string (EVITAR)

```js
new WebSocket(`wss://...?token=${jwt}`)
```

**Problema**: tokens em URL → logs proxy, log server, history. Só use se absolutamente necessário e com tokens **curtíssimos** (< 1min) que expiram logo.

#### D. Authenticate-after-upgrade

Server aceita conexão; primeira mensagem DEVE ser `{ type: 'auth', token }` em < 5s. Server fecha (4001) se não autenticar. Flexível mas adiciona round-trip.

### 2. Origin check (sempre)

CSWSH (Cross-Site WebSocket Hijacking) — site malicioso abre WebSocket para seu server com cookie do user. WebSocket **não** obedece CORS. **Você** valida `Origin` header no handshake:

```ts
wss.on('connection', (ws, req) => {
  const origin = req.headers.origin
  if (!['https://app.example.com', 'https://admin.example.com'].includes(origin)) {
    ws.close(4003, 'forbidden origin'); return
  }
  // ...
})
```

### 3. Rate limit no handshake

HTTP `/ws` upgrade é endpoint HTTP normal; atacante pode flood. Rate limit por IP:

```ts
// Express/Hono middleware antes do upgrade
rateLimit({ windowMs: 60_000, max: 60 })  // 60 upgrade/min/IP
```

Depois de conectado, rate limit **por connection** em mensagens (ex.: max 100 msg/s).

## Load balancing

### Layer 4 (TCP) vs Layer 7 (HTTP)

| Dimensão | L4 (TCP) | L7 (HTTP) |
|---|---|---|
| Inspeciona handshake HTTP? | Não | Sim |
| Routing por path | Não (apenas IP:port) | Sim (`/ws/chat` vs `/ws/notifications`) |
| TLS termination | Pode ou passthrough | Termina (decrypt) |
| Sticky session | Por IP ou source port hash | Por cookie ou header |

**Regra prática**: L7 para WebSocket se tem múltiplos paths e quer TLS terminated no LB. L4 se throughput puro importa.

### Sticky session — opção MÁ para escala

```
LB → hash(clientIP) → sempre server X
```

**Problema**: server X cai = todos os clientes que estavam nele desconectam; não podem reconectar em outro server porque **state daquele client estava só no X**.

**Quando tolerar**: app pequena, state in-memory leve, reconnect automático aceitável.

### Sticky session MELHORADA — hash de sessão

LB usa cookie de sessão (não IP) para sticky. Mais estável em NAT/mobile (IP muda).

### Stateless preferível — state externo

**Pattern canônico**:
```
┌────────┐   WS   ┌────────────┐   pub/sub   ┌────────────┐
│ Client │◀───────│ WS Server  │────────────▶│ Redis / NATS│
└────────┘        │ (stateless) │             │  Kafka     │
                  └────────────┘             └────────────┘
                       ▲                           │
                       └───────────────────────────┘
                             fan-out para
                             conexões locais
```

- Server **não guarda** state do user além de "qual conexão → qual user".
- Mensagem recebida → publica em canal Redis `user:<uid>` (ou `room:<rid>`).
- **Todos** os servers consomem o canal; fazem fan-out local para suas conexões do user/room.
- Client reconecta em qualquer server; state vem de Redis/DB.

## Pub/sub broker — escolha

### Redis Pub/Sub

- **Prós**: familiar, setup trivial, latência sub-ms.
- **Cons**: sem persistência (mensagem perdida se subscriber não conectado); fan-out CPU-heavy em milhões msg/s.
- **Quando**: cluster pequeno-médio (< 100 WS servers).

### Redis Streams

- **Prós**: persistent + consumer groups; replay.
- **Cons**: consumer logic mais complexa.
- **Quando**: quer durabilidade sem Kafka.

### NATS

- **Prós**: subject-based routing; JetStream para durability; escrito em Go, muito leve.
- **Cons**: conceitos próprios (subject, stream, consumer).
- **Quando**: green field, Go-shop, quer algo moderno.

### Kafka

- **Prós**: throughput massivo; partitioning + replay; ecosystem.
- **Cons**: complexo operacionalmente; overkill para fan-out simples.
- **Quando**: > 1M msg/s; já tem Kafka.

### RabbitMQ

- **Prós**: maturidade; plugins (MQTT bridge, Stomp).
- **Cons**: mais pesado que NATS; fan-out tem nuance.

### Cloudflare Durable Objects (pattern alternativo)

Ver [[../providers/cloudflare/durable-objects]]. **1 DO por room/user** = state + fan-out em 1 isolate. Elimina pub/sub broker. Única alternativa "serverless" real para WebSocket. Limite: DO single-threaded per instance; shard se hot.

## Heartbeat — obrigatório

```ts
// server
ws.isAlive = true
ws.on('pong', () => { ws.isAlive = true })

setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) { ws.terminate(); return }
    ws.isAlive = false
    ws.ping()
  })
}, 30_000)   // 30s — alinhado com menor timeout LB
```

### Timeouts típicos de LB/proxy

| LB/Proxy | Default idle timeout |
|---|---|
| AWS ALB | 60s (configurável até 4000s) |
| GCP Global LB | 30s (default) — aumentar se WS |
| Cloudflare | 100s |
| Nginx | 60s |
| HAProxy | 50s |

**Regra**: ping ≤ timeout LB / 2. 30s é seguro na maioria.

## Reconnect — cliente

```js
function connect() {
  const ws = new WebSocket('wss://...')
  let retries = 0
  const MAX_BACKOFF = 30_000

  ws.addEventListener('close', (event) => {
    if (event.code === 4001 || event.code === 1008) {
      // Permanent — auth expired / policy. Don't retry.
      return
    }
    const backoff = Math.min(1000 * 2 ** retries, MAX_BACKOFF)
    const jitter = Math.random() * 1000
    setTimeout(() => { retries++; connect() }, backoff + jitter)
  })

  ws.addEventListener('open', () => { retries = 0 })
}
```

**Jitter obrigatório** — evita thundering herd quando server reinicia e milhões reconectam no mesmo segundo.

### Resume missed messages após reconnect

Server mantém stream de eventos por user/room em Redis Stream / Kafka / DB. Cliente envia `lastMessageId` no reconnect; server replay desde lá.

## Backpressure

**Problema**: server envia mais rápido que client recebe (slow network). TCP buffer cresce → memória explode.

### Monitorar `bufferedAmount`

```js
// client
if (ws.bufferedAmount > 1_000_000) {   // 1MB
  // Drop messages ou alertar
}
```

### Server-side (Node `ws`)

```ts
ws.on('error', (err) => { /* bufferedAmount high → disconnect slow client */ })

// Check antes de enviar
if (ws.bufferedAmount > THRESHOLD) {
  // skip this message, log
  return
}
```

### Aplicar **flow control**

Para streams contínuos (stock ticker):
- **Throttle** (server): max N msg/s por client.
- **Debounce** (server): agrupar em batch a cada X ms.
- **Drop**: priority queue; drop msg antigas quando buffer cheio.
- **Selective**: client subscreve "qualidades" diferentes (simulcast-style).

## Connection limits por server

| Componente | Limite típico |
|---|---|
| File descriptors | `ulimit -n` — aumentar (65536+) |
| TCP sockets (Linux) | `net.ipv4.ip_local_port_range`, `net.core.somaxconn` |
| Memory per conn idle | 2 KB kernel + 5-30 KB user space (lib dependent) |
| Memory per conn com compression (permessage-deflate) | +50 KB-1 MB context |
| CPU per ping cycle | µs → negligível até 500k+ |
| CPU per message routing | depende do work |

**Realístico 2026**:
- Node.js well-tuned: **50k-100k** connections/core.
- Go (gnet, coder/websocket): **200k-500k+** connections/core.
- Erlang/Elixir (Phoenix Channels): **milhões** (WhatsApp fez 2M em 1 node em 2012).
- Rust (tokio + tungstenite): **200k-500k** connections/core.

Discord usa Elixir/OTP por isso. ~5M conexões por node em picos.

## Deploy patterns

### Kubernetes

- **Service** `type: ClusterIP` + **Ingress** com WebSocket support.
- Nginx Ingress: `nginx.org/websocket-services: "chat-service"`.
- `keepalive_timeout` e `proxy_read_timeout` altos (600s+).
- HPA (horizontal autoscaler) baseado em connection count métrica custom (não só CPU).
- **Graceful shutdown**: no SIGTERM, send close frame 1012 (server restart) + wait X s + kill. Client reconecta em outro pod.

### AWS ALB

- ALB suporta WebSocket nativamente.
- `target_group` com `deregistration_delay` = 120s (tempo pra conexões drenarem).
- Sticky: "Application-based cookie" se precisa.
- CloudWatch custom metrics por connection count.

### Cloudflare

- [[../providers/cloudflare/workers|Workers]] suporta WebSocket.
- [[../providers/cloudflare/durable-objects|Durable Objects]] + Hibernation API — milhões de conexões idle sem custo; wake on message.
- Tunnel: `cloudflared` proxies WS nativamente.
- Worker limit: 32768 open WebSockets per Worker instance; use DO para mais.

### Fly.io / Render / Railway

PaaS com suporte WebSocket nativo (passthrough). Region stickiness ajuda latência.

## Monitoring obrigatório

| Métrica | Alerta |
|---|---|
| **Active connections per server** | > 80% capacidade → scale out |
| **Message latency p99** | > 500ms → investigate |
| **Reconnect rate** | > 5% / min → problema de server/LB |
| **Close code distribution** | muito `1006` → LB/network issue |
| **Messages per sec (in/out)** | drift indica backpressure |
| **Memory per connection** | crescendo indica leak |
| **Ping RTT p99** | > 1s → slow network / CPU pegged |

## Degradation / fallback

Alguns proxies corporativos bloqueiam WebSocket. Libs como **Socket.IO** tentam WebSocket → falha → fallback HTTP long-polling automático. Native WS não tem fallback; aplicação decide se cai para [[server-sent-events|SSE]] ou long-poll.

## Como grandes empresas operam

- **WhatsApp (2012, famoso)**: 2M WebSocket connections em 1 node Erlang. FreeBSD + custom tunings. Hoje > 2B users com clusters.
- **Discord**: Elixir/OTP; 5M concurrent conexões por node; gateway sharding por guild.
- **Slack**: WebSocket para message delivery; RedisPub/Sub backplane; geo-distributed; long-poll fallback em mobile com battery-saver.
- **Figma**: WebSocket para multi-user cursors/selections; Postgres + in-memory fan-out; CRDTs no client.
- **Binance / Coinbase / Bybit**: market data feeds via WebSocket; 3s heartbeat (mais agressivo); rate limits rigorosos.
- **OpenAI realtime API**: WebSocket para agent tool calls interleaved com audio streaming.
- **GitHub Copilot Chat**: WebSocket para streaming tokens + LSP-like operations.
- **AWS IoT Core**: MQTT over WebSocket para milhões de devices.

## Checklist produção

- [ ] Auth via cookie httpOnly **ou** subprotocol; **não** query string.
- [ ] Origin check no handshake.
- [ ] Rate limit no upgrade endpoint + por connection.
- [ ] Heartbeat ping/pong 30s.
- [ ] Reconnect com exponential backoff + jitter.
- [ ] State externo (Redis/DB) — server stateless.
- [ ] Pub/sub broker para fan-out cross-server.
- [ ] `wss://` (TLS) sempre.
- [ ] Close codes semânticos (4000+ para app-specific).
- [ ] Message size limit (`maxPayload`).
- [ ] Backpressure: `bufferedAmount` check.
- [ ] Graceful shutdown (SIGTERM → close 1012 → drain).
- [ ] Monitoring: active conn, latency p99, reconnect rate.
- [ ] `ulimit -n` 65536+.
- [ ] LB idle timeout > heartbeat interval.
- [ ] Audit log de conexões (user_id, IP, duration).

## Anti-patterns

Ver [[websocket|websocket.md §Anti-patterns]] — tudo lá se aplica. Adicionais específicos a escala:

1. **State in-memory sem sync** entre servers — `user:online` map inconsistente.
2. **Broadcast N² sem broker** — 10k conns × 10k msgs = 100M ops.
3. **Sticky session permanente** — impede drain em deploy.
4. **Pub/sub sem particionamento** — 1 Redis instance vira bottleneck.
5. **Log por mensagem** — 1M msg/s × log entry = disk full em horas.
6. **Close code padrão `1000` para tudo** — client não distingue user banido vs restart planejado.
7. **SIGTERM brusco** — clients veem 1006 (abnormal close); retry storm.

## Relações

- **Protocol**: [[websocket]]
- **Alternativas**: [[server-sent-events]] (one-way; mesmo problema de scaling stateful), [[webhooks]] (sem stateful connection)
- **WebRTC signaling**: [[webrtc]] usa WebSocket para signaling com mesmas considerações
- **Provider**: [[../providers/cloudflare/durable-objects]] (pattern alternativo sem broker), [[../providers/cloudflare/workers]] (WS serverless)
- **Segurança**: [[../security/security-principles]], [[../runtime-antipatterns/op-024-rate-limit-ausente|OP-024]], [[../runtime-antipatterns/op-005-cache-sem-versionamento|OP-005]]

## Fontes

- [websocket.org — WebSockets at scale](https://websocket.org/guides/websockets-at-scale) (consultada 2026-04-24)
- [Socket.IO — scaling guide](https://socket.io/docs/v4/tutorial/step-9) (consultada 2026-04-24)
- [Phoenix Channels overview](https://hexdocs.pm/phoenix/channels.html) (consultada 2026-04-24)
- [Discord engineering — how we handle 5M concurrent users](https://discord.com/blog/how-discord-handles-two-and-half-million-concurrent-voice-users-using-webrtc) (consultada 2026-04-24)
- [WhatsApp Erlang scaling (2012 talk)](https://www.erlang-factory.com/upload/presentations/558/efsf2012-whatsapp-scaling.pdf) (consultada 2026-04-24)
- [Nginx WebSocket proxy docs](http://nginx.org/en/docs/http/websocket.html) (consultada 2026-04-24)
- [AWS ALB WebSocket support](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-listeners.html) (consultada 2026-04-24)
- [Cloudflare — scaling WebSocket with Durable Objects](https://developers.cloudflare.com/durable-objects/examples/websocket-hibernation-server/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[websocket]]
- [[server-sent-events]]
- [[../providers/cloudflare/durable-objects]]
- [[../providers/cloudflare/workers]]
- [[../security/security-principles]]
- [[../runtime-antipatterns/op-024-rate-limit-ausente]]
