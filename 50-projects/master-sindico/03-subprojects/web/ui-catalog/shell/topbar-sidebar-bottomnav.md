---
title: Topbar / Sidebar / Bottom Nav — Shell consolidado
type: ui-spec
tags: [ui-catalog, master-sindico, shell, layout, cross]
bc: cross
source: _chaos/MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md §3 (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# Shell — Topbar, Sidebar, Bottom Nav (consolidado)

> **Origem**: `_chaos/MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md` §3 + diversos addendums (Avalanche / VoiceLabs / Genaigurus / Flavor refinements).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Status**: o conteúdo de §3 é praticamente 100 % redundante com [[./app-shell|./app-shell]] (que já foi absorvido do `_chaos/02-APP-SHELL.md`). Este doc consolida apenas **deltas** específicos do FRONTEND_DESIGN_SYSTEM monolito (Sprint 1-2 spec) que não entraram no app-shell.

---

## Conteúdo principal

A spec canônica de Topbar / Sidebar / Bottom Nav vive em:

- **[[./app-shell|app-shell]]** — Topbar (desktop / mobile), Sidebar (240 px / 64 px collapsed), Bottom Nav mobile (5 items + FAB), Drawer mobile, Breadcrumb, regras de visibilidade ABAC por `plan_tier`.
- **[[../../../../02-architecture/design-tokens|design-tokens]]** §18 — tokens, CVA Button / Badge, animações, ícones Iconify.

---

## Deltas absorvidos do monolito

### 1. Sidebar com User Switcher (multi-condomínio)

Padrão refinado VoiceLabs aplicado ao Síndico que opera múltiplos condomínios:

```
┌──────────────────────────┐
│  ┌─────────────────────┐ │
│  │ [👤] João Silva   ▾ │ │  User switcher
│  │      Síndico Pro    │ │  (dropdown lista condos)
│  └─────────────────────┘ │
│                          │
│  ▣ Início                │ ← gold bg pill se ativo
│  ...                     │
```

```css
.user-switcher {
  display: flex; align-items: center; gap: 10px;
  padding: 12px 14px; margin: 16px 12px;
  background: var(--surface-variant);
  border: 1px solid var(--border);
  border-radius: var(--radius-md);
  cursor: pointer;
}
.user-switcher-name {
  font-family: 'Manrope'; font-size: 14px; font-weight: 600;
  color: var(--surface-foreground);
}
.user-switcher-role {
  font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground);
}
```

**Dropdown** (Kobalte Popover): bg `var(--popover)`, border 1 px, shadow-lg, rounded-lg. Item selecionado: dot gold + texto bold. Hover: bg `var(--muted)`.

### 2. Sidebar — Badges "NEW" e numéricos (notificações pendentes)

```css
.sidebar-badge-new {
  padding: 1px 6px; border-radius: var(--radius-full);
  background: var(--accent); color: var(--secondary);
  font-family: 'Manrope'; font-size: 10px; font-weight: 700;
  text-transform: uppercase;
  margin-left: auto;
}
.sidebar-badge-count {
  min-width: 20px; height: 20px;
  display: inline-flex; align-items: center; justify-content: center;
  border-radius: 50%;
  background: var(--destructive); color: white;
  font-family: 'Manrope'; font-size: 11px; font-weight: 700;
  margin-left: auto;
}
```

Aplicação típica: "📋 Assembleias  •3" (3 pendentes), "📻 Podcast  NEW" (feature recém-lançada).

### 3. Tooltip on collapsed sidebar items

Quando sidebar 64 px (colapsada): tooltip lateral aparece on hover com label completa.

```css
.sidebar-tooltip {
  position: absolute; left: calc(100% + 8px); top: 50%;
  transform: translateY(-50%);
  background: var(--popover); color: var(--popover-foreground);
  padding: 6px 10px; border-radius: var(--radius-sm);
  box-shadow: var(--shadow-md);
  font-family: 'Manrope'; font-size: 13px; white-space: nowrap;
  opacity: 0; pointer-events: none;
  transition: opacity var(--duration-quick), transform var(--duration-quick);
}
.sidebar.collapsed .sidebar-item:hover .sidebar-tooltip {
  opacity: 1; transform: translateY(-50%) translateX(0);
}
```

### 4. Bottom Nav FAB Action Sheet

Ao clicar no `[⊕]` central no bottom nav mobile, abre Bottom Sheet (Kobalte Dialog/Sheet) com ações por `role`:

| Role | Ações no Action Sheet |
|---|---|
| Síndico | Upload Vídeo · Nova Assembleia · Criar Comunicado · Criar Link |
| Empresa | Upload Vídeo · Criar Curso (`pro` only) · Registrar Execução |
| Morador (com Banco Talentos) | Atualizar Vídeo-Currículo (gated por trava 90 d) |
| Morador (sem Banco Talentos) | Ativar Banco de Talentos · Buscar Empresas |

### 5. Card do plano — Variações

Footer da sidebar exibe card de plano consoante `plan_tier`. Substitui o nome da label "Síndico N3" por **Plano Pro** (D-081/D-096):

```
┌────────────────┐
│ Plano Pro      │  Michroma 12px gold
│ Status: 🥇 Ouro│  Manrope 12px
│ [Gerenciar →]  │
└────────────────┘
```

Ver detalhamento de tiers em [[../../../../00-product/business-model]].

---

## Componentes

`UserSwitcher`, `SidebarBadgeNew`, `SidebarBadgeCount`, `SidebarTooltip`, `BottomNavFabActionSheet`, `SidebarPlanCard` (com variants por `plan_tier`).

## Links

- [[./app-shell|app-shell]] — spec canônica completa
- [[../../../../02-architecture/design-tokens|design-tokens]] §18 — tokens consumidos
- [[../../../../00-product/business-model]] — `plan_tier`
- [[../../patterns/ui-states|patterns/ui-states]]
