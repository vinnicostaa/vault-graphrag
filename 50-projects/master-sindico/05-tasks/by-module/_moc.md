---
title: MOC — 05-tasks/by-module
type: moc
tags: [moc, master-sindico, v2, tasks, module]
created: 2026-04-23
updated: 2026-04-23
---

# MOC — 05-tasks/by-module

Tasks agrupadas por bounded context. Espelha `04-requirements/functional/<module>.md`.

## Arquivos nesta pasta

- [[identity]] — BC identity (Zitadel OIDC, 1-device, ABAC claims, anti-fraude)
- [[billing]] — BC billing (Stripe, planos, trial persona-aware, quotas, soft-block)
- [[institutional]] — BC institutional (Condomínio, Unidades, Timeline imutável, Plano Diretor, Governance Score, Compliance)
- [[commercial]] — BC commercial (Connect Me 4 fluxos, Reputação Bronze→Diamante, Vizinhança, Banco Talentos, Supplier Vote)
- [[content]] — BC content (vídeos Mux, moderação 48h, lock 90d, LMS, busca, ebooks)
- [[assembly]] — BC assembly (Assembleias Livekit, votação fracional, procuração, ata imutável, relatórios)
- [[cross-domain]] — transversal (IAuditLogger, ABAC, Event Bus, LGPD Registry, Antifraude, Providers compartilhados)

## Vizinhos

- [[../_moc]] — MOC tasks
- [[../backlog]] — backlog global
- [[../by-sprint/_moc]]
- [[../../01-domain/bounded-contexts]]
- [[../../04-requirements/functional/_moc]]
