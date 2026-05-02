---
title: Destilado — TikTok/ByteDance aplicável ao Master Síndico
type: note
tags: [research, tiktok, bytedance, recsys, video, live-streaming, moderation, graph-db, scheduler, master-sindico, v2]
source: 13-research/tiktok/source-01..08
created: 2026-04-23
updated: 2026-04-23
aliases: [tiktok-destilado]
---

# Destilado — TikTok/ByteDance aplicável ao Master Síndico

Síntese de 8 fontes (paper VLDB ByteGraph, arXiv Monolith, CNCF Gödel, TikTok Transparência, Medium LLM moderação, arXiv playback, Comissão Europeia DSA, análise arquitetural TikTok LIVE). Foco: o que é reaproveitável no **master-sindico** vs. o que é overkill. Escala ByteDance é 3-5 ordens de grandeza acima da nossa — a maior parte do valor vem de **padrões conceituais**, não de infra.

> Volume master-sindico M1: ~50 condomínios, ~5k moradores ativos, ~500 vídeos-currículo (Banco de Talentos), ~100 vídeos institucionais de empresas/síndicos. Volume TikTok: 34M vídeos/dia, bilhões de usuários. **Diferença: ~10^6**.

---

## 1. For You Page (Monolith) — recsys real-time

**O que é**: framework recsys baseado em TensorFlow com **embedding table sem colisão** (cuckoo hashing) e **online training** via Kafka → Flink → parameter server. Atualiza embeddings em minutos, parâmetros densos em dias/semanas. Trata concept drift (trend que surge/morre em horas).

**Aplicável ao master-sindico**:

- **Connect Me (marketplace síndico↔empresa)**: **potencialmente aplicável** como feed de descoberta de empresas. Match por similaridade (histórico de contratações do condomínio + avaliações + categoria de serviço + localização). **Mas em M1**: não precisa de recsys real-time. Um ranking por regras + score (Elo/Bayesian avg de reviews) resolve. **Revisitar em M3+** se volume > 1k empresas ativas e engajamento de descoberta for métrica-chave.
- **Feed de descoberta de vídeos** (vídeos de empresas + síndicos influenciadores): **conceito aplicável**, implementação não. Em M2, um feed cronológico + boost por engagement (like/comment count) cobre. Só depois de 10k+ vídeos faria sentido embedding-based retrieval.
- **Lição canônica**: **separar "serving model" de "training model"** e atualizar incrementalmente só as embeddings que mudaram. Padrão aplicável a qualquer sistema de ranking no master-sindico (ex: reputação de síndicos).

**Overkill**: parameter server, cuckoo hashing, Flink streaming. Não temos escala pra justificar. Postgres + job batch noturno de re-ranking atende M1-M2.

**Referências**: [[source-01-monolith-recsys]].

---

## 2. Vídeo short-form 90s — paridade com Banco de Talentos

**TikTok pipeline**: upload → chunking MPEG-DASH (1-5 MB) → transcoding GPU edge-node → 720p prioritário + resoluções completas em background → hot cache NVMe SSD (~48h) → tiered storage → CDN + PCDN híbrido com 200+ PoPs. Codecs H.264 (compatibilidade), H.265/HEVC, AV1 (devices novos), H.266/VVC (testes). Pulsar como backbone de fila.

**Aplicável ao master-sindico (Banco de Talentos + vídeos empresas/síndicos)**:

- **Mux já entrega tudo que precisamos em M1-M2**: upload direto do cliente, transcoding multi-resolução, adaptive bitrate, thumbnail, HLS/DASH, assinatura de URLs. Cobre 100% do caso vídeo-currículo 90s de morador e vídeo-pitch 60-90s de empresa/síndico.
- **Paridade conceitual TikTok**: limite de 90s, vertical 9:16, compressão agressiva pra reduzir fricção em upload mobile (relevante em condomínio BR onde morador pode estar em 4G). Mux cobre com `playback_policy: signed` + resoluções 360p-1080p.
- **Lição crítica**: **priorizar 720p pra disponibilidade imediata** (em TikTok, usuário vê o vídeo 3-5s após upload). Em Mux isso é o default (`ready` event antes de todas resoluções completarem).
- **Quando reconsiderar infra custom**: só se custo Mux > US$ 3k/mês (dezenas de milhares de vídeos ativos). Até lá, Mux é economicamente racional.

**Não faz sentido copiar**: CDN próprio, PCDN, codecs custom, GPU edge-nodes. Requer equipe dedicada e escala que não teremos antes de 2028.

**Referências**: [[source-04-video-delivery]], [[source-08-architecture-overview]].

---

## 3. Moderação multi-idioma em escala

**TikTok**: 2-stage cascade — (1) retrieval rápido por embedding comparando vs. "banco de exemplos de alto risco" → (2) MLLM (Multimodal LLM baseado em LLaVA) analisa só o subconjunto suspeito. 96% do conteúdo removido é derrubado antes de 0 views. Humanos anotam "golden seeds" e tratam apelações.

**Aplicável ao master-sindico**:

- **Master-sindico é PT-BR only** → problema de multilingual não se aplica.
- **MAS lições de ML aplicam**:
  - **Cascade 2-stage** é padrão útil: modelo barato filtra, modelo caro julga. Aplicável a **moderação de vídeo-currículo** (Banco de Talentos) e **review de anúncios/propostas** do Connect Me. Barato = heurísticas (palavrões, nudity hash, CNPJ inválido). Caro = LLM multimodal (Claude Sonnet / GPT-4o) julgando contexto.
  - **Golden seed set**: cultivar bank de exemplos curados (vídeos OK + vídeos rejeitados com razão) pra fine-tuning / few-shot prompting.
  - **Appeal flow obrigatório**: humano revisa decisões do modelo. Alinha com LGPD Art. 20 (direito à revisão de decisão automatizada).
- **Escala realista M1**: ~500 vídeos-currículo + ~100 vídeos empresa/mês. Volume permite moderação humana auxiliada por LLM, não precisa de MLLM próprio. OpenAI moderation API + Claude classificação cobre.

**Referências**: [[source-05-content-moderation]].

---

## 4. ByteGraph — graph DB in-house

**O que é**: graph DB da ByteDance, paper VLDB 2022. 10^10 vértices, 10^12 arestas, 30M queries/s, SLA 99.99%. Arquitetura 3 camadas: BGE (execução), cache in-memory, BGS (storage). Edge-trees pra adjacency lists. Replicação geográfica. Suporta 3 workloads: OLAP + OLTP + OLSP (serving real-time).

**Aplicável ao master-sindico**: **overkill absoluto pra M1-M2**.

- **Nosso "grafo"**: condomínio → moradores (10-500) → unidades, síndico → condomínios administrados (1-20), empresa → avaliações → condomínios que contrataram. Tudo cabe em Postgres com índices bem feitos. Milhões de vértices, não bilhões.
- **Quando reconsiderar**: se o **Vizinhança** (sub-produto social vizinho-vizinho) crescer pra rede de milhões de usuários com traversals profundas (ex: "amigos dos amigos do meu condomínio"), aí ArangoDB / Neo4j / Dgraph fazem sentido. Antes disso, recursive CTE em Postgres resolve.
- **Lição conceitual aplicável**: **modelar explicitamente o grafo de relações** no domínio (condomínio ↔ síndico ↔ empresa ↔ avaliação). Mesmo em Postgres, tratar isso como grafo facilita queries de "empresa mais recomendada entre condomínios similares ao meu".

**Referências**: [[source-02-bytegraph]].

---

## 5. Live streaming global (TikTok LIVE)

**Pilha TikTok LIVE**: ingestão via **RTMP/SRT**, transcoding, delivery via **LL-HLS** (passivo, 2-5s latência) + **WebRTC** (interativo, 200-500ms). WebSocket pra chat/gifts. Message queue (Pulsar/Kafka) pra eventos. Origin transcoda uma vez, edge servers fazem cache 30-60s e distribuem. PoP tree: origin → regional → edge.

**Master-sindico hoje (assembleias virtuais — M3)**: **LiveKit** (WebRTC SFU).

**Comparação**:

| Aspecto | TikTok LIVE | Master-sindico (LiveKit) |
|---|---|---|
| Modelo | 1 streamer → N viewers (pode ir a milhões) | N ↔ N participantes (assembleia é conversacional) |
| Latência necessária | 200ms se interativo, 2-5s se passivo | **< 400ms obrigatório** (deliberação, quórum, voto ao vivo) |
| Protocolo | RTMP ingest + WebRTC/LL-HLS delivery | WebRTC ponta-a-ponta (SFU) |
| CDN | Próprio + comerciais, 200+ PoPs | Não aplicável (SFU não usa CDN) |
| Escala típica | Milhões de viewers por live | 10-500 participantes por assembleia |

**Conclusão**: **pilhas diferentes por natureza do caso de uso**. LiveKit/WebRTC é a escolha certa pra assembleia (baixa latência + multi-party). Se master-sindico tiver futuramente **broadcast 1→N** (ex: síndico transmitindo evento do condomínio pra todos moradores como espectadores), aí **LL-HLS entra** — provavelmente via Mux Live (já contratado) em vez de montar infra própria.

**Lições aplicáveis**:

- **Separar ingest de delivery** por protocolo (RTMP entra, HLS sai) é desacoplamento útil mesmo em escala pequena.
- **Chat/gifts em WebSocket + message queue, não no mesmo canal do vídeo**: arquitetura limpa que vale replicar em assembleias (voto/chat em canal separado do áudio/vídeo).
- **Edge cache 30-60s de segments**: paradigma aplicável se master-sindico fizer replay de assembleias (gravação + playback).

**Referências**: [[source-03-tiktok-live]], [[source-08-architecture-overview]].

---

## 6. Task scheduling em escala (Gödel / ByteTask)

**ByteDance Gödel Scheduler**: open-source (CNCF/kubewharf), extensão do K8s scheduler. Unifica workloads online + offline. 2k+ pods/s single shard, 5k+ pods/s multi-shard. Maior cluster: 20k+ nós, 1M+ pods. 30x throughput vs. K8s scheduler nativo.

**Aplicável ao master-sindico**: **overkill hoje**, inspiração pra futuro.

- **Hoje (M1-M2)**: jobs batch pequenos (reprocessar score de reputação, cleanup de dados LGPD, envio de emails). Kubernetes CronJob ou um simples scheduler Go (ex: `robfig/cron`) resolve.
- **Quando reconsiderar**: se tivermos centenas de tenants, cada um com jobs customizados (ex: agendar votação recorrente, backup noturno por condomínio), aí um scheduler próprio com isolamento por tenant faz sentido. Mas Temporal.io / river-queue cobrem esse gap bem antes de precisar algo como Gödel.
- **Lição conceitual aplicável**: **two-tier scheduling (Unit + Pod)** — agrupar tasks correlatas em "unit" que precisa rodar atomicamente. Útil em sagas/workflows (ex: envio de convocação de assembleia envolve email + push + WhatsApp, todos precisam completar ou compensar).

**Referências**: [[source-06-godel-scheduler]].

---

## 7. Trust & Safety em plataforma social

**TikTok**: times globais de "milhares de safety professionals"; 96% automated takedown pre-view (2024); AI moderation + appeals humanos; policies por categoria (integrity, authenticity, minors, hate). LLMs multimodais desde 2024.

**Aplicável ao master-sindico**:

- **Sub-produto Banco de Talentos** (vídeo-currículo morador): precisa moderação leve — anti-nudity, anti-info sensível (CPF exposto, endereço completo), anti-linguagem imprópria. Volume mensal pequeno (<500) → moderação híbrida humana + OpenAI Moderation + classificação LLM cobre.
- **Connect Me** (marketplace empresas): review de cadastro (CNPJ ativo via API Receita, docs fiscais) > moderação de conteúdo. Problema é mais fraude que toxic speech.
- **Vizinhança** (social vizinho-vizinho — futuro): esse sim vai precisar moderação séria. Padrões TikTok aplicáveis: bank de golden seeds, cascade filtering, appeal obrigatório.
- **LGPD Art. 20**: **todo modelo de moderação automatizada precisa appeal humano**. Não-negociável no Brasil.

**Referências**: [[source-05-content-moderation]], [[source-08-architecture-overview]].

---

## 8. Transparência algorítmica e debate público

**Estado atual**:

- TikTok abriu **Transparency and Accountability Centers** (Dublin, LA, DC, Singapura) — visitantes veem source code e fluxo do algoritmo FYP sob NDA.
- **Dedicated Transparency Centers (DTC)**: Oracle + inspetores independentes têm acesso full-time ao source code (Projeto Texas).
- **EU DSA (fevereiro 2026)**: Comissão Europeia **achou TikTok preliminarmente em violação** por "design viciante" — infinite scroll, autoplay, push notifications persistentes, recsys altamente personalizado. Multa possível: até 6% do faturamento global anual.
- **Debate**: críticos argumentam que "transparência" via TAC é teatro (NDA impede publicação); algoritmo real ainda é caixa-preta.

**Aplicável ao master-sindico**:

- **Não somos plataforma de massa** — regulação DSA não nos atinge (limiar 45M usuários na UE). Mas vem **marco regulatório brasileiro similar** (PL 2.630, discussão em 2025-2026). Melhor já operar alinhado.
- **Decisões automatizadas que precisam de transparência**:
  - Ranking de empresas no Connect Me (por que essa empresa apareceu primeiro pro síndico?).
  - Moderação de conteúdo em Banco de Talentos (por que meu vídeo foi rejeitado?).
  - Score de reputação de síndico (como é calculado?).
- **Padrão canônico**: **cada decisão automatizada expõe os top-K fatores** (feature importance) pro usuário final, e tem **appeal humano**.
- **LGPD Art. 20 alinhado**: "direito à revisão por pessoa natural de decisões tomadas unicamente com base em tratamento automatizado".
- **Evitar dark patterns DSA-style**: não usar infinite scroll viciante no feed de vídeos do master-sindico. Preferir listas paginadas + "carregar mais" explícito. Especialmente relevante porque síndico é audiência adulta/profissional, não quer experiência "TikTok-ificada".

**Referências**: [[source-07-transparency-dsa]].

---

## Resumo executivo — 1 página

| Área TikTok | Nosso equivalente | Aplicável M1 | Aplicável M3+ | Overkill |
|---|---|---|---|---|
| Monolith recsys | Feed descoberta Connect Me | Não (ranking por regras) | Sim (quando >1k empresas) | Parameter server, Flink |
| Vídeo short-form pipeline | Banco Talentos (Mux) | Mux entrega 100% | Manter Mux | CDN próprio, PCDN, GPU edge |
| Moderação multi-idioma | Moderação BR-PT (LLM cascade) | 2-stage simples (heur+LLM) | Refinar golden seed bank | MLLM próprio, LLaVA fine-tune |
| ByteGraph | Postgres + recursive CTE | Postgres | Avaliar Neo4j se Vizinhança escalar | Graph DB in-house |
| TikTok LIVE | LiveKit (assembleias M3) | — | LiveKit + Mux Live (broadcast) | Infra LIVE própria |
| Gödel scheduler | K8s CronJob / Temporal | CronJob | Temporal | Scheduler custom |
| Trust & Safety | Moderação + LGPD Art. 20 | OpenAI Mod + Claude + appeal humano | Escalar com Vizinhança | Times globais de milhares |
| Transparência algorítmica | Explicabilidade + appeal | Top-K features + appeal | Alinhar a PL 2.630 quando sair | TAC físico |

---

## Links

- [[_moc]] — MOC da pasta
- [[../../02-architecture/research-inspirations]] — destino na migração (Fase 6)
- [[../instagram/_destilado]] — compare com Instagram (social feed paralelo)
- [[../youtube/_destilado]] — compare com YouTube (vídeo long-form)
- [[../netflix/_destilado]] — compare com Netflix (streaming video + recsys)
- Plano: `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`
