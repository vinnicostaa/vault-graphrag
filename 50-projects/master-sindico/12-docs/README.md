---
title: README — Master Síndico v2
type: guide
tags: [readme, master-sindico, v2, onboarding, docs]
source: legado README.md + portfolio-de-produtos.md + STEERING
created: 2026-04-23
updated: 2026-04-23
---

# Master Síndico v2

> Plataforma SaaS brasileira de **governança condominial**: conecta síndicos, moradores, empresas do mercado condominial, agências de marketing e comércio local via critério na contratação (reputação auditável), memória institucional imutável (timeline) e ecossistema unificado.

**Não é**: ERP condominial, chat, sistema financeiro, app de comunicação interna.

**É**: 7 sub-produtos integrados (Governança, Connect Me, Reputação, Vídeos, Assembleias Live, LMS+Banco Talentos, Vizinhança) rodando sobre 6 bounded contexts.

Este README é o **entry point** do repositório. Aponta pro resto.

---

## Pitch curto

- 🟢 **Timeline imutável** como prova documental — governança auditável, não ERP
- 🎯 **Reputação Bronze→Diamante** determinística — critério na escolha de fornecedor
- 🤝 **Connect Me unidirecional** (não chat) — formaliza contratação, preserva auditabilidade
- 📹 **Vídeos com lock 90d** — churn prevention + curadoria editorial
- 🗳️ **Assembleia live** com quórum fracional + ata imutável — aderência à Lei BR
- 👥 **Identidade única, contextos múltiplos** — 1 CPF/CNPJ = 1 User, vários papéis

---

## Estrutura da pasta (Johnny.Decimal)

```
master-sindico-v2/
├── CLAUDE.md, STATE.md, AUDIT.md, SESSION_CHARTER.md, STEERING.md, ROADMAP.md, DoR-DoD.md
├── _moc.md, _ingestion-log.md
├── 00-product/       visão, personas, business-model, portfolio-de-produtos, glossary
├── 01-domain/        bounded-contexts, ubiquitous-language, invariants, domain-events, business-rules
├── 02-architecture/  C4 (context/containers/components), clean-arch, patterns, event-driven, multi-tenant, ADRs
├── 03-subprojects/   backend (Go), web (SolidJS), mobile (Flutter)
├── 04-requirements/  global + functional/<bc>.md (identity, billing, institutional, commercial, content, assembly)
├── 05-tasks/         backlog.md + by-sprint/ + by-module/
├── 06-execution/     STEPS zero→prod, dev, qa, deploy, rollback, incident + playbooks
├── 07-quality/       CLEAN_CODE, CLEAN_ARCH, CODE_CALISTHENICS, PATTERNS
├── 08-security/      BEYOND_CORP, threat-model, OWASP Top 10, LGPD, CVE process
├── 09-testing/       test-strategy, unit/integration/e2e/load/security, coverage
├── 10-schedule/      cronograma-macro, milestones, calendar
├── 11-audit/         AUDIT (espelho raiz), audit-log/, dual-check-log/, postmortems/
├── 12-docs/          README (este), adr-index, changelog, how-to/, runbooks/
├── 13-research/      Beyond-Corp + big techs (Google, Netflix, YouTube, Instagram, TikTok, Meet, LinkedIn) + condominial BR
└── 90-ingested/      material-fonte herdado do legado (vault/50-projects/master-sindico/) + _archive/ + _source-material/
```

Detalhamento navegacional em [[../_moc]].

---

## Como rodar local (quando código existir)

**Contexto**: esta pasta (`master-sindico-v2/`) é **remontagem arquitetural** (só docs). O código real vive em `vault/50-projects/master-sindico/backend/web/app/` (legado). A migração completa acontece na Fase 6 do plano.

### Pré-requisitos (legado que vai ser herdado)

- Go 1.26+
- Bun 1.x (web)
- Flutter 3.x + Dart 3.5+
- Docker + Docker Compose (PG, Redis, Zitadel, MinIO, Livekit locais)
- Stripe CLI (webhooks locais)
- Railway CLI (deploy)

### Setup

Ver [[../06-execution/playbooks/onboarding-novo-projeto]].

```bash
# Backend
cd backend
cp .env.example .env     # editar com keys de teste
docker compose up -d     # PG + Redis + Zitadel + MinIO + Livekit
make migrate
make run                 # :8080

# Web
cd web
bun install
bun dev                  # :5173

# Mobile
cd app
flutter pub get
flutter run -d chrome
```

### Smoke test

```bash
curl localhost:8080/health          # → 200
curl localhost:8080/health/ready    # → 200 (PG + Redis + Zitadel ok)
```

---

## Como contribuir

1. **Ler primeiro** (obrigatório):
   - [[../CLAUDE]] — contrato do agente desta pasta
   - [[../STEERING]] — não-negociáveis
   - [[../DoR-DoD]] — Definition of Ready/Done
   - [[../06-execution/dev]] — fluxo dev diário (SDD 5-fase)
2. **Gate 0** toda sessão: abrir [[../11-audit/AUDIT]]; 🔴🟡 bloqueia
3. **Task pick**: backlog em [[../05-tasks/backlog]]
4. **Ciclo SDD**: Discuss → Plan → Execute → Verify → Ship
5. **PR**: ≥ 1 reviewer + CI verde + AC verificados
6. **Documentação**: atualizar doc correspondente ao mesmo PR

Onboarding: [[../06-execution/playbooks/onboarding-novo-projeto]].

---

## Stack

| Camada | Tech |
|---|---|
| Backend | Go 1.26 + Gin (abstraído) + pgx/v5 + sqlc + goose + zerolog |
| DB | PostgreSQL 18 |
| Cache | Redis 7 |
| Auth | Zitadel (OIDC Authorization Code + PKCE) |
| Billing | Stripe |
| Vídeo | Mux (HLS, upload presigned, lock 90d) |
| Live (assembleias) | Livekit (WebRTC SFU) |
| Busca | `ISearchProvider` real — OpenSearch ou Meilisearch (ADR-0042 pending dual-check; D-120 revoga D-067 PG tsvector) |
| Email | Resend |
| SMS | Twilio |
| Push | FCM |
| Storage | MinIO (dev) / AWS S3 (prod) |
| Messaging | NATS JetStream (quando escalar) |
| Observability | OpenTelemetry + Grafana + Sentry |
| Frontend | SolidJS + TypeScript + Vite + Tailwind |
| Mobile | Flutter + Dart |
| Infra | Railway (migração AWS ECS em discussão — D-024) |

Decisões arquiteturais em [[../02-architecture/adr/_moc]] + [[../STATE]].

---

## Marcos

- 🟢 **M1** — 2026-05-08 — MVP contratual (Identity + Billing + Institutional base; frontend web + mobile)
- 🟡 **M2** — 2026-06-20 — Connect Me 4 fluxos + Reputação + Vídeos + Vizinhança + Banco Talentos
- 🔵 **M3** — 2026-07-20 — Assembleia Live + LMS thin + a11y WCAG AA + SLO 99.5%

Detalhe em [[../ROADMAP]] e [[../10-schedule/milestones]].

---

## Links essenciais

### Governança viva
- [[../CLAUDE]] — contrato agente
- [[../STATE]] — decisões (D-###) + dívidas (DT-###)
- [[../11-audit/AUDIT]] — findings (A-###)
- [[../SESSION_CHARTER]] — ordens da sessão
- [[../STEERING]] — não-negociáveis

### Produto
- [[../00-product/vision]]
- [[../00-product/personas]]
- [[../00-product/portfolio-de-produtos]]
- [[../00-product/business-model]]
- [[../00-product/glossary]]

### Domínio + arquitetura
- [[../01-domain/bounded-contexts]]
- [[../01-domain/invariants]]
- [[../01-domain/domain-events]]
- [[../02-architecture/c4-context]]
- [[../02-architecture/patterns]]
- [[../02-architecture/event-driven]]
- [[adr-index]] — índice ADRs

### Requirements + tasks
- [[../04-requirements/global]]
- [[../04-requirements/functional/_moc]]
- [[../05-tasks/backlog]]
- [[../05-tasks/by-sprint/_moc]]

### Execução + schedule
- [[../06-execution/STEPS]]
- [[../06-execution/dev]]
- [[../06-execution/deploy]]
- [[../10-schedule/cronograma-macro]]
- [[../10-schedule/milestones]]

### Segurança
- [[../08-security/BEYOND_CORP]]
- [[../08-security/threat-model]]
- [[../08-security/lgpd]]

### Docs operacionais (esta pasta)
- [[how-to/_moc]] — guias passo-a-passo
- [[runbooks/_moc]] — runbooks de operação
- [[changelog]] — changelog (seed; populado por deploy)
- [[adr-index]] — índice de ADRs

### Vault raiz
- `../vault/CLAUDE.md` — contrato raiz
- `../vault/30-agents/playbooks/_moc.md` — playbooks genéricos
- `../vault/10-knowledge/` — conhecimento atemporal
- `../vault/50-projects/master-sindico/` — legado vivo (substituído em Fase 6)
