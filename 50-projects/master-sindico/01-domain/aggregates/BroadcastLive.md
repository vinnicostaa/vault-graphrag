---
title: BroadcastLive (aggregate stub)
type: aggregate
tags:
  - aggregate
  - master-sindico
  - stub
  - content
bc: content
status: stub
created: '2026-04-27'
updated: '2026-04-27'
---

# BroadcastLive

> 🟡 **Stub** criado em 2026-04-27 (D-133) — Lives institucionais broadcast (1→N viewers) separadas de [[LiveSession]] (que mantém Livekit conferência N↔N para Assembly).

Aggregate root para **Lives institucionais broadcast** (Empresa Pro, Admin Master Síndico) com audiência grande (centenas a milhares de viewers concorrentes). Provider Mux Live (decisão pendente em [[../../02-architecture/adr/0044-broadcast-live-provider]]).

**BC**: content.

**Invariantes esperadas**:
- Owner: Company (kind=Pro) ou User (kind=admin_ms).
- Estado: state machine `scheduled | live | ended | archived`.
- Live → VOD automático (gravação vira [[Video]] na mesma conta).
- Privacidade: `public | members-only | empresa-only`.

**Eventos**: `BroadcastScheduled`, `BroadcastStarted`, `BroadcastEnded`, `BroadcastArchived`.

## Links

- [[_moc]]
- [[Video]]
- [[LiveSession]]
- [[../../02-architecture/adr/0044-broadcast-live-provider]]
- [[../../STATE]] §D-133
