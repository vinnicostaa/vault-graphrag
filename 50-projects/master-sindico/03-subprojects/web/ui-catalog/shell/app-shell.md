---
title: App Shell — Sidebar / Topbar / Bottom Nav
type: ui-spec
tags: [ui-catalog, master-sindico, shell, layout, cross]
bc: cross
source: _chaos/02-APP-SHELL.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 02 — App Shell (Sidebar, Topbar, Bottom Nav)

> **Origem**: `_chaos/02-APP-SHELL.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "My Síndico" → "Master Síndico".

> Transversal · Layout principal da aplicação autenticada (apps `cms`, `lms`, `forum`, `assembly`).
> Componentes: `AppShell`, `Sidebar`, `Topbar`, `BottomNav`, `Drawer`.

---

## Estrutura macro — Desktop (≥ 1024 px)

```
┌─────────────────────────────────────────────────────────────┐
│                     TOP BAR (64px)                           │
│  [☰] [🛡 MASTER SÍNDICO]  [🔍 Buscar...]  [🌙] [🔔•] [👤▼]│
├────────────┬────────────────────────────────────────────────┤
│  SIDEBAR   │            MAIN CONTENT AREA                   │
│  (240px)   │            (flex-1, overflow-y-auto)           │
│  Navy bg   │                                                │
│            │  Breadcrumb / Page Title                       │
│  [🏠 Início]│  [ Content ]                                   │
│  [🔍 Buscar]│                                                │
│  [👤 Perfil]│                                                │
│  [📋 Assemb]│                                                │
│  [⚙ Config] │                                                │
│            │                                                │
│  ──────── │                                                │
│  Plano: Pro│                                                │
│  Status: 🥇│                                                │
├────────────┴────────────────────────────────────────────────┤
│                     FOOTER (opcional)                        │
└─────────────────────────────────────────────────────────────┘
```

## Estrutura macro — Mobile (< 768 px)

```
┌──────────────────────┐
│  TOP BAR (56px)      │
│  [☰] [🛡] [🔍] [🔔]│
├──────────────────────┤
│                      │
│  MAIN CONTENT        │
│  (full-width)        │
│  (overflow-y-auto)   │
│                      │
├──────────────────────┤
│  BOTTOM NAV (64px)   │
│  [🏠][🔍][⊕][📋][👤]│
└──────────────────────┘
```

## Topbar

### Desktop

```
┌──────────────────────────────────────────────────────────────┐
│  [☰]  [🛡 MASTER SÍNDICO]         [🔍 Buscar síndicos,     │
│         Michroma gold               empresas, serviços...]   │
│                                   [🌙/☀️] [🔔 •] [Avatar▼]  │
└──────────────────────────────────────────────────────────────┘
```

```css
.topbar {
  height: 64px; /* 56px mobile */
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px;
  position: sticky; top: 0; z-index: var(--z-sticky);
}
.topbar-logo {
  font-family: 'Michroma'; color: var(--primary);
  font-size: 14px; letter-spacing: 0.05em;
}
.topbar-search {
  flex: 1; max-width: 480px; margin: 0 24px;
  height: 40px; padding: 0 12px;
  background: var(--surface-variant); border: 1px solid var(--border);
  border-radius: var(--radius-md); color: var(--surface-foreground);
  font-family: 'Manrope'; font-size: 14px;
}
.topbar-search:focus { border-color: var(--ring); box-shadow: 0 0 0 2px oklch(0.715 0.120 84.2 / 0.2); }
.topbar-search::placeholder { color: var(--muted-foreground); }
.topbar-actions { display: flex; align-items: center; gap: 8px; }
.topbar-icon-btn {
  width: 36px; height: 36px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  color: var(--surface-foreground); transition: background var(--duration-moderate) var(--ease-out);
}
.topbar-icon-btn:hover { background: var(--surface-variant); }
.topbar-avatar {
  width: 32px; height: 32px; border-radius: 50%;
  border: 2px solid transparent; cursor: pointer;
}
/* Plan tier pro: gold border on avatar */
.topbar-avatar[data-plan-tier="pro"] { border-color: var(--primary); }
.topbar-notification-dot {
  position: absolute; top: 6px; right: 6px;
  width: 8px; height: 8px; border-radius: 50%;
  background: var(--destructive);
}
```

### Mobile

- Logo: apenas brasão (sem texto).
- Search: ícone que abre fullscreen search overlay.
- Hamburger: abre `Drawer` (sidebar overlay).
- Height: 56 px.

## Sidebar

### Estrutura de itens

```
PRINCIPAL
  🏠 Início
  🔍 Buscar
  📹 Vídeos
  👤 Meu Perfil

MINHA GESTÃO (apenas Síndico plus/pro)
  📋 Assembleias
  📊 Transparência
  ⭐ Avaliações
  📄 Exportar PDF

CONTEÚDO
  🎓 Cursos (apenas com acesso LMS)
  💬 Fórum (apenas roles com acesso)
  🎙 Podcast
  📺 Lives

COMUNIDADE
  🏪 Vizinhança
  💼 Banco de Talentos

CONFIGURAÇÕES
  ⚙️ Configurações
  ❓ Ajuda
```

### CSS

```css
.sidebar {
  width: 240px; height: calc(100vh - 64px);
  background: var(--surface); border-right: 1px solid var(--border);
  display: flex; flex-direction: column;
  overflow-y: auto; position: fixed; top: 64px; left: 0;
  transition: width var(--duration-slow) var(--ease-in-out);
  z-index: var(--z-dropdown);
}
.sidebar.collapsed { width: 64px; }
.sidebar-section-label {
  font-family: 'Michroma'; font-size: 10px; text-transform: uppercase;
  letter-spacing: 0.1em; color: var(--muted-foreground);
  padding: 24px 16px 8px; user-select: none;
}
.sidebar.collapsed .sidebar-section-label { display: none; }
.sidebar-item {
  display: flex; align-items: center; gap: 12px;
  padding: 10px 16px; margin: 2px 8px;
  border-radius: var(--radius-md); color: var(--surface-foreground);
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out); position: relative;
  text-decoration: none;
}
.sidebar-item:hover { background: oklch(0.274 0.077 256.3 / 0.5); }
.sidebar-item.active {
  background: oklch(0.715 0.120 84.2 / 0.15);
  color: var(--primary);
  border-left: 3px solid var(--primary);
  margin-left: 5px; /* compensar border */
}
.sidebar-item .icon { width: 20px; height: 20px; flex-shrink: 0; }
.sidebar.collapsed .sidebar-item .label { display: none; }
.sidebar.collapsed .sidebar-item { justify-content: center; padding: 10px; }
```

### Widget de plano (fundo da sidebar)

```css
.sidebar-plan-card {
  margin: auto 8px 16px; padding: 12px;
  background: var(--surface-variant); border: 1px solid oklch(0.715 0.120 84.2 / 0.3);
  border-radius: var(--radius-md);
}
.sidebar-plan-name { font-family: 'Michroma'; font-size: 12px; color: var(--primary); }
.sidebar-plan-status { font-family: 'Manrope'; font-size: 12px; color: var(--surface-foreground); margin-top: 4px; }
```

### Badge "NEW" em item

```css
.sidebar-new-badge {
  font-size: 10px; font-weight: 700;
  background: var(--primary); color: var(--primary-foreground);
  padding: 1px 6px; border-radius: var(--radius-full);
  margin-left: auto;
}
```

### Visibilidade ABAC

Itens NÃO mostram cadeado. Se o `plan_tier` não permite, o item NÃO renderiza (decisão UX: zero ruído visual de bloqueio em navegação primária).

| Item | Visível para |
|---|---|
| Assembleias | Síndico `plus`, `pro` |
| Transparência | Síndico `pro` |
| Avaliações | Síndico `pro` |
| Exportar PDF | Síndico `pro` |
| Cursos | Síndico `plus`+, Empresa `pro`, Marketing |
| Fórum | Síndico `base`+, Empresa `plus`+, Marketing |
| Lives | Todos exceto `base` |
| Banco de Talentos | Empresa `plus`+, Marketing, Vizinhança (Parceiro) |

> Detalhamento completo da matriz em [[../../../../00-product/business-model#41-matriz-consolidada]].

## Bottom Navigation (Mobile)

```
┌─────────────────────────────────────────┐
│  [🏠]   [🔍]   [⊕]   [📋]   [👤]     │
│ Início  Buscar  Ação  Gestão  Perfil   │
└─────────────────────────────────────────┘
```

```css
.bottom-nav {
  position: fixed; bottom: 0; left: 0; right: 0;
  height: 64px; padding-bottom: env(safe-area-inset-bottom);
  background: var(--surface); border-top: 1px solid var(--border);
  display: flex; align-items: center; justify-content: space-around;
  z-index: var(--z-sticky);
}
.bottom-nav-item {
  display: flex; flex-direction: column; align-items: center; gap: 2px;
  color: var(--muted-foreground); font-size: 10px; font-family: 'Manrope';
  font-weight: 500; padding: 4px 12px; transition: color var(--duration-moderate) var(--ease-out);
}
.bottom-nav-item.active { color: var(--primary); }
.bottom-nav-item .icon { width: 24px; height: 24px; }
.bottom-nav-fab {
  width: 48px; height: 48px; border-radius: 50%;
  background: var(--primary); color: var(--primary-foreground);
  display: flex; align-items: center; justify-content: center;
  margin-top: -24px; box-shadow: 0 4px 12px oklch(0.715 0.120 84.2 / 0.4);
}
```

### Itens por role

| Role | Bottom nav |
|---|---|
| Morador | [Início] [Buscar] [+Ação] [Benefícios] [Perfil] |
| Síndico | [Início] [Buscar] [+Criar] [Gestão] [Perfil] |
| Empresa | [Início] [Buscar] [+Upload] [Conteúdo] [Perfil] |

### FAB Action Sheet

Ao clicar no **⊕**, abre Bottom Sheet com ações por role:

- **Síndico**: Upload Vídeo, Nova Assembleia, Criar Link
- **Empresa**: Upload Vídeo, Criar Curso
- **Morador**: Gravar Vídeo-Currículo (Banco de Talentos)

## Drawer (mobile sidebar)

```css
.drawer-overlay {
  position: fixed; inset: 0; background: var(--overlay);
  z-index: var(--z-overlay); animation: fadeIn var(--duration-moderate);
}
.drawer {
  position: fixed; top: 0; left: 0; bottom: 0;
  width: 280px; background: var(--surface);
  z-index: var(--z-modal); animation: slideInFromLeft var(--duration-slow) var(--ease-out);
  overflow-y: auto;
}
@keyframes slideInFromLeft {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}
```

## Breadcrumb

```css
.breadcrumb {
  display: flex; align-items: center; gap: 8px;
  padding: 16px 0; font-family: 'Manrope'; font-size: 14px;
}
.breadcrumb-item { color: var(--muted-foreground); }
.breadcrumb-item:hover { color: var(--primary); }
.breadcrumb-separator { color: var(--muted-foreground); }
.breadcrumb-current { color: var(--foreground); font-weight: 600; }
```

## Componentes

`AppShell`, `Sidebar`, `SidebarItem`, `SidebarSectionLabel`, `SidebarPlanCard`, `Topbar`, `TopbarSearch`, `BottomNav`, `BottomNavItem`, `BottomNavFab`, `Drawer`, `DrawerOverlay`, `Breadcrumb`.

## Links

- [[../../../web/ui-catalog|web/ui-catalog]] — catálogo de telas (shell aplica em `cms`, `lms`, `forum`, `assembly`)
- [[../../../web/design-system|web/design-system]] — tokens consumidos
- [[../../../../02-architecture/design-tokens|design-tokens]] — fonte canônica de tokens OKLCH
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
- [[../../../../00-product/business-model#41-matriz-consolidada]] — matriz visibilidade
