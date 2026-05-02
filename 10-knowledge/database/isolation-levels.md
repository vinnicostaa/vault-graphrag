---
title: Isolation Levels
type: concept
tags: [database, transactions, isolation, concurrency]
created: 2026-04-24
updated: 2026-04-24
aliases: [ANSI isolation levels, transaction isolation, SSI]
---

# Isolation Levels

Conceito: **quanto uma transação enxerga do estado intermediário de outras transações concorrentes**. Trade-off central: mais isolamento = menos anomalia + menos concorrência + mais retry.

## Os 4 níveis ANSI

Definidos pelo SQL standard em função das **anomalias** que previnem:

| Nível | Dirty Read | Non-repeatable Read | Phantom | Write Skew |
|---|---|---|---|---|
| Read Uncommitted | permite | permite | permite | permite |
| Read Committed | bloqueia | permite | permite | permite |
| Repeatable Read | bloqueia | bloqueia | permite¹ | permite |
| Serializable | bloqueia | bloqueia | bloqueia | bloqueia |

¹ Postgres Repeatable Read (via MVCC snapshot) **também bloqueia phantom**, divergindo do standard.

### Anomalias explicadas

- **Dirty read** — T1 lê linha que T2 escreveu mas ainda não commitou; se T2 rollback, T1 leu dado que nunca existiu
- **Non-repeatable read** — T1 lê linha X=10; T2 commita X=20; T1 relê X=20. Mesma query, resultados diferentes dentro da mesma transação
- **Phantom read** — T1 faz `SELECT WHERE status='open'` retorna 3 linhas; T2 insere uma 4ª; T1 repete → 4 linhas
- **Write skew** — duas transações leem o mesmo estado, tomam decisões baseadas nele, e escrevem valores que **juntos** violam invariante. Clássico: dois médicos on-call, cada um se desmarca acreditando que o outro está. Ambas commitam. Zero médico.

## Postgres default: Read Committed

Postgres default é **Read Committed** com MVCC. Cada statement vê um snapshot do momento em que **ele** começou (não o da transação). Isso é suficiente pra maioria dos casos quando combinado com:

- **SELECT FOR UPDATE** — lock pessimista na linha lida; próxima transação que tentar ler com FOR UPDATE espera
- **SELECT FOR NO KEY UPDATE** — mais permissivo, não bloqueia FK check
- **Advisory locks** — lock nomeado pela aplicação, fora das linhas

Regra prática: **prefira lock explícito a subir isolation**. Lock é cirúrgico (escopo conhecido); isolation é global (afeta toda a transação).

```sql
BEGIN;
SELECT balance FROM accounts WHERE id = $1 FOR UPDATE;
-- outras transações bloqueiam aqui até commit/rollback
UPDATE accounts SET balance = balance - $2 WHERE id = $1;
COMMIT;
```

## Serializable (SSI) em Postgres

**Serializable Snapshot Isolation** (SSI) dá garantia de que a execução concorrente produz o **mesmo resultado** de alguma ordem serial, sem precisar de locks pessimistas em tudo.

Postgres implementa via predicate locking + detecção de dependency cycle. Quando cycle é detectado, uma das transações recebe `could not serialize access due to read/write dependencies` (SQLSTATE `40001`) e **precisa retry**.

**Custo**: mais memória (rastreio de predicate locks), mais CPU em commit time, **retry obrigatório** no client.

**Quando usar**: invariantes multi-linha difíceis de expressar com FOR UPDATE. Casos canônicos:

- Transferência de dinheiro entre N contas com invariante de total
- Booking de recurso escasso (quota concorrente, assentos, slots)
- Balanceamento de quota compartilhada
- Qualquer cenário de write skew que seria complexo resolver com lock

Cliente **precisa** implementar retry:

```go
for attempt := 0; attempt < 3; attempt++ {
    err := runTx(ctx, sql.LevelSerializable, func(tx *sql.Tx) error { ... })
    if err == nil { return nil }
    if !isSerializationError(err) { return err }
    time.Sleep(backoff(attempt))
}
```

## MVCC vs locking

- **MVCC** (Postgres, Oracle) — cada escrita cria nova versão; leitura usa snapshot. Readers não bloqueiam writers e vice-versa. Custo: VACUUM (limpar versões mortas), bloat
- **Locking** (MySQL MyISAM historicamente, SQL Server default) — reads pegam shared lock, writes pegam exclusive. Readers bloqueiam writers

Postgres e MySQL InnoDB modernos são MVCC. SQL Server oferece MVCC opt-in (`READ_COMMITTED_SNAPSHOT`).

## Comparação entre DBs

| DB | Default | Níveis suportados | Observação |
|---|---|---|---|
| PostgreSQL | Read Committed | RC, RR, SER | RR não tem phantom (forte); SER via SSI |
| MySQL InnoDB | Repeatable Read | RU, RC, RR, SER | RR usa gap locks — phantom bloqueado via lock |
| Oracle | Read Committed | RC, SER | Sem RU nem RR; "Serializable" é snapshot |
| SQL Server | Read Committed | RU, RC, RR, SER, Snapshot | Snapshot isolation separado |

## Regra de escolha

1. Comece em **Read Committed** (default)
2. Precisa de consistência em read múltiplo dentro da tx? `SELECT FOR UPDATE` nas linhas relevantes
3. Invariante multi-linha difícil? **Serializable + retry**
4. Nunca suba pra SER em tudo "por segurança" — impacto de throughput é real

## Links

- [[_moc]]
- [[../_moc]]
- [[database-patterns]]
- [[../patterns/transaction-patterns]]
