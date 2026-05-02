---
title: "Source 03 — Google Cloud Media CDN (infra de edge do YouTube exposta como produto)"
type: source
tags: [research, youtube, cdn, media-cdn, edge-network, google-cloud, delivery]
source: https://cloud.google.com/media-cdn
created: 2026-04-23
updated: 2026-04-23
aliases: [media-cdn, google-edge]
---

# Source 03 — Google Cloud Media CDN

## Metadados

- **URL oficial**: https://cloud.google.com/media-cdn
- **Docs**: https://docs.cloud.google.com/media-cdn/docs/overview
- **Blog announcement**: https://cloud.google.com/blog/products/networking/media-cdn-and-trends-in-content-delivery/
- **Lançamento**: 2022 (GA); baseado na infra usada pelo YouTube

## O que é

CDN de mídia **construído sobre a mesma edge network que o YouTube usa**. Google expôs como produto pra clientes externos fazerem VoD/live streaming em escala.

## Escala

- **3.179+ localizações edge** globais (declaração Google).
- **206+ países/territórios**, **1300+ cidades**.
- **100+ Tbps** de egress capacity (reportado DCD).
- Backbone privado Google (non-internet) entre regiões.

## Componentes

- **Edge Cache Service** — endpoint público (domain + TLS).
- **Edge Cache Origin** — referência ao origin HTTP(S) onde está a mídia.
- **Edge Cache Route** — roteamento de path → origin.

## Protocolos e performance

- **HTTP/3 + QUIC** client→edge (out-of-the-box).
- **TLS 1.3** default.
- **BBR** transport (Google congestion control) entre edge e backbone.
- **HTTP/2** suportado.
- Adaptação automática de protocolo por user agent / network conditions.
- Max segment size **25 MiB** (suporta 4K/8K).

## Features pra streaming

- **Flexible Shielding** (origin shield regional): cache intermediário reduz hit no origin. Disponível South Africa, Middle East, US.
- **Multi-part range requests**: HEAD + range pra chunks.
- **Signed URLs / signed cookies**: auth no edge.
- **Monitoring-as-a-Service**: dashboard broadcast-grade pra live events (origin health + QoE do user final).
- **Monthly savings plans**: pricing commit vs pay-as-you-go.

## Relevância para Master Síndico

### Aplicável? **Não diretamente — Mux resolve isto.**

- Mux já entrega HLS via sua própria infra multi-CDN (Fastly + Cloudflare + Akamai under the hood).
- Master-sindico **não consome Media CDN**.

### Lição arquitetural aplicável

1. **Edge-first delivery é requisito**, não nice-to-have. Mux já faz — confirma a decisão de terceirizar.
2. **QUIC/HTTP3 é baseline em 2026**, não opcional. Validar que Mux Player habilita HTTP/3 (é default desde 2024).
3. **Signed URLs no edge** — padrão que master-sindico precisa usar pra lock 90d + tenant isolation. Mux entrega via **signed playback URLs** (JWT assinado). Padrão: backend emite JWT curto (TTL 5-15min), player renova on demand.
4. **Origin shield** — conceito irrelevante pra Mux (a própria Mux é o "origin" pra nós).
5. **Monitoring de QoE** — Mux Data (produto separado da Mux) entrega metrics similares (startup time, rebuffer ratio, bitrate distribution). Habilitar pra sub-produto Conteúdo/Vídeos + Assembleias Live.

## O que NÃO aplicar

- Não operar CDN próprio. Mux + pricing incluso >> complexidade de rodar edge network.
- Não usar Media CDN direto antes de justificar custo (Mux bundle é mais barato em volume MVP).

## Links internos

- [[source-05-mux-hls-best-practices]]
- [[_destilado]]
- [[../_moc]]
