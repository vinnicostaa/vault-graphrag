---
title: ASM-FILA-FALA — Fila de Fala (Mobile)
type: ui-screen
tags: [master-sindico, ui-catalog, mobile, assembly]
project: master-sindico
persona: morador
screen_id: ASM-FILA-FALA
sub_produto: assembleia-live
plan_requirement: premium
status: specification
stack: mobile
created: 2026-04-24
---

# ASM-FILA-FALA — Fila de Fala (Mobile)

## Finalidade

Morador "levanta a mão" para entrar na fila de falas da assembleia. Síndico visualiza e gerencia ordem (via web ou mobile modo apresentação M3+).

## Persona e ABAC

- Morador check-in ativo.
- `assembly.speaker_queue.raise`.

## Fluxo mobile

1. Durante ASM-VOTO, toca ícone "mão levantada" na AppBar.
2. `POST /speaker-queue/raise` — adiciona na fila.
3. Exibe card status: "Você é #4 na fila. Tempo estimado ~8 min."
4. Pode cancelar (`DELETE /speaker-queue/me`).
5. Recebe push quando é sua vez: "Você foi chamado. Aguarde microfone."

## Widget Flutter principal

`SpeakerQueueButton` (FAB ou AppBar action) + `SpeakerQueueDialog` com posição atual.

## Platform-specific notes

- Haptic heavy ao entrar fila.
- Notificação critical quando é a vez.

## Estados

- Idle / Raising / InQueue(pos, eta) / Called / Cancelling / Error.

## Offline behavior

- Online obrigatório.

## Push notification trigger

- `assembly.{id}.speaker.called.{user_id}` canal `ms_assembly_live`.

## WebSocket events

- `speaker_queue.updated { queue }` — recalcula posição.

## Gaps/ressalvas

- Tempo por fala configurável pelo síndico server-side.
- Fala remota (via áudio app) = M4+ (requer Livekit full).

## Links

- [[_moc]] · [[ASM-VOTO]]
