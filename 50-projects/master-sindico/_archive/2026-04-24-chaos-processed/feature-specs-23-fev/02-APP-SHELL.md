# 02 — APP SHELL (Sidebar, Topbar, Bottom Nav)

> Transversal · Layout principal da aplicação autenticada
> Componentes: AppShell, Sidebar, Topbar, BottomNav, Drawer

---

## ESTRUTURA MACRO — DESKTOP (≥1024px)

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

## ESTRUTURA MACRO — MOBILE (<768px)

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

## TOPBAR

### Desktop

```
┌──────────────────────────────────────────────────────────────┐
│  [☰]  [🛡 MASTER SÍNDICO]         [🔍 Buscar síndicos,     │
│         Michroma gold               empresas, serviços...]   │
│                                   [🌙/☀️] [🔔 •] [Avatar▼]  │
└──────────────────────────────────────────────────────────────┘
```

### CSS

```css
.topbar {
  height: 64px; /* 56px mobile */
  background: var(--surface);
  border-bottom: 1px solid var(--border);
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 16px;
  position: sticky; top: 0; z-index: 50;
}
.topbar-logo {
  font-family: 'Michroma'; color: var(--primary);
  font-size: 14px; letter-spacing: 0.05em;
}
.topbar-search {
  flex: 1; max-width: 480px; margin: 0 24px;
  height: 40px; padding: 0 12px;
  background: var(--surface-variant); border: 1px solid var(--border);
  border-radius: var(--radius); color: var(--surface-foreground);
  font-family: 'Manrope'; font-size: 14px;
}
.topbar-search:focus { border-color: var(--ring); box-shadow: 0 0 0 2px oklch(0.715 0.120 84.2 / 0.2); }
.topbar-search::placeholder { color: var(--muted-foreground); }
.topbar-actions { display: flex; align-items: center; gap: 8px; }
.topbar-icon-btn {
  width: 36px; height: 36px; border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  color: var(--surface-foreground); transition: background 200ms;
}
.topbar-icon-btn:hover { background: var(--surface-variant); }
.topbar-avatar {
  width: 32px; height: 32px; border-radius: 50%;
  border: 2px solid transparent; cursor: pointer;
}
/* Pro plan: gold border on avatar */
.topbar-avatar[data-plan="pro"] { border-color: var(--primary); }
.topbar-notification-dot {
  position: absolute; top: 6px; right: 6px;
  width: 8px; height: 8px; border-radius: 50%;
  background: var(--destructive);
}
```

### Mobile

- Logo: apenas brasão (sem texto)
- Search: ícone que abre fullscreen search overlay
- Hamburger: abre Drawer (sidebar overlay)
- Height: 56px

## SIDEBAR

### Estrutura de Itens

```
PRINCIPAL
  🏠 Início
  🔍 Buscar
  📹 Vídeos
  👤 Meu Perfil

MINHA GESTÃO (apenas Síndico N2/N3)
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
  transition: width 300ms cubic-bezier(0.4, 0, 0.2, 1);
  z-index: 40;
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
  border-radius: var(--radius); color: var(--surface-foreground);
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  cursor: pointer; transition: all 200ms; position: relative;
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

### Widget de Plano (fundo sidebar)

```css
.sidebar-plan-card {
  margin: auto 8px 16px; padding: 12px;
  background: var(--surface-variant); border: 1px solid oklch(0.715 0.120 84.2 / 0.3);
  border-radius: var(--radius);
}
.sidebar-plan-name { font-family: 'Michroma'; font-size: 12px; color: var(--primary); }
.sidebar-plan-status { font-family: 'Manrope'; font-size: 12px; color: var(--surface-foreground); margin-top: 4px; }
```

### Badge "NEW" em item

```css
.sidebar-new-badge {
  font-size: 10px; font-weight: 700;
  background: var(--primary); color: var(--primary-foreground);
  padding: 1px 6px; border-radius: 9999px;
  margin-left: auto;
}
```

### Visibilidade ABAC

Itens NÃO mostram cadeado. Se o plano não permite, o item NÃO renderiza.

| Item | Visível para |
|---|---|
| Assembleias | Síndico N2, N3 |
| Transparência | Síndico N3 |
| Avaliações | Síndico N3 |
| Exportar PDF | Síndico N3 |
| Cursos | Síndico N2+, Emp Pro, Marketing |
| Fórum | Síndico N1+, Emp Plus+, Marketing |
| Lives | Todos exceto Base |
| Banco de Talentos | Emp Plus+, Marketing, Vizinhança |

## BOTTOM NAVIGATION (Mobile)

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
  z-index: 50;
}
.bottom-nav-item {
  display: flex; flex-direction: column; align-items: center; gap: 2px;
  color: var(--muted-foreground); font-size: 10px; font-family: 'Manrope';
  font-weight: 500; padding: 4px 12px; transition: color 200ms;
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

### Itens por Role

Morador: [Início] [Buscar] [+Ação] [Benefícios] [Perfil]
Síndico: [Início] [Buscar] [+Criar] [Gestão] [Perfil]
Empresa: [Início] [Buscar] [+Upload] [Conteúdo] [Perfil]

### FAB Action Sheet

Ao clicar no "⊕", abre Bottom Sheet com ações por role:
- Síndico: Upload Vídeo, Nova Assembleia, Criar Link
- Empresa: Upload Vídeo, Criar Curso
- Morador: Gravar Vídeo-Currículo

## DRAWER (Mobile Sidebar)

```css
.drawer-overlay {
  position: fixed; inset: 0; background: rgba(0,0,0,0.5);
  z-index: 60; animation: fadeIn 0.2s;
}
.drawer {
  position: fixed; top: 0; left: 0; bottom: 0;
  width: 280px; background: var(--surface);
  z-index: 61; animation: slideInFromLeft 0.3s ease-out;
  overflow-y: auto;
}
@keyframes slideInFromLeft {
  from { transform: translateX(-100%); }
  to { transform: translateX(0); }
}
```

## BREADCRUMB

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

## COMPONENTES

AppShell, Sidebar, SidebarItem, SidebarSectionLabel, SidebarPlanCard, Topbar, TopbarSearch, BottomNav, BottomNavItem, BottomNavFab, Drawer, DrawerOverlay, Breadcrumb
