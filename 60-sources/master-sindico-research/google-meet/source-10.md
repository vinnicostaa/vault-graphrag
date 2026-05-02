---
title: "Source 10 — WebRTC E2EE (Insertable Streams) — análise e decisão 'off'"
type: research-source
tags: [research, master-sindico, v2, webrtc, e2ee, insertable-streams, seguranca, adr]
created: 2026-04-23
updated: 2026-04-23
source_type: engineering-blog
source_org: "webrtcHacks, Mozilla, W3C"
source_urls:
  - "https://webrtchacks.com/true-end-to-end-encryption-with-webrtc-insertable-streams/"
  - "https://webrtchacks.com/end-to-end-encryption-in-webrtc-4-years-later/"
  - "https://blog.mozilla.org/webrtc/end-to-end-encrypt-webrtc-in-all-browsers/"
  - "https://webrtc.github.io/samples/src/content/insertable-streams/endtoend-encryption/"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 10 — E2EE com Insertable Streams

## Resumo

Análise técnica da W3C Encoded Transform API (antiga "Insertable Streams") pra E2EE real em WebRTC: cifragem frame-a-frame depois do encoder mas antes do packetizer, fazendo o SFU ver só bytes opacos.

## Pontos-chave

### Como funciona

- Hook na pipeline WebRTC **depois do encoder, antes do packetizer**. Opera em encoded frames, não em pacotes RTP.
- Vantagem vs cifrar por pacote: não precisa se preocupar com overhead por fragmentação; packetizer lida.
- SFU recebe frames cifrados — não consegue nem decodar nem gerar thumbnails nem simulcast layer switch inteligente.

### Overhead de banda

- **Áudio**: +12 bytes IV + 16 bytes auth tag por pacote → ~25% overhead (pacotes de áudio são pequenos).
- **Vídeo**: < 1% (frames de vídeo são grandes; overhead relativo irrisório).

### Limitações

- **Chromium-only** completo; Firefox parcial; Safari limitado.
- **Latência**: alguns ms extra por frame em JS/WASM.
- **Cold start**: usuário vê tela preta até key exchange completar. Se signaling de key atrasa vs media, UX quebra.
- **Quebra operações server-side**: gravação, transcrição, moderação automatizada — SFU não pode tocar no conteúdo.
- **Key management é responsabilidade da app**: perder a chave = perder o stream. Difícil de operar bem.

### Quando faz sentido

- Videoconferência **extremamente sensível**: telemedicina com HIPAA estrito + desconfiança do operador, advocacia, jornalismo investigativo, diplomacia.
- Requisito regulatório específico que veda operador ver conteúdo.

## Aplicação ao Master Síndico — decisão OFF

### Por que NÃO ativar E2EE

1. **Requisito de gravação conflitante**: ata legal BR exige anexo de gravação (art. 1.354 CC, interpretação majoritária). E2EE impede gravação server-side via Egress — teríamos que gravar client-side, que é frágil (crash do navegador perde prova).
2. **Modelo de ameaça não justifica**:
   - Não estamos defendendo contra LiveKit Cloud (parceiro com DPA/LGPD assinado; cliente contratual).
   - Estamos defendendo contra: interceptador de rede → SRTP+DTLS já resolve. Tenant hostil → RLS Postgres + JWT per-sala resolve.
3. **UX**: Safari iOS é grande base de usuários BR. E2EE meio quebrado em Safari vira churn.
4. **Custo operacional**: gestão de chaves é trabalho dedicado; não temos headcount M1.

### O que entregamos em vez

- **Transport-layer security**: DTLS + SRTP (padrão WebRTC; LiveKit default). Provedor não consegue interceptar — só o SFU "termina" o SRTP pra fazer routing, e LiveKit Cloud tem DPA.
- **Isolamento por tenant**: sala LiveKit = 1 condomínio; JWT scope por convenção; RLS Postgres no app-layer.
- **Auditabilidade**: logs de quem entrou (signaling events) + gravação cifrada at-rest (SSE-KMS S3) + ledger append-only de votos.

### ADR a criar

`/home/vinni/Documentos/Projetos/Privados/automation-agents/master-sindico-v2/00-product/adr/ADR-XXXX-assembleias-live-e2ee-off.md`

## Citações relevantes

> "Insertable Streams [...] provides a manipulation hook deep within the WebRTC pipeline after the encoder but before the packetizer, allowing operation on complete encoded frames rather than fragmented RTP packets."

> "Appending a 12-byte IV and a 16-byte Authentication Tag to every audio packet introduces approximately 25% overhead for audio, while for video the overhead is negligible (less than 1%)."

> "Users cannot see video until the key exchange completes, and if the signaling for keys lags behind the media connection, the user sees a black screen."

## URL(s)

- https://webrtchacks.com/true-end-to-end-encryption-with-webrtc-insertable-streams/
- https://webrtchacks.com/end-to-end-encryption-in-webrtc-4-years-later/
- https://blog.mozilla.org/webrtc/end-to-end-encrypt-webrtc-in-all-browsers/
- https://webrtc.github.io/samples/src/content/insertable-streams/endtoend-encryption/
