---
title: OP-002 — Check-then-act não atômico
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - concurrency
id: OP-002
severity: High
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - TOCTOU
  - Check-then-act
doc-consulted: '2026-04-24'
---

# OP-002 — Check-then-act não atômico

**Severidade**: High
**Categoria**: Concorrência

## Sintoma

`if (not exists) insert` fora de transação única. Entre o check e o insert, outro request insere. O segundo também executa o insert — viola invariante de unicidade (duplica registro, cria estado inconsistente).

Também conhecido como **TOCTOU** (Time-Of-Check to Time-Of-Use).

## Por que é problema

- Invariantes quebradas: dois usuários com mesmo email/CPF, dois convites ativos pra mesma pessoa.
- Dificíl de cobrir em teste unitário — precisa injeção temporal.
- Quando a invariante é de negócio (unicidade de slug), o bug só aparece sob concorrência real.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO
const exists = await userRepo.findByEmail(email)
if (exists) throw new ConflictError('email já cadastrado')
await userRepo.create({ email, name })
```

## Fix preferido — constraint `UNIQUE` + tratar violação

```sql
-- Schema
CREATE UNIQUE INDEX idx_users_email ON users(email) WHERE deleted_at IS NULL;
```

```ts
try {
  await userRepo.create({ email, name })
} catch (err) {
  if (isUniqueViolation(err)) throw new ConflictError('email já cadastrado')
  throw err
}
```

## Alternativa

- `INSERT ... WHERE NOT EXISTS` (Postgres).
- `INSERT ... ON CONFLICT DO NOTHING RETURNING id` — linha vazia no retorno = já existia.
- `MERGE` em bancos que suportam.

## Quando tolerar

Nunca de graça. Se a aplicação **precisa** do check antes (ex: exibir mensagem diferente pra usuário banido vs novo), fazer check **dentro da mesma transação** com `SELECT FOR UPDATE` ou isolamento `SERIALIZABLE` + retry.

## Relações

- **Patterns que resolvem**: [[../patterns/transaction-patterns]] (UoW + isolation), [[../database/isolation-levels]] (SERIALIZABLE)
- **OPs relacionados**: [[op-001-race-condition-contador]] (mesma raiz), [[op-018-state-machine-sem-guards]] (invariante cruza transição)

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-002
- OWASP TOCTOU — [link](https://owasp.org/www-community/vulnerabilities/Time_of_check_time_of_use)
- PostgreSQL — [ON CONFLICT clause](https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT)

## Links

- [[_moc]]
- [[../antipatterns/_moc]]
- [[../database/isolation-levels]]
- [[../patterns/transaction-patterns]]
