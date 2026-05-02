# 03 — HOME / DASHBOARD

> Transversal · Rota: / (index), /explorar (mobile discover)
> Referências: YouTube Home + VoiceLabs Dashboard + Humanified Discover

---

## CONCEITO

Home é HÍBRIDA: dashboard + feed. Adapta seções conforme role do usuário logado.
NÃO é puro YouTube feed — tem greeting personalizado, quick actions e seções contextuais.

## LAYOUT — DESKTOP

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Bom dia, João! 👋                             │
│          │  Manrope 400 20px foreground                   │
│          │  Síndico Pro • Status Ouro ← badges inline     │
│          │                                                │
│          │  QUICK ACTIONS (grid 2x3 ou 3x2)              │
│          │  ┌──────┐ ┌──────┐ ┌──────┐                   │
│          │  │📹    │ │📋    │ │📊    │                   │
│          │  │Upload │ │Assemb│ │Gestão │                   │
│          │  └──────┘ └──────┘ └──────┘                   │
│          │  ┌──────┐ ┌──────┐ ┌──────┐                   │
│          │  │🔍    │ │💬    │ │⚙     │                   │
│          │  │Buscar │ │Fórum │ │Config │                   │
│          │  └──────┘ └──────┘ └──────┘                   │
│          │                                                │
│          │  ── CATEGORY ICONS (scroll horizontal) ──      │
│          │  [Para Você] [Popular] [Manutenção] [Elétrica] │
│          │                                                │
│          │  ── Recomendados para você ──  [Ver todos →]   │
│          │  [VideoCard] [VideoCard] [VideoCard] [VC]      │
│          │                                                │
│          │  ── Assembleias Recentes ──   [Ver todas →]    │
│          │  [AssemblyCard] [AssemblyCard]                  │
│          │                                                │
│          │  ── Vídeos de Empresas ──     [Ver todos →]    │
│          │  [VideoCard] [VideoCard] [VideoCard] [VC]      │
│          │                                                │
│          │  ── Comércio Local ──         [Ver todos →]    │
│          │  [PromoCard] [PromoCard] [PromoCard]           │
└──────────────────────────────────────────────────────────┘
```

## LAYOUT — MOBILE (/explorar)

```
┌────────────────────────────┐
│  Bom dia, João! 👋         │
│  Síndico Pro               │
│                            │
│  [🔍 Buscar...]            │
│                            │
│  CATEGORY ICONS (scroll)   │
│  [🏠][🔍][📋][🎓][🏪][📺]│
│  Home Busca Assemb Cursos..│
│                            │
│  Recomendados  [Ver todos] │
│  [VC] [VC] [VC] → scroll  │
│                            │
│  Assembleias   [Ver todas] │
│  [AC] [AC]                 │
│                            │
│  Vizinhança    [Ver todos] │
│  [PC] [PC] [PC]           │
│                            │
│  BOTTOM NAV               │
└────────────────────────────┘
```

## SAUDAÇÃO PERSONALIZADA

```css
.greeting { font-family: 'Manrope'; font-size: 20px; font-weight: 400; color: var(--foreground); margin-bottom: 4px; }
.greeting-subtitle { display: flex; align-items: center; gap: 8px; font-size: 14px; color: var(--muted-foreground); }
```

Período: "Bom dia" (5-12h), "Boa tarde" (12-18h), "Boa noite" (18-5h)

## QUICK ACTION CARDS

```css
.quick-actions { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin: 16px 0; }
@media (max-width: 768px) { .quick-actions { grid-template-columns: repeat(2, 1fr); } }
.quick-action-card {
  display: flex; flex-direction: column; align-items: center; gap: 8px;
  padding: 20px 16px; background: var(--card); border: 1px solid var(--border);
  border-radius: 12px; cursor: pointer; transition: all 200ms;
}
.quick-action-card:hover { border-color: var(--primary); transform: translateY(-1px); box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1); }
.quick-action-icon {
  width: 48px; height: 48px; border-radius: 12px;
  display: flex; align-items: center; justify-content: center;
  font-size: 24px;
}
/* Cores variadas por ação */
.quick-action-icon.upload { background: oklch(0.715 0.120 84.2 / 0.15); color: var(--primary); }
.quick-action-icon.assembly { background: oklch(0.627 0.170 149.2 / 0.15); color: var(--success); }
.quick-action-icon.search { background: oklch(0.568 0.200 26.4 / 0.1); color: #6366F1; }
.quick-action-label { font-family: 'Manrope'; font-size: 13px; font-weight: 600; color: var(--foreground); }
```

### Por Role

Síndico N3: Meus Vídeos, Assembleias, Transparência, Cursos, Buscar, Connect Me
Empresa Pro: Meus Vídeos, Meus Cursos, Currículos, Fórum, Métricas, Buscar
Morador Pg: Assembleias, Vídeo-Currículo, Vizinhança, Buscar, Perfil, Configurações
Base: Buscar, Vídeos, Assembleias (se vinculado), Vizinhança

## CATEGORY ICONS ROW

```css
.category-icons {
  display: flex; gap: 12px; overflow-x: auto; padding: 8px 0;
  scrollbar-width: none; scroll-snap-type: x mandatory;
}
.category-icons::-webkit-scrollbar { display: none; }
.category-icon {
  display: flex; flex-direction: column; align-items: center; gap: 6px;
  flex-shrink: 0; cursor: pointer; scroll-snap-align: start;
}
.category-icon-circle {
  width: 56px; height: 56px; border-radius: 12px;
  display: flex; align-items: center; justify-content: center;
  background: var(--muted); transition: all 200ms;
}
.category-icon-circle .icon { width: 24px; height: 24px; color: var(--muted-foreground); }
.category-icon.active .category-icon-circle { background: var(--primary); }
.category-icon.active .category-icon-circle .icon { color: var(--primary-foreground); }
.category-icon-label { font-family: 'Manrope'; font-size: 11px; color: var(--muted-foreground); white-space: nowrap; }
.category-icon.active .category-icon-label { color: var(--primary); font-weight: 600; }
```

## SEÇÕES DA HOME

```css
.home-section { margin-bottom: 32px; }
.home-section-header {
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 16px;
}
.home-section-title { font-family: 'Michroma'; font-size: 18px; color: var(--foreground); }
.home-section-link { font-family: 'Manrope'; font-size: 14px; color: var(--primary); font-weight: 500; cursor: pointer; }
.home-section-link:hover { text-decoration: underline; }
```

## SEÇÕES CONDICIONAIS

| Seção | Visível para |
|---|---|
| Assembleias Recentes | Moradores/Síndicos com condomínio vinculado |
| Comércio Local | Todos com CEP cadastrado |
| Cursos Recomendados | Síndico N2+, Emp Plus+ |
| Lives Ativas | Todos exceto Base (badge "AO VIVO" vermelho pulsante) |
| Vídeos de Síndicos | Todos |
| Vídeos de Empresas | Todos (preview 25% para Base) |

## QUOTA WIDGET (sidebar ou inline)

Aparece quando uso > 50%.

```css
.quota-widget {
  display: flex; align-items: center; gap: 12px;
  padding: 12px; background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius);
}
.quota-circle { width: 48px; height: 48px; position: relative; }
.quota-circle svg { transform: rotate(-90deg); }
.quota-circle-track { stroke: var(--muted); stroke-width: 4; fill: none; }
.quota-circle-fill { stroke: var(--primary); stroke-width: 4; fill: none; stroke-linecap: round; transition: stroke-dashoffset 500ms; }
.quota-text { font-family: 'Manrope'; }
.quota-title { font-size: 14px; font-weight: 600; color: var(--foreground); }
.quota-description { font-size: 12px; color: var(--muted-foreground); }
```

## COMPONENTES

HomePage, HomeGreeting, QuickActionGrid, QuickActionCard, CategoryIconRow, CategoryIcon, HomeSection, HomeSectionHeader, QuotaWidget, LiveBadge
