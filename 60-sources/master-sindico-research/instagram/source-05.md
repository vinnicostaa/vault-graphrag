---
title: Akkio — Managing data store locality at scale
type: source
tags: [research, meta, akkio, data-placement, multi-region, ushards, wan, source]
source: https://engineering.fb.com/2018/10/08/core-infra/akkio/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-05-akkio]
---

# source-05 — Akkio: data placement service

- **URL**: https://engineering.fb.com/2018/10/08/core-infra/akkio/
- **Publicado**: 2018-10-08
- **Autor/Canal**: Engineering at Meta
- **Tipo**: post técnico

## Sinopse

Akkio é o serviço de placement dinâmico de dados do Facebook que move dados em granularidade fina (μshards, tipicamente 1 usuário) entre regiões geográficas conforme padrão de acesso. Permite ter 2-3 réplicas próximas ao invés de full replication em todas as regiões.

## Pontos-chave

1. **μshards** — unidade mínima de placement, contém dados com access locality (ex: 1 usuário = 1 μshard). Sistema gerencia trilhões.
2. **Multi-region strategy** — dados ficam em 2-3 regiões próximas, não em todas. Reflete observação: "a maioria das pessoas fica dentro de um grupo previsível de no máximo 2-3 regiões ao acessar o Facebook".
3. **Resultados**:
   - **−40%** storage footprint
   - **−50%** tráfego WAN cross-region
   - **~−50%** latência percebida pelo usuário
4. **Arquitetura**:
   - **Metadata tier** — mapeia μshard → placement
   - **Access DB** — histórico de 10 dias de access patterns
   - **DPS (Data Placement Service)** — decide movimentações
   - Integra com **ZippyDB** (KV store consistente da Meta)
5. **Strong consistency** suportada por μshard (apesar de eventual consistency global).

## Aplicabilidade ao Master Síndico

Detalhado em `_destilado.md` §5.

**M1-M2 (BR only)**: **Não implementar.** Uma região AWS sa-east-1. Seria over-engineering.

**Arquitetura permite regionalização futura se**:
1. **Tenant = μshard equivalente** — `tenant_id` é chave de sharding desde o D1. Isso é requisito multi-tenant de todo jeito, e casa com o padrão Akkio: 1 condomínio = μshard, move entre regiões SP/RJ/NE sem afetar outros.
2. **ID globalmente único** — ULID/UUID prefixado, não sequence regional.
3. **Outbox + eventos idempotentes** — habilita replicação assíncrona entre regiões no futuro.
4. **Evitar cross-tenant joins** — mesma razão que exigiria cross-region join no futuro.

**Quando reconsiderar**: M3+ se houver condomínios em regiões distintas com latência inaceitável na região única, ou exigência regulatória de residência de dado regional (ex: cliente exige dado em datacenter específico).

## Citações

> "Most people accessing Facebook stay within a small, fairly predictable group of no more than two or three regions."

> "40 percent smaller footprint, which resulted in a 50 percent reduction of the corresponding WAN traffic and an approximately 50 percent reduction in perceived latency."

## Confiança / datas

- Data confirmada via fetch (2018-10-08) — **alta confiança**.
- Fonte primária Meta Engineering.
