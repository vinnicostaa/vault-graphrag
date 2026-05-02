---
title: Journey to 1000 models — Scaling Instagram's recommendation system
type: source
tags: [research, instagram, meta, ml, model-registry, configerator, production-engineering, source]
source: https://engineering.fb.com/2025/05/21/production-engineering/journey-to-1000-models-scaling-instagrams-recommendation-system/
created: 2026-04-23
updated: 2026-04-23
aliases: [source-02-1000-models]
---

# source-02 — Journey to 1000 models

- **URL**: https://engineering.fb.com/2025/05/21/production-engineering/journey-to-1000-models-scaling-instagrams-recommendation-system/
- **Publicado**: 2025-05-21
- **Autor/Canal**: Engineering at Meta
- **Tipo**: post técnico

## Sinopse

Como o Instagram escalou pra 1000+ modelos ML em produção (Feed, Stories, Reels, comments, notifications, tagging) sem sacrificar qualidade nem confiabilidade.

## Pontos-chave

1. **Três gaps críticos**: descoberta (não havia source of truth de modelos em produção), release (lançamentos lentos/inconsistentes), health (sem métrica padronizada de qualidade).
2. **Model Registry** — ledger central em cima do sistema Configerator da Meta. Catálogo de importância, função de negócio, metadados.
3. **Launch automation** — reduziu tempo de deploy de dias para horas. Suporta 10+ lançamentos por semana. Inclui estimativa de capacidade, avaliação offline, traffic shifting gradual.
4. **Model Stability Metric** — indicador binário de saúde do modelo baseado em calibration + normalized entropy. Mesma definição em todos os ranking models.
5. **Lição**: infra understanding é fundamento pra ferramentas. Iteração mais rápida paradoxalmente aumenta confiabilidade (menos erros de coordenação).

## Aplicabilidade ao Master Síndico

- **Não aplicável diretamente em M1-M2** — ainda não temos modelos ML em produção.
- **Quando M3+ trouxer primeiros modelos** (moderação, ranking Connect Me ML, possivelmente recommendation de empresas): começar **já** com model registry simples (tabela `ml_models`, versioning, feature flag por modelo). Não esperar atingir 10 modelos para criar.
- **Calibration + normalized entropy** como métrica padrão de saúde é boa prática universal, vale adotar desde o primeiro modelo.
- **Traffic shifting gradual** (canary/shadow) precisa do mesmo pipeline de feature flags que já teremos para features normais — reaproveitar.

## Citações

> "Within each surface and layer, there is constant experimentation, and these permutations create a severe infrastructure challenge."

> "Infra understanding is the foundation to building the right tools."

## Confiança / datas

- Data confirmada via fetch (2025-05-21) — **alta confiança**.
- Fonte primária Meta Engineering.
