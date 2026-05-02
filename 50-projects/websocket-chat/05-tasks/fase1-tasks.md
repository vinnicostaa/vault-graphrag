---
title: Fase 1 — Tasks de implementação
type: note
tags: [tasks, websocket-chat, phase-1, backlog]
created: 2026-04-24
updated: 2026-04-24
---

# Fase 1 — Tasks

Lista executável da Fase 1 do projeto `websocket-chat`. Stub criado em 2026-04-24 para fechar link quebrado ([[../../../00-meta/AUDIT|A-vault-021]]).

> **Status**: a detalhar pelo operador. Arquivo criado como âncora do grafo até o planner produzir breakdown formal.

## Escopo da Fase 1 (conforme [[../CLAUDE]])

UUIDv7 por cliente, mensagens diretas, SQLite (modernc.org/sqlite, pure Go), entrega offline, CLI (stdin/stdout).

## Tasks (alto nível — breakdown detalhado a fazer)

- [ ] **T1-01** — Resolver DT-001 (escolher lib UUIDv7). Critério: decisão em [[../STATE]].
- [ ] **T1-02** — Criar `02-architecture/system-overview.md` (design técnico detalhado com schema SQLite + protocolo JSON + write-ahead).
- [ ] **T1-03** — Implementar servidor: registro thread-safe (map+RWMutex), handler WebSocket, persistência SQLite.
- [ ] **T1-04** — Implementar cliente CLI (stdin/stdout).
- [ ] **T1-05** — Fluxo write-ahead (persistir antes de entregar; status `pending`/`delivered`).
- [ ] **T1-06** — Entrega offline (pending messages na reconexão).
- [ ] **T1-07** — Testes `go test ./... -race -count=1` + gates `go build`, `go vet`.
- [ ] **T1-08** — README atualizado com instruções de uso.

## DoD por task

Cada task fecha quando:
- Código compila (`go build ./...`).
- Testes verdes com `-race`.
- Sem `else` após `if err != nil`.
- Goroutines de sessão com `recover`.
- Commit atômico.

## Links

- [[_moc]]
- [[../CLAUDE]]
- [[../STATE]]
- [[../AUDIT]]
- [[../ROADMAP]]
- [[../04-requirements/fase1-requisitos]]
- [[../02-architecture/system-design-referencias]]
