---
title: MOC — Assembly Mobile (M3)
type: moc
tags: [moc, master-sindico, mobile, flutter, assembly]
project: master-sindico
stack: mobile
created: 2026-04-24
---

# MOC — Assembly Mobile (M3)

Telas da assembleia live no mobile. **Fora do escopo M1** (ver D-versão-assembly-fora-escopo-M1 = A-FASE10-DEV-004). Entrega M3 (Sprint 12).

## Telas

- [[ASM-CHKIN]] — Check-in ao vivo (QR / código numérico)
- [[ASM-VOTO]] — Voto item em aberto (WebSocket real-time)
- [[ASM-FILA-FALA]] — Levantar mão / fila de falas
- [[ASM-CIEN]] — Ciência de item não-votável
- [[ASM-PROC-CAD]] — Cadastrar procuração (subset mobile; completo = web)

## Integridade (D-050)

M3 gated: `integrity=hooked` ou `jailbroken` → bloquear acesso a voto (modal "dispositivo não seguro").

## WebSocket

- Endpoint `/ws/assembly/{id}` com Bearer token.
- Reconnect backoff 1s → 30s max.

## Links

- [[../../../../_moc]]
- [[../../requirements/assembly-check-in]] · [[../../requirements/assembly-vote]]
- [[../../../STATE#D-070|D-070 WebSocket]]
- [[../../../AUDIT#A-FASE10-DEV-004|A-FASE10-DEV-004]]
