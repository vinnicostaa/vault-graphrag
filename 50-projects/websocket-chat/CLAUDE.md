---
created: '2026-04-24'
tags:
  - claude
  - agent
  - contract
  - websocket
  - go
title: CLAUDE.md — websocket-chat
type: agent-contract
updated: '2026-04-24'
---

# CLAUDE.md — Projeto `websocket-chat`

> **Leia este arquivo inteiro antes de qualquer ação no projeto.**
> Este é o único arquivo que o agente precisa para saber tudo sobre como operar aqui.

---

## 1. O que é este projeto

Sistema de mensagens diretas via WebSocket em Go, operado via CLI. Inspirado na arquitetura do WhatsApp/Signal/Telegram. Construído em 3 fases incrementais:

| Fase | Status | Objetivo |
|---|---|---|
| **Fase 1** | 🔨 Em andamento | UUIDv7 por cliente, mensagens diretas, SQLite, entrega offline, CLI |
| **Fase 2** | 📋 Planejada | Autenticação com username + Argon2id |
| **Fase 3** | 📋 Planejada | E2E com X25519 + ChaCha20-Poly1305 (Double Ratchet simplificado) |

---

## 2. Onde fica cada coisa

### No vault (fonte de verdade do projeto)

```
50-projects/websocket-chat/
├── CLAUDE.md                          ← este arquivo (leia primeiro)
├── _moc.md                            ← índice de navegação
├── STATE.md                           ← decisões vivas (D-XXX) + dívidas (DT-XXX)
├── AUDIT.md                           ← findings de revisão (A-XXX)
├── SESSION_CHARTER.md                 ← histórico de sessões
├── ROADMAP.md                         ← fases e entregas
├── 02-architecture/
│   ├── system-design-referencias.md  ← WhatsApp/Signal/Telegram (pesquisa)
│   └── system-overview.md            ← design técnico da Fase 1 (a criar)
├── 04-requirements/
│   └── fase1-requisitos.md           ← 11 requisitos com critérios de aceitação
└── 05-tasks/
    └── fase1-tasks.md                ← tasks de implementação (a criar)
```

### No repositório (código)

```
websockets/
├── go.work              ← workspace Go (client + server)
├── client/
│   ├── go.mod           ← module: github.com/vinniciusolliveiracostaa/websocket/client
│   └── main.go
├── server/
│   ├── go.mod           ← module: github.com/vinniciusolliveiracostaa/websocket/server
│   └── main.go
└── vendor/              ← dependências vendorizadas (go work vendor)
```

---

## 3. Regras de operação

### O agente SEMPRE faz ao iniciar uma sessão

1. Ler este arquivo (`CLAUDE.md`)
2. Ler `STATE.md` — ver decisões ativas e dívidas abertas
3. Ler `AUDIT.md` — se houver 🔴 aberto, resolver antes de qualquer task nova
4. Ler `SESSION_CHARTER.md` — entender o que foi feito na sessão anterior
5. Identificar a próxima task em `05-tasks/fase1-tasks.md`

### O agente SEMPRE faz ao terminar uma sessão

1. Atualizar `STATE.md` com decisões tomadas (D-XXX) e dívidas descobertas (DT-XXX)
2. Atualizar `AUDIT.md` com findings encontrados (A-XXX)
3. Atualizar `SESSION_CHARTER.md` com o que foi feito
4. Atualizar `05-tasks/fase1-tasks.md` com status das tasks

### O agente NUNCA faz

- ❌ Criar arquivos `.kiro/` — tudo vai no vault via MCP Obsidian
- ❌ Criar steering files no repositório — vai no vault
- ❌ Escrever diretamente no filesystem do vault — sempre via `mcp_obsidian_*`
- ❌ Tomar decisão arquitetural sem registrar em `STATE.md` como D-XXX
- ❌ Usar `else` após `if err != nil` — early return sempre
- ❌ Deixar goroutine sem recover de panic

---

## 4. Stack e dependências

| Componente | Escolha | Motivo |
|---|---|---|
| Linguagem | Go 1.26.2 | — |
| WebSocket | `github.com/coder/websocket` | já instalado e vendorizado |
| Banco | `modernc.org/sqlite` | pure Go, sem CGO |
| UUID | a definir (DT-001) | pesquisar suporte a UUIDv7 |
| Interface | CLI (stdin/stdout) | Fase 1 sem UI |

### Comandos essenciais

```bash
# Instalar dependência em um módulo
cd server && go get <pkg>
cd client && go get <pkg>

# Sincronizar vendor após instalar
go work vendor   # na raiz da workspace

# Build
go build ./...   # na raiz da workspace

# Testes
go test ./... -race -count=1

# Vet
go vet ./...
```

---

## 5. Padrões de código obrigatórios

### Tratamento de erro

```go
// ✅ correto — early return
result, err := doSomething()
if err != nil {
    return fmt.Errorf("contexto: %w", err)
}

// ❌ proibido — else após erro
if err != nil {
    log.Printf("erro: %v", err)
} else {
    // continua...
}
```

### Goroutines de sessão

```go
// ✅ toda goroutine de sessão tem recover
go func() {
    defer func() {
        if r := recover(); r != nil {
            log.Printf("panic recuperado na sessão %s: %v", clientID, r)
        }
    }()
    // lógica da sessão
}()
```

### Registro de clientes (thread-safe)

```go
type Server struct {
    mu      sync.RWMutex
    clients map[string]*Client  // clientID → *Client
    db      *sql.DB
}

// leitura
s.mu.RLock()
client, ok := s.clients[id]
s.mu.RUnlock()

// escrita
s.mu.Lock()
s.clients[id] = client
s.mu.Unlock()
```

### Protocolo JSON

Todo payload tem campo `type` obrigatório:

```go
type Payload struct {
    Type string `json:"type"`
}

// Tipos Fase 1:
// "welcome"               servidor → cliente (boas-vindas + ID)
// "send_message"          cliente → servidor (enviar mensagem)
// "receive_message"       servidor → cliente (receber mensagem)
// "delivery_confirmation" servidor → remetente (confirmação de entrega)
// "pending_messages"      servidor → cliente (mensagens offline na reconexão)
// "error"                 servidor → cliente (erro)
```

### Write-ahead (padrão WhatsApp)

```
1. Recebe mensagem do remetente
2. Persiste no SQLite com status "pending"   ← ANTES de tentar entrega
3. Envia ACK ao remetente
4. Tenta entregar ao destinatário
5. Se entregue: atualiza status para "delivered"
6. Se offline: mantém "pending" para entrega na reconexão
```

---

## 6. Schema SQLite

```sql
CREATE TABLE clients (
    id         TEXT PRIMARY KEY,   -- UUIDv7
    created_at DATETIME NOT NULL
);

CREATE TABLE messages (
    id           TEXT PRIMARY KEY,  -- UUIDv7
    from_id      TEXT NOT NULL REFERENCES clients(id),
    to_id        TEXT NOT NULL REFERENCES clients(id),
    body         TEXT NOT NULL,
    status       TEXT NOT NULL DEFAULT 'pending',  -- pending | delivered
    sent_at      DATETIME NOT NULL,
    delivered_at DATETIME
);

CREATE INDEX idx_messages_to_pending ON messages(to_id, status)
    WHERE status = 'pending';
```

---

## 7. Gates duros (antes de marcar qualquer task como concluída)

1. `go build ./...` — sem erros
2. `go vet ./...` — sem warnings
3. `go test ./... -race -count=1` — todos verdes
4. Nenhuma goroutine de sessão sem `recover`
5. Nenhum `else` após `if err != nil`
6. `STATE.md` atualizado com decisões da task

---

## 8. Decisões arquiteturais ativas

Ver detalhes completos em [[STATE]].

| ID | Decisão |
|---|---|
| D-001 | UUIDv7 como identidade de cliente (lib a definir — DT-001) |
| D-002 | SQLite via `modernc.org/sqlite` (pure Go, sem CGO) |
| D-003 | Write-ahead: persiste antes de entregar |
| D-004 | Protocolo JSON com campo `type` obrigatório |
| D-005 | Servidor como relay cego (preparação para E2E na Fase 3) |

---

## 9. Próximos passos (Fase 1)

1. Resolver DT-001 — escolher lib UUIDv7
2. Criar `02-architecture/system-overview.md` — design técnico detalhado
3. Criar `05-tasks/fase1-tasks.md` — tasks de implementação
4. Implementar

---

## 10. Links essenciais

- [[_moc]] — índice do projeto
- [[STATE]] — decisões e dívidas vivas
- [[AUDIT]] — findings abertos
- [[ROADMAP]] — visão das 3 fases
- [[04-requirements/fase1-requisitos]] — 11 requisitos com critérios de aceitação
- [[02-architecture/system-design-referencias]] — pesquisa WhatsApp/Signal/Telegram
- [[02-architecture/system-overview]] — design técnico (a criar)
- [[05-tasks/fase1-tasks]] — tasks (a criar)
