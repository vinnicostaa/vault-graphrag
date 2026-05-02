---
created: '2026-04-24'
tags:
  - state
  - governance
  - websocket-chat
title: STATE — websocket-chat
type: note
updated: '2026-04-24'
---

# STATE — websocket-chat

Arquivo **vivo** de bloqueadores, decisões em aberto e dívidas técnicas.

Última revisão: `2026-04-24` (sessão de arquitetura — system-overview criado)

---

## 🚧 Bloqueadores Ativos

*Nenhum bloqueador crítico no momento.*

---

## 🎯 Progresso — Fase 1

### Setup do projeto — ✅ CONCLUÍDO 2026-04-24

- ✅ Go workspace configurado (`go.work` com `./client` e `./server`)
- ✅ Dependência `github.com/coder/websocket` instalada e vendorizada
- ✅ `server/main.go` — servidor WebSocket básico funcional
- ✅ `client/main.go` — cliente WebSocket básico funcional
- ✅ Requisitos da Fase 1 documentados no vault (11 requisitos)
- ✅ System design de referência (WhatsApp/Signal/Telegram) documentado
- ✅ Arquitetura de solução completa (`system-overview.md`)
- 📋 Resolver DT-001 (lib UUIDv7) — próximo
- 📋 Tasks de implementação (`fase1-tasks.md`) — próximo
- 📋 Implementação

---

## 📐 Decisões Arquiteturais (D-0XX)

### D-001 — UUIDv7 como identidade de cliente
- **Data**: 2026-04-24
- **Contexto**: Precisamos de IDs únicos por cliente que sejam ordenáveis cronologicamente
- **Decisão**: UUIDv7 gerado pelo servidor no momento da conexão
- **Alternativas rejeitadas**: UUIDv4 (não ordenável), ID incremental (colisão em multi-instância futura)
- **Reversibilidade**: baixa (afeta schema SQLite)
- **Lib a definir**: ver DT-001

### D-002 — SQLite como persistência (Fase 1)
- **Data**: 2026-04-24
- **Contexto**: Precisamos de persistência simples sem overhead de servidor de banco
- **Decisão**: SQLite via `modernc.org/sqlite` (pure Go, sem CGO)
- **Alternativas rejeitadas**: PostgreSQL (overhead desnecessário para Fase 1), arquivo JSON (sem queries)
- **Reversibilidade**: média (schema pode ser migrado para Postgres na Fase 4+)

### D-003 — Write-ahead antes de entrega
- **Data**: 2026-04-24
- **Contexto**: Garantir que mensagens não sejam perdidas em caso de falha na entrega
- **Decisão**: Persistir mensagem no SQLite **antes** de tentar entrega ao destinatário
- **Fontes**: [ByteByteGo — WhatsApp 40B messages](https://blog.bytebytego.com/p/how-whatsapp-handles-40-billion-messages) (baseado em talks oficiais WhatsApp Engineering)
- **Reversibilidade**: alta

### D-004 — Protocolo JSON com campo `type`
- **Data**: 2026-04-24
- **Contexto**: Precisamos de um protocolo extensível para as fases futuras
- **Decisão**: Todo payload JSON tem campo `type` obrigatório. Tipos Fase 1: `welcome`, `send_message`, `ack`, `receive_message`, `delivery_confirmation`, `offline_notice`, `pending_messages`, `error`
- **Reversibilidade**: baixa (clientes dependem do protocolo)

### D-005 — Servidor como relay cego (preparação Fase 3)
- **Data**: 2026-04-24
- **Contexto**: Na Fase 3, o servidor não poderá ler o conteúdo das mensagens (E2E)
- **Decisão**: Servidor não processa conteúdo, apenas roteia. Facilita adição de E2E na Fase 3 sem refactor
- **Fontes**: [Signal.org — X3DH](https://signal.org/docs/specifications/x3dh/); [Signal.org — Double Ratchet](https://signal.org/docs/specifications/doubleratchet/)
- **Reversibilidade**: alta

### D-006 — 1 goroutine readLoop + 1 writeLoop por sessão
- **Data**: 2026-04-24
- **Contexto**: WebSocket não é thread-safe para writes concorrentes. Precisamos de isolamento por sessão.
- **Decisão**: Cada sessão tem 2 goroutines: `readLoop` (lê do WebSocket) e `writeLoop` (escreve via channel). Writes nunca diretos — sempre via `session.send <- payload`.
- **Fontes**: WhatsApp/Erlang model (1 processo por conexão); Go WebSocket best practices
- **Reversibilidade**: baixa

### D-007 — Channel por sessão para writes
- **Data**: 2026-04-24
- **Contexto**: Múltiplas goroutines podem querer escrever para o mesmo cliente (router, heartbeat, pending messages)
- **Decisão**: `Session.send chan []byte` — qualquer goroutine envia para o channel, writeLoop serializa os writes
- **Fontes**: Go concurrency patterns — channel como mutex implícito
- **Reversibilidade**: média

### D-008 — Organização em pacotes separados
- **Data**: 2026-04-24
- **Contexto**: Preparar para crescimento e reutilização entre client/server
- **Decisão**: `server/`, `message/`, `store/`, `uuid/` como pacotes. `message/` compartilhado entre client e server
- **Fontes**: Clean Architecture — separação de responsabilidades
- **Reversibilidade**: alta

### D-009 — Roadmap de clientes: CLI → Web → Desktop Rust
- **Data**: 2026-04-24
- **Contexto**: Meta do projeto é recriar o WhatsApp com múltiplos clientes
- **Decisão**: Fase 1 CLI (Go), Fase 4 Web (browser), Fase 5 Desktop (Rust). Protocolo WebSocket/JSON é agnóstico de cliente.
- **Fontes**: Decisão do usuário
- **Reversibilidade**: alta

### D-010 — Status de mensagem: 3 estados (como WhatsApp)
- **Data**: 2026-04-24
- **Contexto**: WhatsApp usa 3 estados visuais (✓ enviado, ✓✓ entregue, ✓✓ azul lido)
- **Decisão**: `pending` → `delivered` → `read`. Fase 1 implementa pending/delivered. Fase 2+ adiciona read.
- **Fontes**: [WhatsApp FAQ — Read Receipts](https://faq.whatsapp.com/665923838265756)
- **Reversibilidade**: média

---

## 💳 Dívidas Técnicas (DT-0XX)

### DT-001 — Validar lib UUIDv7 em Go
- **Data**: 2026-04-24
- **O quê**: Pesquisar e decidir qual lib usar para UUIDv7 (`github.com/google/uuid` v7 support, `github.com/gofrs/uuid` ou outra)
- **Por que adiado**: Foco no design primeiro
- **Sprint alvo**: Fase 1 — antes de implementar

---

## 📚 Histórico (resolvidos)

*Vazio por enquanto.*

---

## Links

- [[_moc]]
- [[AUDIT]]
- [[SESSION_CHARTER]]
- [[04-requirements/fase1-requisitos]]
- [[02-architecture/system-overview]]
