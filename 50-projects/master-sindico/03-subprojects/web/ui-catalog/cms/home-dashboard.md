---
title: Home / Dashboard — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, cms, home, dashboard, cross]
bc: cross
source: _chaos/03-HOME-DASHBOARD.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 03 — Home / Dashboard

> **Origem**: `_chaos/03-HOME-DASHBOARD.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".

> Transversal · App `cms` · Rota: `/` (index), `/explorar` (mobile discover)
> Referências: YouTube Home + VoiceLabs Dashboard + Humanified Discover.

---

## Conceito

A Home é **híbrida**: dashboard + feed. Adapta seções conforme `role` e `plan_tier` do usuário logado.
NÃO é puro YouTube feed — tem greeting personalizado, quick actions e seções contextuais.

## Layout — Desktop

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

## Layout — Mobile (`/explorar`)

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

## Saudação personalizada

```css
.greeting {
  font-family: 'Manrope'; font-size: 20px; font-weight: 400;
  color: var(--foreground); margin-bottom: 4px;
}
.greeting-subtitle {
  display: flex; align-items: center; gap: 8px;
  font-size: 14px; color: var(--muted-foreground);
}
```

Período: "Bom dia" (5–12 h), "Boa tarde" (12–18 h), "Boa noite" (18–5 h).

## Quick Action Cards

```css
.quick-actions {
  display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; margin: 16px 0;
}
@media (max-width: 768px) {
  .quick-actions { grid-template-columns: repeat(2, 1fr); }
}
.quick-action-card {
  display: flex; flex-direction: column; align-items: center; gap: 8px;
  padding: 20px 16px; background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.quick-action-card:hover {
  border-color: var(--primary);
  transform: translateY(-1px);
  box-shadow: var(--shadow-md);
}
.quick-action-icon {
  width: 48px; height: 48px; border-radius: var(--radius-xl);
  display: flex; align-items: center; justify-content: center;
  font-size: 24px;
}
.quick-action-icon.upload   { background: oklch(0.715 0.120 84.2 / 0.15); color: var(--primary); }
.quick-action-icon.assembly { background: oklch(0.627 0.170 149.2 / 0.15); color: var(--success); }
.quick-action-icon.search   { background: oklch(0.568 0.200 26.4 / 0.10); color: var(--info); }
.quick-action-label {
  font-family: 'Manrope'; font-size: 13px; font-weight: 600; color: var(--foreground);
}
```

### Por role

| Role × tier | Quick actions |
|---|---|
| Síndico `pro` | Meus Vídeos, Assembleias, Transparência, Cursos, Buscar, Connect Me |
| Empresa `pro` | Meus Vídeos, Meus Cursos, Currículos (Banco Talentos), Fórum, Métricas, Buscar |
| Morador (todos) | Assembleias, Banco de Talentos, Vizinhança, Buscar, Perfil, Configurações |
| `base` (qualquer role) | Buscar, Vídeos, Assembleias (se vinculado), Vizinhança |

> Morador é gratuito por padrão (`plan_tier=base`). Banco de Talentos é addon livre (D-099). Não há mais "Morador Pagante" (D-126).

## Category Icons Row

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
  width: 56px; height: 56px; border-radius: var(--radius-xl);
  display: flex; align-items: center; justify-content: center;
  background: var(--muted); transition: all var(--duration-moderate) var(--ease-out);
}
.category-icon-circle .icon { width: 24px; height: 24px; color: var(--muted-foreground); }
.category-icon.active .category-icon-circle { background: var(--primary); }
.category-icon.active .category-icon-circle .icon { color: var(--primary-foreground); }
.category-icon-label {
  font-family: 'Manrope'; font-size: 11px;
  color: var(--muted-foreground); white-space: nowrap;
}
.category-icon.active .category-icon-label { color: var(--primary); font-weight: 600; }
```

## Seções da Home

```css
.home-section { margin-bottom: 32px; }
.home-section-header {
  display: flex; justify-content: space-between; align-items: center;
  margin-bottom: 16px;
}
.home-section-title { font-family: 'Michroma'; font-size: 18px; color: var(--foreground); }
.home-section-link {
  font-family: 'Manrope'; font-size: 14px; color: var(--primary);
  font-weight: 500; cursor: pointer;
}
.home-section-link:hover { text-decoration: underline; }
```

## Seções condicionais

| Seção | Visível para |
|---|---|
| Assembleias Recentes | Moradores / Síndicos com condomínio vinculado |
| Comércio Local | Todos com CEP cadastrado |
| Cursos Recomendados | Síndico `plus`+, Empresa `plus`+ |
| Lives Ativas | Todos exceto `base` (badge "AO VIVO" vermelho pulsante) |
| Vídeos de Síndicos | Todos |
| Vídeos de Empresas | Todos (preview 25 % para `base`) |

> Restrição "preview 25 % para `base`" → ver [[../../../../00-product/business-model#41-matriz-consolidada]].

## Quota Widget (sidebar ou inline)

Aparece quando uso > 50 %. Útil para Connect Me anual e vídeos mensais.

```css
.quota-widget {
  display: flex; align-items: center; gap: 12px;
  padding: 12px; background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md);
}
.quota-circle { width: 48px; height: 48px; position: relative; }
.quota-circle svg { transform: rotate(-90deg); }
.quota-circle-track { stroke: var(--muted); stroke-width: 4; fill: none; }
.quota-circle-fill {
  stroke: var(--primary); stroke-width: 4; fill: none;
  stroke-linecap: round; transition: stroke-dashoffset var(--duration-slower);
}
.quota-text { font-family: 'Manrope'; }
.quota-title { font-size: 14px; font-weight: 600; color: var(--foreground); }
.quota-description { font-size: 12px; color: var(--muted-foreground); }
```

## Componentes

`HomePage`, `HomeGreeting`, `QuickActionGrid`, `QuickActionCard`, `CategoryIconRow`, `CategoryIcon`, `HomeSection`, `HomeSectionHeader`, `QuotaWidget`, `LiveBadge`.

## Links

- [[../../../web/ui-catalog#b1-morador-m1-m15]] — telas M1-M15 morador
- [[../../../web/ui-catalog#b2-síndico-s1-s31--compliance-c1-c11]] — S1-S31 síndico
- [[../../../../00-product/business-model]] — matriz `plan_tier`
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
- [[../../../../02-architecture/design-tokens|design-tokens]] — tokens
