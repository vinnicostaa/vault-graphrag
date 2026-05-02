---
title: "Source 03 — Google Meet Codecs (VP9 SVC, AV1) — webrtcHacks"
type: research-source
tags: [research, master-sindico, v2, webrtc, google-meet, codecs, vp9, av1, svc]
created: 2026-04-23
updated: 2026-04-23
source_type: engineering-blog
source_org: "webrtcHacks"
source_urls:
  - "https://webrtchacks.com/the-hidden-av1-gift-in-google-meet/"
  - "https://developer.chrome.com/blog/av1"
  - "https://developers.google.com/workspace/meet/media-api/guides/codecs"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 03 — Google Meet Codecs (VP9 SVC + AV1)

## Resumo

Análise da webrtcHacks (blog técnico referência em WebRTC, mantido por engenheiros da área) sobre como Google Meet escolhe codecs em produção. Complementado com docs oficiais Google pra Meet Media API e Chrome AV1.

## Pontos-chave

- **VP9 com SVC** (modo `L3T3_KEY` = 3 camadas espaciais + 3 temporais) é o padrão pra webcam quando há 2+ participantes. SVC permite SFU escolher qual camada mandar pra cada subscriber → economiza ~40–60% do upload do publisher vs simulcast VP8.
- **VP8 simulcast** continua em screen-share (RTCPeerConnection separado). Tab/screen sharing não migrou pra VP9 ainda.
- **AV1** está em uso experimental: Meet liga AV1 só em fase pré-call (usuário sozinho), pra testar encoder e fazer estimativa de banda sem arriscar sessão real.
- AV1 tem "screen content coding" que reduz 25%+ em conteúdo de tela; 40 kbps foi testado em condições extremas.
- VP9 SVC força decode em software (hardware decode não suporta SVC) — trade-off com CPU cliente.

## Aplicação ao Master Síndico

- Em assembleia com 200 passivos só vendo slide do síndico, **VP9 SVC no publisher do moderador** é ganho enorme. LiveKit suporta VP9 simulcast/SVC via SDK.
- Mobile BR (maioria Android): maioria aparelhos recentes decoda VP9 em HW; decode em SW de SVC pesa em dispositivos antigos. Monitorar.
- AV1 não vale ainda em 2026 pra nosso caso — encoder pesado demais no publisher mobile.
- **Estratégia recomendada M1**: VP8 simulcast baseline; M2 ativar VP9 SVC opt-in no host da assembleia.

## Citações relevantes

> "Google Meet uses VP9 Scalable Video Coding (SVC) in production, with L3T3_KEY and similar modes."

> "VP9's SVC capability reduces publisher upload bandwidth by 40–60% compared to VP8 simulcast in multi-party conferences."

## URL(s)

- https://webrtchacks.com/the-hidden-av1-gift-in-google-meet/
- https://developer.chrome.com/blog/av1
- https://developers.google.com/workspace/meet/media-api/guides/codecs
