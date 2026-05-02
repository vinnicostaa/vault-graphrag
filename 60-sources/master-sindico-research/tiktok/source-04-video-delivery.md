---
title: Source — ByteDance short-video playback infra (arXiv 2410.17073)
type: source
tags: [research, tiktok, bytedance, video, cdn, pcdn, transcoding, codecs]
source: https://arxiv.org/html/2410.17073v1
created: 2026-04-23
updated: 2026-04-23
aliases: [personalized-playback-paper]
---

# Source — Personalized Playback Technology: How Short Video Services Create Excellent User Experience

## Metadados

- **URL**: https://arxiv.org/html/2410.17073v1
- **Autores**: ByteDance research team (2024)
- **Autoridade**: alta — paper técnico da ByteDance descrevendo infra de playback real.

## Componentes end-to-end do pipeline de vídeo short-form

A ByteDance descreve pipeline com os seguintes componentes:

1. **Metadata service** — ingestão de metadados de vídeo (uploader, hashtags, duração).
2. **Upload service** — recebe o vídeo do device do criador.
3. **Transcoding** — converte pra múltiplas resoluções e codecs.
4. **Delivery** — CDN + PCDN (Peer CDN).
5. **Player SDK** — player custom no app que coordena chunks, buffer, fallback.
6. **Analytics dashboard** — telemetria de QoE (Quality of Experience).

## CDN + PCDN híbrido

- Tráfego CDN tradicional atende a maior parte da demanda.
- **PCDN (Peer CDN)** usa "edge resources menos performantes" — **routers domésticos, pequenos servidores** — pra servir chunks a usuários próximos.
- Player SDK coordena: divide vídeo em chunks e atribui **cada chunk** a CDN ou PCDN, otimizando custo + latência.

### Vantagens de PCDN
- **Custo**: recursos de edge são ordens de grandeza mais baratos que CDN comercial.
- **Latência**: recursos mais próximos geograficamente do usuário.

### Server-side do PCDN
- **Tracker service** — indexa qual conteúdo está em qual edge node.
- **Log service** — coleta estatísticas de cada download (user info, node info).

## Chunking e buffer

- Vídeos divididos em múltiplos chunks.
- Chunks baixados ficam em **data buffer do player** (estratégia pull-based).
- Player aplica **task-level scheduling**: cada chunk = uma task, atribuída a CDN ou PCDN.

## Codecs usados (consenso de fontes sobre TikTok)

- **H.264 (AVC)** — recomendado pra upload por criadores (compatibilidade máxima).
- **H.265 (HEVC)** — resoluções altas, devices recentes.
- **AV1** — devices mais novos; melhor compressão, livre de royalties.
- **H.266/VVC** — testes; ~50% menos bandwidth que AV1 (citação de análise secundária, verificar com cautela).

## Transcoding e availability

- **Priorização 720p** — primeira resolução disponível após upload (usuário vê vídeo ~3-5s depois de publicar).
- Full resolution processada em background.
- GPU edge-nodes usados pra transcoding real-time (fonte secundária, plausível mas não confirmado no paper).

## Aplicabilidade ao master-sindico (Banco de Talentos + vídeos empresa/síndico)

### Decisão canônica M1-M2: **Mux cobre 100% do caso de uso**

- Upload direto (Mux Direct Uploads): upload do device → Mux, sem passar pelo nosso backend.
- Transcoding automático multi-resolução (360p-1080p).
- Adaptive bitrate HLS + DASH out of the box.
- Signed URLs pra privacy (vídeo-currículo não é público).
- Player SDK (Mux Player) ou HLS.js.
- Thumbnail + animated GIF preview auto-gerados.
- Analytics (QoE metrics): bitrate, buffer, rebuffer events.

### Custos comparativos

- **Mux pricing (2026)**: ~US$ 0.04/min de vídeo armazenado + US$ 0.001/min de streaming.
- Para M1 (~500 vídeos de 90s = 750 min de armazenamento + ~5000 views/mês = ~450 min streaming): **< US$ 50/mês**.
- Racional econômico: só reconsiderar infra própria se custo Mux passar US$ 3k/mês (dezenas de milhares de vídeos ativos + centenas de milhares de views).

### Lições da arquitetura ByteDance aplicáveis

- **Priorizar resolução baixa pra disponibilidade imediata** — Mux já faz (ready event antes de todas resoluções completarem).
- **Chunking + adaptive bitrate** — HLS padrão, Mux entrega.
- **CDN próprio + PCDN**: overkill. Mux usa Fastly/Cloudflare; cobre.

### Parâmetros a fixar na spec do Banco de Talentos

- **Duração máxima**: 90s (paridade com TikTok).
- **Formato**: vertical 9:16 preferencial (consumo mobile do síndico/empresa avaliando candidato).
- **Resoluções oferecidas**: 360p, 540p, 720p, 1080p.
- **Codec**: H.264 + H.265 (AV1 ainda não universalmente suportado em players web).
- **Upload size limit**: 200MB (cobre 90s em 4K).
- **Retry/resume**: obrigatório (condomínio BR frequentemente em 4G instável).

## Fontes complementares

- [Async Thinking — TikTok architecture secrets](https://asyncthinking.com/p/tiktok-architecture-secrets) — cobertura adicional (números dali são estimativas, tratar com cautela).
- [USENIX ATC 2024 — Zhang et al.](https://www.usenix.org/system/files/atc24-zhang-rui-xiao.pdf) — paper correlato sobre video delivery ByteDance.
- [Dev.to — Designing TikTok short video architecture](https://dev.to/matt_frank_usa/designing-tiktok-short-video-platform-architecture-401m).

## Links

- [[_destilado#2. Vídeo short-form 90s — paridade com Banco de Talentos]]
- [[_moc]]
