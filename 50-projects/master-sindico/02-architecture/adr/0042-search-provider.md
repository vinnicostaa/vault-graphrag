---
adr: 0042
title: ISearchProvider real desde M1 (revoga D-067) — OpenSearch vs MeliSearch a fechar
status: proposed (pending dual-check)
date: 2026-04-24
deciders: ["@vinni"]
consulted: []
informed: []
supersedes: []
superseded-by: []
amends: ["0018-opensearch-tsvector-dev"]
tags: [adr, master-sindico, v2, search, opensearch, meilisearch, fase-14]
---

# ADR-0042 — ISearchProvider real desde M1 (revoga D-067 PG tsvector dev)

## Status

**proposed (pending dual-check)** — 2026-04-24

Não pode ser promovida a `accepted` sem dual-check em contexto limpo (STEERING §32 — *toda decisão arquitetural com impacto cross-BC e provider externo passa por 2º agente*). Itens a fechar no dual-check:

1. **OpenSearch vs MeliSearch** — escolha do adapter inicial.
2. **Confirmação de revogação parcial de D-067** — D-067 abolia OpenSearch em dev usando Postgres tsvector como mock; esta ADR torna search uma capability de M1 real (não mockada) em todos os ambientes.
3. **Hosting/região BR** — AWS OpenSearch Service `sa-east-1` vs MeliSearch Cloud (US/EU) vs self-hosted Railway side-car.

## Contexto

### Por que search é vital, não acessório

Master Síndico tem 5 superfícies de busca **first-class**, não secundárias:

| Superfície | Volume estimado M1 | Latência esperada | Features mínimas |
|---|---|---|---|
| **Vizinhança** (comércio local + cupons) | 5k-20k lojas | < 200ms p95 | full-text + geo (raio km) + facets categoria |
| **Connect Me** (catálogo de empresas/prestadores) | 100-2k empresas | < 200ms p95 | full-text + facets categoria/região + ranking determinístico ([[0023-recsys-deterministic-m1]]) |
| **Banco de Talentos** (currículos/vagas) | 1k-10k currículos | < 300ms p95 | full-text + facets área/cidade |
| **Biblioteca de atas + timeline** (governança) | 500-5k docs por condomínio | < 500ms p95 | full-text PT-BR com stemming + filtro por período |
| **Prestadores avaliados** (reputação Bronze→Diamante) | overlap com Connect Me | < 200ms p95 | filter por tier + ranking |

### Por que D-067 (PG tsvector como mock dev) precisa ser revogado

[[0018-opensearch-tsvector-dev]] (herdada do legado) define um modelo `PgSearchProvider` (dev) + `OpenSearchProvider` (prod), trocados via `SEARCH_BACKEND=pg|opensearch`. **Problemas**:

1. **Dev/prod parity quebrada**. Stemming PT-BR diverge entre PG `to_tsvector('portuguese', ...)` e OpenSearch analyzer `brazilian` — features que passam em dev podem falhar em staging/prod (ranking, fuzzy match, sinônimos).
2. **Implementação dupla cara**. Manter dois adapters drena tempo (caso extremo: facet aggregations só funcionam num lado; cliente reporta bug em prod que dev não reproduz).
3. **Search não é mock-able**. Diferente de email (Resend → Noop), pagamento (Stripe → mock), search **define a UX**. Vizinhança sem busca usável é Vizinhança quebrada — não há fallback aceitável "feature flag off".
4. **D-067 surgiu pré-Fase 8**. Reanálise de research (LinkedIn, Connect Me unidirecional, ranking determinístico — [[../research-inspirations]]) mostrou que **retrieval ≠ ranking** e que retrieval engine maduro (BM25, ngram, fuzzy) é alavanca direta para conversão e satisfação do morador/síndico.
5. **Vizinhança e Banco de Talentos pedem geo + facets nativos** — `tsvector` não tem geo; precisaria PostGIS + lógica composta no app, dobrando complexidade.

### Restrições

- **M1 (2026-05-18)** — search precisa estar em produção real, com índice populado e UI integrada.
- **Origin Railway** ([[0017-railway-initial-aws-m4]]) + **edge Cloudflare** ([[0040-cloudflare-edge-primary-railway-origin]]) — search engine pode rodar como side-car Railway, AWS managed (`sa-east-1`), ou Meili Cloud.
- **Dev local** — engine deve subir em `docker-compose` < 30s sem RAM > 1GB.
- **Multi-tenant** — todo doc indexado tem `tenant_id` (CondominiumID/CompanyID/NeighborhoodID — INV-108) e queries filtram por tenant antes de relevância.
- **LGPD** — DPA do provider precisa cobrir BR; index não pode conter PII bruta (ver [[0028-lgpd-keyed-hmac]] — pseudonimização para campos sensíveis).

## Decisão

**Adotar `ISearchProvider` (port em `application/ports/search.go`) como contrato canônico de busca, com adapter REAL desde M1 em TODOS os ambientes (dev, staging, prod). Revogar a parte de D-067 / ADR-0018 que usa Postgres tsvector como adapter dev.**

### Contrato `ISearchProvider`

```go
type ISearchProvider interface {
    Index(ctx context.Context, index string, docs []Document) error
    Search(ctx context.Context, q SearchQuery) (SearchResult, error)
    Delete(ctx context.Context, index string, ids []string) error
    Reindex(ctx context.Context, index string) error
}

type SearchQuery struct {
    Index       string
    TenantID    types.TenantID  // INV-108: VO selado
    QueryText   string
    Filters     map[string]any
    Facets      []string
    GeoRadius   *GeoFilter      // opcional (Vizinhança)
    Pagination  Page
    SortBy      []SortField
}
```

### Índices canônicos M1

- `idx-vizinhanca` (lojas + cupons + benefícios)
- `idx-connect-me` (empresas/prestadores)
- `idx-talentos` (currículos)
- `idx-timeline` (atas + entries de governança)

### Atualização: outbox → indexer worker

Eventos `*.Created`, `*.Updated`, `*.Deleted` em cada BC publicam para outbox ([[event-driven]] §3) → worker `search-indexer` aplica upsert no índice. **Reindex full diário** às 03:00 BRT como safety net.

### Adapter inicial: **a decidir no dual-check**

Esta ADR **não fecha** a escolha entre OpenSearch e MeliSearch. Fecha apenas: (a) capability M1 real, (b) contrato `ISearchProvider`, (c) revogação do mock PG tsvector. O dual-check decide o adapter (ver Alternatives).

## Consequências

### Positivas

- **Dev/prod parity** total — stemming, ranking, facets idênticos.
- **Capability M1** — Vizinhança, Connect Me, Banco de Talentos, biblioteca de atas funcionam com search robusto desde o lançamento.
- **Adapter trocável** — se OpenSearch ficar caro em M3+, troca-se para Meili (ou vice-versa) com refactor trivial via porta isolada.
- **Geo + facets nativos** sem precisar PostGIS no app.
- **Outbox → indexer** integra com pattern existente ([[event-driven]]) sem novo paradigma.
- **Origin agnóstico** — search engine vive ao lado do origin (Railway hoje, ECS amanhã — [[0017-railway-initial-aws-m4]]).

### Negativas / trade-offs

- **+1 dependência runtime** em dev (containers `docker-compose`) e em prod (managed service ou self-host).
- **+1 DPA** (LGPD) com provider escolhido — bloqueio pré-M1.
- **Custo M1**: $30-200/mês conforme adapter (OpenSearch managed BR `sa-east-1` ~$80-150 t3.small.search; Meili Cloud `Build` $30-50; self-host Railway side-car ~$10-30 RAM-bound).
- **Reindex full** consome I/O — agendar fora de pico, monitorar.
- **Relevância tuning** vira disciplina nova — antes era só `ts_rank`; agora há analyzers, synonym filters, BM25 params.

### Risks

- **R-SEARCH-001**: Se MeliSearch escolhido e volume crescer > 10M docs em M3+, migração para OpenSearch terá custo (estimar refactor de queries + reindex). **Mitigação**: porta `ISearchProvider` isola; reindex via outbox replay.
- **R-SEARCH-002**: PT-BR stemming acurácia varia entre engines — testar com dataset real (atas + timeline) antes de fechar adapter no dual-check.
- **R-SEARCH-003**: Outbox lag em pico → search "stale" por X segundos. SLO indexer p95 < 30s. Alerta se > 5min.
- **R-SEARCH-004**: Revogar D-067 tem **impacto retroativo** em [[../../03-subprojects/backend/requirements/content]] (CNT-008+ menciona PG tsvector) — task em [[../../05-tasks/_moc]] para reescrever.

## Alternatives considered

### Opção A — **OpenSearch managed AWS `sa-east-1`** ⭐ (mais conservador)

**Pros**:
- Maduro, stack Apache Lucene; ecossistema gigante; Kibana/dashboards inclusos.
- Hosting BR (`sa-east-1`) — latência + LGPD residency.
- Suporta volumes 100M+ docs sem refactor.
- Stack-fit com [[0018-opensearch-tsvector-dev]] que já mencionava OpenSearch para prod.
- Brazilian analyzer nativo + plugins (icu, phonetic).

**Cons**:
- Custo base mais alto (~$80-150/mês t3.small.search single-node M1).
- Learning curve maior (analyzers, ILM policies, shard sizing).
- Time backend Go-first; cluster ops requer atenção dedicada.
- Container dev mais pesado (~600-800MB RAM).

### Opção B — **MeliSearch (Cloud Build $30-50/mês ou self-host Railway)** ⭐ (mais leve)

**Pros**:
- Setup trivial — 1 binário Rust, < 100MB RAM dev, < 30s subir.
- DX excelente — REST/typed clients, schema-less com auto-attribute detection.
- "Instant search" UX (typo-tolerance built-in, < 50ms).
- Custo mínimo M1.

**Cons**:
- Menos maduro em aggregations/facets complexos vs OpenSearch.
- Escala bem até ~10-50M docs/index — projeção M3+ pode pressionar.
- Cloud hosting é US/EU (Meili Cloud sem BR-region em 2026-04) — **risco LGPD residency** e latência (+50-150ms BR↔US).
- Self-host em Railway resolve LGPD mas exige ops (backup, snapshot, upgrade).

### Opção C — Manter ADR-0018 (PG tsvector dev / OpenSearch prod) — **rejeitada**

Motivos no Contexto: dev/prod parity, custo manutenção dupla, search não é mock-able, geo/facets ausentes.

### Opção D — **Typesense** — viável, não escolhida nesta ADR

Similar a Meili (Go-friendly, leve, fast). Comunidade menor que Meili; menos referências BR. Pode entrar como adapter futuro se Meili decepcionar.

### Opção E — **ElasticSearch** — rejeitada

Elastic License 2.0 (não Apache 2.0 — restrições comerciais). OpenSearch é fork Apache 2.0 — dominante no segmento "drop-in ES" sem amarras.

### Opção F — **Algolia** — rejeitada

SaaS premium; custo $0.50/1k searches escala caro; vendor lock alto; LGPD residency US.

## Implementation notes

- **Porta**: `internal/application/ports/search.go` — `ISearchProvider` interface.
- **Adapters**: `internal/infrastructure/search/opensearch/` ou `internal/infrastructure/search/meilisearch/` (decisão dual-check).
- **DI**: `cmd/server/main.go` resolve por `SEARCH_PROVIDER=opensearch|meilisearch` (sem `pg` na lista).
- **Indexer worker**: `cmd/search-indexer/main.go` — consome outbox, processa batched (500 docs/op).
- **Migrations**: nenhuma DB-side (índices são schema do search engine, versionados em `internal/infrastructure/search/<provider>/schema.go`).
- **Observability**: métricas `search.query.latency_ms` (p50/p95/p99 por índice + tenant), `search.index.lag_seconds`, `search.errors_total` ([[observability]]).
- **Multi-tenant**: filter `tenant_id` aplicado SEMPRE (middleware envolve `Search` call); cross-tenant test em CI ([[../../09-testing/_moc]]).
- **Refs C4**: [[c4-components]] §3.4 commercial — adapter substitui PG tsvector + integra com retrieval≠ranking pattern.

## References

- D-067 (STATE.md) — abolição OpenSearch em dev a ser **revogada** parcialmente por esta ADR (busca volta a ser real, mas adapter dev/prod = mesmo provider).
- D-066, D-066/D-067 — planos globais Stripe-style ([[0032-global-plans-stripe-style]]) — não conflita.
- D-117 — Cloudflare edge ([[0040-cloudflare-edge-primary-railway-origin]]) — search engine fica origin-side, não edge.
- INV-049, INV-060, INV-061, INV-089 — tenant isolation aplicada também a search.
- INV-108 — `TenantID` selado (compile-time safety nas queries).
- [[0018-opensearch-tsvector-dev]] — **amends** (parte dev tsvector revogada; prod OpenSearch path mantido como Opção A).
- [[0023-recsys-deterministic-m1]] — ranking determinístico; retrieval (esta ADR) ≠ ranking.
- [[0019-nats-jetstream-threshold]] — outbox → indexer pode usar NATS quando ativado.
- [[../research-inspirations]] §retrieval-ranking-split (LinkedIn destilado).
- [[../../00-product/sub-produtos/08-vizinhanca]] — search é UX core.
- [[../../00-product/sub-produtos/07-banco-de-talentos]] — search é UX core.
- [[../../03-subprojects/backend/requirements/content]] CNT-008 — atualizar pós-dual-check.
- OpenSearch BR analyzer: https://opensearch.org/docs/latest/analyzers/language-analyzers/brazilian/
- MeliSearch PT-BR: https://www.meilisearch.com/docs/learn/configuration/typo
- LGPD residency guidance ANPD: https://www.gov.br/anpd/pt-br
