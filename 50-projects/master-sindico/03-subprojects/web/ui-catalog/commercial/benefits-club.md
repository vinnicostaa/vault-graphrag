---
title: Benefits Club / Vizinhança Hub — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, commercial, vizinhanca]
bc: commercial
source: _chaos/14-BENEFITS-CLUB.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 14 — Clube de Benefícios / Vizinhança

> **Origem**: `_chaos/14-BENEFITS-CLUB.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 3 · App `forum` (porta 3003) / `cms` · Rotas: `/vizinhanca`, `/vizinhanca/estabelecimento/:id`, `/vizinhanca/meu-negocio` (lojista)
> Hub de integração com comércio local. Gera uso diário e pertencimento.

---

## Regras de negócio

### Conceito
- Aba **Vizinhança** é o hub de comércio local na plataforma.
- **Busca filtrada por CEP** — morador vê apenas benefícios da sua região.
- Estabelecimentos = perfis `role=local_company` (Parceiro Vizinhança).
- Promoções vivem em [[./coupon-engine|coupon-engine]] (13). Este doc cobre: navegação, perfil do estabelecimento, listagem, descoberta por CEP.

### Permissões

| Ação | `base` (resident) | Síndico | Empresa `plus`/`pro` | `local_company` |
|---|---|---|---|---|
| Ver benefícios locais | ✅ | ✅ | ✅ | ✅ |
| Visualizar currículos (Banco Talentos) | — | — | — | — |
| Perfil público | — | — | — | ✅ |
| Cadastrar promoções | — | — | — | ✅ |

### CEP como critério principal
- Morador define CEP no cadastro / configurações.
- Busca retorna estabelecimentos dentro do raio do CEP (raio backend).
- Estabelecimento cadastra CEP de atuação (pode ter mais de um).
- Sem CEP: banner pedindo configuração.

### Perfil do estabelecimento
Nome, Logo / foto, Descrição, Endereço, CEP, Categoria, Horário de funcionamento, Promoção ativa do dia (se houver), conteúdo limitado (até 2 fotos + 1 vídeo 60 s).

---

## Layout — Hub Vizinhança

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  VIZINHANÇA                                            │
│          │  Descubra o melhor da sua região                       │
│          │                                                        │
│          │  📍 CEP: 01310-100 • Bela Vista, SP       [Alterar]   │
│          │                                                        │
│          │  🔍 [Buscar estabelecimento...]                       │
│          │                                                        │
│          │  ─── CATEGORIAS ───                                    │
│          │  [🍕Alim.] [💈Serv.] [🏋️Saúde] [🎭Lazer]              │
│          │  [🛒Merc.] [🧹Limp.] [🔧Rep.] [📦Outros]              │
│          │                                                        │
│          │  ─── PROMOÇÕES DO DIA 🔥 ─── ← horizontal scroll      │
│          │  [Mini Pad. Sol] [Mini Pizza] [Mini Barb.] →          │
│          │                                                        │
│          │  ─── PERTO DE VOCÊ ───                                 │
│          │  ┌────────┐ ┌────────┐                                 │
│          │  │ Mercad. │ │ Lavand. │                                │
│          │  │ Maria   │ │ Express │                                │
│          │  │ ⭐ 4.8  │ │ ⭐ 4.5  │                                │
│          │  │ 200m    │ │ 350m    │                                │
│          │  └────────┘ └────────┘                                 │
└──────────────────────────────────────────────────────────────────┘
```

### CSS — Hub layout

```css
.vizinhanca-hub { max-width: 900px; margin: 0 auto; padding: 0 16px; }
.vizinhanca-cep-bar {
  display: flex; align-items: center; gap: 8px;
  padding: 10px 14px; background: var(--muted);
  border-radius: var(--radius-md); margin-bottom: 20px;
}
.vizinhanca-search-input {
  width: 100%; padding: 12px 16px 12px 44px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  font-family: 'Manrope'; font-size: 15px; color: var(--foreground);
  outline: none; transition: border-color var(--duration-moderate) var(--ease-out);
}
.vizinhanca-search-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.1);
}

/* Category Grid */
.vizinhanca-category-grid {
  display: grid; grid-template-columns: repeat(4, 1fr); gap: 10px;
}
@media (min-width: 768px) {
  .vizinhanca-category-grid { grid-template-columns: repeat(8, 1fr); }
}
.vizinhanca-category-item {
  display: flex; flex-direction: column; align-items: center; gap: 6px;
  padding: 12px 8px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.vizinhanca-category-item:hover {
  border-color: var(--primary); background: oklch(0.715 0.120 84.2 / 0.05);
}
.vizinhanca-category-item.active {
  border-color: var(--primary); background: oklch(0.715 0.120 84.2 / 0.1);
}

/* Promo scroll horizontal */
.vizinhanca-promos-scroll {
  display: flex; gap: 12px; overflow-x: auto;
  scroll-snap-type: x mandatory;
  -webkit-overflow-scrolling: touch; padding-bottom: 8px;
}
.vizinhanca-promos-scroll::-webkit-scrollbar { display: none; }
.vizinhanca-promo-mini {
  min-width: 160px; max-width: 160px; scroll-snap-align: start;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); padding: 12px; flex-shrink: 0;
}

/* Grid "Perto de Você" */
.vizinhanca-nearby-grid {
  display: grid; grid-template-columns: repeat(2, 1fr); gap: 16px;
}
@media (min-width: 1024px) {
  .vizinhanca-nearby-grid { grid-template-columns: repeat(3, 1fr); }
}
@media (max-width: 767px) {
  .vizinhanca-nearby-grid { grid-template-columns: 1fr; }
}
.vizinhanca-store-card {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); overflow: hidden;
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.vizinhanca-store-photo {
  width: 100%; height: 130px; object-fit: cover; background: var(--muted);
}
```

---

## Layout — Perfil do estabelecimento

Rota: `/vizinhanca/estabelecimento/:id`.

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ┌─────────────────────────────────────────────┐   │
│          │  │      [FOTO BANNER / FACHADA — 300px]       │   │
│          │  └─────────────────────────────────────────────┘   │
│          │  ┌────────┐  Padaria Sol                           │
│          │  │ [logo] │  ⭐ 4.8                                │
│          │  │  80px  │  📍 Rua Augusta, 1200 • 200m          │
│          │  └────────┘  🕐 Seg-Sex 6h-20h • Sáb 6h-14h      │
│          │              📂 Alimentação • Padaria              │
│          │                                                    │
│          │  ┌── PROMOÇÃO ATIVA 🔥 ──────────────────┐         │
│          │  │  10% OFF em toda loja                  │         │
│          │  │  Válido para compras > R$30            │         │
│          │  │  ⏰ Até 28/02/2026                     │         │
│          │  │  [🎟️ Pegar Cupom]                      │         │
│          │  └────────────────────────────────────────┘         │
│          │                                                    │
│          │  SOBRE                                              │
│          │  GALERIA (máx 2 fotos)                             │
│          │  VÍDEO institucional (máx 60s)                     │
└──────────────────────────────────────────────────────────────┘
```

### CSS — Store profile (excerto)

```css
.store-banner {
  width: 100%; height: 300px; object-fit: cover;
  border-radius: var(--radius-2xl); background: var(--muted);
}
.store-logo {
  width: 80px; height: 80px;
  border-radius: var(--radius-2xl);
  border: 3px solid var(--card); background: var(--card);
  object-fit: cover; box-shadow: var(--shadow-md);
}
.store-promo-highlight {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border: 1px solid var(--primary);
  border-radius: var(--radius-xl);
  padding: 20px; margin: 24px 0;
}
.store-gallery-img {
  width: 200px; height: 150px;
  object-fit: cover; border-radius: var(--radius-md);
  background: var(--muted); cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.store-video-container {
  width: 100%; aspect-ratio: 16/9;
  background: var(--surface); border-radius: var(--radius-xl); overflow: hidden;
}
```

---

## Layout — Painel "Meu Negócio" (Lojista)

Rota: `/vizinhanca/meu-negocio` — tabs: **Perfil** / **Promoções** / **Métricas**.

Tab **Perfil** inclui: foto, nome, descrição (max 500), endereço, CEP, categoria, horário de funcionamento (grid editável), galeria (máx 2 fotos), vídeo institucional (máx 60 s).

Tab **Promoções** reutiliza componentes de [[./coupon-engine|coupon-engine]].

Tab **Métricas**: cupons gerados (mês), visualizações do perfil, promoções ativas.

### CSS — Tabs (Kobalte)

```css
.meu-negocio-tabs {
  display: flex; gap: 0; border-bottom: 1px solid var(--border);
  margin-bottom: 24px;
}
.meu-negocio-tab {
  padding: 12px 20px; font-family: 'Manrope';
  font-size: 14px; font-weight: 500; color: var(--muted-foreground);
  background: none; border: none;
  border-bottom: 2px solid transparent; cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.meu-negocio-tab:hover { color: var(--foreground); }
.meu-negocio-tab[data-selected] {
  color: var(--primary); border-bottom-color: var(--primary); font-weight: 600;
}

/* Horario grid */
.horario-grid {
  display: grid; grid-template-columns: 80px 1fr 30px 1fr;
  gap: 8px; align-items: center;
}
.horario-input {
  padding: 8px 12px; background: var(--background);
  border: 1px solid var(--input); border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  text-align: center;
}
```

---

## Banner — Sem CEP

```
┌──────────────────────────────────────────────────────────────┐
│  📍 Cadastre seu CEP para ver os benefícios da sua região    │
│     Acesse configurações e defina seu endereço.              │
│                                          [Configurar →]      │
└──────────────────────────────────────────────────────────────┘
```

```css
.vizinhanca-no-cep-banner {
  background: oklch(0.715 0.120 84.2 / 0.08);
  border: 1px solid var(--primary);
  border-radius: var(--radius-xl); padding: 16px 20px;
  display: flex; align-items: center; gap: 12px; margin-bottom: 24px;
}
```

## Componentes

`VizinhancaHub`, `CEPBar`, `CategoryGrid`, `PromoScroll`, `StoreCard`, `StoreProfile`, `StorePromoHighlight`, `MeuNegocioForm`, `GalleryUpload`, `HorarioGrid`, `NoCEPBanner`.

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[./coupon-engine|coupon-engine]] — engine de cupons (13)
- [[../../../../00-product/sub-produtos/08-vizinhanca]]
- [[../../../../00-product/business-model#41-matriz-consolidada]]
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
