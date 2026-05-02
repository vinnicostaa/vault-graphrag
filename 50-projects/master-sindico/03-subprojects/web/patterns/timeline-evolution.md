---
title: Pattern — Timeline Evolution
type: pattern
tags:
  - pattern
  - ui
  - web
  - master-sindico
  - timeline
created: '2026-04-27'
updated: '2026-04-27'
---

# Pattern — Timeline Evolution

Visualização temporal append-only para Linha do Tempo institucional, histórico de Plano Diretor, audit trail e mandatos. Apresenta itens em ordem cronológica reversa (mais recente no topo) com agrupamento por mês/ano.

## Estrutura

- **Item**: ícone-tipo + título + autor (síndico/empresa/sistema) + timestamp relativo + corpo + ações (ver detalhes, comentar quando aplicável).
- **Agrupamento**: header `Abril 2026`, `Março 2026` separa runs.
- **Tipos visuais**: cor/ícone diferente por tipo de evento (atividade, validação, comunicado, evento, adendo).
- **Filtros**: tipo, autor, período (sticky no topo).
- **Imutabilidade visual**: item não tem CTA "editar" — só "ver versão" se houver edição vinculada (ADR-0033 ata DB-immutable).

## Quando usar

- Linha do Tempo do condomínio (governança).
- Histórico de Plano Diretor (status changes).
- Audit trail (mostrar para admin/síndico em modal).

## Quando evitar

- Lista que precisa de ordering customizável (drag-drop) — Timeline é cronológica por definição.
- Volume muito grande sem paginação — usar [[feed-infinite-scroll]].

## Stack

SolidJS + Kobalte + UnoCSS + `@motionone` para entrada animada de novos itens em real-time (WebSocket).

## Cross-refs

- [[../../../01-domain/aggregates/TimelineEntry]]
- [[../../../02-architecture/adr/0033-ata-timeline-db-immutability]]
- [[feed-infinite-scroll]] — quando volume cresce
- [[../../07-quality/PATTERNS]]

## Links

- [[_moc]]
