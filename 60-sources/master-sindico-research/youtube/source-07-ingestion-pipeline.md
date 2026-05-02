---
title: "Source 07 — YouTube video ingestion pipeline (upload → transcode → storage)"
type: source
tags: [research, youtube, ingestion, transcoding, vcu, pipeline, bigtable, gcs]
source: https://bytebytego.com/guides/how-does-youtube-handle-massive-video-content-upload/
created: 2026-04-23
updated: 2026-04-23
aliases: [youtube-ingestion-pipeline]
---

# Source 07 — YouTube ingestion pipeline

## Metadados

- **Fontes** (engineering digests — primary sources do YouTube são fechadas):
  - https://bytebytego.com/guides/how-does-youtube-handle-massive-video-content-upload/
  - https://www.vdocipher.com/blog/youtube-tech-stack-architecture/
  - https://sderay.com/youtube-video-processing-architecture-how-video-goes-from-upload-to-play/
- **Papers oficiais Google sobre VCU**:
  - "Warehouse-scale video acceleration: Co-design and deployment in the wild" (Google, ISCA 2021).

## Escala declarada

- **500+ horas de vídeo upload por minuto** em média (YouTube, 2022).
- Catálogo em exabytes.

## Pipeline (síntese multi-source)

```
[Creator] → chunked HTTPS upload (resumable) → edge server mais próximo
   ↓
[GCS (Google Cloud Storage) staging] 
   ↓
[Pub/Sub distributed queue] → publish "new upload" event
   ↓
[Transcoding workers (VCU clusters)] — pick up job via DAG scheduler
   ↓
[Split em chunks paralelos] (vídeo 2GB → 100-200 segments processados em paralelo)
   ↓
[Gera múltiplas renditions] 144p → 8K, múltiplos codecs (H.264 / VP9 / AV1)
   ↓
[Merge + manifest generation (DASH/HLS)]
   ↓
[Metadata → Bigtable] (playback ID, durations, resolutions, thumbnails)
   ↓
[Storage final (Colossus / GCS blobs)]
   ↓
[CDN warm-up pra edge]
```

## Componentes-chave

- **Resumable upload HTTPS**: retoma em caso de falha de rede (vital pro user móvel).
- **Edge ingest**: chunks vão pro POP mais próximo — reduz latência de upload.
- **DAG execution**: pipeline é um grafo de tasks (transcode rendition N depende de extract audio, gerar thumb depende de extract frame, etc.). Maximiza paralelismo.
- **VCU (Video transCoding Unit)**: ASIC custom do Google, 20-33x mais eficiente que software encoding (FFmpeg CPU-only).
- **Bigtable**: metadata (playback ID, manifest URLs, ownership, analytics).
- **Colossus**: file system distribuído interno do Google pra storage bruto.

## Decisões arquiteturais notáveis

1. **Immutable pipeline** — transcode nunca muta arquivo de entrada; só gera outputs.
2. **Stateless workers** — VCU workers não têm state; pegam job da queue, processam, empurram resultado.
3. **Failure isolation** — falha em 1 chunk reprocessa só aquele chunk, não o vídeo inteiro.
4. **Backpressure via queue depth** — Pub/Sub inspeciona depth; if queue cresce, scheduler escala workers.
5. **Warm CDN pós-processing** — manifest publica só depois de copia nos edges populares.

## Relevância para Master Síndico

### Arquitetura Mux espelha isto (terceirização)

Mux é o YouTube ingestion pipeline **como serviço**. O que Mux faz por nós:

| Função YouTube | Mux equivalente |
|----------------|-----------------|
| Chunked upload edge | `mux.createDirectUpload()` — signed URL pré-assinada pra cliente |
| GCS staging | Mux storage interno |
| Pub/Sub + workers | Mux engine (opaque) |
| VCU transcode | Mux transcoding (cpu/gpu opaque pro cliente) |
| Bigtable metadata | Mux API (`assets/{id}`) |
| Manifest HLS | Mux entrega `/playback/{id}.m3u8` |
| CDN warm | Mux multi-CDN |
| Webhooks | `video.asset.ready` evento pro nosso backend |

### O que aplicar (mesmo terceirizado)

1. **Event-driven pós-upload** — webhooks Mux (`asset.created`, `asset.ready`, `asset.errored`) entram em fila nossa (SQS/RabbitMQ) pra:
   - Trigger moderação.
   - Indexar no OpenSearch (título, descrição, transcript).
   - Atualizar Bigtable-equivalente nosso (Postgres): `video_assets` com playback_id, duration, status.
   - Emitir evento de domínio `VideoProcessed` → handlers downstream.
2. **Resumable upload no cliente** — usar o componente Mux Upload (suporta tus.io resumable) **sempre**. Condomínio brasileiro tem conexão instável; upload que não resuma é UX catastrófico.
3. **Moderação acontece entre `asset.ready` e `published`** — estado separado `pending_moderation`.
4. **Immutable asset** — alinha natural com **lock 90d**. Nosso modelo: asset.replace gera **new asset**, old mantém playback_id com redirect interno (não remove — compliance).
5. **Idempotência dos webhooks** — Mux retry; handler precisa `UPSERT BY webhook_id`.
6. **DLQ pra webhook handler** — se falhar reprocessar, vai pra dead-letter queue manual.

### Arquitetura event-driven que isto implica

```
[Cliente] → Mux Upload signed URL → Mux
                                      ↓
                                  (processa)
                                      ↓
                                  [Webhook Mux] → [Nosso /webhook/mux] 
                                                          ↓
                                                  [Validate signature]
                                                          ↓
                                                  [Enqueue event]
                                                          ↓
                           ┌──────────────┬──────────────┴──────────────┬─────────────┐
                           ↓              ↓                              ↓             ↓
                  [Moderation job]  [Index OpenSearch]         [Update Postgres]  [Notify owner]
```

## O que NÃO aplicar

- Custom VCU / transcoding próprio — zero ROI; Mux resolve.
- DAG executor custom (Airflow overkill) — webhooks + queue simples bastam pro nosso volume.
- Colossus / file system distribuído — S3/GCS bucket comum basta.

## Links internos

- [[source-05-mux-hls-best-practices]]
- [[source-03-media-cdn]]
- [[_destilado]]
- [[../_moc]]
