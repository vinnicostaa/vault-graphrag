---
title: MOC — Mux (Video on-demand + Live streaming)
type: moc
tags: [moc, provider, mux, video, streaming, live]
category: video
mcp: null
sdk-official-go: mux-go (github.com/muxinc/mux-go)
sdk-official-node: "@mux/mux-node"
doc-oficial: https://docs.mux.com
doc-consulted: 2026-04-23
created: 2026-04-22
updated: 2026-04-23
---

# Mux

**Categoria**: Video streaming (VOD + Live) · encoding · analytics · player.

**MCP disponível**: ❌ — usar WebFetch em https://docs.mux.com ou Context7 `resolve-library-id "mux"`.

**SDKs oficiais**:
- Go: `github.com/muxinc/mux-go`
- Node: `@mux/mux-node`
- Python: `mux-python`
- Mux Player: `@mux/mux-player` (web) · `mux_video_player` (Flutter community)

**Doc oficial**: https://docs.mux.com — consultada em 2026-04-23.

**Sandbox**: Chaves `free` (modo dev) — limite de minutos mensais; acima migra pra `pay-as-you-go`.

**Modelo de preço**: por minuto de encoding/storage/delivery. ~$0.005/min encoding + $0.0016/min delivery.

---

## Quando usar Mux

1. **Live streaming de alta qualidade** (RTMP ingest → HLS delivery) — concorrente: AWS IVS, Livepeer
2. **Video on-demand com analytics embutido** (Mux Data) — vê heatmap de drop-off por segundo
3. **Simulcast** (RTMP pra múltiplos destinos)
4. **Thumbnail/preview automático**
5. **DRM / signed playback URLs** para conteúdo privado

## Quando NÃO usar Mux

- **Vídeos curtos public (TikTok-like)** → Cloudflare Stream é mais barato
- **Tudo dentro da AWS** → IVS (live) + MediaConvert (VOD) casa melhor com resto
- **Self-hosted requerido** → Livepeer ou SRS self-host

---

## Sub-docs do Mux

- [[overview]] — arquitetura (Asset → Playback ID → Delivery URL)
- [[integration-guide]] — upload direto + webhook + Mux Player
- [[patterns]] — signed playback, webhook lifecycle (asset.created → asset.ready), simulcast
- [[antipatterns]] — ex: não usar playback_id como identificador primário no DB
- [[usage-in-projects]] — quais projetos usam

## Glossário interno do Mux

- **Asset** — representação interna do vídeo (uuid Mux); tem múltiplos Playback IDs.
- **Playback ID** ≠ Asset ID. Playback ID é público (vai na URL HLS); Asset ID é privado.
- **Live Stream** — endpoint RTMP + playback gerado automaticamente; fica em `idle` → `active` → `idle`.
- **Signed Playback** — JWT com `sub: <playback_id>` + `aud: "v"` + HMAC SHA256 com key do Mux dashboard. Obrigatório pra conteúdo privado.
- **Simulcast Target** — destino adicional (RTMP) onde Mux reenvia a live (YouTube, Twitch, etc.).

## Fontes brutas (pendente destilar — fora do vault)

Material-fonte bruto preservado em filesystem (`_source-material/`, não indexado pelo Obsidian):

- `_source-material/2026-04-23-raw-source/from-master-sindico/contextos/MUX_VIDEO_CONTEXT.md` — 33K de contexto do Master Síndico (assembleia live via Mux RTMP + playback signed)

Path absoluto: `/home/vinni/Documentos/Projetos/Privados/automation-agents/_source-material/2026-04-23-raw-source/from-master-sindico/contextos/`

> **Estado**: destilação pendente. Fonte tem cenário completo de live streaming de assembleia condominial. Agente acessa via Read/Bash no filesystem. Aplicação concreta do Mux no projeto: [[../../../50-projects/master-sindico/01-domain/bounded-contexts|bounded context Assembly]].

## Vizinhos no grafo

- [[../_moc]]
- [[../livekit/_moc]] — concorrente pra realtime peer-to-peer (WebRTC); Mux é broadcast-scale
- [[../../security/_moc]] — signed playback = assinatura tokenizada
