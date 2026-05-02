---
created: '2026-04-24'
tags:
  - session
  - charter
  - websocket-chat
title: SESSION_CHARTER — Setup inicial (2026-04-24)
type: note
updated: '2026-04-24'
---

# SESSION_CHARTER — Setup inicial do projeto (2026-04-24)

**Escopo da sessão**: Setup do projeto websocket-chat no vault, migração dos requisitos do `.kiro` para o vault Obsidian, e preparação para o design técnico da Fase 1.

**Data de início**: 2026-04-24. **Última atualização**: 2026-04-24.

---

## 1. Ordens do usuário

### 1.1 Configuração da workspace Go
> "irmao, quero que veja se essa go workspace esta configurada corretamente"

### 1.2 Dependências
> "se eu precisar instalar uma dependecia global para todos os packages da workspace, eu dou o go get na raiz da workspace ou manualmente em cada module"

### 1.3 Análise do código
> "pode analisar o server/main.go?" / "agora pode olhar o client/main.go"

### 1.4 Planejamento do sistema
> "quero melhorar e documentar cada coisa, estava pensando em adicionar o recurso de cada cliente ter um id [...] podemos criar um user para cada cliente, como funciona no whatsapp [...] criptografamos com chave publica e privada"

### 1.5 Pesquisa de referências
> "quero sim, pode estudar o system design do whatsapp e telegram e signal e wechat para se espelharmos na arquitetura de solucao deles?, pode usar o vault via mcp do obsidian"

### 1.6 Migração para o vault
> "voce vai trabalhar 100% usando o vault, vai organizar o projeto la dentro como o vault 'e organizado, nada de criar pastas .kiro com spec ou steering aqui, vai usar graphrag de la"

---

## 2. O que foi feito nesta sessão

| ID | Título | Status |
|---|---|---|
| F1 | Verificação da Go workspace | ✅ |
| F2 | Correção do `server/main.go` | ✅ |
| F3 | Correção do `client/main.go` | ✅ |
| F4 | Resolução do vendor inconsistente | ✅ |
| F5 | Pesquisa WhatsApp/Signal/Telegram system design | ✅ |
| F6 | Requisitos Fase 1 documentados | ✅ |
| F7 | Migração completa para o vault | ✅ |
| F8 | Deletar `.kiro/specs/` | ✅ |
| F9 | Arquitetura de solução completa (system-overview) | ✅ |
| F10 | Tasks Fase 1 criadas (05-tasks/fase1-tasks.md) | ✅ |
| F11 | Links quebrados corrigidos (A-vault-021) | ✅ |

---

## 3. Decisões tomadas

- **D-001** — UUIDv7 como identidade de cliente
- **D-002** — SQLite via `modernc.org/sqlite`
- **D-003** — Write-ahead antes de entrega
- **D-004** — Protocolo JSON com campo `type`
- **D-005** — Servidor como relay cego (prep Fase 3)

---

## 4. Próxima sessão

1. Deletar `.kiro/specs/websocket-chat-phase1/`
2. Criar design técnico (`02-architecture/system-overview`)
3. Resolver DT-001 (lib UUIDv7)
4. Criar tasks de implementação (`05-tasks/fase1-tasks`)

---

## Links

- [[_moc]]
- [[STATE]]
- [[AUDIT]]
