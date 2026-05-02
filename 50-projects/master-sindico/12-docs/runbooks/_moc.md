---
title: MOC — 12-docs/runbooks
type: moc
tags: [moc, master-sindico, v2, docs, runbooks]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 12-docs/runbooks

Runbooks operacionais. Passos concretos pra situações específicas — escritos pensando em **dev/ops acordado às 3h da manhã**.

## Arquivos nesta pasta

- [[incident-lgpd-breach]] — incident LGPD com notificação ANPD em 72h + titulares + postmortem
- [[rollback-deploy]] — rollback operacional em Railway (staging ou prod), incluindo migration destrutiva
- [[webhook-reprocessing]] — reprocessar webhooks Stripe/Mux/Livekit via admin endpoints + DLQ
- [[dr]] — **Disaster Recovery** (DR/BCP): RTO 4h crítico + RPO 1h; PITR Postgres via WAL archive S3 + full diário; quarterly DR drill; LGPD Art. 48 em perda de writes ou indisponibilidade > 24h
- [[secret-rotation]] — Rotação de secrets: cadências 90d/180d + dual-key overlap 7d + emergency hard cutover em compromise; inventário Stripe/Zitadel/Mux/Livekit/Twilio/cookie/JWT/LGPD HMAC

## Runbooks planejados (backlog)

- `pgsql-backup-restore.md` — backup/restore PG detalhado (complementa `dr.md` com passos granulares de backup pipeline)
- `zitadel-failover.md` — Zitadel Railway instability (T-### M2)
- `onboarding-oncall.md` — onboarding de novo oncall dev (pré-M3)
- `assembly-live-troubleshoot.md` — debug assembleia live Livekit em prod (pré-M3)

## Vizinhos

- [[../_moc]]
- [[../README]]
- [[../how-to/_moc]]
- [[../../06-execution/incident]]
- [[../../06-execution/rollback]]
- [[../../08-security/lgpd]]
