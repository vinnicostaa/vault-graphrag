---
title: MOC — 12-docs
type: moc
tags: [moc, master-sindico, v2, docs]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 12-docs

README principal, how-to guides, runbooks operacionais, ADR-index, changelog.

## Arquivos nesta pasta

- [[README]] — README principal do projeto (pitch + stack + como rodar + como contribuir + links essenciais)
- [[adr-index]] — índice de ADRs (27 ADRs catalogadas — 14 já implementadas no legado, 13 a escrever ou pendentes research)
- [[changelog]] — changelog (seed); populado conforme deploys M1/M2/M3/hotfixes
- [[how-to/_moc]] — guias passo-a-passo
  - [[how-to/criar-novo-bc]] — adicionar bounded context novo
  - [[how-to/adicionar-novo-provider]] — adicionar provider externo (interface + adapter + webhook)
  - [[how-to/criar-adr]] — criar ADR (MADR simplificado)
- [[runbooks/_moc]] — runbooks operacionais
  - [[runbooks/incident-lgpd-breach]] — incident LGPD + ANPD 72h notification
  - [[runbooks/rollback-deploy]] — rollback operacional Railway
  - [[runbooks/webhook-reprocessing]] — reprocessar webhooks Stripe/Mux/Livekit

## Templates LGPD/Compliance (stubs — D-vault-025 2026-04-27)

- [[dpia-template]] — DPIA (LGPD art. 38) — risco alto de tratamento
- [[privacy-policy-template]] — política de privacidade pública (LGPD art. 9º)
- [[incident-communication-template]] — comunicação de incidente (LGPD art. 48 + ANPD)

## Vizinhos

- [[../_moc]]
- [[../CLAUDE]]
- [[../SESSION_CHARTER]]
- [[../02-architecture/adr/_moc]]
- [[../06-execution/_moc]]
- [[../11-audit/_moc]]
