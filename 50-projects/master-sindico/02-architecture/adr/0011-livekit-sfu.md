---
title: ADR-0011 — LiveKit Cloud (SFU WebRTC) em M3; self-host M3+ sob gatilho
type: adr
tags: [adr, assembly, webrtc, livekit, master-sindico, v2]
status: accepted
date: 2026-04-23
created: 2026-04-23
updated: 2026-04-23
---

# ADR-0011 — LiveKit como SFU WebRTC (Cloud M1; self-host sob gatilho)

## Status (atualizado 2026-04-27)

⚠️ **Atualização D-141 (2026-04-27)**: LiveKit é a **implementação inicial**, encapsulada como adapter de `IRealtimeProvider`. Cliente decidiu adicionar **Cloudflare Calls/Realtime** como segundo adapter (alinhado D-138). Domínio NÃO conhece LiveKit SDK. Refactor obrigatório M1: port `IRealtimeProvider` domain-typed; `LiveKitProvider` em `infrastructure/providers/livekit/`; `CloudflareCallsProvider` em `infrastructure/providers/cloudflare_calls/` será adicionado M2-M3 após validar feature parity (recording, simulcast, screen share, transcrição). Ver [[../../STATE#D-141]].

### Status original

accepted — 2026-04-23

## Contexto

Sub-produto **Assembleias Live** (M3) precisa baixa latência (< 400ms), bidirecional (deliberação + voto), gravação para ata (Código Civil art. 1.354), escala 10-500 participantes, TURN fallback para rede corporativa bloqueando UDP.

## Decisão

**LiveKit SFU** (Selective Forwarding Unit) como engine WebRTC — LiveKit Cloud em M1/M3 primeira assembleia; self-host K8s + Redis + STUNner só M3+ sob gatilho de custo.

### Configuração

- **Cloud single-region SP** em primeiras assembleias (até ~200h/mês + 30 participantes ativos/sala).
- **Gravação**: LiveKit Egress Room Composite → MP4 em S3 + MP3 áudio paralelo (redundância barata).
- **Signaling**: WebSocket nativo LiveKit; master-sindico API só emite **JWT de acesso** (`room_join`, `canPublish` condicional, metadata: `tenant_id`, `unidade_id`, `fracao_ideal_mask`).
- **Votação RT**: DataStream LiveKit reliable para UI otimista; **fonte de verdade em Postgres via POST HTTP** ([[../../13-research/google-meet/_destilado]] §4).
- **Fila de fala**: attributes por participante + `UpdateParticipant` API (quem tem `canPublish=true` fala; moderador alterna).
- **E2EE**: **off** — quebra gravação server-side, que é requisito legal da ata.
- **TURN**: incluso LiveKit Cloud; self-host STUNner em K8s só quando custo justificar.
- **Codec**: VP9 SVC em multi-party; AV1 avaliado em M4+.

## Consequências

### Positivas

- Zero operação SFU/TURN em M1-M2.
- Egress Room Composite entrega MP4 pronto para anexar ata.
- Latência <400ms em SP — adequado para voto ao vivo.
- Dimensionamento folgado: LiveKit single node roteia centenas de streams; Cloud vai a 100k por sessão.

### Negativas

- Dependência de LiveKit Cloud availability — ADR futuro: cenário "assembleia crítica ocorre durante outage LK" → fallback manual (gravação por síndico local + upload).
- Custo: transcode $0.02/min overage + bandwidth. Assembleia 2h × 50 participantes ~$5-10 por sessão (baseline).

### Trade-off E2EE off

- **Privacidade**: tenant isolation (sala = condomínio; RLS na camada de dados), DPA LGPD com LiveKit, SRTP+DTLS em transporte.
- **Ameaça modelada**: não defendemos contra operador hostil SFU; defendemos contra interceptação rede. SRTP/DTLS resolve.

## Alternativas consideradas

1. **Daily.co** — SFU similar; preço similar; API pronta.
2. **Jitsi** — OSS; requer operar JVS + Jibri; complexo.
3. **Zoom SDK** — $$ e lock-in; UX excelente; reavaliar se cliente pedir.
4. **Mux Real-Time** — recém-lançado 2025; ainda imaturo para ata legal.
5. **WebRTC mesh P2P** — cai com 10+ participantes.

## Referências

- [[../../13-research/google-meet/_destilado]] §2, §4, §6, §7
- [[../c4-components]] §3.6 assembly BC
- [[0001-clean-architecture-ddd-cqrs]]
