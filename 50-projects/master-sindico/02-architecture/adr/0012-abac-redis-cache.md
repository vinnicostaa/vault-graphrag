---
title: ADR-0012 — ABAC engine + Redis cache 5min + invalidação via webhook
type: adr
tags: [adr, security, abac, cache, redis, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0012 — ABAC engine + Redis cache 5 min + invalidação webhook

## Status

accepted — 2026-04-23 (herdado de legado)

## Contexto

Autorização baseada apenas em RBAC é insuficiente: precisamos ABAC (attribute-based) para expressar "síndico do condomínio X pode ler timeline do condomínio X se membership ativa". Ordem de avaliação estrutura decisões + debug.

## Decisão

**ABAC matrix ordenada** avaliada em middleware:

```
admin_bypass  → se user.role = platform_admin AND /admin/v1/*, pass
role          → user.role in allowed_for_endpoint
action        → action_permitted(user.role, endpoint)
tenant        → request.tenant_id = user.active_tenant_id (ou matching membership)
scope         → resource.scope_attributes satisfy request context
```

### Cache

Decisão ABAC cacheada em **Redis 5 min** com key `abac:{user_id}:{tenant_id}:{endpoint}:{method}`. Write-through — atualização síncrona com mudança.

### Invalidação

- **Webhook Stripe** `customer.subscription.updated` → invalida `abac:{user_id}:*` (scope muda com plano).
- **Webhook Zitadel Actions** `user.role.changed` → invalida `abac:{user_id}:*`.
- **Mudança local de role** (síndico promove conselheiro) → invalida direto (mesmo processo).
- TTL 5 min garante consistência eventual se webhook falhar.

## Consequências

### Positivas

- Avaliação ABAC < 1ms (cache hit) vs ~5ms (DB lookup).
- Ordem determinística — bugs de authz raros e diagnosticáveis.
- Invalidação via webhook garante fresh data em mudança de plano/role.

### Negativas

- 5 min max de janela de staleness se webhook falhar — aceitável para master-sindico (não é trading algorítmico).
- Cache explode com muitos endpoints × users: mitigação com TTL curto + eviction LRU no Redis.
- Mudança de política (nova regra) requer **cache flush global** — operação coordenada.

## Fail-closed

- Cache miss + Redis down → avaliação DB direta; se DB também falhar → **deny**.
- Ambiguidade na matrix → **deny**.

## Alternativas consideradas

1. **OPA (Open Policy Agent)** — portável, poderoso. M2+ considerar migração se políticas ficarem complexas.
2. **AWS Cedar / Verified Permissions** — gerenciado; AWS-only lock.
3. **Casbin** — Go-native, RBAC+ABAC; considerado; simplicidade da matriz custom ganhou.
4. **Cache in-memory (LRU)** — perde multi-instance sync; Redis é melhor.

## Referências

- [[../../13-research/beyond-corp/_destilado]] §2.2 Princípio 3 + §3.2 (policy engine distribuído)
- [[../../13-research/instagram/_destilado]] §3 (TAO write-through em autorização)
- [[0016-beyondcorp-security-posture]]
