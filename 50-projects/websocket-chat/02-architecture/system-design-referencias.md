---
tags:
  - system-design
  - websocket
  - chat
  - go
status: em-progresso
created: '2026-04-24'
---
# System Design — Chat WebSocket (Referências)

## Visão Geral

Este documento consolida os padrões arquiteturais dos principais apps de mensagens (WhatsApp, Signal, Telegram) para guiar o design do nosso sistema de chat WebSocket em Go.

---

## WhatsApp — Arquitetura

### Camadas principais

```
[Cliente CLI]
     |
[WebSocket Connection]
     |
[Connection Gateway]  ← servidor WebSocket stateful
     |
[Message Router]
     |          |
[Message     [Presence
  Store]       Service]
     |
[Offline Queue]
```

### Conceitos-chave

- **Gateway stateful**: cada servidor WebSocket sabe quais clientes estão conectados a ele
- **User-to-Server Mapping**: mapa `clientID → serverID` para rotear mensagens entre gateways (Redis em produção, em memória para nós)
- **Heartbeat**: ping a cada 30s, fecha conexão após 3 pongs perdidos
- **ACK de entrega**: mensagem persiste antes do ACK → garante at-least-once delivery
- **Deduplicação no cliente**: UUID por mensagem, cliente ignora duplicatas → effectively exactly-once

### Fluxo de mensagem (Alice → Bob)

```
1. Alice → Gateway via WebSocket
   { id, from: alice, to: bob, body, timestamp }

2. Gateway:
   - Valida mensagem
   - Persiste no Message Store (write-ahead)
   - Envia ACK para Alice ✓

3. Router busca conexão de Bob:
   - Bob ONLINE  → entrega direta → Bob envia ACK → Alice vê ✓✓
   - Bob OFFLINE → salva na Offline Queue → notificação push
                 → quando Bob reconecta, recebe mensagens pendentes em ordem
```

### Ordenação de mensagens

- Sequence number por conversa (atomic increment)
- Chave de partição: `hash(sorted(userA, userB))`
- Cliente reordena por seq se chegarem fora de ordem

---

## Signal — Criptografia E2E

### Protocolo (para Fase 3)

**X3DH (Extended Triple Diffie-Hellman)** — handshake inicial:
- Cada cliente gera par de chaves (pública + privada)
- Chaves públicas ficam no servidor (Key Server)
- No primeiro contato, os dois derivam uma shared session key via X3DH
- Servidor nunca vê a plaintext

**Double Ratchet** — criptografia por mensagem:
- Nova chave derivada para cada mensagem
- Forward secrecy: chaves passadas não podem ser recalculadas
- Break-in recovery: chaves futuras não podem ser calculadas a partir de uma chave comprometida
- Dois ratchets combinados: Symmetric-key ratchet + Diffie-Hellman ratchet

**Primitivas criptográficas recomendadas:**
- Troca de chaves: `X25519` (Curve25519)
- Criptografia de mensagens: `AES-256-GCM` ou `ChaCha20-Poly1305`
- Hash/MAC: `HMAC-SHA256` / `HKDF`
- Senhas: `Argon2id` (Fase 2)

### Implicações de design

- Servidor é um **relay cego** — armazena e repassa ciphertext, nunca plaintext
- Servidor não pode indexar ou buscar conteúdo de mensagens
- Cada par de usuários tem sua própria sessão criptográfica

---

## Telegram — MTProto

### Diferenças do Signal

- MTProto é protocolo proprietário (não usa Signal Protocol)
- Chats normais: criptografados servidor-cliente (servidor vê plaintext)
- Secret Chats: E2E com Diffie-Hellman, não sincroniza entre dispositivos
- Foco em velocidade e confiabilidade em conexões móveis fracas

### Transporte

- Suporta: TCP, HTTP, WebSocket (WS/WSS), UDP
- Conexão persistente ao servidor MTProto quando app está em foreground
- Latência típica < 100ms

---

## Nosso Sistema — Mapeamento por Fase

| Conceito | WhatsApp/Signal | Nossa Fase 1 | Nossa Fase 2 | Nossa Fase 3 |
|---|---|---|---|---|
| Identidade | Phone number + device key | UUIDv7 por sessão | Username + Argon2 | Keypair X25519 |
| Transporte | WebSocket + TLS | WebSocket | WebSocket | WebSocket |
| Persistência | Cassandra + S3 | SQLite | SQLite | SQLite |
| Presença | Redis TTL | Map em memória | Map em memória | Map em memória |
| Criptografia | Signal Protocol (E2E) | Nenhuma | TLS (transporte) | X25519 + ChaCha20-Poly1305 |
| Entrega offline | Offline Queue durável | SQLite pending | SQLite pending | SQLite pending (ciphertext) |
| ACK | 3 estados (sent/delivered/read) | sent/delivered | sent/delivered | sent/delivered |

---

## Decisões de Design para Fase 1

### Protocolo de mensagens (JSON sobre WebSocket)

Todo payload tem campo `type`. Tipos suportados:

```json
// Servidor → Cliente (boas-vindas)
{ "type": "welcome", "client_id": "01966..." }

// Cliente → Servidor (enviar mensagem)
{ "type": "send_message", "to": "01966...", "body": "oi" }

// Servidor → Cliente (receber mensagem)
{ "type": "receive_message", "id": "msg_uuid", "from": "01966...", "body": "oi", "sent_at": "..." }

// Servidor → Cliente (confirmação de entrega)
{ "type": "delivery_confirmation", "message_id": "msg_uuid", "status": "delivered" }

// Servidor → Cliente (mensagens pendentes na reconexão)
{ "type": "pending_messages", "messages": [...] }

// Servidor → Cliente (erro)
{ "type": "error", "code": "INVALID_PAYLOAD", "message": "..." }
```

### Schema SQLite

```sql
CREATE TABLE clients (
    id          TEXT PRIMARY KEY,  -- UUIDv7
    created_at  DATETIME NOT NULL
);

CREATE TABLE messages (
    id           TEXT PRIMARY KEY,  -- UUIDv7
    from_id      TEXT NOT NULL REFERENCES clients(id),
    to_id        TEXT NOT NULL REFERENCES clients(id),
    body         TEXT NOT NULL,
    status       TEXT NOT NULL DEFAULT 'pending', -- pending | delivered
    sent_at      DATETIME NOT NULL,
    delivered_at DATETIME
);

CREATE INDEX idx_messages_to_id_status ON messages(to_id, status);
```

### Concorrência no servidor

```
Server {
    clients: sync.RWMutex + map[string]*Client  // Registro em memória
    db:      *sql.DB                             // SQLite (serializado)
}

Por conexão:
    goroutine readLoop  → lê mensagens do cliente
    goroutine writeLoop → escreve mensagens para o cliente (via channel)
```

---

## Referências

- [WhatsApp Architecture — kindatechnical.com](https://kindatechnical.com/system-design-interview/designing-a-real-time-chat-system-whatsapp-architecture.html)
- [Signal Double Ratchet Specification](https://signal.org/docs/specifications/doubleratchet/)
- [Signal Protocol — Wikipedia](https://en.wikipedia.org/wiki/Signal_Protocol)
- [Telegram MTProto](https://blogfork.telegram.org/mtproto)
