---
title: GraphQL — A Data Query Language
type: source
tags: [research, meta, graphql, api, rest, mobile, source]
source: https://engineering.fb.com/2015/09/14/core-infra/graphql-a-data-query-language/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-06-graphql]
---

# source-06 — GraphQL: A Data Query Language

- **URL**: https://engineering.fb.com/2015/09/14/core-infra/graphql-a-data-query-language/
- **Publicado**: 2015-09-14 (open-source announcement)
- **Autor/Canal**: Engineering at Meta — Lee Byron
- **Tipo**: post de lançamento / design
- **Origem interna**: GraphQL nasceu no Facebook em 2012. Criadores: Dan Schafer, Lee Byron, Nick Schrock.

## Sinopse

Lançamento oficial do GraphQL como padrão aberto. Contextualiza que foi criado em 2012 pra suportar News Feed nativo mobile, resolvendo problemas de over/underfetching e multiplicidade de endpoints REST.

## Pontos-chave

1. **Origem**: 2012, esforço de reescrever Facebook mobile nativo (antes eram wrappers da web). News Feed precisou de API de dados, não HTML.
2. **Problema REST**: over-fetching (trazer demais), under-fetching (N round-trips), versioning.
3. **Design principles**:
   - Query shape = response shape.
   - Hierárquico (objetos relacionados em uma query).
   - Strong typing + schema.
   - Introspection (cliente descobre schema).
   - Version-free (cliente escolhe fields, backward-compat natural).
4. **GraphQL não dita storage** — é protocolo sobre qualquer backend.

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §8.

**Decisão**: **NÃO adotar GraphQL no M1/M2.** Stack canônica é REST + OpenAPI 3.1 (regra §11 do CLAUDE.md).

**Motivos**:
- Volume de clientes no M1 é pequeno (web admin + mobile síndico). Não há heterogeneidade que justifique GraphQL.
- Cache HTTP (CDN, browser) funciona nativamente com REST GET; com GraphQL (POST único) requer layer custom.
- Segurança: N+1, query complexity limiting, depth limiting — toda uma disciplina extra.
- OpenAPI 3.1 já atende contrato-first + code-gen.

**Observação importante**: Instagram (produto) em si usa **Django + REST internamente**, não GraphQL. GraphQL é mais usado em Facebook/Meta apps. Isso reforça que REST escala com Python e com múltiplos frontends nativos.

**Quando reconsiderar**: M3+ com 3+ clientes heterogêneos (web admin, síndico mobile, morador mobile, API parceiros, widget embed). Nesse caso avaliar BFF com GraphQL pontual, não troca total.

## Citações

> "We developed GraphQL three years ago to fill this need." (2015 → origem 2012)

> "GraphQL queries mirror their response."

## Confiança / datas

- Data blog confirmada via fetch (2015-09-14) — **alta confiança**.
- Origem em 2012 bem documentada (múltiplas fontes: Postman blog, entrevistas Lee Byron). **Alta confiança**.
- Fonte primária Meta Engineering.
