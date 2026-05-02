---
title: MOC — 11-audit/dual-check-log
type: moc
tags: [moc, master-sindico, v2, audit, dual-check]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 11-audit/dual-check-log

Registros cronológicos de dual-checks executados (propositor + revisor + consolidação). Formato: `YYYY-MM-DD-<slug>.md`.

Processo: [[../../06-execution/playbooks/dual-check]].

## Arquivos nesta pasta

- [[template]] — template a ser copiado pra cada dual-check novo
- [[2026-04-23-decisoes-arquiteturais]] — dual-check consolidado de ADR-0001..0025 + D-020..D-043 + 10 P-### de domínio (47 decisões revisadas; 6 A-DC-### abertos; 10 D-### sugeridos pra STATE)

## Dual-checks pendentes (D-### abertos)

Cada decisão aberta em [[../../STATE]] que for arquitetural vai gerar um arquivo aqui quando dual-check rodar:

- D-006 — topologia multi-tenant (RLS vs discriminator vs schema vs pool)
- D-007 — event-driven vs request-response por fluxo
- D-008 — feed/recomendação Connect Me
- D-009 — Livekit vs WebRTC custom vs Meet SFU
- D-010 — Reputação parametrizável vs congelada
- D-011 — Agência Marketing tenant vs actor
- D-012 — Empresa-síndica MVP ou backlog
- D-013 — LMS M3 thin vs M4 pleno
- D-014 — Moderação manual vs ML híbrido
- D-015 — Observability stack

Também: D-004 (research web amplo; 🟡 aberto) — resultará em vários dual-checks por vertical.

## Vizinhos

- [[../_moc]]
- [[../AUDIT]]
- [[../audit-log/_moc]]
- [[../../STATE]]
- [[../../06-execution/playbooks/dual-check]]
- [[../../02-architecture/adr/_moc]]
