---
title: Case study — Primer (GitHub Design System Architecture)
type: note
tags:
  - ui-design
  - case-study
  - primer
  - github
  - design-system
  - multi-product
doc-oficial: https://primer.style
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Primer case study
  - GitHub DS case
  - Primer architecture
---

# Case Study — Primer (GitHub Design System)

**Contexto**: GitHub opera como platform com **múltiplos produtos e superfícies**:
- **github.com app** (main product — repo browsing, issues, PRs, Actions) — Ruby on Rails + ViewComponents + React islands.
- **github.com marketing** (landing pages, About, Enterprise, Pricing) — Next.js.
- **GitHub Mobile** (iOS, Android) — nativo.
- **VS Code extension (Copilot, PR review)** — web views.
- **GitHub Desktop** — Electron.
- **Enterprise Cloud + Enterprise Server** (on-prem) — mesmo stack que github.com.

**Desafio**: 50+ teams, 10+ ambientes técnicos distintos, bilhões de interações diárias, **UX consistente + brand identity + acessibilidade WCAG obrigatória**. Como?

**Resposta**: **Primer** — arquitetura multi-camada de design system, open-source, adotada como reference pela indústria.

> Primer é excelente **case concreto** de [[design-system-architecture]] aplicado em escala real. Este doc extrai as decisões arquiteturais + como elas atendem multi-product, multi-platform, multi-brand.

## Arquitetura Primer — 6 packages fundamentais

```
primer/primitives     ── JSON design tokens (fonte de verdade)
       ▲
       │
   ┌───┴───────────────────┬───────────────────┬─────────────────┐
   │                       │                   │                 │
primer/css           primer/react       primer/view_components   primer/brand
(utility CSS)        (React components) (Rails ViewComponents)   (marketing)
                                                                      │
                                                                      ▼
                                                                primer/figma
                                                                (Figma library)
```

- **`primer/primitives`** — JSON tokens (cor, spacing, typography, motion). Source of truth. Output build: CSS vars, JS constants, Figma plugin format. Todas as outras libs **consomem** este package.
- **`primer/react`** — React component library (TypeScript). Componentes ricos (Button, Dialog, DataTable, etc.). Primary em apps novas github.com.
- **`primer/css`** — utility CSS classes (Tailwind-like). Para páginas server-rendered sem React (Rails ViewComponents consumer). Gradualmente sendo deprecated em favor de Rails + ViewComponents.
- **`primer/view_components`** — Rails ViewComponents (gem `primer/view_components`). Equivalente a primer/react mas para ERB/Rails. Usado em grande parte do github.com monolith.
- **`primer/brand`** — React components e primitives para github.com marketing (landing pages). Tokens **diferentes** de primer product — tipografia mais expressiva, hero patterns, marketing-specific components.
- **`primer/octicons`** — biblioteca de ícones SVG (scalable handcrafted icons). Distribuída como React components, CSS sprite, Figma assets.

## Layers de tokens (primer/primitives)

Primer adotou arquitetura **em camadas** canônica (ver [[design-tokens]]):

```
base/
  scale/color/gray/0..10
  scale/color/blue/0..9
  scale/typography/*

functional/
  themes/light.json
    "fgColor.default": "{scale.color.gray.9}"
    "bgColor.default": "{scale.color.gray.0}"
    "accent.fgColor":  "{scale.color.blue.5}"
  themes/dark.json
    "fgColor.default": "{scale.color.gray.1}"
    "bgColor.default": "{scale.color.gray.9}"

component/
  button/
    primary/
      fgColor.rest:    "{fgColor.onEmphasis}"
      bgColor.rest:    "{bgColor.accent.emphasis}"
      bgColor.hover:   "{control.bgColor.primary.hover}"
```

**3 camadas: base → functional → component**. Themes substituem **apenas functional** (dark/high-contrast/etc.). Components herdam automaticamente.

Temas suportados:
- `light`
- `light_colorblind` (protanopia/deuteranopia friendly)
- `light_tritanopia`
- `light_high_contrast`
- `dark`
- `dark_dimmed` (menos contraste que dark)
- `dark_high_contrast`
- `dark_colorblind`
- `dark_tritanopia`

**9 temas** mantidos — acessibilidade é first-class, não afterthought.

## Multi-product com 1 fonte de verdade

```
                        primer/primitives
                          (JSON tokens)
                                │
                    ┌───────────┼───────────┬─────────────────┐
                    │           │           │                 │
         ┌──────────▼─┐  ┌──────▼──────┐  ┌─▼──────────┐   ┌─▼──────────┐
         │ primer/    │  │ primer/react │  │ primer/css │   │ primer/view│
         │ figma      │  │ (TS)         │  │ (utility)  │   │ _components│
         │ (design)   │  │              │  │            │   │ (Ruby)     │
         └──────┬─────┘  └──────┬───────┘  └─────┬──────┘   └──────┬─────┘
                │               │                │                  │
                ▼               ▼                ▼                  ▼
         Figma libraries    github.com     Enterprise         github.com Rails
         Spec.github.com    (React apps)   legacy pages       monolith
                            VS Code ext
                            primer.style
                            (docs site)
```

Designer edita **uma biblioteca Figma** → sync com primitives JSON → todas as apps consomem.

**Não existe**:
- `@github/button-v2` que é versão "nova" reinventada.
- Componente "similar" replicado entre apps.
- Color picker livre no Figma (só paleta aprovada).

## primer/react — arquitetura interna

### Componentes organizados por tipo

```
packages/react/src/
  Button/
    Button.tsx
    IconButton.tsx
    Button.stories.tsx          ← Storybook
    Button.test.tsx
    Button.docs.json             ← docs generation
  FormControl/
  DataTable/
  ...
```

### Props API — composição > configuração

Princípios:
- **`sx` prop** para overrides pontuais (styled-system heritage).
- **Composição** via slots em vez de mega-props. Ex.:

```tsx
// Bad: 30 props
<Dialog
  title="..."
  titleIcon={<X />}
  primaryButton={{ label: 'Ok', onClick: ... }}
  secondaryButton={...}
  showFooter
/>

// Good: composição
<Dialog>
  <Dialog.Header>
    <Dialog.Title icon={<X />}>...</Dialog.Title>
  </Dialog.Header>
  <Dialog.Body>...</Dialog.Body>
  <Dialog.Footer>
    <Button variant="primary">Ok</Button>
    <Button variant="secondary">Cancel</Button>
  </Dialog.Footer>
</Dialog>
```

- **Variants** declarativas: `<Button variant="primary|secondary|danger|invisible">`.
- **Sizes**: `small`, `medium`, `large`.
- **Responsive props** (aceita objeto `{ narrow, wide }`).

### Accessibility first-class

Cada componente:
- ARIA roles/states configurados corretamente.
- Keyboard navigation testada.
- Focus management.
- Reduced motion respected.
- Testing: `@storybook/addon-a11y`, `axe-core` em CI.

## primer/view_components — ponte para Rails monolith

github.com é **monolith Ruby on Rails** há 15 anos. Frontend é **mix** de Rails views (ERB + ViewComponents) e React islands (Hotwire-adjacent).

ViewComponents são Ruby classes com templates ERB:

```ruby
# app/components/alert_component.rb
class AlertComponent < Primer::Alpha::Dialog
  def initialize(title:, severity: :info)
    @title = title
    @severity = severity
  end
end
```

```erb
<%# app/components/alert_component.html.erb %>
<div class="Alert Alert--<%= @severity %>">
  <h3><%= @title %></h3>
  <%= content %>
</div>
```

Uso em view Rails:
```erb
<%= render AlertComponent.new(title: "Deploy failed", severity: :error) do %>
  Rollback em progresso.
<% end %>
```

Isso permite **reuso entre páginas server-rendered e client-rendered**. Tokens compartilhados garantem visual idêntico.

## primer/brand — why separate from product

**Produto (github.com app)**:
- **Denso, funcional**, text-heavy.
- Paleta neutra + accent.
- Typography optimized para leitura longa.
- Layouts grid rígido.

**Marketing (github.com/home, /enterprise, /pricing)**:
- **Expressive, visual**, image-heavy.
- Paleta vibrante + gradients + brand colors.
- Typography display (large hero type).
- Layouts editorial (breakouts, full-bleed).

**Não faz sentido** usar os mesmos componentes. Mas **compartilham foundations**: Octicons, brand guidelines, accessibility, motion principles.

Primer Brand:
- Hero patterns (RiverBreakout, Hero, Pillar).
- MinimalFooter, SubNav.
- Animation primitives (AnimationProvider).

Tokens separados mas **derivados** de primer/primitives base.

## Governance GitHub

- **Primer team** dedicado (~15 engs + designers).
- **Design Infrastructure** team separado (build, CI, docs tooling).
- **Contribution model**: PR aberto; reviewers Primer team; critérios públicos.
- **RFC process** para componentes novos (proposta em issue → discussion → aprovação → implementação).
- **Office hours** semanais para Q&A de teams de produto.
- **Adoption metrics** — dashboards internos de "% componentes Primer" por app; uso direto tracked.

## Release process

- **Changesets** — cada PR com change relevante inclui `.changeset/<hash>.md` descrevendo change + type (patch/minor/major).
- **Bot aggregates** changesets → "Release" PR com CHANGELOG gerado.
- **Merge → publish** automático npm + GitHub Release.
- **Canary builds** em cada PR para teste em consumidor antes de merge.
- **Deprecation**: `@deprecated` JSDoc + CHANGELOG flag + migration guide + codemod (às vezes).

## Material pulled from GitHub engineering (public posts)

Exemplos documentados publicamente:
- **Migration Primer CSS → Primer View Components** (multi-year migration; github/github.com + github/primer blogs).
- **Primer React v32 → v33** breaking changes com codemods.
- **Adoption of Hotwire + Turbo** para pages server-rendered (reduzindo React surface).
- **Monorepo structure** (pnpm workspaces; Turborepo build).
- **Docs site primer.style** gerado do source (React Static + MDX).

## Lições aplicáveis (replicar em outro produto)

### 1. Multi-camada é o padrão em escala

Single package "@company/ui" funciona até ~5 apps. Acima disso, separar em:
- `@company/tokens` (JSON source of truth).
- `@company/ui` (React, main product).
- `@company/brand` (marketing) — opcional.
- `@company/icons`.
- `@company/patterns` (recipes) — opcional.

### 2. Tokens em JSON + Style Dictionary = multiplicador

1 JSON gera CSS, JS, iOS Swift, Android XML, Figma. Rebrand = edit 1 arquivo.

### 3. Composição > Configuração

Mega-props (`<Dialog props1={...} props2={...} ... props30>`) viram legacy. Slot pattern (`<Dialog.Header>`) escala.

### 4. Governance é arquitetura

Sem time dedicado + RFC process + adoption metrics, DS vira pet project. Primer tem ~15 devs full-time.

### 5. Server + Client components

Não force um stack. Primer tem React, ViewComponents Ruby, e CSS puro — serve qualquer app GitHub.

### 6. Acessibilidade built-in

Não é feature; é **requisito** de cada componente. Primer valida automaticamente em CI (axe-core + visual regression).

### 7. Multi-theme é a prova de fogo

9 temas no Primer (light + dark + colorblind variants + high contrast × 2). Se arquitetura token aguenta isso, aguenta white-label multi-tenant depois.

### 8. Documentação é produto

primer.style é site com docs por componente, examples, guidelines, Figma assets. Investimento proporcional ao código.

## Patterns replicáveis para SaaS multi-app (ex.: Master Síndico style)

Quando produto tem **multiple apps** compartilhando stack (auth/cms/lms/forum/assembly/landing):

1. **`packages/ui`** em monorepo — Primer role.
2. **`packages/tokens`** em JSON — Primer Primitives role.
3. **`packages/brand`** separado se marketing tem UX distinto.
4. **Figma sync** via Tokens Studio (ou similar).
5. **Changesets** + automated release.
6. **Storybook** em subdomain (`design.company.com`).
7. **Contribution docs** + RFC template.
8. **CI a11y check** obrigatório.

## Anti-patterns observados (mesmo no Primer)

- **Breaking changes sem codemod** — Primer React v33 teve churn significativo; time produto gastou sprints migrando.
- **Docs site FOC** em algumas eras — updates lentos após component release.
- **CSS utility (primer/css) conflito com React styled** — duas maneiras de fazer mesma coisa; deprecation complexa.
- **Rails + React islands** — 2 rendering models no mesmo page; state management spaghetti em alguns pontos.

Reconhecer que **mesmo Primer não é perfeito** ajuda a calibrar expectations.

## Referências oficiais

- [primer.style (docs)](https://primer.style/) (consultada 2026-04-24)
- [github.com/primer organization](https://github.com/primer) (consultada 2026-04-24)
- [primer/primitives](https://github.com/primer/primitives) (consultada 2026-04-24)
- [primer/react](https://github.com/primer/react) (consultada 2026-04-24)
- [primer/view_components](https://github.com/primer/view_components) (consultada 2026-04-24)
- [primer/brand](https://github.com/primer/brand) (consultada 2026-04-24)
- [primer/octicons](https://github.com/primer/octicons) (consultada 2026-04-24)
- [GitHub Design](https://design.github.com/) (consultada 2026-04-24)
- [github.blog Primer category](https://github.blog/category/engineering/ui-design/) (consultada 2026-04-24)

## Relações

- **Cluster**: [[_moc]]
- **Teoria**: [[design-system-architecture]]
- **Fundação**: [[design-tokens]]
- **Aplicado**: [[../../20-stacks/typescript/frontend-solidjs]] (stack SolidJS pode adotar padrão similar)
- **Arquitetura**: [[../architecture/clean-architecture-layers]] (DS é separação de concerns real)
- **Anti-patterns evitados**: [[../antipatterns/shotgun-surgery|AP-008]], [[../antipatterns/duplicate-code|AP-011]]

## Links

- [[_moc]]
- [[design-system-architecture]]
- [[design-tokens]]
- [[../../20-stacks/typescript/frontend-solidjs]]
- [[../architecture/_moc]]
