---
title: source-03 — Search Federation Architecture at LinkedIn
type: source
tags: [research, linkedin, search, galene, opensearch, federation, ranking]
source: https://engineering.linkedin.com/blog/2018/03/search-federation-architecture-at-linkedin
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-search-federation, galene]
---

# source-03 — Search Federation Architecture at LinkedIn

**URL (atual, pós-redirect)**: https://www.linkedin.com/blog/engineering/search/search-federation-architecture-at-linkedin
**Data original**: 2018-03
**Fonte primária**: sim (LinkedIn Engineering)
**Complementares**: [Galene](https://engineering.linkedin.com/search/did-you-mean-galene), [InSearch](https://www.linkedin.com/blog/engineering/search/introducing-insearch)

## Por que essa fonte importa

Referência prática de arquitetura de busca em camadas — federator, mid-tier ranker, backend indexes — aplicável ao nosso OpenSearch. Contém lições de **migração** e **rollout incremental** generalizáveis.

## Pontos extraídos

### Componentes
- **Galene** = nome da arquitetura de busca; Lucene como camada de indexação mantida.
- Histórico de serviços consolidados: `federated-search-rest` (2012), `typeahead-rest` (2014), `seas-federated-search` (2014).
- Eventualmente tudo unificado sob SeaS workflow framework — "composição de workflow sem if-else clauses", isolamento entre verticais.

### Camadas
1. **Frontend clients + API** — routing request, stitching responses.
2. **Mid-tier federator** — scatter-gather dos verticais (People, Jobs, Content, Groups…), ML ranking pipeline combinando resultados.
3. **Backend indexes** — online + offline Lucene indexes; ML pipeline separado pra maximizar successful searches.

### Query understanding
- Spell check, synonym expansion, intent classification **antes** da query bater no índice.
- "Did you mean 'Galene'?" (artigo de spell correction dedicado).

### Pattern de federação
- Query única → fan-out pra N verticais (scatter) → merge ranked (gather) → apresenta blend.
- Cada vertical tem próprio schema, próprio ranker.

### Migração & rollout
- **Parity check framework** — comparar legado vs novo **antes** de A/B test.
- Rollout incremental: 5% → monitora GCNs (garbage collection notices), estabilidade → aumenta.
- Evita big-bang.

## Aplicabilidade master-sindico

Detalhada em [[_destilado#3-busca-voyagergalene--opensearch-já-é-stack-lições-aplicam]].

- **Separar document fetching de ranking**: OpenSearch retorna candidatos (BM25 + filters), re-rank determinístico em aplicação aplica boosts (reputação, proximidade, skill match). Pattern direto do LinkedIn — não tentar fazer tudo na query DSL.
- **Query understanding barato**: sinônimos curados ("encanador" ↔ "hidráulico") em index-time analyzer do OpenSearch. Não precisa ML classifier.
- **Federator é M2+**: M1 um multi-index simples basta (empresas + talentos). Quando houver 3+ verticais (empresas, talentos, assembleias, posts), um gateway de busca unificado emerge.
- **Parity check é obrigação** em qualquer mudança de ranking/indexação — compara scores antigos vs novos em dataset gravado, detecta regressão antes de release.

## Lições explícitas do texto

1. **Centralizar federation logic cedo** — evita duplicação entre search e typeahead.
2. **Workflow composition > conditional branching** — escala com verticais.
3. **Parity check antes de A/B** — detecta regressão sem expor usuário.
4. **Rollout staged** — 5% → ramp, monitora GCs.
5. **Ranking separado do fetching** — permite variar latência e heurística independentemente.

## Quando releer

- ADR de ranking do Connect Me search.
- Quando adicionar 3º índice distinto (após empresas + talentos).
- Quando surgir discussão de "busca personalizada" (perfil do buscador afeta ranking).

## Vizinhos

- [[_destilado]]
- [[source-08]] — Skill Assessments alimenta Galene via Kafka (integração índice)
- [[../../02-architecture/patterns]] — pattern "fetch vs rank separation"
