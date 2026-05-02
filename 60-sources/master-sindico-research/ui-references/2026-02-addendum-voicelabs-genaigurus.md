---
title: ADDENDUM v1.1 — VoiceLabs + Genaigurus (UI Refs Feb 2026)
type: source-reference
tags: [source, ui-reference, master-sindico, design-system, voicelabs, genaigurus]
source: _chaos/MASTER_SINDICO_DESIGN_SYSTEM_ADDENDUM_v1.1.md (2026-02-23)
created: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: external-reference
---

# ADDENDUM v1.1 — Padrões UI extraídos de VoiceLabs & Genaigurus

> **Referências externas (Fevereiro 2026)** — análise de UIs de terceiros (**VoiceLabs** + **Genaigurus**) usada como inspiração visual para o design system Master Síndico.
> **Origem**: `_chaos/MASTER_SINDICO_DESIGN_SYSTEM_ADDENDUM_v1.1.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação. Material aqui é **referência histórica de inspiração**, não canônico técnico.

> **Vocabulário do doc original**: usava `N1/N2/N3` (abolido D-081/D-096) e "Morador Pagante" (removido D-126). Para vocabulário canônico atual, ver [[../../../50-projects/master-sindico/00-product/business-model]].

---

## Padrões absorvidos no canônico

A maioria destes padrões já está consolidada nos artefatos canônicos:

| Padrão | Origem ref | Destino canônico |
|---|---|---|
| Quota / Usage Widget (circular progress) | VoiceLabs "Credit Uses" | `QuotaWidget` em [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cms/home-dashboard]] |
| Quick Actions Grid (2×3 / 4×2) | VoiceLabs Dashboard | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cms/home-dashboard#quick-action-cards]] |
| Sidebar Refinada (User Switcher + seções uppercase + quota) | VoiceLabs | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/shell/app-shell#sidebar]] |
| Status Badges Bronze→Diamante (gradient + glow) | Genaigurus Leaderboard | [[../../../50-projects/master-sindico/02-architecture/design-tokens#182-status-badges—reputação-bronze-diamante]] |
| Tab "Status" no perfil + critérios de evolução | Genaigurus Badges | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cms/profile-management#status-dashboard-meu-status]] |
| Explore page com curated collections (tabs + trending + categorias) | VoiceLabs Voices | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cross/search-engine]] |
| Stepper de evolução (4 pontos conectados) | Genaigurus | adaptável a wizards (assembly, course creation) |
| Mobile Video Player (portrait / landscape / gold bar) | Genaigurus | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/content/video-player#tiktok-style-fullscreen]] |
| Banner de Comunicado dismissible | Genaigurus Newsletter | parte do [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cross/notifications]] |
| Modal de Denúncia com motivos pré-definidos | Genaigurus Feedback | [[../../../50-projects/master-sindico/03-subprojects/web/ui-catalog/cross/private-feedback#modal-de-denúncia]] |
| Heart Bounce, Shimmer, Stagger, Sidebar Collapse | Genaigurus + VoiceLabs | [[../../../50-projects/master-sindico/02-architecture/design-tokens#183-keyframes-canônicos]] |

---

## Padrões úteis não-canonizados ainda

Itens da ref que poderiam ser usados em features futuras (M3+ ou M4+):

- **Card Stagger Animation** (animação escalonada por `nth-child` em listagens de até 8 items) — útil para skeleton-to-real transition em grids.
- **Tooltip on Collapsed Sidebar Items** (tooltip lateral em sidebar 64 px colapsada) — útil quando D-058 evoluir.
- **Diamante Shimmer** (gradient animado 3 s) — já mencionado em design-tokens; adicionar `@keyframes shimmer` específico para badge Diamante quando RBAC permitir.

---

## Conteúdo histórico completo

Documento original arquivado em:

```
50-projects/master-sindico/_archive/2026-04-24-chaos-processed/feature-specs-23-fev/MASTER_SINDICO_DESIGN_SYSTEM_ADDENDUM_v1.1.md
```

(Fluxos detalhados, ASCII art, especificações de Quick Action Card por persona, etc. preservados na íntegra para referência histórica.)

## Links

- [[../_toc|60-sources/master-sindico-research/_toc]]
- [[../../../50-projects/master-sindico/02-architecture/design-tokens|design-tokens]]
- [[../../../50-projects/master-sindico/03-subprojects/web/design-system|web/design-system]]
- [[../../../50-projects/master-sindico/03-subprojects/web/patterns/ui-states|patterns/ui-states]]
- [[../../../50-projects/master-sindico/STATE]]
