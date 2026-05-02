---
title: Monolito → microservices Netflix (quando decompor e pitfalls)
type: source
tags: [netflix, microservices, monolith, decomposition, pitfalls, research]
source: https://medium.com/@yashbatra11111/how-netflix-accidentally-proved-monoliths-scale-better-than-microservices-f2d66f0a0bb5
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-monolith-lessons]
---

# Monolito → microservices Netflix: quando decompor

> Fontes: análises de caso Netflix (dev.to, Medium, AWS blog) + InfoQ — síntese destilada em 2026-04-23.

## Trigger de decomposição (Netflix)

- **2008**: corrupção de banco derrubou o serviço por 3 dias. Esse foi o gatilho claro — ponto-único-de-falha insustentável.
- Antes disso, scaling vertical dava conta. Crescimento de streaming expôs limites.

## Pitfalls documentados

1. **Rushing** — Netflix levou 7 anos. Big-bang é receita pra desastre.
2. **Sem alinhamento organizacional** — Conway's law: team boundaries = service boundaries. Sem reorg, microserviços viram "distributed monolith".
3. **Bad design** — "pior que monolito bem mantido". Módulos coacoplados via import compartilhado destroem a autonomia que microserviços prometem.
4. **Cascading failures** — sem circuit breaker / bulkhead, uma falha derruba cadeia inteira.
5. **API Gateway virando novo monolito** — se o gateway tem lógica de negócio, concentra risco e deploy.

## Success factors

- Começar pela parte que **não é core** — reduz risco.
- Cultura de ownership/responsabilidade — cada time responsável pelo seu serviço 24/7.
- Observabilidade **antes** da decomposição — sem ela, você cega.
- Circuit breaker + bulkhead + timeout — obrigatório em qualquer call cross-service.

## Counter-take importante

Alguns analistas argumentam que Netflix "provou que monolitos escalam melhor" — não porque monolito é superior, mas porque **a maioria dos projetos que migram não tem a escala nem o problema que justificava o custo da migração**. Traduzindo: não migre porque está na moda.

## Aplicabilidade ao Master Síndico

- Hoje (M1): monolito em Go modular é **correto**. Bounded contexts separados dentro do monolito dão 80% do benefício com 20% do custo operacional.
- Trigger pra extrair micro: deploy do módulo Conteúdo afetando build time global; pipeline Mux precisar de scale ops-independente; tenant isolation requerendo DB separado.
- Ordem provável: Conteúdo/Vídeos (primeiro, por stack diferente) → Assembleias Live (Livekit, scale burst) → Connect Me (quando marketplace crescer).

## Links

- [[source-01]] — evolução API layer
- [[source-03]] — service mesh moderno
- [[_destilado]]
