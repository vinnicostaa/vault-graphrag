---
title: MOC — ADRs Master Síndico
type: moc
tags: [moc, master-sindico, adr, architecture-decision]
project: master-sindico
created: 2026-04-22
---

# ADRs — Architecture Decision Records

Registros **permanentes** de decisões arquiteturais. Formato: `NNNN-slug-descritivo.md` (numeração sequencial). **Imutáveis** — mudança gera ADR superseder (`Supersedes: ADR-NNNN`).

## Template

Ver [[../../../../40-templates/adr-template]] *(a criar)*.

Estrutura básica:

```yaml
---
title: ADR NNNN — <título>
type: adr
adr: NNNN
status: proposed | accepted | deprecated | superseded
supersedes: NNNN | null
superseded-by: NNNN | null
date: YYYY-MM-DD
context-links: [D-###, requirement-id]
---
```

Corpo: **Contexto** (forças) · **Decisão** (o que) · **Consequências** (positivas/negativas) · **Alternativas consideradas**.

## ADRs atuais (a criar com base em D-### do STATE)

Levantamento inicial — migrar D-### de alta relevância arquitetural pra ADRs formais:

- ADR-0001 — Clean Architecture + DDD + CQRS como base obrigatória
- ADR-0002 — Gin abstraído via `contracts.HTTPRouter` (sem acoplamento direto)
- ADR-0003 — Zitadel como provider OIDC (vs Auth0/Clerk)
- ADR-0004 — Stripe via `IPaymentGateway` (abstração para troca futura)
- ADR-0005 — sqlc para queries tipadas (vs ORM)
- ADR-0006 — pgx/v5 (vs database/sql+lib padrão)
- ADR-0007 — goose para migrations (vs migrate)
- ADR-0008 — zerolog (vs slog)
- ADR-0009 — UUIDv7 como PK (vs UUIDv4 / incremental)
- ADR-0010 — Mux como provider de vídeo (vs Cloudflare Stream / AWS IVS)
- ADR-0011 — Livekit self-hosted (vs Agora / Twilio Video)
- ADR-0012 — ABAC com cache Redis 5min invalidado por webhook Stripe
- ADR-0013 — Timeline imutável em `institutional` (sem UPDATE, sem DELETE)
- ADR-0014 — 1-device policy via `device_fingerprint` na sessão
- ADR-0015 — UoW dentro de bounded context; Saga ao cruzar serviço externo
- ADR-0016 — BeyondCorp adaptado como security posture
- ADR-0017 — Railway como hospedagem inicial (migração ECS planejada em M4)
- ADR-0018 — OpenSearch vs PG tsvector para busca
- ADR-0019 — NATS JetStream pra mensageria (quando necessário)
- ADR-0020 — OpenTelemetry como observability padrão

## Vizinhos

- [[../_moc]]
- [[../../STATE]] — decisões vivas (D-###) antes de virarem ADR
- [[../../../../30-agents/playbooks/research-loop]] — pré-requisito pra abrir ADR
