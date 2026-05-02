---
created: '2026-04-24'
tags:
  - roadmap
  - websocket-chat
title: ROADMAP — websocket-chat
type: note
updated: '2026-04-24'
---

# ROADMAP — websocket-chat

---

## Fase 1 — Identidade e Mensagens Diretas 🔨

**Objetivo**: sistema funcional via CLI com identidade UUIDv7, mensagens diretas, histórico SQLite e entrega offline.

**Status**: em design

### Entregas
- [ ] UUIDv7 por cliente gerado pelo servidor
- [ ] Registro de clientes em memória (thread-safe)
- [ ] Persistência SQLite (clientes + mensagens)
- [ ] Mensagens diretas entre clientes pelo ID
- [ ] Entrega offline (mensagens pendentes na reconexão)
- [ ] Protocolo JSON com campo `type`
- [ ] Heartbeat (ping/pong a cada 30s)
- [ ] CLI funcional (enviar/receber mensagens)

---

## Fase 2 — Autenticação 📋

**Objetivo**: sistema de usuário com credenciais persistentes.

### Entregas
- [ ] Username único por usuário
- [ ] Senha com hash Argon2id
- [ ] Login/registro via CLI
- [ ] Sessão vinculada ao usuário (não à conexão)
- [ ] Histórico de mensagens por usuário (não por sessão)

---

## Fase 3 — Criptografia E2E 📋

**Objetivo**: mensagens criptografadas ponta a ponta, servidor como relay cego.

### Entregas
- [ ] Geração de keypair X25519 por cliente
- [ ] Troca de chaves públicas via servidor (Key Server)
- [ ] Handshake X3DH simplificado no primeiro contato
- [ ] Criptografia de mensagens com ChaCha20-Poly1305
- [ ] Double Ratchet simplificado (forward secrecy)
- [ ] Servidor armazena e repassa apenas ciphertext

---

## Links

- [[_moc]]
- [[STATE]]
- [[04-requirements/fase1-requisitos]]
- [[02-architecture/system-design-referencias]]
