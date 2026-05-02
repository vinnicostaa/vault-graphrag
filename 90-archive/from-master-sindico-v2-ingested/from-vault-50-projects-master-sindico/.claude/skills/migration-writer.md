# Skill: Migration Writer (goose v3)

**Referência obrigatória**: `.kiro/steering/database.md` — padrões completos de schema, pgx, sqlc, goose.

Quando solicitado a criar uma migration, siga este protocolo:

## Convenção de numeração por bounded context

| Bounded Context | Faixa de números |
|----------------|-----------------|
| identity       | 00001 – 00099   |
| billing        | 00100 – 00199   |
| institutional  | 00200 – 00299   |
| commercial     | 00300 – 00399   |
| content        | 00400 – 00499   |
| assembly       | 00500 – 00599   |

## Localização
Todas as migrations ficam em:
`internal/shared/providers/database/migrations/`

## Formato obrigatório

```sql
-- +goose Up
-- Descrição do que esta migration faz e POR QUÊ

CREATE TABLE nome_da_tabela (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    -- campos...
    created_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE NOT NULL DEFAULT NOW()
    -- deleted_at apenas se a tabela suporta soft delete
    -- NÃO adicionar deleted_at em: timeline_entries, audit_trail, assembly_votes
);

-- Índices parciais excluem soft-deleted para queries do dia a dia
CREATE INDEX idx_nome ON tabela(coluna) WHERE deleted_at IS NULL;

-- +goose Down
DROP TABLE nome_da_tabela;
```

## Regras

- PKs: UUID com `DEFAULT gen_random_uuid()`
- Nomenclatura: `snake_case`
- Valores monetários: `INTEGER` (centavos) — nunca `FLOAT` ou `DECIMAL` para dinheiro
- Fração ideal: `NUMERIC(10,6)` — nunca `FLOAT`
- Enums PostgreSQL: definir em migration separada (00001_create_enums.sql já existe)
- Nunca destrutivas em produção: sem `DROP TABLE`, sem `DROP COLUMN`, sem `ALTER COLUMN` que perde dados
- Sempre incluir `-- +goose Down` mesmo que seja apenas `SELECT 1;` (para migrations irreversíveis)

## Tabelas imutáveis (sem deleted_at, sem UPDATE)
- `timeline_entries`
- `audit_trail`
- `assembly_votes`

## Após criar a migration

Verifique que o embed.FS compila:
```bash
go build ./...
```

Se falhar com erro de embed, verifique o nome do arquivo e o path.
