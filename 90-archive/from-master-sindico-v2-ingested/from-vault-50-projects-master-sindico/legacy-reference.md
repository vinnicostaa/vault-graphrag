---
title: Master Síndico — Legacy como referência
type: project
tags: [master-sindico, legacy, reference]
source: .kiro/steering/fullstack-reference.md
created: 2026-04-22
project: master-sindico
---

# Legacy TypeScript como referência

O projeto anterior (`full-stack/` — Fastify + SolidJS + Drizzle em Turborepo) é **referência de domínio e regras de negócio**, não template de código. Foi validado em produção; tem **bugs identificados** que o Go reimplementa com arquitetura superior.

Aplicação do padrão geral em [[../../10-knowledge/methodology/legacy-as-reference]].

## O que usar (artefatos do legacy)

| Artefato | Localização | Usar para |
|---|---|---|
| Análise técnica | `contextos/ANALISE_COMPLETA.md` | Bugs e decisões do projeto anterior |
| Análise de modelagem | `contextos/ANALISE_COMPLETA_MODELAGEM.md` | Schema DB, relacionamentos |
| Stripe context | `contextos/stripe-context.md` | Fluxos de billing, webhooks, planos |
| Mux context | `contextos/MUX_VIDEO_CONTEXT.md` | Upload, HLS, webhooks vídeo |
| Telas e fluxos | `contextos/telas.md` | Jornadas por persona |
| Perfis/onboarding | `contextos/ONBOARDING_CADASTRO_VS_PERFIL.md` | Regras onboarding por persona |
| Arquitetura proposta | `legacy-docs/arquitetura-moderna-master-sindico.md` | Padrões a adotar no Go |
| Plano implementação | `legacy-docs/master-sindico-implementation-plan.md` | Roadmap e prioridades |

## O que NÃO copiar (antipatterns identificados)

- ❌ **God Objects** — `Subscription` (450 linhas), `Invoice` (550 linhas). No Go: entidades enxutas + domain services separados. Ver [[../../10-knowledge/antipatterns/_moc]] AP-014.
- ❌ **Vazamento de domínio** — repositories retornando tipos do ORM. No Go: mappers explícitos, domínio nunca conhece pgx/sqlc. Ver AP-009.
- ❌ **Ausência de testes** — 0% cobertura. No Go: testes entram após base sólida.
- ❌ **N+1 queries** — múltiplos use cases com queries em loop. No Go: sqlc com JOINs explícitos. Ver AP-012.
- ❌ **Logs com PII** — email e dados pessoais em logs. No Go: mascarar CPF/email. Ver AP-022.
- ❌ **Módulo billing sem use cases** — lógica direto no domain service. No Go: use cases obrigatórios.
- ❌ **Inconsistência estrutural** — módulos com estruturas diferentes. No Go: todos seguem o mesmo padrão.

## O que melhorar (vs legacy)

- ✅ **Domain Events** — o TS não tinha. Go: `IDomainEvent` + event bus
- ✅ **CQRS estrito** — TS misturava read/write. Go: commands e queries em arquivos separados
- ✅ **Tenant isolation** — TS não tinha multi-tenancy real. Go: `WHERE condominium_id = $tenantID` em toda query
- ✅ **1 device por conta** — TS tinha session limit mas não device fingerprint. Go: BeyondCorp completo
- ✅ **Imutabilidade** — TS não tinha `timeline_entries` imutável. Go: sem `deleted_at`, sem UPDATE
- ✅ **Property-based testing** — TS tinha 0% cobertura. Go: PBT para regras críticas

## Mapeamento de módulos TS → Go

| TypeScript (full-stack) | Go (backend) | Observações |
|---|---|---|
| `modules/auth/` | `modules/identity/` | Zitadel substitui JWT próprio |
| `modules/billing/` | `modules/billing/` | Mesma lógica Stripe, arquitetura melhor |
| `modules/profile/` | `modules/identity/` + `modules/institutional/` | Perfis distribuídos por contexto |
| `modules/communication/` | `modules/commercial/` | Connect Me + contratos |
| `modules/search-engine/` | `modules/content/` | OpenSearch substitui PG FTS |
| `modules/onboarding/` | `modules/identity/` | Onboarding como use case |
| `modules/user/` | `modules/identity/` | User entity no identity |

## Value Objects validados (replicar lógica no Go)

- `EmailVO` — lowercase, regex RFC 5322
- `CPFVO` — algoritmo de validação + máscara (dígitos verificadores BR)
- `CNPJVO` — algoritmo de validação + máscara
- `PasswordVO` — mínimo 8 chars, maiúscula, minúscula, número
- `MoneyVO` — int64 centavos, operações seguras (já em `pkg/money/`)
- `PhoneVO` — E.164, validação por país

Ver [[../../10-knowledge/patterns/value-object]] *(a criar)* pro padrão genérico.

## Regras de negócio críticas (validadas em produção)

- **Session limit**: máximo N sessões ativas por usuário (configurável via env)
- **Quota reset**: Connect Me reseta anualmente, uploads mensalmente
- **Trial por persona**: Síndico 15d, Empresa 7d, Parceiro 30d — **não** é global
- **Visibility ready**: usuário só aparece na busca após completar onboarding
- **ABAC com cache**: permissões cacheadas no Redis com TTL 5min, invalidadas no webhook Stripe. Ver AP-005 (cache versioning).
- **Forgot password cooldown**: não enviar novo código se o anterior ainda é válido

## Links

- [[_moc]] — MOC do projeto
- [[overview]] — visão geral
- [[../../10-knowledge/methodology/legacy-as-reference]] — padrão genérico
- [[../../10-knowledge/antipatterns/_moc]] — catálogo cross-linguagem
