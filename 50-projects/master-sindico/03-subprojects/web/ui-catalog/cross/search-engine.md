---
title: Search Engine & Catálogo — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, cross, search]
bc: cross
source: _chaos/07-SEARCH-ENGINE.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 07 — Busca & Catálogo

> **Origem**: `_chaos/07-SEARCH-ENGINE.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: vocabulário neutro; "Morador Pagante" removido (D-126).

> Sprint 1 · App `cms` · Rotas: `/buscar`, `/buscar/:categoria`
> Backend: `ISearchProvider` (D-120) — adapter inicial OpenSearch ou Meilisearch (ADR-0042 pending).

---

## Regras de negócio

- **Busca geral retorna mix**: Síndicos, Empresas, Comércio Local (Parceiros Vizinhança), Vídeos.
- **Filtro por categoria técnica** (Manual Cap. 6 — 31 categorias).
- **Filtro por CEP** (exclusivo para Comércio Local / Vizinhança — feed bairro).
- **Visibilidade ABAC**: perfis só aparecem se a matriz `(viewer_role × viewer_plan_tier × target_role)` permitir.
- **Empresas de Marketing** (`role=marketing`) NÃO aparecem na busca pública (anti-prospecção — ver [[../commercial/connect-me|connect-me]]).

## Layout — Desktop

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  SEARCH BAR (ampliada, full-width)              │
│          │  [🔍 Buscar síndicos, empresas, serviços...]   │
│          │                                                │
│          │  FILTER PILLS (horizontal scroll)              │
│          │  [Todos] [Síndicos] [Empresas] [Comércio]     │
│          │  [+ Categorias ▼]                              │
│          │                                                │
│          │  ┌──────────┐  ┌──────────────────────────┐   │
│          │  │ FILTROS   │  │    RESULTADOS (grid)      │   │
│          │  │           │  │                           │   │
│          │  │ Categorias│  │  [Card] [Card] [Card]     │   │
│          │  │ ☐ Hidrául.│  │  [Card] [Card] [Card]     │   │
│          │  │ ☐ Elétrica│  │                           │   │
│          │  │ ☐ Seguros │  │  Infinite scroll /        │   │
│          │  │ ...       │  │  pagination               │   │
│          │  │           │  │                           │   │
│          │  │ CEP       │  │                           │   │
│          │  │ [_______] │  │                           │   │
│          │  │           │  │                           │   │
│          │  │ Tipo      │  │                           │   │
│          │  │ ○ Vídeos  │  │                           │   │
│          │  │ ○ Perfis  │  │                           │   │
│          │  │ ○ Ambos   │  │                           │   │
│          │  └──────────┘  └──────────────────────────┘   │
└──────────────────────────────────────────────────────────┘
```

## CSS

```css
.search-bar-page {
  width: 100%; max-width: 640px; margin: 0 auto 24px;
  height: 48px; padding: 0 16px;
  background: var(--card); border: 2px solid var(--border); border-radius: 24px;
  font-family: 'Manrope'; font-size: 16px;
  transition: border-color var(--duration-moderate) var(--ease-out);
}
.search-bar-page:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}

.filter-pills {
  display: flex; gap: 8px; overflow-x: auto; padding: 8px 0; scrollbar-width: none;
}
.filter-pill {
  flex-shrink: 0; padding: 8px 16px; border-radius: var(--radius-full);
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  border: 1px solid var(--border); background: var(--muted);
  color: var(--foreground); cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.filter-pill:hover { background: oklch(0.715 0.120 84.2 / 0.1); }
.filter-pill.active {
  background: var(--primary); color: var(--primary-foreground);
  border-color: var(--primary); font-weight: 600;
}

.search-filters-sidebar {
  width: 240px; flex-shrink: 0; padding: 16px;
  border-right: 1px solid var(--border);
}
.search-filter-group { margin-bottom: 24px; }
.search-filter-group-title {
  font-family: 'Michroma'; font-size: 12px; color: var(--muted-foreground);
  text-transform: uppercase; margin-bottom: 12px;
}

.search-results { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; flex: 1; }
@media (max-width: 1024px) { .search-results { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 768px)  { .search-results { grid-template-columns: 1fr; } }
```

## Cards de resultado

### Card de Empresa
Avatar / logo + nome + categorias (tags) + badge `Plus`/`Pro` + descrição truncada.

### Card de Síndico
Avatar + nome + badge status (Bronze / Prata / Ouro / Diamante) + nº condomínios + [Connect Me].

### Card de Comércio Local (Parceiro Vizinhança)
Logo + nome + endereço + CEP + badge "Promoção ativa" se houver.

```css
.search-result-card {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); padding: 16px; display: flex; gap: 16px;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.search-result-card:hover {
  border-color: var(--primary); box-shadow: var(--shadow-sm);
}
.search-result-avatar { width: 48px; height: 48px; border-radius: 50%; flex-shrink: 0; }
.search-result-name { font-family: 'Manrope'; font-size: 16px; font-weight: 600; }
.search-result-meta { font-size: 13px; color: var(--muted-foreground); margin-top: 4px; }
.search-result-tags { display: flex; gap: 4px; flex-wrap: wrap; margin-top: 8px; }
.search-result-tag {
  padding: 2px 8px; font-size: 11px; border-radius: var(--radius-full);
  background: var(--muted); color: var(--muted-foreground);
}
```

## Browse by Categories (Mobile)

```
[Grid de ícones com label]
[Admin] [Jurídico] [Contab.] [Elétrica]
[Hidrául] [Seguros] [Elevad.] [Securit.]
```

Grid 4 colunas, ícones 56 px em circle, label 11 px abaixo.

## Componentes

`SearchPage`, `SearchBar`, `FilterPills`, `FilterSidebar`, `SearchResultCard`, `CategoryBrowseGrid`, `CEPFilter`, `ResultsGrid`.

## Links

- [[../../../../STATE]] — D-120 ISearchProvider
- [[../../../../00-product/business-model#41-matriz-consolidada]] — visibilidade ABAC
- [[../commercial/connect-me|connect-me]] — fluxo pós-busca
- [[../../patterns/ui-states|patterns/ui-states]] — estados obrigatórios
- [[../../../../02-architecture/design-tokens|design-tokens]] — tokens
