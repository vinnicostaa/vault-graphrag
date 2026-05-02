---
title: _ingested/refactoring-guru
type: note
tags:
  - ingested
  - refactoring-guru
  - patterns
  - tracking
source: ../refactoring-guru
created: 2026-04-22T00:00:00.000Z
updated: '2026-04-24'
---

# Ingestão — refactoring-guru

**Tracker** das destilações. Fila viva em [[../refactoring-guru/_toc#Prioridade de destilação]].

**Nota de copyright**: site é comercial (livro pago). Destilar **em pt-BR próprio**, sem copiar parágrafos nem exemplos de código integralmente. Atribuir com link ao original.

## Dívidas prioritárias (P0/P1)

Esta fonte tem **deuda larga** (muitas notas possíveis). A prioridade é **pull-based**: destilar quando um pattern/smell for citado numa revisão ou AP-### real, não "adiantar" tudo.

| Prio | Origem | Destino | Status |
|---|---|---|---|
| P0 | 22 GoF patterns em [[../refactoring-guru/raw/design-patterns/]] | `10-knowledge/patterns/<pattern>.md` (on-demand) | ✅ 22/22 (2026-04-24, D-vault-001) |
| P0 | 23 Code smells em `60-sources/refactoring-guru/raw/` (flat, sem subpasta `smells/`) | `10-knowledge/antipatterns/AP-<smell>.md` (on-demand) | 🟡 13/23 destilados em AP-001..AP-013 (2026-04-24, D-vault-009) |
| P1 | 66 refactorings Fowler em `60-sources/refactoring-guru/raw/` (flat, sem subpasta `refactorings/`) | catálogo em `10-knowledge/methodology/refactoring-techniques.md` | ⏳ pendente |
| P1 | Grupos de smells (bloaters, change-preventers, couplers, dispensables, oo-abusers) | taxonomia em `10-knowledge/antipatterns/_moc` | ⏳ pendente |

## Destiladas em staging local (`_notes/`)

*(nenhuma ainda)*

## Promovidas pro vault canônico

### 2026-04-24 — 22 GoF patterns + 3 categorias (D-vault-001)

Todos em `10-knowledge/patterns/`:

- **Creational (5)**: [[../../10-knowledge/patterns/singleton]], [[../../10-knowledge/patterns/factory-method]], [[../../10-knowledge/patterns/abstract-factory]], [[../../10-knowledge/patterns/builder]], [[../../10-knowledge/patterns/prototype]]
- **Structural (7)**: [[../../10-knowledge/patterns/adapter]], [[../../10-knowledge/patterns/bridge]], [[../../10-knowledge/patterns/composite]], [[../../10-knowledge/patterns/decorator]], [[../../10-knowledge/patterns/facade]], [[../../10-knowledge/patterns/flyweight]], [[../../10-knowledge/patterns/proxy]]
- **Behavioral (10)**: [[../../10-knowledge/patterns/chain-of-responsibility]], [[../../10-knowledge/patterns/command]], [[../../10-knowledge/patterns/iterator]], [[../../10-knowledge/patterns/mediator]], [[../../10-knowledge/patterns/memento]], [[../../10-knowledge/patterns/observer]], [[../../10-knowledge/patterns/state]], [[../../10-knowledge/patterns/strategy]], [[../../10-knowledge/patterns/template-method]], [[../../10-knowledge/patterns/visitor]]
- **Categorias (3)**: [[../../10-knowledge/patterns/creational-patterns]], [[../../10-knowledge/patterns/structural-patterns]], [[../../10-knowledge/patterns/behavioral-patterns]]

## Vizinhos

- [[../refactoring-guru/_toc]]
- [[_moc]]
- [[../../00-meta/STATE]] — D-vault-001
