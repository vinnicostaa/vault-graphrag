---
title: Aplicações LinkedIn → Master Síndico (recorte profissional/marketplace/network)
type: note
tags: [research, linkedin, aplicacoes, master-sindico, v2, connect-me, banco-de-talentos, reputacao, marketing-agencias]
source: 13-research/linkedin/_destilado.md + 00-product/sub-produtos/{02,03,07,10} + 02-architecture/event-driven.md + STATE.md
created: 2026-04-23
updated: 2026-04-23
aliases: [linkedin-aplicacoes, aplicacoes-linkedin]
---

# Aplicações LinkedIn → Master Síndico

Cruzamento pragmático do [[_destilado]] com os sub-produtos de natureza **profissional/marketplace/network**: Connect Me, Banco de Talentos, Reputação e Marketing/Agências. Foco M1 (2026-05-08). Cinco aplicações diretas; o resto vira gap/backlog.

> Regra herdada: reputação determinística (D-051, ADR-0023), eventos via Outbox M1 → NATS JetStream só quando 3×3 real ([[../../02-architecture/event-driven]] §2), PG tsvector em vez de OpenSearch (D-067).

---

## 1. Kafka/"The Log" → NATS JetStream quando 3×3 real (reafirmação)

**Analogia**: Kafka do LinkedIn = NATS JetStream nosso; o **outbox Postgres** é o equivalente honesto em M1. [[_destilado]] §1 + [[source-01]].

**Aplicação concreta (cross sub-produtos)**:
- **Connect Me + Banco de Talentos + Reputação + Marketing** são, juntos, os maiores produtores de eventos (connect_me.*, proposal.*, closed_deal.*, evaluation.*, reputation.*, video.*, delegation.*, content.talent.*, curriculum_search_performed, harassment_report.*). Em M1, cada um é ainda 1×1..1×2 (handler síncrono UoW + outbox → 1-2 side effects). **Mantém Outbox** ([[../../02-architecture/event-driven]] §3).
- **Gatilho real de NATS no M2**: quando `ReputationTierChanged` for consumido por ≥ 3 subscribers (perfil público + feed Connect Me + badge Marketing + indexer busca), OU quando `content.talent.curriculum-published` alimentar ≥ 3 projeções (search index + notificação empresa + analytics agência). Aí o 3×3 fica real — ligar stream `events.commercial` e `events.content` ([[../../02-architecture/event-driven]] §9).
- **Retenção/Replay**: quando entrar, retenção 30d padrão para commercial/content; 90d para ata assembleia. Consumer groups por subscriber; replay histórico via `DeliverByStartTime` (LGPD auditoria).

**Veredito M1**: ❌ não ligar NATS. ✅ manter outbox. D-LKD-001 do destilado já cobre.

---

## 2. Social graph / Voyager search → PG tsvector M1, federator M2 (revisão de D-067)

**Analogia**: LinkedIn tem grafo denormalizado member↔member + Galene (Lucene) para busca com ranking em camada separada ([[source-02]], [[source-03]]).

**Aplicação concreta**:
- **Grafo Master Síndico** (síndico↔condomínio↔empresa↔parceiro↔agência↔morador-talento) é **raso** (3-4 hops fixos) e < 1M edges em horizonte M3 — **Postgres + FKs + `WITH RECURSIVE`** basta. D-LKD-008.
- **Busca Connect Me (empresas)** + **busca Banco de Talentos (currículos)** + busca Parceiro Vizinhança = 3 verticais distintas já em M1/M2. D-067 trocou OpenSearch por PG tsvector — **decisão permanece válida para M1** (stack reduzida, vocabulário limitado, volume baixo).
- **Revisão pendente (GAP registrar como A-###)**: aplicar a lição Galene de **separar document fetching de ranking** (D-LKD-005). Query tsvector devolve candidatos; re-ranking determinístico em Go aplica boosts: `reputation_tier_bonus` (ADR-0023 já previsto w6: Bronze=0…Diamante=0.20), proximidade CEP, match de skill, tenure. Evita acoplar regra de negócio ao índice.
- **Federator M2+**: quando virar 3+ índices (empresas + talentos + posts/comunicados), introduzir gateway `/search` que faz scatter-gather. Até lá, endpoints separados por vertical (`/companies/search`, `/talent-bank/search`).
- **Query understanding BR** (sinônimos curados, sem ML): "encanador↔hidráulico↔bombeiro hidráulico", "diarista↔doméstica↔faxineira", "porteiro↔zelador noturno". Seed manual, 200-500 entradas cobrem o universo ([[source-08]] + destilado §6). Aplicável tanto no Connect Me quanto no Banco de Talentos (TalentArea aliases).

**Ação M1**: manter PG tsvector + camada de re-rank determinístico no application. **M2**: avaliar OpenSearch só se p95 tsvector > 300ms com >50k currículos/empresas.

---

## 3. Trust/Safety anti-fake CNPJ (Connect Me + Agências) — gap real

**Analogia**: LinkedIn combina validação documental + **device fingerprint + IP velocity + behavioral post-signup** porque documento sozinho é furável ([[source-05]], [[source-06]], destilado §4).

**Aplicação concreta**:
- **Connect Me já tem**: CNPJ + validação Receita + HarassmentReport aggregate + admin bypass (D-058/D-072). ✅
- **Falta (abrir como A-### em AUDIT)**:
  1. **Device fingerprint + IP velocity** no cadastro de empresa (Connect Me) e no aceite de DelegationGrant (Agência) — N cadastros/aceites do mesmo device/IP em janela curta ⇒ CAPTCHA forte + fila de revisão manual (admin).
  2. **Behavioral signals pós-cadastro**: empresa nova que dispara ≥ 10 propostas em 24h, ou agência aceita ≥ 5 DelegationGrants em 1h → shadow-ban até revisão.
  3. **Velocity counters** no Banco de Talentos: empresa Pro que faz > 200 buscas/dia ou abre > 100 currículos sem favoritar nenhum → rate limit suave + flag ("harvesting" de base LGPD). Já temos `curriculum_search_logs` (INV-BT-07); adicionar thresholds.
  4. **Unified content filtering** (`internal/safety/content`) reutilizada por: `HarassmentReport.description`, `ConnectMeRequest.message`, `Proposal.description`, `CompanyEvaluation.comment`, `CurriculumProfessionalProfile.answer`, `MarketingLinkRequest.message`. Mesma lib, mesmo log. D-LKD-003.
- **NÃO replicar ML classifier** (CNN LinkedIn) em M1 — palavras-chave + denúncia humana basta até > 100 denúncias/semana não resolvidas em 24h ([[source-05]] §4, destilado §4).

**Veredito**: abrir **A-LKD-001** (velocity/fingerprint anti-fake) para AUDIT, escopo Sprint 7 Connect Me + Sprint 10 Banco de Talentos. Não bloqueia M1.

---

## 4. Endorsements/Seals — mecânica LinkedIn adaptada ao "Síndico-aprovado"

**Analogia**: LinkedIn endorsements = relação `endorser → recipient + skill` com fraude alta (amigos endossam amigos); qualidade só aparece quando o endorser **conhece a pessoa E o skill** ([[source-10]], destilado §7).

**Aplicação concreta**:
- **Master Síndico já tem 2 mecânicas análogas**, ambas **anti-fraude by design** melhores que LinkedIn:
  1. **CompanyEvaluation** (Req 26) — 5 critérios 1-10, **gatilho = ClosedDeal.signed imutável** (síndico só avalia se contratou). Resolve self-endorsement + conluio que LinkedIn não resolve bem. Alimenta o peso 40% da fórmula determinística de Reputação (D-051).
  2. **Seal "Síndico-aprovado"** (peso 10% `content_engagement`) — síndico `plan_tier ≥ plus` concede selo a empresa. Equivalente direto a endorsement LinkedIn; ranqueia no score de reputação.
- **Lição LinkedIn aplicável** (adicionar ao cálculo quando houver dado, M3+):
  - Priorizar avaliações/seals de síndicos **com reputação consolidada** (plan_tier pro + tempo plataforma > 1 ano) — feature interpretável, sem ML.
  - Priorizar avaliações de **contratos maiores** (já feito: `rating_avg_weighted` pondera por `totalAmountCents` — D-051 ✅).
  - Priorizar avaliações **com texto** sobre estrelas-sem-texto (sinal de engajamento real).
- **Dupla-cega 14d** (D-LKD-004, padrão Airbnb): avaliações bilaterais síndico↔empresa permanecem ocultas até ambos submeterem **ou** 14 dias. Mitiga retaliação. **Registrar como D-### / ADR pendente** — hoje CompanyEvaluation é unidirecional (síndico→empresa); o inverso (empresa→síndico sobre pagamento em dia/clareza) pode entrar M3+ para alimentar reputação do síndico (opcional §3 Reputação).

**Veredito**: mecânicas atuais **superam LinkedIn em anti-fraude** porque exigem contrato fechado. Nenhuma mudança M1. Abrir A-LKD-002 para dupla-cega 14d e avaliação empresa→síndico (backlog M3+).

---

## 5. PYMK/People You May Know → **fora de M1/M2** + lateral para Connect Me M3+

**Analogia**: LinkedIn PYMK usa graph walks (PPR) + two-tower neural + heurísticas; treinamento com impressions como positivas ([[source-09]], destilado §9).

**Aplicação concreta**:
- **Direto em M1/M2**: ❌ nada. Não temos rede social interna; síndico↔síndico não conversam; morador↔morador idem.
- **Lateral Connect Me M3+**: "Empresas que atenderam condomínios similares ao seu" (cidade + tipo + tier reputação) e "Síndicos perto de você" (bairro + plan_tier) são ganchos óbvios quando houver > 10k empresas e > 1k condomínios. Implementação **começa por heurísticas** (3ª família LinkedIn), nunca neural:
  - PPR Postgres: `WITH RECURSIVE` em `deals JOIN companies JOIN condominiums` — devolve top-N empresas mais próximas no grafo.
  - Ranking: features interpretáveis (reputation_tier, geo_distance_km, mesma categoria, volume_cases). Zero ML.
- **Feed "artigos"/newsletter** do LinkedIn → no Master Síndico já existe como **vídeos institucionais (Req 29)** + **comunicados síndico→moradores** + **cases de empresa**. Feed próprio não é prioridade; exposição por perfil público (síndico Pro, empresa Plus/Pro) cobre a equivalência.
- **Verified badges** (LinkedIn verified profiles) ≡ **reputation_tier + selo Síndico-aprovado + CNPJ verificado + agência com DelegationGrant ativo**. Verificação manual admin (BIL-###/IDN-###) já prevista no fluxo Connect Me (status `pending_verification → active`) e em Marketing Link E14 (empresa convida agência).

**Veredito**: ❌ M1/M2. Registrar como idéia M3+ em [[../../10-schedule/ROADMAP]] (sem ADR enquanto não houver volume).

---

## Gaps a registrar em AUDIT

| ID proposto | Assunto | Origem |
|---|---|---|
| **A-LKD-001** | Anti-fake além de CNPJ: device fingerprint + IP velocity + behavioral post-signup para empresa (Connect Me) e agência (DelegationGrant) | §3 |
| **A-LKD-002** | Avaliação dupla-cega 14d síndico↔empresa (Airbnb-style) + avaliação empresa→síndico para reputação síndico M3+ | §4 |
| **A-LKD-003** | Re-ranking determinístico separado do document fetching (PG tsvector + camada app boost) — aplicar em Connect Me + Banco de Talentos | §2 |
| **A-LKD-004** | Unified content filtering library (`internal/safety/content`) compartilhada por HarassmentReport, ConnectMe, Proposal, Evaluation, Curriculum, MarketingLink | §3, D-LKD-003 |
| **A-LKD-005** | Velocity counters no Banco de Talentos (empresa Pro harvesting de base LGPD) | §3 |

---

## Resumo decisões confirmadas/novas

- **D-LKD-001..D-LKD-008** do destilado seguem válidos. Aplicação específica confirmada aqui para Connect Me, Banco de Talentos, Reputação e Marketing/Agências.
- **Nenhuma mudança em M1** nos sub-produtos cobertos — arquitetura atual (PG + tsvector + Outbox + DelegationGrant ABAC + ReputationCalculator determinístico) é **superior ou equivalente** às lições LinkedIn para o porte/segmento/regulação brasileira.
- **Prioridade pós-M1**: A-LKD-001 (anti-fake) e A-LKD-003 (re-rank determinístico) entram antes de qualquer outro item deste recorte.

---

## Vizinhos

- [[_destilado]] — síntese das 10 fontes LinkedIn
- [[_moc]] — MOC pasta linkedin
- [[../../STATE]] — D-051, D-056..D-068, D-LKD-001..008
- [[../../AUDIT]] — destino das A-LKD-###
- [[../../00-product/sub-produtos/02-connect-me]]
- [[../../00-product/sub-produtos/03-reputacao]]
- [[../../00-product/sub-produtos/07-banco-de-talentos]]
- [[../../00-product/sub-produtos/10-marketing-agencias]]
- [[../../02-architecture/event-driven]] — Outbox + critério 3×3 NATS
- [[../../02-architecture/patterns]] — Observer/Outbox/Specification/Idempotency
- [[../../02-architecture/adr/0023-recsys-deterministic-m1]] — reputação determinística
