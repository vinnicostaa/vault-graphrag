# 07 — BUSCA & CATÁLOGO

> Sprint 1 · Rotas: /buscar, /buscar/:categoria
> Referências: YouTube Search + Genaigurus Browse + VoiceLabs Explore

---

## REGRAS DE NEGÓCIO

- Busca geral retorna mix: Síndicos, Empresas, Comércio Local, Vídeos
- Filtro por categoria técnica (Manual Cap. 6 — 31 categorias)
- Filtro por CEP (exclusivo para Comércio Local/Vizinhança)
- Perfis só aparecem se Matriz Funcional permitir (middleware de visibilidade)
- Empresas de Marketing NÃO aparecem na busca pública

## LAYOUT — DESKTOP

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
  transition: border-color 200ms;
}
.search-bar-page:focus { border-color: var(--primary); box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15); }

.filter-pills { display: flex; gap: 8px; overflow-x: auto; padding: 8px 0; scrollbar-width: none; }
.filter-pill {
  flex-shrink: 0; padding: 8px 16px; border-radius: 9999px;
  font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  border: 1px solid var(--border); background: var(--muted);
  color: var(--foreground); cursor: pointer; transition: all 200ms;
}
.filter-pill:hover { background: oklch(0.715 0.120 84.2 / 0.1); }
.filter-pill.active { background: var(--primary); color: var(--primary-foreground); border-color: var(--primary); font-weight: 600; }

.search-filters-sidebar {
  width: 240px; flex-shrink: 0; padding: 16px;
  border-right: 1px solid var(--border);
}
.search-filter-group { margin-bottom: 24px; }
.search-filter-group-title { font-family: 'Michroma'; font-size: 12px; color: var(--muted-foreground); text-transform: uppercase; margin-bottom: 12px; }

.search-results { display: grid; grid-template-columns: repeat(3, 1fr); gap: 16px; flex: 1; }
@media (max-width: 1024px) { .search-results { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 768px) { .search-results { grid-template-columns: 1fr; } }
```

## CARDS DE RESULTADO

### Card de Empresa
Avatar/logo + nome + categorias (tags) + badge "Pro" + descrição truncada

### Card de Síndico
Avatar + nome + badge status (Bronze/Prata/Ouro/Diamante) + nº condomínios + [Connect Me]

### Card de Comércio Local
Logo + nome + endereço + CEP + badge "Promoção ativa" se houver

```css
.search-result-card {
  background: var(--card); border: 1px solid var(--border);
  border-radius: 12px; padding: 16px; display: flex; gap: 16px;
  cursor: pointer; transition: all 200ms;
}
.search-result-card:hover { border-color: var(--primary); box-shadow: 0 2px 8px rgb(0 0 0 / 0.05); }
.search-result-avatar { width: 48px; height: 48px; border-radius: 50%; flex-shrink: 0; }
.search-result-name { font-family: 'Manrope'; font-size: 16px; font-weight: 600; }
.search-result-meta { font-size: 13px; color: var(--muted-foreground); margin-top: 4px; }
.search-result-tags { display: flex; gap: 4px; flex-wrap: wrap; margin-top: 8px; }
.search-result-tag {
  padding: 2px 8px; font-size: 11px; border-radius: 9999px;
  background: var(--muted); color: var(--muted-foreground);
}
```

## BROWSE BY CATEGORIES (Mobile)

```
[Grid de ícones com label]
[Admin] [Jurídico] [Contab.] [Elétrica]
[Hidrául] [Seguros] [Elevad.] [Securit.]
```

Grid 4 colunas, ícones 56px em circle, label 11px abaixo.

## COMPONENTES

SearchPage, SearchBar, FilterPills, FilterSidebar, SearchResultCard, CategoryBrowseGrid, CEPFilter, ResultsGrid
