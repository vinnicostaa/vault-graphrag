---
created: '2026-04-24'
tags:
  - moc
  - websocket
  - go
  - chat
title: MOC — websocket-chat
type: moc
updated: '2026-04-24'
---

# MOC — websocket-chat

Sistema de chat CLI em Go via WebSocket. Construído em fases incrementais inspiradas na arquitetura do WhatsApp/Signal/Telegram.

---

## Navegação

| Área | Nota | Status |
|---|---|---|
| Contrato do agente | [[CLAUDE]] | ✅ |
| Estado vivo | [[STATE]] | ✅ |
| Audit | [[AUDIT]] | ✅ |
| Sessão corrente | [[SESSION_CHARTER]] | ✅ |
| Roadmap | [[ROADMAP]] | ✅ |
| Requisitos Fase 1 | [[04-requirements/fase1-requisitos]] | ✅ |
| System Design (referências) | [[02-architecture/system-design-referencias]] | ✅ |
| Overview arquitetural | [[02-architecture/system-overview]] | 📋 |
| Tasks Fase 1 | [[05-tasks/fase1-tasks]] | ✅ |

---

## Fases do projeto

- **Fase 1** 🔨 — UUIDv7, mensagens diretas, SQLite, entrega offline, CLI
- **Fase 2** 📋 — Autenticação (username + Argon2id)
- **Fase 3** 📋 — E2E encryption (X25519 + ChaCha20-Poly1305)

## Links

- [[CLAUDE]]
- [[STATE]]
- [[../../_index]]
