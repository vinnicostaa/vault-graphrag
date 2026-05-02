---
title: Netflix Personalization & Recommendation System
type: source
tags: [personalization, recommendation, machine-learning, contextual-bandits, hydra, research]
source: https://www.brainforge.ai/blog/how-netflix-uses-machine-learning-ml-to-create-perfect-recommendations
created: 2026-04-23
updated: 2026-04-23
aliases: [netflix-recommendation]
---

# Netflix Personalization & Recommendation

> Fontes: Netflix Research (recommendations area), Netflix TechBlog "System Architectures for Personalization", BrainForge síntese, Medium Shilpa Thota — destilado 2026-04-23.

## Impacto

- Em 2024, **> 80% do conteúdo consumido vem de recomendação** (não de busca direta).

## Arquitetura em 3 camadas (canônico Netflix)

1. **Offline** — batch ML training (Spark, GPU). Modelos grandes, dados históricos completos.
2. **Nearline** — event-driven (Kafka → feature store). Atualiza features em segundos após ação do usuário.
3. **Online** — request-time serving. Modelos leves/pre-computed, p99 < 100ms.

## Modelos que compõem o sistema

- **Collaborative filtering** — "quem viu X também viu Y".
- **Content-based** — features extraídas do próprio item (microgenres, scene pacing, artwork).
- **Deep learning** — neural nets sobre embeddings de usuário e item.
- **Contextual bandits** — trade-off explore/exploit pra artwork personalizado (cada título tem N versões de poster; bandit aprende qual converte melhor pra cada segmento).
- **Hydra (multi-task learning)** — um único modelo faz homepage ranking + search ordering + notification personalization simultaneamente (economia operacional, consolidação de models).

## Feature store

- Stream-processing (Kafka + Flink equivalent) atualiza features em segundos.
- Features consumidas por modelos online sem re-query a stores de dado crus.
- **Separa feature engineering de serving** — reuso entre modelos.

## Pipeline simplificado

```
User action (play, rate, browse) → Kafka →
  Feature Store (updates)
  + Offline training batch
→ Online model serving
  → Personalization API
  → UI (homepage rows, artwork variant)
```

## Lições aplicáveis

1. **3 camadas (offline/nearline/online)** separam preocupações de latência vs freshness.
2. **Contextual bandit > A/B test** pra otimização contínua de apresentação.
3. **Multi-task > N models** quando features são compartilhadas.
4. **Feature store** evita duplicação de feature engineering entre modelos.
5. **> 80% do consumo via recomendação** mostra impacto de negócio real → vale o investimento ML.

## Aplicabilidade ao Master Síndico

### Cenário 1 — Ranking de empresas no Connect Me (marketplace)

**Faz sentido simplificar:**

- M1: ordenação determinística (relevância textual + geo + reputação). Sem ML.
- M2: contextual signal simples — boost por tenant (condomínio X viu empresa Y antes?).
- M3+: **eventual** model ranker com features (distância, nota, histórico de contratações). LambdaMART ou XGBoost simples. **Não deep learning**.
- Nunca: deep learning / embeddings / bandit em escala Netflix — temos centenas de empresas, não milhões.

### Cenário 2 — Feed de vídeos (sub-produto Conteúdo)

- M1: ordenação manual por curadoria + recency + module sequence (curso tem ordem).
- M2: "continue watching" + "recomendado pro seu cargo" (filtro por persona).
- M3+: se biblioteca crescer > 1000 vídeos, considerar content-based simples (tags + embeddings do título/descrição).

### Cenário 3 — Artwork personalization

- **Não aplicar.** Não temos volume de views que justifique bandit.

### Insight aplicável desde já

- **Separar offline/nearline/online mentalmente** mesmo sem ML:
  - Offline: relatórios semanais (reputação, stats).
  - Nearline: Kafka-like → atualiza contadores (views, likes).
  - Online: request → busca pré-computada, não recalcula.

## Anti-padrão pra evitar

- Construir ML platform custom antes de ter dado suficiente — falha garantida.
- Usar GPT/LLM pra tudo que parece "recomendação" — custo alto, latência alta.

## Links

- [[source-07]] — EVCache (cache de feature store)
- [[source-08]] — polyglot (ML feature store é um datastore)
- [[_destilado]]
