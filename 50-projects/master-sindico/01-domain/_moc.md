---
title: MOC — 01-domain
type: moc
tags: [moc, master-sindico, v2, domain, ddd]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 01-domain

Bounded contexts, ubiquitous language, aggregates, invariants, domain events, state machines, business rules.

**Fase 3/4 do plano** (`/home/vinni/.claude/plans/squishy-drifting-avalanche.md`): destilação completa a partir do legado em `90-ingested/from-vault-50-projects-master-sindico/**` + specs `specs/requirements/*.md`.

**Produto**: SaaS governança condominial BR, multi-tenant, 7 sub-produtos integrados. Ver [[../00-product/portfolio-de-produtos]].

---

## Artefatos canônicos (Fase 3)

### Arquivos-raiz

| Arquivo | Descrição | Destaques |
|---|---|---|
| [[bounded-contexts]] | Catálogo dos 6 BCs + cross-domain | 6 BCs: identity, billing, institutional, commercial, content, assembly |
| [[context-map]] | Relações U/D, Shared Kernel, ACL, Partnership, Conformist, Published Language | Matriz 7×7 + grafo textual |
| [[ubiquitous-language]] | Glossário técnico por BC (complementa glossary cross-BC) | ~150 termos catalogados |
| [[invariants]] | Invariantes testáveis com enforcement + teste | **100 invariantes canônicos** (INV-001..INV-100) |
| [[business-rules]] | Regras de negócio destiladas | **74 regras** (BR-001..BR-074) |
| [[domain-events]] | Catálogo de eventos cross-BC | **82 eventos** (E-001..E-082) |
| [[state-machines]] | Máquinas de estado (10 entidades dinâmicas) | Subscription, VideoModeration, Assembly, ConnectMeRequest, VoteSession, Company Reputation, User Session, MasterPlan Activity, Proposal, HarassmentReport |
| [[aggregates/_moc]] | MOC dos aggregates | **34 aggregates** em 6 BCs |

### Aggregates (pasta [[aggregates/_moc]])

**identity (2)**: [[aggregates/User]] · [[aggregates/Session]]

**billing (3)**: [[aggregates/Subscription]] · [[aggregates/Plan]] · [[aggregates/FeatureUsage]]

**institutional (8)**: [[aggregates/Condominium]] · [[aggregates/Unit]] · [[aggregates/Membership]] · [[aggregates/TimelineEntry]] · [[aggregates/MasterPlan]] · [[aggregates/Announcement]] · [[aggregates/Event]] · [[aggregates/Poll]]

**commercial (9)**: [[aggregates/Company]] · [[aggregates/ConnectMeRequest]] · [[aggregates/Proposal]] · [[aggregates/ClosedDeal]] · [[aggregates/CompanyEvaluation]] · [[aggregates/SupplierVoteSession]] · [[aggregates/HarassmentReport]] · [[aggregates/Partner]]

**content (5)**: [[aggregates/Video]] · [[aggregates/VideoModeration]] · [[aggregates/Course]] · [[aggregates/Enrollment]] · [[aggregates/Certificate]]

**assembly (7)**: [[aggregates/Assembly]] · [[aggregates/AgendaItem]] · [[aggregates/Vote]] · [[aggregates/Minutes]] · [[aggregates/LiveSession]] · [[aggregates/Proxy]] · [[aggregates/AttendanceRecord]]

---

## Métricas (Fase 3 resultado)

| Métrica | Valor |
|---|---|
| Bounded contexts | 6 + cross-domain |
| Aggregates canônicos | 34 |
| Invariantes testáveis | 100 |
| Regras de negócio | 74 |
| Eventos cross-BC | 82 |
| State machines | 10 |

---

## Pendências de dual-check (agregadas)

Cada arquivo lista suas próprias pendências (`P-BC-###`, `P-CM-###`, `P-UL-###`, `P-INV-###`, `P-EVT-###`, `P-BR-###`, `P-SM-###`). Agregar e cruzar na Fase 5.

Principais riscos pra M1:
- Outbox pattern pra eventos (P-EVT-002) — precisa antes de escalar
- Parâmetros exatos Reputação Bronze→Diamante (P-BR-001)
- Timezone BRT vs UTC em resets de quota (P-INV-002)
- Ambiguidade `TenantID` (`condominium_id` vs `company_id`) (P-CM-003)

---

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
- [[../STEERING]] — não-negociáveis
- [[../00-product/portfolio-de-produtos]] — 7 sub-produtos
- [[../00-product/glossary]] — glossário cross-BC
- [[../02-architecture/_moc]] — arquitetura (a destilar; referencia patterns DDD/CQRS/UoW/Saga)
- [[../04-requirements/_moc]] — requirements Req 1-103 (a destilar)
