---
title: OP-005 — Cache sem versionamento quando regra muda
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - cache
  - security
id: OP-005
severity: Critical
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Stale cache permissions
  - Cache-invalidation-on-rule-change
doc-consulted: '2026-04-24'
---

# OP-005 — Cache sem versionamento quando regra muda

**Severidade**: Critical (segurança — permissões stale)
**Categoria**: Cache

## Sintoma

Cache key é `abilities:{userId}`. Admin muda permissões do plano no banco; cache já tem valor calculado com regras antigas. Até o TTL expirar (minutos/horas), usuário opera com permissões desatualizadas.

Quando a regra **restringe** permissão, é brecha de segurança: usuário que perdeu feature ainda consegue executar por quanto durar o cache.

## Por que é problema

- **Segurança**: permissões stale = bypass autorização por tempo de cache.
- **Compliance**: audit trail vai mostrar "usuário X executou Y às 10h", regra o proibia às 9:55 — difícil justificar.
- **TTL curto não resolve**: reduz janela mas não zera; aumenta load no banco.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — abilities cacheada por user, sem hash das regras
const key = `abilities:${userId}`
const cached = await cache.get(key)
if (cached) return cached
const abilities = await computeAbilities(userId)
await cache.set(key, abilities, { ex: 600 })
return abilities
```

## Fix preferido — hash das regras na key

```ts
// Hash determinístico das permissões efetivas do plano + role
const plan = await planRepo.findByUserId(userId)
const permissionsHash = hashPermissions(plan.permissions)   // sha256 estável

const key = `abilities:${userId}:${permissionsHash}`
const cached = await cache.get(key)
if (cached) return cached

const abilities = await computeAbilities(userId, plan)
await cache.set(key, abilities, { ex: 3600 })   // TTL mais longo agora
return abilities
```

Quando admin muda regras do plano → `hashPermissions()` muda → cache miss automático, sem invalidação manual. **Cache self-invalidating** é mais robusto que invalidação ativa.

## Alternativa

- Invalidação proativa via evento: emitir `PlanPermissionsChanged` → worker invalida pattern `abilities:*:{oldHash}`.
- TTL curto + `stale-while-revalidate` quando consistência estrita não é crítica.

## Quando tolerar

Dado cacheado sem implicação de segurança e com tolerância explícita a staleness (ex: contagem de visualizações públicas, featured products). Sempre documentar.

## Relações

- **Patterns**: [[../patterns/cqrs]] (read model separado do write), [[../patterns/observer]] (publish cache invalidation)
- **OPs relacionados**: [[op-008-autorizacao-apenas-cache-hit]] (fallback quando não dá pra versionar), [[op-006-cache-falha-quebra-fluxo]], [[op-007-cache-key-json-gigante]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-005
- Martin Kleppmann — *Designing Data-Intensive Applications*, Ch. 5 (Replication)
- Redis — [Cache invalidation patterns](https://redis.io/docs/latest/develop/use/keyspace-notifications/)

## Links

- [[_moc]]
- [[../patterns/cqrs]]
- [[../security/security-principles]]
- [[op-008-autorizacao-apenas-cache-hit]]
