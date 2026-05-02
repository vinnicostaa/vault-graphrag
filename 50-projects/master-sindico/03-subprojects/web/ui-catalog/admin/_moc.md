---
title: MOC — Admin Master Síndico (ui-catalog web)
type: moc
tags: [master-sindico, ui-catalog, web, admin, gap, moc]
project: master-sindico
status: gap
stack: web
created: 2026-04-24
---

# MOC — Admin Master Síndico (Web) — GAP CRÍTICO

Este MOC documenta **ausência de telas** do cliente para o portal administrativo interno. Ver arquivo único e explicativo:

- [[GAP-admin-master-sindico]] — GAP CRÍTICO + 5 capacidades legadas + checklist de perguntas para cliente

## Contexto canônico existente (para consulta)

- [[../../../admin-privilegios]] — Canon D-058: Admin é role transversal, não app separada. Lista AD1–AD8 conceitual (feature-slice `admin-panel` em `web/`). Sub-papéis ABAC (super-admin, moderador, suporte, financeiro).
- [[../../../../02-architecture/adr/0026-admin-mfa-m1]] — MFA obrigatório M1 + step-up em `/admin/v1/*`.
- [[../../../../02-architecture/adr/0024-moderation-cascade]] — Moderação evolutiva.
- [[../../../../STATE]] — D-054 (4-eyes), D-056 (agnosticismo), D-058 (role transversal).

## AD1–AD8 conceitualmente

Referência em [[../../../admin-privilegios]] §4.1 — placeholders sem wireframe cliente:

| ID | Tela | Rota | Sub-papel principal |
|---|---|---|---|
| AD1 | Painel admin (home) | `/admin` | todos |
| AD2 | Moderação de vídeos | `/admin/moderacao/videos` | moderador |
| AD3 | Harassment reports | `/admin/harassment-reports` | moderador |
| AD4 | ABAC matrix (override) | `/admin/abac` | super-admin |
| AD5 | Usuários internos | `/admin/users` | suporte + super-admin |
| AD6 | Dashboards técnicos | `/admin/dashboards` | super-admin |
| AD7 | LGPD console | `/admin/lgpd` | suporte + super-admin |
| AD8 | Billing admin | `/admin/billing` | financeiro + super-admin |

**Nenhuma dessas telas foi especificada pelo cliente** — todas necessitam wireframes+campos+fluxos pós-M1.

## Sprints previstos

- **Sprint 8–9 (M2)**: backend (`/admin/v1/*`, `admin_profiles` table) + AD5 + AD8.
- **Sprint 10 (M1, 2026-05-08)**: MFA required em Zitadel + step-up endpoints (ADR-0026 closure).
- **Sprint 11–12 (M2)**: AD2 + AD3 + AD7 + 4-eyes workflow.
- **Sprint 13–14 (M3)**: AD1 + AD4 + AD6 + IP allowlist.

## Links

- [[GAP-admin-master-sindico]]
- [[../../../admin-privilegios]]
- [[../../../../_moc]]
