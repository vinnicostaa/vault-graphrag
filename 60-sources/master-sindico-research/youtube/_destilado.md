---
title: "Destilado — arquitetura YouTube aplicada ao Master Síndico"
type: note
tags: [research, youtube, destilado, master-sindico, video, recsys, moderation, cdn]
source: 13-research/youtube/source-01..08
created: 2026-04-23
updated: 2026-04-23
aliases: [youtube-destilado]
---

# Destilado — YouTube aplicado ao Master Síndico

> Síntese de 8 fontes oficiais/engineering (Google Research, cloud.google.com, developers.google.com, support.google.com, aws.amazon.com, mux.com, vitess.io, bytebytego). Cada afirmação está ancorada em `source-0X.md` desta pasta.

## TL;DR de decisões propostas

| Tópico | Decisão proposta | Fonte |
|---|---|---|
| Ingestion video | **Mux resolve tudo** (upload → transcode → HLS → CDN). Nosso trabalho: event handler dos webhooks. | [[source-07-ingestion-pipeline]] |
| Player | **Mux Player** (`<mux-player>`). Não rodar hls.js custom. | [[source-05-mux-hls-best-practices]] |
| Codecs | **H.264 AVC baseline**, habilitar **H.265** opt-in quando volume justificar (reduz custo banda Mux). AV1 só premium tier. Nunca AV1-only. | [[source-05-mux-hls-best-practices]] |
| ABR ladder | 4 tiers: **360p / 480p / 720p / 1080p**. 240p desnecessário (banda brasileira residencial suporta); 4K injustificado (vídeo institucional condominial). | [[source-05-mux-hls-best-practices]] |
| Segment / keyframe | **6s segment / 2s keyframe** — default Mux, sem tuning. | [[source-05-mux-hls-best-practices]] |
| DRM | **NÃO no MVP**. Signed URLs JWT (Mux signed playback) + lock 90d bastam. | [[source-05-mux-hls-best-practices]] |
| Moderação hoje | **Manual 48h** — mantém. Volume MVP (15-60 min/mês) está abaixo do limiar de custo-benefício ML. | [[source-06-rekognition-moderation]] |
| Moderação futura | Gatilho: ≥300 min/mês OU backlog manual > 48h. Primeira iteração: **AWS Rekognition Video** ($0.10/min) + **OpenAI Moderation** pra transcript (grátis). Custom classifier só fase 3. | [[source-06-rekognition-moderation]] |
| Copyright detection | **NÃO construir**. Cláusula contratual + moderação manual cobrem. Fingerprint SaaS (AudD/ACRCloud) só se problema emergir. | [[source-02-content-id]] |
| Recommendation Connect Me | **Two-stage simplificado**: retrieval determinístico (filtro região+categoria) + ranking heurístico (score composto). **Zero ML** no MVP. Otimizar **proxy certo** (contato > click). | [[source-01-deep-nn-recommendations]] |
| Assembleias Live | **WebRTC SFU** (LiveKit/Daily) pra call bidirecional; Mux Live só pra broadcast monológico. Gravação → VoD asset → lock 90d. | [[source-08-live-streaming]] |
| DB scaling | Postgres single + read replica + pgbouncer basta por anos. **Modelar tenant_id em toda tabela** desde dia 1 (sharding key futura). Vitess/Citus só >10k QPS sustained. | [[source-04-vitess-mysql]] |
| CDN | **Não operar própria**. Mux bundle cobre; validar que HTTP/3 está habilitado no player. | [[source-03-media-cdn]] |

## 1. Pipeline de vídeo: Mux como YouTube-as-a-Service

### O que YouTube faz (fonte 07)

Chunked resumable upload → GCS staging → Pub/Sub queue → VCU (ASIC transcoding, 20-33x mais eficiente que software) processando chunks paralelos → Bigtable metadata → Colossus storage → CDN warm.

### O que nós fazemos

Master-sindico **terceiriza tudo até o manifest HLS**. Nosso escopo fica em:

1. **Cliente**: componente Mux Upload (tus.io resumable) — obrigatório pra UX Brasil (conexões instáveis).
2. **Backend**: endpoint que emite signed Direct Upload URL via API Mux (`mux.createDirectUpload()`).
3. **Webhook handler** (`/webhook/mux`):
   - Valida assinatura HMAC Mux.
   - Idempotência (`UPSERT BY mux_webhook_id`).
   - Enfileira evento (SQS/RabbitMQ) → workers:
     - Indexar no OpenSearch (título, descrição, transcript auto-caption).
     - UPSERT em `video_assets` (playback_id, duration, codec, resolution, status).
     - Trigger moderação.
     - Emitir evento domínio `VideoProcessed` pros outros BCs.
4. **DLQ** obrigatória pra webhook que falhar ≥N retries.

### Lições "YouTube-style" a adotar

- **Event-driven, não sync** — pipeline é async por natureza; forçar sync em UI é bug de UX (vídeo "travado em processamento" → mostrar progress).
- **Immutable assets** — replace gera new playback_id, old fica com redirect. Alinha com **lock 90d**.
- **Idempotência em tudo** — webhooks Mux retry; handler precisa ser safe to repeat.
- **Stateless workers** — workers pegam job, processam, soltam.

## 2. ABR / HLS / codecs — Mux entrega, a gente configura bem

Decisão por default Mux (source 05) resolve 95% dos casos. Pontos específicos:

- **Nunca MP4 raw** em produção — sem ABR, vídeo quebra em rede móvel.
- **Signed playback URL JWT** — TTL 5-15min, player renova. Isto serve pra **tenant isolation** (cada JWT carrega tenant_id + user_id) e indiretamente pra lock 90d (acesso só por via autenticada permite auditoria).
- **Captions auto**: habilitar — acessibilidade + base pra moderação de áudio (transcript → OpenAI Moderation).
- **Mux Data (QoE)**: habilitar desde dia 1 — startup time, rebuffer ratio, bitrate distribution. SRE pro sub-produto Vídeos.

## 3. Moderação: manual hoje, ML quando?

### Princípio central

**Moderação é síncrona ao publish** (ou async com estado `pending_moderation` que bloqueia visibilidade). Porque o **lock 90d** torna erro irreversível → nunca autopublish sem checkpoint de moderação.

### Trilha proposta

| Fase | Volume mensal | Stack | Custo est./mês |
|------|---------------|-------|----------------|
| M1 (hoje) | <100 min | Manual 48h SLA | $0 + 2h/semana humano |
| Fase 2 (ano 1) | 100-1000 min | Rekognition Video + OpenAI Moderation (transcript) + fila manual pra casos mid-confidence | $10-100 |
| Fase 3 (ano 2+) | >1000 min | Idem + custom classifier treinado em dados próprios | $100-1000 + ML-ops |

### Arquitetura Fase 2 (sketch)

```
[Upload Mux] → asset.ready webhook
    ↓
[Moderation orchestrator]
    ├─→ Rekognition Video API ($0.10/min) → score {nudity, violence, hate}
    ├─→ Mux caption auto → OpenAI Moderation (texto, grátis) → flags
    └─→ ensembler
            ↓
        decision:
          score > 0.85 → bloqueia publish, fila manual PRIORITY
          0.20 < score < 0.85 → fila manual normal
          score < 0.20 → libera auto, moderação passiva (spot-check)
```

### Por que Rekognition e não Hive/custom

- Rekognition: API transparente, free tier útil, $0.10/min. Hive = enterprise contract, OpenAI não faz vídeo, custom = ML-ops injustificável em MVP.
- Ver [[source-06-rekognition-moderation]] pra comparativo completo.

## 4. Content ID / copyright — NÃO construir

YouTube investiu $100M+ em fingerprinting (source 02). Pra nós:

- Público-alvo (empresas subindo institucional) tem incentivo baixo pra piratear.
- **Risco real**: trilha sonora licenciada de fundo. **Solução**: moderador humano pega 90%.
- **Tech solution só se problema emergir**: AudD (~$10/mês low volume) pra fingerprint de áudio em spot-check.
- **Cláusula contratual** (termo de uso): empresa declara posse de direitos; violação = takedown + ban. Mais barato e eficaz.

**Lição arquitetural aplicável**: configurabilidade das ações (block/monetize/track) é bom padrão. Para master-sindico, rights holder hipotético deveria poder **block em tenant específico** sem derrubar em todos.

## 5. Recommendation: Connect Me two-stage, sem ML

### Insight do paper Covington 2016 (source 01)

- **Retrieval + Ranking** é o padrão robusto.
- **Otimize proxy certo**: YouTube escolheu expected watch time (não CTR) pra combater clickbait.

### Aplicação em Connect Me

- **Retrieval** (determinístico, zero ML):
  - Filtro por região do condomínio (raio ~20km).
  - Filtro por categoria solicitada (ex: "desentupidora").
  - Filtro por disponibilidade (empresa ativa no tenant).
- **Ranking** (heurístico, zero ML):
  ```
  score = w1 * rating_avg
        + w2 * recency_of_last_service
        + w3 * 1/distance_km
        + w4 * historical_success_rate
        + w5 * premium_tier_multiplier
  ```
- **Proxy a otimizar**: **taxa de "contato iniciado" + "serviço concluído"**, NÃO click no card. Lição direta do paper YouTube.

### Quando ML entra

- Quando tivermos **≥10k interações/mês** pra treinar.
- Mesmo aí, começar com **collaborative filtering clássico** (matrix factorization, ALS), não DNN.

## 6. DB scaling: Vitess não, mas aprender os truques

Vitess (source 04) é overkill. Mas as lições aplicam:

1. **Connection pooling dia 1** (pgbouncer).
2. **statement_timeout em todo statement** — query killer preventivo.
3. **tenant_id em toda tabela** como se fosse sharding key — abre caminho pra sharding físico futuro sem refactor.
4. **Online schema migrations** (pg-online-schema-change) — evitar lock longo.
5. **Metrics Prometheus desde dia 1** (postgres_exporter + pgbouncer_exporter).

## 7. Live streaming: Assembleias é WebRTC, não YouTube Live

YouTube Live (source 08) suporta RTMP/RTMPS/HLS/DASH ingest. **Irrelevante diretamente** pra nós porque **assembleia é bidirecional** (debate + voto), não broadcast.

- **WebRTC SFU** (LiveKit ou Daily) é o caminho — latência <500ms, bidirecional.
- **Gravação da assembleia** → VoD asset Mux → aplica lock 90d (ata oficial gravada).
- **Broadcast-only** (ex: síndico apresentando obra sem Q&A live): Mux Live RTMPS basta.

Lição aplicável: **separar ingest de delivery** — protocolos de entrada (RTMPS/WebRTC) não dependem de protocolo de saída (HLS). Permite trocar sem breaking change.

## 8. Lock 90d — o que é único, não copiar de ninguém

YouTube **não faz lock 90d**. Instagram/TikTok/Netflix também não. É requisito **regulatório/governança** específico do master-sindico (ata digital condominial, auditabilidade jurídica).

### Implicações arquiteturais (não vêm de research, são decisão nossa)

1. **Assets immutable** casa perfeitamente com lock 90d — Mux já é assim.
2. **Moderação pré-publish obrigatória** — pós-publish é tarde demais.
3. **Signed URLs com audit trail** — toda playback registrada (quem, quando, de onde) por 90d+.
4. **Workflow de "correção pós-lock"**: precisa processo formal (assinatura digital, nova votação, ou compliance/LGPD request). NÃO um botão de edit.

**Registro**: lock 90d é diferenciador do produto — não tentar espelhar YouTube (que prioriza freshness de content).

## 9. Anti-padrões explícitos (NÃO fazer)

- ❌ **Operar CDN própria** — Mux + Cloudflare/Fastly já feito.
- ❌ **Transcoding custom** — Mux custa menos que a conta AWS da sua ECS EC2 rodando FFmpeg.
- ❌ **Fingerprinting copyright próprio** — $100M do YouTube não é nosso budget.
- ❌ **DNN recommendation no MVP** — feature engineering + heurística + 6 meses de dados primeiro.
- ❌ **Vitess/Citus no MVP** — Postgres vanilla resolve por anos.
- ❌ **hls.js custom em vez de Mux Player** — perde Mux Data, aumenta bug surface.
- ❌ **Autopublish sem moderação** — lock 90d torna o erro irreversível.
- ❌ **Copiar "freshness feed" YouTube** — nosso modelo é "ata" (immutable) não "feed".

## Mapa das fontes

| Arquivo | Tema | Confiança fonte |
|---------|------|-----------------|
| [[source-01-deep-nn-recommendations]] | RecSys two-stage | **Paper oficial Google Research** |
| [[source-02-content-id]] | Copyright detection | **Doc oficial YouTube support** |
| [[source-03-media-cdn]] | Edge CDN | **Doc + blog oficial Google Cloud** |
| [[source-04-vitess-mysql]] | DB sharding | **Docs oficiais Vitess + CNCF** |
| [[source-05-mux-hls-best-practices]] | Player + HLS | **Mux eng blog** (primary for our stack) |
| [[source-06-rekognition-moderation]] | ML moderation | **AWS docs oficiais + comparativo 2026** |
| [[source-07-ingestion-pipeline]] | Upload pipeline | Engineering digests (primary YouTube fechado) |
| [[source-08-live-streaming]] | Live protocols | **Doc oficial YouTube Developers** |

## Links internos

- [[../_moc]]
- [[../../02-architecture/patterns]] (a preencher)
- [[../../03-subprojects/conteudo-videos]] (a preencher)
- [[../../03-subprojects/connect-me]] (a preencher)
- [[../../03-subprojects/assembleias-live]] (a preencher)
