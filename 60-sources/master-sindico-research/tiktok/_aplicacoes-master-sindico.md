---
title: Aplicações TikTok/ByteDance — Master Síndico v2
type: note
tags: [research, tiktok, bytedance, aplicacoes, recsys, bytegraph, moderation, master-sindico, v2]
source: 13-research/tiktok/_destilado.md + cross 00-product/sub-produtos/{02,04,07,08} + 02-architecture/patterns.md + STATE.md
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-aplicacoes, tiktok-master-sindico]
---

# Aplicações TikTok/ByteDance — Master Síndico v2

Cruzamento do [[_destilado]] com sub-produtos Master Síndico. **Tese**: MS é **governança condominial**, não mídia viral — escala ~10⁶ menor que TikTok, audiência adulta/profissional. Aproveitamos padrões conceituais; infra não.

> ⚠️ **Shorts NÃO existe canonicamente** — [[../../00-product/sub-produtos/04-conteudo-videos#3-não-escopo]]. Vídeos institucionais (Req 29, 12 tipos) + vídeo-currículo 90s (BT) têm **lock 90d** + **visibilidade Rule 4** — *oposto* da mecânica de consumo infinito do TikTok.

---

## Top-5 aplicações

### 1. Recsys híbrido para Banco de Talentos — **M3+ pós-PMF**

Hoje busca é filtro estruturado + `tsvector` (D-067) — suficiente. **M3+ (>1k currículos + >100 empresas Plus/Pro)**: recsys híbrido leve — embeddings `text-embedding-3-small` em `pgvector` sobre `professional_profile.answer` + collaborative filtering batch noturno sobre `company_curriculum_favorites`. **Sem Flink, sem parameter server, sem Monolith**. Cuidados LGPD: top-3 fatores visíveis + appeal humano (Art. 20). → [[_destilado#1-for-you-page-monolith-recsys]]

### 2. ByteGraph vs PostgreSQL — **Postgres vence por ~10⁶ margem**

M1: 50 condomínios, 5k moradores, 500 empresas. **Postgres 18 + índices compostos + recursive CTE + PostGIS (ST_DWithin para Vizinhança raio 2km) + pgvector (recsys)** resolve tudo até M4+. Reconsiderar Neo4j/Dgraph só se Vizinhança virar rede social vizinho-vizinho (improvável — DNA §3 [[../../00-product/sub-produtos/08-vizinhanca]] é curadoria do síndico, não rede aberta). **Lição conceitual válida**: modelar grafo explicitamente no DDD (aggregates `Company.Reviews`, `Partner.Seals`), não no DB. → [[_destilado#4-bytegraph-graph-db-in-house]]

### 3. TikTok LIVE — **não aplica; Livekit cobre assembleia (M3)**

Assembleia é **N↔N conversacional** (10-500 participantes, <400ms), não broadcast 1→milhões. Livekit (WebRTC SFU) é a escolha certa — CDN+200 PoPs é desperdício. Saga `AssemblySaga` já prevê compensação ([[../../02-architecture/patterns#7-saga]]). Se surgir broadcast 1→N (síndico transmite evento pra moradores espectadores): Mux Live (já contratado via `ILiveStreamProvider`), sem infra própria. Padrões transferíveis: ingest separado de delivery; chat/voto em WebSocket fora do canal A/V (já adotado); edge cache 30-60s para replay assembleia. → [[_destilado#5-live-streaming-global-tiktok-live]]

### 4. Cascade moderation 2-stage — **M1-M2 suficiente com humano**

Escala real: ~14 vídeos/dia (500 BT + 100 empresa/mês). Moderação humana auxiliada por LLM cobre. **Cascade canônico**: (1) Stage barato síncrono — CHECK duração ≤90s + AWS Rekognition face/nudity + OpenAI Moderation em transcript Mux; (2) Stage caro async — Claude Sonnet julga 10% amostragem + 100% sinalizados contra **golden seed bank** cultivado pelo admin (Strategy pattern); (3) **Stage humano obrigatório** — admin D-058 revisa rejeitados + apelados em 48h (LGPD Art. 20 não-negociável). Overkill evitado: MLLM próprio, LLaVA fine-tune, times globais. → [[_destilado#3-moderação-multi-idioma-em-escala]]

### 5. Transparência algorítmica + anti-dark-patterns — **PL 2.630 ready**

DSA (45M UE) não nos atinge, mas **operar transparente desde M1** evita migração emergencial quando PL 2.630 virar lei. **Proibido**: infinite scroll, autoplay, push persistentes, recsys otimizando "tempo na plataforma". **Obrigatório em toda decisão automatizada** (Connect Me ranking, BT ranking, reputação, feed Vizinhança): top-K fatores visíveis + botão "Contestar decisão" + audit trail com `decision_id` + snapshot features (retention LGPD 5 anos). Transparência vira **invariante arquitetural**, não feature — todo ranker expõe `.Explain()`. → [[_destilado#8-transparência-algorítmica-e-debate-público]]

---

## Não-aplicações explícitas (anti-catálogo)

| TikTok faz | MS explicitamente NÃO faz | Fonte |
|---|---|---|
| Shorts / feed 15-60s viciante | ❌ canonicamente proibido | [[../../00-product/sub-produtos/04-conteudo-videos#3-não-escopo]] |
| Infinite scroll + autoplay + push persistente | ❌ paginação + click-to-play + push só-acionável | §5 |
| FYP otimiza tempo-na-plataforma | ❌ otimizar tarefa resolvida | §5 |
| Monolith real-time (parameter server, Flink) | ❌ batch noturno M3+ | §1 |
| ByteGraph graph DB próprio | ❌ Postgres + CTE + PostGIS | §2 |
| CDN+PCDN / GPU edge-nodes / codecs custom | ❌ Mux cobre 100% | [[../../00-product/sub-produtos/04-conteudo-videos]] |
| MLLM moderation própria (LLaVA) | ❌ OpenAI Mod + Claude + humano | §4 |
| Gödel Scheduler K8s custom | ❌ CronJob / Temporal M3+ | [[_destilado#6-task-scheduling-em-escala-gödel-bytetask]] |
| Live 1→milhões RTMP+LL-HLS+WebRTC | ❌ Livekit assembleia; Mux Live se broadcast | §3 |
| Likes públicos / endorsements / feed social | ❌ anti-fraude by design | [[../../00-product/sub-produtos/07-banco-de-talentos#16-anti-fraude]] |
| Transparency Accountability Centers físicos | ❌ explicabilidade in-app basta | §5 |

---

## Cuidados de foco

- **MS = governança condominial** (LGPD + CC Art. 1.336/1.350). Não é rede social, não é entretenimento.
- **Recsys é M3+ pós-PMF**. M1 (2026-05-08) entrega ranking determinístico. ML antes de 1k currículos + 100 empresas Plus/Pro é over-engineering.
- **Escala TikTok ~10⁶ acima da nossa** — toda infra ByteDance é overkill. Conceito sim; infra não.
- **Audit + appeal humano não-negociáveis** (LGPD Art. 20 + D-058). "Automação pre-view 96%" sem appeal é ilegal no BR.
- **Cultivar golden seed bank desde M1** mesmo com volume baixo — paga em M3 no cascade.

---

## Links

- [[_destilado]] — síntese 8 fontes TikTok/ByteDance
- [[../../00-product/sub-produtos/04-conteudo-videos]] — "Shorts NÃO" canônico
- [[../../00-product/sub-produtos/07-banco-de-talentos]] — target #1
- [[../../00-product/sub-produtos/02-connect-me]] — target #4/5
- [[../../00-product/sub-produtos/08-vizinhanca]] — target #2
- [[../../02-architecture/patterns]] — Strategy + Specification + Saga
- [[../../STATE]] — D-056 / D-058 / D-062 / D-067
- Plano: `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`
