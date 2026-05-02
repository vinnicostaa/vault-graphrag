---
title: Legacy Summary — o que sobreviveu do full-stack
version: 1.0
status: stable
type: reference
---

# Legacy Summary — full-stack TypeScript (arquivado)

## O que era

`full-stack/` em `../../../full-stack/` é uma iteração anterior do Master Síndico, em **TypeScript** com Fastify + Drizzle ORM + SolidJS (no mesmo Turborepo). Foi desenvolvida durante ~2 meses e **abandonada** em favor do backend Go atual.

## Stack do legacy

- Backend: Fastify 5, TypeScript 5.9, Awilix DI, Drizzle ORM v1-beta
- Frontend: SolidJS + Rsbuild (mesma do atual — herdado)
- Auth: JWT próprio (jose) + Argon2 + OAuth Google via Arctic
- Permissions: CASL-based ABAC (comentado no startup — ver antipattern A1)
- Billing: Stripe (arquitetura similar ao atual)
- Dados: PostgreSQL via Drizzle + Redis

## Estado ao ser abandonado

- **17 use cases** implementados (13 auth + 2 profile + 1 onboarding + 1 user)
- **5 repositórios** de perfil (Resident, Syndic, Enterprise, LocalCompany, Marketing)
- **11 migrations** Drizzle aplicadas
- **0 testes de integração**, coverage unitário indeterminado
- Dashboard de produto admin, Connect Me, Vídeos, Search — **não implementados**

## O que sobreviveu como **conhecimento**

1. **Modelo de domínio** — definição de personas, planos, quotas, matriz funcional. Migrou para `requirements/personas-and-plans.md` + `requirements/functional-matrix.md`.

2. **Regras de negócio conceituais** — 1-device, trava 90 dias, Connect Me unidirecional, imutabilidade Timeline. Vieram do discovery com o cliente, persistem no atual.

3. **Decisões resolvidas** com o cliente — Decision 1-12 no `requirements.md` atual vieram dessa fase. Algumas refinadas, todas mantidas em essência.

4. **Analises técnicas** — `full-stack/contextos/ANALISE_COMPLETA.md` e similares contêm mapeamento de gaps entre escopo contratado e implementação. Úteis como histórico, desatualizados como tech.

## O que **não** sobreviveu (e por quê)

| Stack / decisão | Motivo de descarte |
|---|---|
| **Drizzle ORM v1-beta** | Instável, breaking changes entre releases, SQL gerado com bugs. Substituído por sqlc v1.30 (estável, deterministic). |
| **CASL ABAC** | Complexidade para o que o MS precisa; engine próprio em Go é mais previsível, testável e performático. |
| **Awilix DI** | Boilerplate + runtime overhead. Go usa DI manual no `main.go` — explícito e type-safe. |
| **JWT próprio + Arctic OAuth** | Reinventar auth é anti-padrão. Zitadel cloud assume responsabilidade completa de IAM (MFA, reset, OAuth Google/Apple). |
| **Fastify schemas + jsonschema** | Substituído por validator Go + sqlc tipado + Zod no frontend. |
| **Node.js/TypeScript runtime** | Go tem concorrência nativa (goroutines) para WebSocket de assembly, binário único, deploy mais simples, performance superior em throughput. |
| **Turborepo gerenciando TS + Go** | Overhead desnecessário; hoje `full-stack/` está no mesmo workspace mas sem build compartilhado. |

## Principais erros documentados

Compilados em `antipatterns-to-avoid.md`. Resumo rápido:

- A1 — `PermissionService` comentado sem teste de regressão
- A2 — Vazamento de domínio entre bounded contexts via import direto
- A3 — If/else hell em use cases de update (switch de 5 cases)
- A4 — `authorize()` duplicado em 4 métodos
- A5 — Ordem inconsistente do fluxo Fetch/Validate/Authorize/Mutate
- A6 — ABAC sem log estruturado de denial
- A7 — Mutations não invalidavam cache (sem domain events)
- A8 — Race condition em `incrementUsage` sem lock
- A9 — Validação só na aplicação, DB aceitava lixo
- A10 — Logs sem campos mínimos (user_id, tenant_id)
- A11 — Drizzle v1-beta instável
- A12 — User virou God Object com 40 colunas

## Destino do diretório `full-stack/`

**Arquivo histórico**, intocado. Não editamos, não rodamos. Permanece no repo pela decisão **D2** (aprovada por João): extrair conhecimento útil para `.kiro/specs/master-sindico/reference/` e depois arquivar/deletar. Extração **concluída**. Deletar quando João aprovar — não é ação que agente faz sozinho (destrutivo).

## Mapeamento de arquivos legacy → specs atuais

| Arquivo legacy | Migrado para |
|---|---|
| `full-stack/contextos/CONTEXT_BRAIN.md` | `requirements/` genérico (conceitual) |
| `full-stack/contextos/ONBOARDING_CADASTRO_VS_PERFIL.md` | `requirements/institutional.md` (futuro) + `personas-and-plans.md` |
| `full-stack/contextos/PROFILE_MODULE_DOCS.md` | Superseded pelo `data-model.md` (futuro) com sqlc |
| `full-stack/contextos/DATABASE_VALIDATION.md` | Parte validada está em `requirements.md` atual; resto obsoleto |
| `full-stack/contextos/stripe-context.md` | Superseded por Context7 MCP (sempre docs oficiais atualizados) |
| `full-stack/contextos/MUX_VIDEO_CONTEXT.md` | Parcialmente em `requirements/content.md` (futuro) |
| `full-stack/contextos/ANALISE_COMPLETA*.md` | Destilado em `antipatterns-to-avoid.md` |
| `full-stack/plans/architecture-refined.md` | Superseded por `design.md` (raiz) + `design/modules-architecture.md` (futuro) |
| `full-stack/plans/master-sindico-implementation-plan.md` | Superseded por `tasks/sprint-*.md` |
| `full-stack/docs/status-api-vs-escopo.md` | Obsoleto (status do legacy; projeto atual tem seus próprios marcos) |

## Referências

- `reference/antipatterns-to-avoid.md` — para qualquer implementação sensível
- `requirements/personas-and-plans.md` — modelo de domínio herdado
- `steering/fullstack-reference.md` — padrões do legacy mantidos no `steering/` quando aplicáveis
