---
title: MOC — 03-subprojects
type: moc
tags: [moc, master-sindico, subprojects]
project: master-sindico
created: 2026-04-22
---

# 03-subprojects — Backend, Web, Mobile

Três sub-projetos independentes mas coesos:

| Sub-projeto | Pasta no repo | README (vault) | CLAUDE.md | Extras |
|---|---|---|---|---|
| Backend | `Development/backend/` | [[backend/README]] | — | |
| Web | `Development/web/` | [[web/README]] | [[web/CLAUDE]] | |
| Mobile | `Development/app/` (Flutter) | [[mobile/README]] | [[mobile/CLAUDE]] | [[mobile/ARCHITECTURE]] + [[mobile/CODING_MANUAL]] |

Cada um tem sua pasta com README (stack, escopo), architecture.md (camadas e padrões aplicados), patterns.md (quais patterns se aplicam aqui), requirements.md (requisitos específicos — **complementam** os de [[../04-requirements/_moc]]), tasks.md (tasks específicas), security.md (posture específica).

### Workspace-wide (raiz git multi-repo)

Contratos que atravessam backend + web + mobile + legacy fullstack:

- [[workspace-CLAUDE]] — CLAUDE.md da raiz do workspace git (cobre setup cross, stack matrix, commands por sub-projeto, invariantes Go backend)
- [[workspace-AGENTS]] — AGENTS.md com papel do agente (arquiteto + dev sênior + DevOps + sec) e coding guidelines transversais

## Princípios cross-subproject

- **Specs canônicas são no backend**. Web/mobile são vertical slices; consomem as mesmas specs (ver `web/.kiro/specs/README-DEPRECATED.md` e `app/.kiro/specs/README-DEPRECATED.md`)
- **OpenAPI** é contrato entre backend e web/mobile
- **Design tokens** compartilhados (cores, tipografia, espaçamento) via design system ([[web/design-system]])
- **i18n keys** compartilhadas (pt-BR first)
- **Auth flow** idêntico: web usa cookie httpOnly `.mastersindico.com.br`, mobile usa Bearer + refresh
- **Telemetry schema** consistente entre backend/web/mobile (OpenTelemetry)

## Vizinhos

- [[../_moc]]
- [[workspace-CLAUDE]] · [[workspace-AGENTS]]
- [[backend/README]]
- [[web/README]] · [[web/CLAUDE]]
- [[mobile/README]] · [[mobile/CLAUDE]] · [[mobile/ARCHITECTURE]] · [[mobile/CODING_MANUAL]]
- [[../02-architecture/_moc]]
- [[../04-requirements/_moc]]
- [[../05-tasks/_moc]]
