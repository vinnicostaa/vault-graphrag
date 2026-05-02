# Skill: sqlc Query Writer

**Referência obrigatória**: `.kiro/steering/database.md` — overrides em `sqlc.yaml`, pgx/v5, padrões de query.

Quando solicitado a escrever queries sqlc, siga este protocolo:

## Localização das queries

```
internal/modules/{bounded_context}/infrastructure/database/queries/
  {entidade}.sql
```

Exemplo: `internal/modules/identity/infrastructure/database/queries/user.sql`

## Formato obrigatório

```sql
-- name: NomeDaQuery :one|:many|:exec|:execrows

SELECT
    coluna1,
    coluna2,
    -- liste colunas explicitamente — nunca SELECT *
FROM tabela
WHERE condicao = $1
    AND deleted_at IS NULL;  -- sempre filtrar soft-deleted
```

## Tipos de retorno sqlc

| Tipo | Quando usar |
|------|-------------|
| `:one` | Retorna uma linha (usa `pgx.ErrNoRows` se não encontrar) |
| `:many` | Retorna múltiplas linhas |
| `:exec` | Não retorna linhas (INSERT/UPDATE/DELETE) |
| `:execrows` | Retorna número de linhas afetadas |

## Regras

- Sempre listar colunas explicitamente — nunca `SELECT *`
- Sempre filtrar `deleted_at IS NULL` em tabelas com soft delete
- Sempre filtrar `expires_at > NOW()` em sessões
- Parâmetros posicionais: `$1`, `$2`, etc. — nunca concatenação de string
- Tenant isolation: toda query de dados de condomínio inclui `WHERE condominium_id = $N`

## Após modificar queries

```bash
sqlc generate
go build ./...
```

Se `sqlc generate` falhar:
1. Verifique o nome da query (formato `-- name: NomeDaQuery :tipo`)
2. Verifique se os tipos dos parâmetros batem com o schema
3. Verifique os overrides em `sqlc.yaml`

## Mapeamento de tipos (sqlc.yaml overrides)

Os overrides já configurados em `sqlc.yaml`:
- Enums PostgreSQL → `string` (não o tipo gerado)
- UUID → `string`
- Campos nullable → `*string`, `*time.Time`, etc.

## Implementação no repository

Após gerar o código sqlc, implemente no repository:

```go
func (r *XRepository) FindByID(ctx context.Context, id string) (*entities.X, error) {
    q := identitydb.New(database.DBFromContext(ctx, r.pool))

    row, err := q.GetXByID(ctx, id)
    if err != nil {
        if errors.Is(err, pgx.ErrNoRows) {
            return nil, apperrors.ErrNotFound.WithMessage("x not found")
        }
        return nil, apperrors.NewInternal(err)
    }

    return mapRowToX(row), nil
}
```

O `mapRowToX` converte o tipo gerado pelo sqlc para a entidade de domínio.
O domínio nunca conhece `pgtype` ou tipos gerados pelo sqlc.
