[FFV Academy](../index.html)/SQL & Databases

🗃️

Blog

# SQL & Databases

Para quem já sabe o básico e quer ir fundo. Aqui o assunto é como os modelos funcionam em produção: memória, roteamento, ferramentas, agentes. O lado técnico que pouca gente explica direito.

10artigos

680XP total

📄PDF da trilha

🖥️Apresentar trilha

[](../aprenda/relacional-vs-nao-relacional/index.html)

01

## ⚖️ Relacional vs NoSQL: quando cada um ganha

PostgreSQL vs MongoDB vs Redis vs DynamoDB: modelos de dados, trade-offs de consistência e quando a escolha importa de verdade.

⏱ 10 min·+50 XP

→[](../aprenda/select-join-na-pratica/index.html)

02

## 🔗 SELECT e JOIN na prática: INNER, LEFT, self-join

Os JOINs que resolvem 90% dos problemas reais: INNER, LEFT/RIGHT, FULL OUTER, self-join para hierarquias, CTEs para legibilidade.

⏱ 13 min·+65 XP

→[](../aprenda/group-by-agregacoes/index.html)

03

## 📊 GROUP BY, HAVING e agregações que resolvem 80% dos casos

COUNT, SUM, AVG, GROUP BY, HAVING, ROLLUP — as funções de agregação que transformam dados brutos em métricas de negócio.

⏱ 11 min·+55 XP

→[](../aprenda/window-functions/index.html)

04

## 🪟 Window functions: ranking, running totals, lead/lag

ROW_NUMBER, RANK, DENSE_RANK, LAG, LEAD, FIRST_VALUE, SUM OVER — as funções de janela que eliminam subqueries e self-joins complicados.

⏱ 15 min·+75 XP

→[](../aprenda/indices-que-funcionam/index.html)

05

## 📇 Índices que funcionam: B-tree, hash, GIN, covering, composto

Quando um índice é usado (e quando não é). B-tree vs hash vs GIN, índice composto e a ordem importa, covering index para evitar heap scan.

⏱ 16 min·+80 XP

→[](../aprenda/explain-analyze/index.html)

06

## 🔬 EXPLAIN ANALYZE: lendo o plano e otimizando query

Seq Scan vs Index Scan vs Bitmap Scan, Nested Loop vs Hash Join vs Merge Join — como ler o plano de execução e transformar queries lentas em rápidas.

⏱ 17 min·+85 XP

→[](../aprenda/transacoes-isolation-levels/index.html)

07

## 🔒 Transações e isolation levels: ACID sem decoreba

ACID não é decoreba — é o que evita dirty reads, non-repeatable reads e phantom reads. Read Committed, Repeatable Read, Serializable: quando usar cada um.

⏱ 15 min·+75 XP

→[](../aprenda/normalizacao-modelagem/index.html)

08

## 📐 Modelagem e normalização: 1NF–3NF + quando desnormalizar

Primeira, segunda e terceira formas normais com exemplos reais. Quando desnormalizar intencionalmente (analytics, performance). ERD na prática.

⏱ 12 min·+60 XP

→[](../aprenda/migrations-profissionais/index.html)

09

## 🔄 Migrations profissionais: reversíveis, zero-downtime

Alembic, Flyway e Liquibase. Migrations reversíveis com up/down. Zero-downtime em produção: adicionar coluna nullable, backfill, NOT NULL depois.

⏱ 13 min·+65 XP

→[](../aprenda/connection-pool-n-plus-1/index.html)

10

## ⚠️ Connection pool, N+1 e o que mata sua API

Por que 100 queries são piores que 1 query com JOIN. N+1 query problem e como detectar. PgBouncer, SQLAlchemy pool, asyncpg — configuração correta.

⏱ 14 min·+70 XP

→

[← Voltar à home](../index.html)
