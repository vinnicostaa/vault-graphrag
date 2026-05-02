---
title: Sprint 9 — Web (audit fallout na sessão autônoma 22/04)
type: sprint-spec
tags: [master-sindico, sprint-9, web, tasks]
project: master-sindico
stack: web
sprint: 9
period: "2026-04-21 → 2026-04-22 (sessão autônoma)"
status: completed
milestone_target: m1
created: 2026-04-24
---

# Sprint 9 Web — Audit fallout (axios CVE + web/AGENTS.md refs stale)

**Período**: 2026-04-21 → 2026-04-22 (parte da sessão autônoma backend)
**Status**: ✅ parcial (apenas audit + doc sweep; sem código de features)
**Objetivo**: O Sprint 9 autônomo focou em backend, mas 2 itens web foram tratados — F25 (auditoria axios 1.15 CVE flagged) e F28 (web/AGENTS.md refs stale Fastify/SES/ECS removidos). Nenhuma feature real entrou.

## Escopo desta sprint (web)

Web recebe um sweep de higiene minimal como efeito colateral do sprint autônomo backend. Não há entrega de UI. A descoberta importante: **apps continuam placeholders** (`<div>Hello "/_auth/sign-in"!</div>`), deps críticos (`@tanstack/solid-query`, `@tanstack/solid-form`, `msw`, `vitest`) não instalados, `AuthProvider`/`QueryClientProvider`/`Toaster`/`errorComponent`/`notFoundComponent` comentados em `index.tsx` dos 6 apps. Isso gera o finding 🔴 A-FASE10-DEV-005.

## Tasks

| ID | Título | Status | Link detalhe | A-### relacionado | D-### |
|---|---|---|---|---|---|
| web-9.1 | F25 Audit web/ qualidade — axios 1.15 CVE flagged | ✅ | [[../../../SESSION_CHARTER#F25]] | — | — |
| web-9.2 | F28 web/AGENTS.md refs stale — Fastify/SES/ECS removidos | ✅ | `Development/web/AGENTS.md` | — | D-067 |
| web-9.3 | Diagnóstico A-FASE10-DEV-005 🔴 (web M1 praticamente não iniciado) | ✅ registrado | [[../../../AUDIT#a-fase10-dev-005]] | A-FASE10-DEV-005 | — |

## Dependências

- Sessão autônoma backend 22/04.

## Entregáveis

- Finding registrado: `A-FASE10-DEV-005` 🔴 bloqueador M1 web — cronograma matematicamente inviável sem Opção A/B/C.
- web/AGENTS.md alinhado com stack real (SolidJS + Rsbuild + Biome 2 + Bun + SolidJS ecosystem).

## Gates (`<verify>`)

- `bun run check` ✅ (código placeholder passa)
- `bun audit` flagged axios 1.15 (dependabot triage em Sprint 10)

## Pós-sprint

- **O que deu certo**: diagnóstico honesto do estado real; AGENTS.md limpo.
- **Bloqueadores**: A-FASE10-DEV-005 🔴 entra em Sprint 10 como decisão stakeholder.
- **Dívidas criadas**: todo o M1 web — FE-001..FE-093.
- **Próxima sprint**: [[sprint-10]] — M1 real ou Opção A/B/C.

## Links

- [[sprint-8]]
- [[sprint-10]]
- [[../../../../_moc]]
- [[../../../../SESSION_CHARTER]]
- [[../../../../AUDIT]]
