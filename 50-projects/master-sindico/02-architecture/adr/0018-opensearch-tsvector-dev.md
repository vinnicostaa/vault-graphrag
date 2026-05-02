---
title: ADR-0018 — PostgreSQL tsvector em dev, OpenSearch em prod
type: adr
tags: [adr, search, opensearch, postgres, tsvector, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0018 — PostgreSQL tsvector em dev, OpenSearch em prod

## Status

accepted — 2026-04-23 (herdado de legado)

## Contexto

Busca full-text em Connect Me (empresas por categoria/região), timeline (governança), biblioteca LMS. Volume M1 baixo (100 empresas, timeline por condomínio); M2+ escala. Dev precisa rodar tudo local sem infraestrutura pesada.

## Decisão

Interface **`ISearchProvider`** em `application/ports/` com 2 impls:

- **`PgSearchProvider`** — usa `tsvector` + `GIN index`. Usado em dev e ambientes sem OpenSearch.
- **`OpenSearchProvider`** — usa OpenSearch managed. Usado em staging/prod.

Env var `SEARCH_BACKEND=pg|opensearch` seleciona.

### PgSearchProvider

```sql
ALTER TABLE commercial.companies ADD COLUMN search_doc tsvector;
CREATE INDEX companies_search_idx ON commercial.companies USING GIN (search_doc);

-- trigger atualiza search_doc
CREATE FUNCTION update_companies_search() RETURNS trigger AS $$
BEGIN
  NEW.search_doc := to_tsvector('portuguese',
    coalesce(NEW.name, '') || ' ' ||
    coalesce(NEW.description, '') || ' ' ||
    coalesce(NEW.category, ''));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;
CREATE TRIGGER companies_search_trig BEFORE INSERT OR UPDATE ON commercial.companies
  FOR EACH ROW EXECUTE FUNCTION update_companies_search();
```

### OpenSearchProvider

- Índices: `idx-empresas`, `idx-timeline`, `idx-biblioteca` (prefixo por tenant para isolamento opcional M3+).
- Atualização via **outbox → indexer worker** (incremental upsert).
- Re-rank determinístico Go-side (separar retrieval de ranking — [[../../13-research/linkedin/_destilado]] §3).
- Full reindex diário 3h BRT.

## Consequências

### Positivas

- Dev local sem dependência externa.
- Transição dev → prod via config, zero refactor.
- pg tsvector escala bem até ~1M docs (suficiente para M1-M2).
- OpenSearch escala milhões + features avançadas (sinônimos, fuzzy, aggregations).

### Negativas

- Duas implementações para manter. Mitigação: interface mínima e enxuta; casos extremos (facets, aggs) só OpenSearch.
- Pequenas diferenças em stemming PT-BR entre pg e OpenSearch; testes documentam.

## Alternativas consideradas

1. **Meilisearch** — rápido, dev-friendly; menos maduro em aggregations e sinônimos.
2. **Typesense** — similar; também menos maduro.
3. **Elasticsearch** — Elastic License 2.0 tem restrições; OpenSearch (fork Apache 2.0) é seguro.
4. **Só pg tsvector** — limita features M2+ (facets, geo search avançada, percolators).
5. **Só OpenSearch** — lock dev em dependência pesada.

## Referências

- [[../../13-research/linkedin/_destilado]] §3
- [[../c4-components]] §3.4 commercial (retrieval ≠ ranking)
- [[../c4-containers]] §2.8
