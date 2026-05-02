---
title: Evolução da arquitetura de API Netflix (monolito → federated gateway)
type: source
tags: [netflix, microservices, api-gateway, graphql, monolith-decomposition, research]
source: https://newsletter.techworld-with-milan.com/p/evolution-of-the-netflix-api-architecture
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-api-evolution]
---

# Evolução da arquitetura de API Netflix

> Fonte: Milan Jovanović, *Evolution of the Netflix API Architecture* — síntese destilada em 2026-04-23.

## 4 estágios canônicos

1. **Monolito (≤ 2008)** — tudo num deploy único; db monolítico falhou por 3 dias em 2008 e disparou a migração.
2. **Direct access (2009-2012)** — cliente bate direto em microserviços; virou inviável pela complexidade de N clients × M serviços e fetching duplicado.
3. **Gateway aggregation layer (2013-2018)** — camada unificada (Falcor/Groovy scripts) resolvendo múltiplas chamadas numa só; reduz round-trips mas centraliza lógica de cliente no gateway.
4. **Federated gateway (2019-atual)** — GraphQL federation (Apollo Federation, implementação Netflix em Kotlin). Backend teams mantêm autonomia sobre seus schemas; gateway compõe.

## Quando cada estágio faz sentido

- Monolito basta até surgir dor real (deploy lento, team bottleneck, escala).
- Direct access só com ≤ 5 microserviços e cliente único bem controlado.
- Gateway agregador aparece quando clients são múltiplos (web, mobile, TV) e duplicação de fetch vira problema.
- Federação só quando nº de domínios e consumidores excede capacidade de um único team manter o gateway.

## Anti-padrão chave

"Startups adotam federação antes de ter os problemas de federação" — over-engineering. Netflix levou **~11 anos** (2008 → 2019) pra chegar ao estágio 4.

## Aplicabilidade ao Master Síndico

- M1 (master-sindico-v2): **ainda monolito deployado em Go**, mesmo com módulos separados. OK enquanto o time for pequeno e os bounded contexts estáveis.
- M2-M3: primeiro candidato a extrair é Conteúdo/Vídeos (I/O intensivo, pipeline Mux, storage diferente), depois Connect Me (marketplace, personalização).
- Federated GraphQL só faz sentido a partir de 50+ serviços; no nosso horizonte de 5 anos, ficamos no estágio 2-3.

## Links

- [[_destilado]]
- [[source-02]] — monolito → micro pitfalls
- [[../../02-architecture/_moc]]
