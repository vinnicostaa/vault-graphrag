---
title: MOC — 04-requirements
type: moc
tags: [moc, master-sindico, requirements]
project: master-sindico
created: 2026-04-22
---

# 04-requirements — Requisitos

Requisitos **canônicos**. O **quê** o sistema deve fazer. Separados em:

## Notas canônicas

### Global
- [[global]] — requisitos **transversais** que todos os módulos respeitam (auth, multi-tenant, tracing, PII, i18n)

### Funcional (por bounded context)
- [[functional/identity]] — Req 1-15 (auth, onboarding, sessão, 1-device, admin)
- [[functional/billing]] — Req 16-30 (planos, trial, Stripe, quotas, feature usage)
- [[functional/institutional]] — Req 31-50 (condomínio, unidades, timeline, plano diretor, anúncios, eventos, compliance)
- [[functional/commercial]] — Req 51-72 (Connect Me, empresas, contratos, reputação, marketplace, empresas-sindicas)
- [[functional/content]] — Req 73-85 (vídeos, busca, trava 90d, moderação)
- [[functional/assembly]] — Req 86-100 (assembleias, votação, atas, fila de fala, live)
- [[functional/cross-domain]] — Req 101-103+ (integrações cross-module, eventos, audit trail)
- [[functional/functional-matrix]] — matriz consolidada feature × persona × plano

### Não-funcional
- [[non-functional/performance]] — SLOs, latência p95/p99, throughput
- [[non-functional/security]] — inventário de controles (ABAC, 1-device, PKCE, HMAC webhook)
- [[non-functional/accessibility]] — WCAG AA target em web/mobile
- [[non-functional/internationalization]] — pt-BR first; EN prep
- [[non-functional/scalability]] — shard por região, read replicas
- [[non-functional/reliability]] — uptime 99.5% M3; SLA interno
- [[non-functional/observability]] — OTel, logs estruturados, métricas de negócio
- [[non-functional/maintainability]] — coverage, docs, onboard time

### Compliance
- [[compliance]] — LGPD (consentimento, exclusão, DPO), audit trail, retenção, PCI (se billing PCI-DSS Level 4 como merchant Stripe)

## Fontes (destilar)

- [[../specs/requirements/README]]
- [[../specs/requirements/product-overview]]
- [[../specs/requirements/identity]] · [[../specs/requirements/billing]] · [[../specs/requirements/institutional]] · [[../specs/requirements/commercial]] · [[../specs/requirements/content]] · [[../specs/requirements/assembly]] · [[../specs/requirements/cross-domain]]
- [[../specs/requirements/functional-matrix]]
- [[../specs/requirements/personas-and-plans]]
- [[../specs/requirements/identity-mirror-pattern]]
- [[../content/requirements]] — fonte gigante (206KB)
- [[../contextos/novo escopo-7]] — escopo detalhado

## Vizinhos

- [[../_moc]]
- [[../00-product/_moc]] — visão (fonte dos requisitos)
- [[../01-domain/_moc]] — domínio (regras que requirements implementam)
- [[../05-tasks/_moc]] — tasks (executam os requirements)
- [[../03-subprojects/_moc]] — sub-projetos implementam os requirements
- [[../09-testing/_moc]] — testes validam requirements
