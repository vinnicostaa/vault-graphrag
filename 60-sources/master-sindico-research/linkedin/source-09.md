---
title: source-09 — PYMK Candidate Generation & Pemberly (combined)
type: source
tags: [research, linkedin, pymk, graph-walks, two-tower, ember, frontend]
source: https://www.linkedin.com/blog/engineering/recommendations/candidate-generation-in-a-large-scale-graph-recommendation-system-people-you-may-know
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-pymk, pemberly]
---

# source-09 — PYMK candidate generation + Pemberly frontend

**Dois temas agrupados** porque ambos aparecem lateralmente no master-sindico (M3+ pra PYMK, stack-reference pra Pemberly).

## 9a — People You May Know: Candidate Generation

**URL**: https://www.linkedin.com/blog/engineering/recommendations/candidate-generation-in-a-large-scale-graph-recommendation-system-people-you-may-know
**Fonte primária**: sim (LinkedIn Engineering — Recommendations)

### Pontos extraídos
- Pipeline: **candidate generation → ranking → final selection** (3 estágios).
- Candidate generation usa 3 famílias:
  1. **Graph-based**:
     - N-hop neighbors (2-hop, 3-hop) scored por common connections.
     - **Personalized PageRank** — probabilistic walks que soft-confinam a vizinhança do viewer; alcança 5-6 hops.
  2. **Similarity-based**:
     - Two-tower neural network aprende embeddings a partir de school, company, geography, activity.
     - **Impressions como positivas** (não invitations) — evita viés contra members raros.
     - 3 negative sampling: in-batch, random, nearest-neighbor.
     - Retrieval via approximate NN search em embeddings pré-computados.
  3. **Heurísticas**:
     - Feed interactions recentes, notification clicks, profile searches.
- Avaliação: Recall@k + entropy (qualidade + diversidade).
- Escala: ~1B members.

### Aplicabilidade master-sindico
- M1/M2: **zero**. Não temos rede social.
- M3+ Connect Me: **heurísticas como família 1** ("empresas bem avaliadas no seu bairro/cidade") cobre 80%. Graph walk como família 2 se o grafo condomínio↔empresa↔síndico ultrapassar 10k nodes. Two-tower é M4+ ou nunca.
- **Lição metodológica**: impression-as-positive > invite-as-positive. Se formos tracking de "síndico olhou perfil da empresa", **esse sinal é mais honesto** que "síndico chamou no chat" (que depende de intenção já formada).

## 9b — Pemberly frontend

**URL (atual, pós-redirect)**: https://www.linkedin.com/blog/engineering/archive/pemberly-at-linkedin
**Data**: 2016-12
**Fonte primária**: sim (LinkedIn Engineering — arquivado)
**Complementar**: https://engineering.linkedin.com/blog/2016/01/smashing-the-monolith

### Pontos extraídos
- Stack LinkedIn web (2016-2018): **Ember.js** frontend + **Play** (Scala/Java) mid-tier.
- Pemberly = nome do ecossistema.
- **Componentes**:
  - API Server (Play): traduz Rest.li interno → JSON público, com auth.
  - Base Page Renderer: 3 modos — Vanilla (CSR), SSR, **BigPipe** (streaming data enquanto Ember boota).
  - Ember Application Layer: Ember Data pra entity/API writes.
  - ArtDeco CSS via Sass/Eyeglass, distribuída como Ember CLI add-on.
- **Glimmer2** — novo renderer Ember pra performance.
- **Lazy/eager loading de engines** — code splitting modular.
- Objetivos: responsiveness (sem full page reload), state management unificado, developer productivity.

### Histórico de migração
- LinkedIn adota Ember cedo (2014-ish), investe pesado (contributors do core team).
- Hoje migrando porções pra React via bridges (ember-engines + react-in-ember).
- **Migração é lenta** — coexistência prolongada, codemods, padrões duplos confundem devs novos.

### Aplicabilidade master-sindico
Detalhada em [[_destilado#8-frontend-ember-pemberly--modern-reactnext]].

- Stack master-sindico (Next.js 15 + Flutter) já está decidida. Não há migração de framework pendente.
- **Lição aplicável**: quando mudar padrão interno (ex: Zustand → Jotai; Riverpod → Signals), **codemods antes de mandar humano refatorar**. LinkedIn aprendeu na dor.
- **BigPipe ≈ Next.js RSC + Suspense** — padrão de SSR streaming com placeholders. Útil pra dashboard de síndico com múltiplos widgets independentes.
- **Lazy engines ≈ Next.js dynamic imports + route groups** — code splitting por feature. Aplicar desde dia 1 (web do master-sindico já usa).

## Quando releer

- PYMK: quando Connect Me M3+ precisar de "recomendação de empresa".
- Pemberly: nunca diretamente (framework decidido), mas como cautionary tale antes de qualquer "grande migração" de stack interna.

## Vizinhos

- [[_destilado]]
- [[source-04]] — Feed ranking (próximo estágio depois de candidate generation)
- [[source-10]] — Endorsements (também usa graph infrastructure)
