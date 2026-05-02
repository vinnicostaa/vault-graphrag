---
title: "Source 07 — LiveKit Participant Management (permissions, remote mute, fila de fala)"
type: research-source
tags: [research, master-sindico, v2, webrtc, livekit, permissoes, moderador, fila-fala]
created: 2026-04-23
updated: 2026-04-23
source_type: oficial
source_org: "LiveKit"
source_urls:
  - "https://docs.livekit.io/home/server/managing-participants/"
  - "https://docs.livekit.io/home/get-started/api-primitives"
  - "https://github.com/livekit/components-js/issues/749"
acessado_em: 2026-04-23
confiabilidade: alta
---

# Source 07 — LiveKit Participant Management

## Resumo

Como LiveKit expõe controle de moderador: permissões granulares (`canPublish`, `canSubscribe`, `canPublishData`), `UpdateParticipant` API, remote mute (opt-in), e padrões de `metadata`/`attributes` pra montar fila de fala.

## Pontos-chave

### Permissions

Três grants principais em `ParticipantPermission`:
- `canPublish` — pode publicar tracks áudio/vídeo.
- `canSubscribe` — pode receber tracks dos outros.
- `canPublishData` — pode enviar data messages.

Revogar `canPublish` **desinscreve automaticamente** todas as tracks que a pessoa estava publicando. Útil pra "cortar microfone" imediato.

### UpdateParticipant

Requer grant `roomAdmin`. Atualiza:
- `Permission` (dispara evento `ParticipantPermissionChanged`)
- `Metadata` (string JSON livre; dispara `ParticipantMetadataChanged`)
- `Attributes` (map key-value; dispara `ParticipantAttributesChanged`)

### Remote mute

`MutePublishedTrack` API permite mutar track remoto. **Remote unmute é desligado por padrão** (UX: não surpreender usuário). Habilitar em config `room.enable_remote_unmute: true` se moderador precisa ligar mic de quem foi chamado.

### Raise hand — NÃO tem built-in

Pattern app-level com `attributes`.

## Aplicação ao Master Síndico — Fila de Fala

**Modelo**:

```json
// attributes do participante morador
{
  "hand_raised": "true",
  "queue_position": "3",
  "requested_at": "2026-04-23T19:32:10-03:00",
  "unidade": "401-A",
  "fracao_ideal": "0.043"
}
```

**Fluxo**:

1. Morador clica "pedir palavra" → client chama API nossa → API chama `UpdateParticipant` setando `attributes.hand_raised=true`.
2. Todos veem evento `ParticipantAttributesChanged` (State Sync) → UI atualiza lista da fila ordenada por `requested_at`.
3. Síndico (tem `roomAdmin`) chama "dar palavra ao próximo" → nossa API:
   - Revoga `canPublish` do falante anterior (corta mic).
   - Concede `canPublish=true` ao próximo da fila.
   - Seta `attributes.speaking=true` + inicia cronômetro server-side.
4. Quando timer expira (ex: 3min) → API revoga `canPublish` automaticamente.

**Vantagem**: enforcement é server-side; cliente malicioso não consegue furar fila.

### Remote mute moderador

Habilitar `enable_remote_unmute: true` — síndico precisa poder ligar mic do chamado sem esperar clique. Documentar no TOS da assembleia que isso é comportamento esperado.

## Citações relevantes

> "When you revoke the CanPublish permission from a participant, all tracks they've published are automatically unpublished."

> "Being remotely unmuted can catch users by surprise, so it's turned off by default."

## URL(s)

- https://docs.livekit.io/home/server/managing-participants/
- https://docs.livekit.io/home/get-started/api-primitives
- https://github.com/livekit/components-js/issues/749
