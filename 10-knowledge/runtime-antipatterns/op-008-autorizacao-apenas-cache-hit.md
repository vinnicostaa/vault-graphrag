---
title: OP-008 — Autorização avaliada apenas no cache hit
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - cache
  - security
  - abac
id: OP-008
severity: Critical
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Stale authorization decision
doc-consulted: '2026-04-24'
---

# OP-008 — Autorização avaliada apenas no cache hit

**Severidade**: Critical (segurança)
**Categoria**: Cache / Segurança

## Sintoma

Cache guarda objeto `{ userId, role, permissions }`. Middleware ABAC lê direto da cache e avalia autorização contra o snapshot cacheado. Se role/permissions mudaram no banco desde último `set`, decisão de autorização fica stale — usuário pode executar ação que acabou de perder.

Variação: `if cache.user.role == 'admin' then allow` sem re-consultar banco.

## Por que é problema

- **Bypass de autorização**: janela do TTL vira brecha de segurança — minutos/horas em que permissões antigas valem.
- **Escalation de privilégio**: role revogada continua válida até expirar.
- **Compliance**: auditor pergunta "como vocês garantem timing de revogação de acesso?" — "até 10 min depois" não costuma passar.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — role cacheada, decisão cega
const user = await cache.get(`user:${userId}`)
if (!user) throw new UnauthorizedError()
if (user.role === 'admin') return allow
// ...
```

## Fix preferido — uma das duas estratégias

### Estratégia A — hash de permissões na key (preferida)

```ts
// Cache key inclui hash do snapshot de permissões do role
const role = await roleRepo.findByUserId(userId)
const key = `abac:${userId}:${hashPermissions(role.permissions)}`
const cached = await cache.get(key)
if (cached) return evaluateABAC(cached, ctx)

const snapshot = await buildABACSnapshot(userId, role)
await cache.set(key, snapshot, { ex: 3600 })
return evaluateABAC(snapshot, ctx)
```

Quando permissões mudam → hash muda → cache miss automático. Ver também [[op-005-cache-sem-versionamento]].

### Estratégia B — re-validar contra fonte de verdade

```ts
// Mantém cache para dados não-sensíveis, mas sempre busca role fresca
const cached = await cache.get(userKey)
const freshRole = await roleRepo.findByUserId(userId)   // query barata, indexada
const snapshot = { ...cached, role: freshRole, permissions: freshRole.permissions }
return evaluateABAC(snapshot, ctx)
```

Custo: 1 query indexada por request protegido — aceitável em muitos sistemas.

## Alternativa

- Invalidação ativa via evento `RoleChanged` → worker `DEL abac:{userId}:*`.
- Cache TTL **muito curto** (30s-60s) quando não dá pra versionar — reduz brecha mas não elimina.

## Quando tolerar

Nunca em sistema com compliance sensível (financeiro, saúde, governamental). Em sistema interno com baixo risco e documentação explícita do trade-off, TTL curto + monitoramento pode ser aceito.

## Relações

- **Patterns**: [[../patterns/cqrs]], [[../patterns/observer]] (publish invalidation)
- **OPs relacionados**: [[op-005-cache-sem-versionamento]] (raiz geral), [[op-021-admin-sem-audit]], [[op-020-erro-engolido]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-008
- [[../security/security-principles]] — ABAC engine
- [[../security/beyond-corp]] — zero-trust implica re-validação

## Links

- [[_moc]]
- [[../security/security-principles]]
- [[op-005-cache-sem-versionamento]]
- [[../security/beyond-corp]]
