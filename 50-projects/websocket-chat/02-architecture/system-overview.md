---
created: '2026-04-24'
tags:
  - architecture
  - system-design
  - websocket-chat
  - go
title: System Overview — Arquitetura de Solução
type: note
updated: '2026-04-24'
---

# System Overview — Arquitetura de Solução

> Baseado em pesquisa de fontes primárias: Meta Engineering Blog, WhatsApp Engineering Team (via ByteByteGo), Signal.org specs oficiais.
> Dual-check aplicado: análise cruzada de 3+ fontes independentes antes de cada decisão.

---

## 1. Visão do Produto

Recriar o sistema de mensagens do WhatsApp, em fases:

| Cliente | Fase | Status |
|---|---|---|
| CLI (Go) | Fase 1 | 🔨 Em andamento |
| Web (browser) | Fase 4 | 📋 Planejada |
| Desktop (Rust) | Fase 5 | 📋 Planejada |

---

## 2. Roadmap de Fases

```
Fase 1 — Fundação (CLI)
  └── UUIDv7 por sessão, mensagens diretas, SQLite, entrega offline, heartbeat

Fase 2 — Identidade Persistente
  └── Username + Argon2id, sessão vinculada ao usuário, histórico por usuário

Fase 3 — Criptografia E2E
  └── X25519 keypair, X3DH handshake, Double Ratchet, servidor relay cego

Fase 4 — Cliente Web
  └── WebSocket no browser, mesma API do servidor, sync de histórico

Fase 5 — Cliente Desktop (Rust)
  └── Mesmo protocolo, multi-device (cada device = identity key própria)
```

---

## 3. Arquitetura do Sistema (inspirada no WhatsApp)

### 3.1 Visão geral

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTES                             │
│  [CLI Go]    [Web Browser]    [Desktop Rust]                │
└──────┬──────────────┬──────────────┬───────────────────────┘
       │              │              │
       └──────────────┴──────────────┘
                      │ WebSocket (TLS na Fase 3+)
                      ▼
┌─────────────────────────────────────────────────────────────┐
│                   CONNECTION GATEWAY                        │
│  • Aceita conexões WebSocket                                │
│  • 1 goroutine por sessão (readLoop + writeLoop)            │
│  • Heartbeat: ping/30s, timeout/10s                         │
│  • Autentica e atribui ID ao cliente                        │
│  • Registro em memória: clientID → *Session                 │
└──────────────────────┬──────────────────────────────────────┘
                       │
          ┌────────────┴────────────┐
          ▼                         ▼
┌─────────────────┐       ┌─────────────────────┐
│  MESSAGE ROUTER │       │   PRESENCE SERVICE  │
│  • Valida msg   │       │  • online/offline   │
│  • Write-ahead  │       │  • last seen        │
│  • Roteia       │       │  • typing indicator │
│  • ACK          │       │  (Fase 2+)          │
└────────┬────────┘       └─────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────────────┐
│                      MESSAGE STORE                          │
│  SQLite (Fase 1-3) → PostgreSQL (Fase 4+)                   │
│  • Tabela clients                                           │
│  • Tabela messages (status: pending | delivered | read)     │
│  • Índice: (to_id, status) WHERE status = 'pending'         │
└─────────────────────────────────────────────────────────────┘
```

### 3.2 Fluxo de mensagem (baseado no WhatsApp)

```
Alice envia mensagem para Bob:

1. Alice → Gateway (WebSocket)
   { type: "send_message", to: "bob-uuid", body: "oi" }

2. Gateway → Message Router
   a. Valida payload (campos obrigatórios, UUID válido)
   b. Gera message_id (UUIDv7)
   c. Persiste no SQLite com status "pending"  ← WRITE-AHEAD
   d. Envia ACK para Alice: { type: "ack", message_id: "..." }  ← ✓ (1 tick)

3. Router verifica presença de Bob:

   BOB ONLINE:
   ├── Envia { type: "receive_message", ... } para Bob via writeLoop channel
   ├── Atualiza status para "delivered" no SQLite
   └── Envia { type: "delivery_confirmation", ... } para Alice  ← ✓✓ (2 ticks)

   BOB OFFLINE:
   ├── Mantém status "pending" no SQLite
   └── Envia { type: "offline_notice", ... } para Alice

4. Bob reconecta:
   ├── Gateway atribui ID (Fase 1: novo UUID; Fase 2+: mesmo userID)
   ├── Router consulta SQLite: SELECT * FROM messages WHERE to_id=? AND status='pending'
   ├── Entrega mensagens em ordem cronológica (sent_at ASC)
   └── Atualiza status para "delivered"
```

### 3.3 Status de mensagem (3 estados — como WhatsApp)

```
pending   → delivered → read
  ✓           ✓✓         ✓✓ (azul)

Fase 1: pending | delivered
Fase 2+: pending | delivered | read
```

---

## 4. Protocolo de Mensagens

### 4.1 Envelope base

Todo payload JSON tem campo `type` obrigatório:

```typescript
interface Payload {
  type: string
}
```

### 4.2 Tipos — Fase 1

```json
// Servidor → Cliente: boas-vindas
{ "type": "welcome", "client_id": "01966c3e-..." }

// Cliente → Servidor: enviar mensagem
{ "type": "send_message", "to": "01966c3e-...", "body": "oi" }

// Servidor → Remetente: ACK (mensagem recebida pelo servidor)
{ "type": "ack", "message_id": "01966c3f-..." }

// Servidor → Destinatário: receber mensagem
{ "type": "receive_message", "message_id": "01966c3f-...", "from": "01966c3e-...", "body": "oi", "sent_at": "2026-04-24T00:00:00Z" }

// Servidor → Remetente: confirmação de entrega
{ "type": "delivery_confirmation", "message_id": "01966c3f-...", "status": "delivered" }

// Servidor → Remetente: destinatário offline
{ "type": "offline_notice", "message_id": "01966c3f-...", "to": "01966c3e-..." }

// Servidor → Cliente: mensagens pendentes (na reconexão)
{ "type": "pending_messages", "messages": [ { ...receive_message }, ... ] }

// Servidor → Cliente: erro
{ "type": "error", "code": "INVALID_PAYLOAD", "message": "campo 'to' obrigatório" }
```

### 4.3 Extensões planejadas (Fase 2+)

```json
// Fase 2 — autenticação
{ "type": "register", "username": "alice", "password": "..." }
{ "type": "login",    "username": "alice", "password": "..." }
{ "type": "auth_ok",  "user_id": "...", "token": "..." }

// Fase 3 — troca de chaves E2E
{ "type": "publish_keys", "identity_key": "...", "prekeys": [...] }
{ "type": "fetch_keys",   "user_id": "..." }
{ "type": "keys_response","identity_key": "...", "prekey": "..." }

// Fase 4+ — presença
{ "type": "presence", "user_id": "...", "status": "online|offline|typing" }
{ "type": "read_receipt", "message_id": "..." }
```

---

## 5. Schema do Banco de Dados

### Fase 1 — SQLite

```sql
-- Clientes conhecidos
CREATE TABLE clients (
    id         TEXT PRIMARY KEY,   -- UUIDv7
    created_at DATETIME NOT NULL DEFAULT (datetime('now'))
);

-- Mensagens
CREATE TABLE messages (
    id           TEXT PRIMARY KEY,                          -- UUIDv7
    from_id      TEXT NOT NULL REFERENCES clients(id),
    to_id        TEXT NOT NULL REFERENCES clients(id),
    body         TEXT NOT NULL,
    status       TEXT NOT NULL DEFAULT 'pending'
                     CHECK(status IN ('pending','delivered','read')),
    sent_at      DATETIME NOT NULL,
    delivered_at DATETIME,
    read_at      DATETIME
);

-- Índice crítico: busca de mensagens pendentes por destinatário
CREATE INDEX idx_messages_pending ON messages(to_id, sent_at)
    WHERE status = 'pending';
```

### Fase 2 — Adicionar usuários

```sql
CREATE TABLE users (
    id           TEXT PRIMARY KEY,   -- UUIDv7
    username     TEXT UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,     -- Argon2id
    created_at   DATETIME NOT NULL DEFAULT (datetime('now'))
);

-- clients passa a referenciar users
ALTER TABLE clients ADD COLUMN user_id TEXT REFERENCES users(id);
```

### Fase 3 — Adicionar chaves E2E

```sql
CREATE TABLE identity_keys (
    user_id     TEXT PRIMARY KEY REFERENCES users(id),
    public_key  BLOB NOT NULL,       -- X25519 public key
    created_at  DATETIME NOT NULL DEFAULT (datetime('now'))
);

CREATE TABLE prekeys (
    id          TEXT PRIMARY KEY,    -- UUIDv7
    user_id     TEXT NOT NULL REFERENCES users(id),
    public_key  BLOB NOT NULL,
    used        INTEGER NOT NULL DEFAULT 0,
    created_at  DATETIME NOT NULL DEFAULT (datetime('now'))
);

-- messages.body passa a ser ciphertext (BLOB)
```

---

## 6. Estrutura do Servidor (Go)

### 6.1 Organização de pacotes

```
server/
├── go.mod
├── main.go              ← entry point: inicializa Server, chama run()
├── server/
│   ├── server.go        ← struct Server, registro de clientes, run()
│   ├── session.go       ← struct Session, readLoop, writeLoop
│   ├── handler.go       ← dispatch de payloads por type
│   └── heartbeat.go     ← ping/pong, detecção de conexão morta
├── message/
│   ├── message.go       ← structs de payload (Payload, SendMessage, etc.)
│   └── types.go         ← constantes de type
├── store/
│   ├── store.go         ← interface Store
│   ├── sqlite.go        ← implementação SQLite
│   └── schema.sql       ← schema inicial
└── uuid/
    └── uuid.go          ← wrapper para geração de UUIDv7
```

### 6.2 Struct principal

```go
type Server struct {
    mu      sync.RWMutex
    clients map[string]*Session  // clientID → *Session
    db      store.Store
}

type Session struct {
    id     string          // UUIDv7
    conn   *websocket.Conn
    send   chan []byte      // writeLoop lê daqui
    server *Server
}
```

### 6.3 Ciclo de vida de uma sessão

```
Accept WebSocket
    │
    ├── Gera UUIDv7 (clientID)
    ├── Persiste client no SQLite
    ├── Registra no Server.clients
    ├── Envia payload "welcome"
    ├── Entrega mensagens pendentes
    │
    ├── go readLoop()   ← lê do WebSocket, despacha para handler
    └── go writeLoop()  ← lê do Session.send, escreve no WebSocket
         │
         └── (ambos terminam quando conn fecha)
              │
              └── defer: remove do Server.clients
```

---

## 7. Estrutura do Cliente (Go CLI)

```
client/
├── go.mod
├── main.go              ← entry point: conecta, inicia loops
├── client/
│   ├── client.go        ← struct Client, connect(), run()
│   ├── reader.go        ← readLoop: recebe payloads do servidor
│   ├── writer.go        ← writeLoop: envia payloads ao servidor
│   └── input.go         ← lê stdin, parseia "<ID> <mensagem>"
└── message/
    └── message.go       ← mesmas structs do server/message/
```

---

## 8. Criptografia — Plano por Fase

### Fase 1 — Sem criptografia de conteúdo
- Transporte: WebSocket plain (localhost/dev)
- Conteúdo: plaintext

### Fase 2 — TLS no transporte
- WebSocket sobre TLS (wss://)
- Conteúdo: ainda plaintext no servidor

### Fase 3 — E2E (Signal Protocol simplificado)

Baseado na spec oficial do Signal ([X3DH](https://signal.org/docs/specifications/x3dh/) + [Double Ratchet](https://signal.org/docs/specifications/doubleratchet/)):

```
Handshake inicial (X3DH simplificado):
  1. Alice gera keypair X25519 (identity key)
  2. Bob publica sua identity key no servidor
  3. Alice busca a public key de Bob
  4. Alice e Bob derivam shared secret via X25519 DH
  5. Shared secret → session key via HKDF

Por mensagem (Double Ratchet simplificado):
  1. Cada mensagem usa uma chave derivada da anterior (KDF chain)
  2. Forward secrecy: chaves passadas não podem ser recalculadas
  3. Mensagem cifrada com ChaCha20-Poly1305

Primitivas:
  - Troca de chaves: X25519 (golang.org/x/crypto/curve25519)
  - Derivação: HKDF-SHA256 (golang.org/x/crypto/hkdf)
  - Cifra: ChaCha20-Poly1305 (golang.org/x/crypto/chacha20poly1305)
  - Senhas (Fase 2): Argon2id (golang.org/x/crypto/argon2)
```

### Fase 4-5 — Multi-device (como WhatsApp 2021)

Baseado no [Meta Engineering Blog](https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device):
- Cada device tem sua própria identity key
- Servidor mantém mapa: userID → [device1_key, device2_key, ...]
- Client-fanout: remetente cifra N vezes para N devices do destinatário
- Linking de device via QR code + provisioning token

---

## 9. Decisões Arquiteturais Registradas

Ver detalhes em [[../STATE]].

| ID | Decisão | Fonte |
|---|---|---|
| D-001 | UUIDv7 como identidade | — |
| D-002 | SQLite via modernc.org/sqlite | — |
| D-003 | Write-ahead antes de entrega | WhatsApp Engineering |
| D-004 | Protocolo JSON com campo `type` | — |
| D-005 | Servidor como relay cego | Signal Protocol |
| D-006 | 1 goroutine readLoop + 1 writeLoop por sessão | WhatsApp/Erlang model |
| D-007 | Channel por sessão para writes (evita race no WebSocket) | Go concurrency patterns |
| D-008 | Pacotes separados: server/, message/, store/, uuid/ | Clean Architecture |

---

## 10. Referências (fontes primárias)

- [Meta Engineering — WhatsApp Multi-Device (2021)](https://engineering.fb.com/2021/07/14/security/whatsapp-multi-device) — arquitetura multi-device, client-fanout, identity keys
- [ByteByteGo — How WhatsApp Handles 40B Messages/Day](https://blog.bytebytego.com/p/how-whatsapp-handles-40-billion-messages) — baseado em talks oficiais do WhatsApp Engineering Team
- [Signal.org — X3DH Specification](https://signal.org/docs/specifications/x3dh/) — protocolo de troca de chaves
- [Signal.org — Double Ratchet Specification](https://signal.org/docs/specifications/doubleratchet/) — criptografia por mensagem
- [WhatsApp FAQ — Read Receipts](https://faq.whatsapp.com/665923838265756) — comportamento oficial dos ticks

## Links

- [[../_moc]]
- [[../STATE]]
- [[system-design-referencias]]
- [[../04-requirements/fase1-requisitos]]
- [[../05-tasks/fase1-tasks]]
