---
title: MOC — Mobile UI Catalog
type: moc
tags: [moc, master-sindico, mobile, flutter, ui-catalog]
project: master-sindico
stack: mobile
created: 2026-04-24
---

# MOC — Mobile UI Catalog

Telas mobile Flutter organizadas por fluxo. Derivadas do web catalog (paridade onde aplicável) **com adaptações mobile**: fontes grandes, touch targets ≥ 48dp, keyboard behavior, SafeArea, platform back button.

## Estrutura

- [[onboarding/_moc]] — 4 fluxos (morador/síndico/empresa/parceiro)
- [[morador/_moc]] — M1..M15 mobile
- [[sindico/_moc]] — S1/S2/S6/S7/S8 (subset M1 mobile; S9-S31 web-only)
- [[assembly/_moc]] — ASM-CHKIN, ASM-VOTO, ASM-FILA-FALA, ASM-CIEN, ASM-PROC-CAD (M3)
- [[vizinhanca/_moc]] — VM1..VM7 (morador feed; M3)

## Convenções

- Cada tela em arquivo `.md` 80-200 linhas.
- Frontmatter obrigatório: `title, type, tags, project, persona, screen_id, stack: mobile, status, created`.
- Template mobile estende web com:
  - **Widget Flutter principal** (classe + tipo)
  - **Platform-specific notes** (iOS vs Android: permissions, back button, SafeArea)
  - **Offline behavior**
  - **Push notification trigger** (quando aplicável)

## Links

- [[../../../_moc]]
- [[../../web/ui-catalog/_moc]] (fonte de jornadas — paridade)
- [[../requirements/_moc]]
