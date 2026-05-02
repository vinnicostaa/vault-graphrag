---
inclusion: always
---

# Full-Stack TypeScript — Projeto de Referência

O projeto anterior (`full-stack/`) é a **referência de domínio e regras de negócio** do Master Síndico. Foi construído em TypeScript (Fastify + SolidJS + Drizzle) e contém a lógica de negócio validada em produção que o backend Go deve reimplementar com arquitetura superior.

## O que usar como referência

| Artefato | Localização | Usar para |
|---|---|---|
| Análise técnica completa | `full-stack/contextos/ANALISE_COMPLETA.md` | Entender bugs e decisões do projeto anterior |
| Análise de modelagem | `full-stack/contextos/ANALISE_COMPLETA_MODELAGEM.md` | Schema de banco, relacionamentos |
| Contexto Stripe | `full-stack/contextos/stripe-context.md` | Fluxos de billing, webhooks, planos |
| Contexto Mux | `full-stack/contextos/MUX_VIDEO_CONTEXT.md` | Upload, HLS, webhooks de vídeo |
| Telas e fluxos | `full-stack/contextos/telas.md` | Jornadas de usuário por persona |
| Perfis e onboarding | `full-stack/contextos/ONBOARDING_CADASTRO_VS_PERFIL.md` | Regras de onboarding por persona |
| Arquitetura proposta | `full-stack/docs/arquitetura-moderna-master-sindico.md` | Padrões a adotar no Go |
| Plano de implementação | `full-stack/plans/master-sindico-implementation-plan.md` | Roadmap e prioridades |

## O que NÃO copiar do projeto anterior

O projeto TypeScript tem problemas identificados na análise técnica. **Não replique:**

- ❌ **God Objects** — `Subscription` (450 linhas), `Invoice` (550 linhas). No Go: entidades enxutas + domain services separados
- ❌ **Vazamento de domínio** — repositories retornando tipos do ORM. No Go: mappers explícitos, domínio nunca conhece pgx/sqlc
- ❌ **Ausência de testes** — 0% de cobertura. No Go: testes entram após implementação base
- ❌ **N+1 queries** — múltiplos use cases com queries em loop. No Go: sqlc com JOINs explícitos
- ❌ **Logs com PII** — email e dados pessoais em logs. No Go: mascarar CPF/email em logs
- ❌ **Módulo billing sem use cases** — lógica direto no domain service. No Go: use cases obrigatórios
- ❌ **Inconsistência estrutural** — módulos com estruturas diferentes. No Go: todos os módulos seguem o mesmo padrão

## O que melhorar em relação ao projeto anterior

O projeto TypeScript identificou problemas que o Go deve resolver desde o início:

- ✅ **Domain Events** — o TS não tinha. Go implementa via `IDomainEvent` + event bus
- ✅ **CQRS estrito** — o TS misturava reads e writes. Go: commands e queries separados
- ✅ **Tenant isolation** — o TS não tinha multi-tenancy real. Go: `WHERE condominium_id = $tenantID` em toda query
- ✅ **1 device por conta** — o TS tinha session limit mas não device fingerprint. Go: BeyondCorp completo
- ✅ **Imutabilidade** — o TS não tinha `timeline_entries` imutável. Go: sem `deleted_at`, sem UPDATE
- ✅ **Property-based testing** — o TS tinha 0% cobertura. Go: PBT para regras críticas

## Módulos do projeto anterior → Bounded contexts Go

| TypeScript (full-stack) | Go (backend) | Observações |
|---|---|---|
| `modules/auth/` | `modules/identity/` | Zitadel substitui JWT próprio |
| `modules/billing/` | `modules/billing/` | Mesma lógica Stripe, arquitetura melhor |
| `modules/profile/` | `modules/identity/` + `modules/institutional/` | Perfis distribuídos por contexto |
| `modules/communication/` | `modules/commercial/` | Connect Me + contratos |
| `modules/search-engine/` | `modules/content/` | OpenSearch substitui PostgreSQL FTS |
| `modules/onboarding/` | `modules/identity/` | Onboarding como use case do identity |
| `modules/user/` | `modules/identity/` | User entity no identity context |

## Value Objects validados no projeto anterior (replicar no Go)

O projeto TypeScript tem VOs bem implementados. Replicar a lógica de validação:

- `EmailVO` — lowercase, regex RFC 5322
- `CPFVO` — algoritmo de validação + máscara
- `CNPJVO` — algoritmo de validação + máscara
- `PasswordVO` — mínimo 8 chars, maiúscula, minúscula, número
- `MoneyVO` — int64 centavos, operações seguras (já existe em `pkg/money/`)
- `PhoneVO` — E.164 format, validação por país

## Regras de negócio críticas do projeto anterior

Estas regras foram validadas em produção e devem ser preservadas:

- **Session limit**: máximo N sessões ativas por usuário (configurável via env)
- **Quota reset**: Connect Me reseta anualmente, uploads mensalmente
- **Trial por persona**: Síndico 15d, Empresa 7d, Parceiro 30d (não é global)
- **Visibility ready**: usuário só aparece na busca após completar onboarding
- **ABAC com cache**: permissões cacheadas no Redis com TTL 5min, invalidadas no webhook Stripe
- **Forgot password cooldown**: não enviar novo código se o anterior ainda é válido
