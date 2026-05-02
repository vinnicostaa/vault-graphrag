---
title: "Source 08 — LiveKit Egress vs Mux (gravação pra ata)"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, egress, gravacao, ata, mux]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "LiveKit, Mux"
source_urls:
  - "https://docs.livekit.io/home/egress/examples/"
  - "https://docs.livekit.io/home/egress/outputs/"
  - "https://github.com/livekit/egress"
  - "https://livekit.com/blog/livekit-universal-egress-launch"
  - "https://www.mux.com/docs/pricing/video"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 08 — LiveKit Egress + comparação Mux

## Resumo

LiveKit Egress é o serviço de gravação/streaming out-of-band (pra não impactar SFU). Oferece Room Composite, Track Composite, Track Egress, Web Egress e Participant Egress. Mux Real-Time é alternativa pagando mais por CDN que nosso caso não demanda.

## Pontos-chave LiveKit Egress

### Tipos

1. **Room Composite** — abre Chrome headless, carrega template HTML, grava layout inteiro (grid, speaker-dark/light) como MP4/HLS.
2. **Track Composite** — combina áudio+vídeo de 1 participante sincronizados.
3. **Track Egress** — track raw, sem transcode. Mais barato ($0.001/min overage).
4. **Web Egress** — grava qualquer URL.
5. **Participant Egress** — todos os tracks de um participante com encode custom.

### Outputs

- Cloud storage: S3, GCS, Azure Blob.
- Streaming: RTMP, SRT, HLS (com playlist "live" pra transmissão).
- Formatos: MP4, HLS segmentado.
- Thumbnails periódicos.

### Arquitetura

> "The overriding design goal of the egress system was to keep load off the SFU. Under no circumstances could we impact real-time audio and video performance or quality."

Egress roda fora do SFU; comunica via Redis. Templates HTML customizáveis via URL param.

### Custo LiveKit Cloud

- Video transcode: **$0.02/min** (overage). Audio-only: $0.005/min. Track egress raw: $0.001/min.
- Bandwidth downstream: $0.10–0.12/GB (depende tier).

## Mux Real-Time

- Pricing 3-dimensional: input (min upload) + storage (min armazenado) + delivery (min entregue).
- Pontos fortes: player pronto, CDN global.
- Pontos fracos pra nosso caso: CDN sobra (assembleia é privada), dois vendors a gerir, DX duplicado, migração do LiveKit (que já temos) custa dev-week.

## Aplicação ao Master Síndico

### Requisito legal BR

- Código Civil art. 1.354 + art. 1.350 + convenção condominial: ata é o documento; gravação é anexo probatório. Não exige qualidade broadcast.
- LGPD: gravação contém biometria facial/vocal → base legal = execução de obrigação decorrente da convenção + interesse legítimo do condomínio. Consent opt-in reforça (botão "estou ciente").

### Decisão M1

- **Primário**: Room Composite layout grid → MP4 → S3 privado (bucket por tenant).
- **Backup áudio**: Track Egress de áudio-only do síndico moderador → MP3 → mesmo S3. Custo marginal ($0.001/min), redundância barata.
- **Template custom**: criar `master-sindico/assembleia-grid.html` com overlay de ata (nome da pauta atual, votação em tempo real sobre o vídeo). Facilita revisão posterior.
- **Retenção**: 5 anos (prazo prescricional de cobranças condominiais) em storage frio (S3 Glacier Instant).

### Quando reavaliar Mux?

- Se adicionarmos live-streaming pra grande público passivo (prestação de contas anual pra todos os moradores com latência importante). Não é cenário M1–M3.

## Citações relevantes

> "Room Composite [...] open an instance of Chrome, load the selected template as a web page and connect to the specified LiveKit room (as a hidden participant) via JavaScript."

> "Track Egress is for when you need a copy of raw track data. Since there's no transcoding or muxing, this is the most computationally efficient option."

## URL(s)

- https://docs.livekit.io/home/egress/examples/
- https://docs.livekit.io/home/egress/outputs/
- https://github.com/livekit/egress
- https://livekit.com/blog/livekit-universal-egress-launch
- https://www.mux.com/docs/pricing/video
