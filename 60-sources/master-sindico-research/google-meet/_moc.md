---
title: MOC — 13-research/google-meet
type: moc
tags: [moc, master-sindico, v2, research,google-meet]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 13-research/google-meet

WebRTC, SFU vs MCU (comparar com Livekit), scalability, E2E encryption.

> Pasta da v2 (remontagem do Master Síndico). Paridade com scaffold canônico em `vault/40-templates/project-scaffold/`. Stub criado na Fase 0 (2026-04-23). Conteúdo entra nas Fases 3/4 conforme plano em `/home/vinni/.claude/plans/squishy-drifting-avalanche.md`.

## Arquivos

- [[_destilado]] — destilado aplicável ao Master Síndico (SFU confirmado, decisões de votação RT, fila de fala, gravação, E2EE off, TURN delegado)
- [[source-01]] — WebRTC fundamentals (ICE/STUN/TURN/SDP) — MDN
- [[source-02]] — LiveKit SFU internals — docs.livekit.io
- [[source-03]] — Google Meet codecs (VP9 SVC, AV1) — webrtcHacks
- [[source-04]] — Escalabilidade: LiveKit Distributed Mesh + Jitsi Octo
- [[source-05]] — Zoom architecture + Daily.co (comparativos)
- [[source-06]] — LiveKit Data Channels (chat, RPC, votação RT)
- [[source-07]] — LiveKit Participant Management (permissões, mute, fila de fala)
- [[source-08]] — LiveKit Egress vs Mux (gravação pra ata)
- [[source-09]] — LiveKit TURN/ICE produção + STUNner
- [[source-10]] — WebRTC E2EE (Insertable Streams) — análise e decisão OFF

## Decisões-chave emergentes (candidatas a D-### em STATE e ADRs em 02-architecture/adr/)

1. Manter LiveKit como SFU — sem mudança arquitetural vs legado.
2. Votação RT = LiveKit DataStream reliable (UI) + API nossa (fonte verdade/ata).
3. Fila de fala = `attributes` por participante + `UpdateParticipant` server-side com enforcement de cronômetro.
4. Gravação = LiveKit Egress Room Composite → MP4 S3 tenant-isolated (não Mux).
5. E2EE desligado — conflita com gravação legal da ata BR; privacidade já tenant-isolated via RLS.
6. TURN = delegado a LiveKit Cloud em M1; self-host só quando gatilhos de custo/soberania disparam.

> Seguir o fluxo §5 do CLAUDE.md: estas decisões precisam virar D-### em [[../STATE]] + ADRs antes de Fase 6.

## Vizinhos

- [[../_moc]] — MOC raiz da v2
- [[../CLAUDE]] — contrato do agente
- [[../STATE]] — decisões vivas
