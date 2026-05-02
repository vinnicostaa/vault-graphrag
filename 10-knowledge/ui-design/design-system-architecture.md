---
title: Design System Architecture — foundations, components, distribution, governance
type: concept
tags:
  - ui-design
  - design-system
  - architecture
  - frontend
  - primer
  - multi-app
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Design System Architecture
  - DS Architecture
---

# Design System Architecture

**O que é**: conjunto de **decisões arquiteturais** que faz um design system escalar de 1 componente a milhares, de 1 produto a N produtos, de 1 time a centenas — sem drift, rework, ou fragmentação. Design system **não é** biblioteca de componentes; é **arquitetura** com tokens, composição, versioning, distribution, governance.

> **Mental model**: "API contract para UI". Tokens = enum. Componentes = funções. Patterns = recipes. Governance = código review. Se você não trata como software real (semver, changelog, migration, deprecation), drift acumula e o DS vira boa ideia morta.

## Por que design system arquitetural importa

Sem DS sistematizado:
- Botão "primário" existe em 14 variantes (cada time fez o seu).
- Mudar cor primária = editar 100 arquivos ([[../antipatterns/shotgun-surgery|AP-008]]).
- Acessibilidade implementada ad-hoc — falha em audits compliance.
- Dark mode exige rewrite.
- Redesign de marca = projeto de 6 meses.

Com DS arquitetural:
- 1 fonte de verdade (tokens).
- Mudança de cor = edit em 1 arquivo de token → propaga tudo.
- Acessibilidade built-in em componentes primitivos.
- Dark mode = theme switch.
- Redesign = novo brand layer, componentes seguem.

## 5 camadas arquiteturais

```
┌──────────────────────────────────────────────────────────┐
│ 5. PATTERNS / TEMPLATES                                  │
│    Login, dashboard, empty-state, settings page…          │
│    Recipes feitos de N componentes                        │
├──────────────────────────────────────────────────────────┤
│ 4. COMPONENTS                                             │
│    Button, Input, Dialog, Menu, Table, DatePicker…        │
│    Unidade de reuso; API estável; variantes documentadas  │
├──────────────────────────────────────────────────────────┤
│ 3. PRIMITIVES                                             │
│    Box, Flex, Grid, Text, Icon…                           │
│    Building blocks atômicos; styling props consistente    │
├──────────────────────────────────────────────────────────┤
│ 2. TOKENS (semantic + primitive)                          │
│    color.fg.default, spacing.medium, radius.small…        │
│    Ver [[design-tokens]] para modelo completo             │
├──────────────────────────────────────────────────────────┤
│ 1. FOUNDATIONS                                            │
│    Typography scale, color palette, spacing system,       │
│    grid, motion curves, iconography, accessibility        │
│    contracts (WCAG), i18n strategy                        │
└──────────────────────────────────────────────────────────┘
```

Cada camada **depende só** das inferiores. Nunca cima → baixo.

## Atomic Design (Brad Frost, 2013)

Taxonomia influente — 5 níveis:
- **Atoms**: inputs básicos (button, input, label).
- **Molecules**: atoms agrupados (search = input + button).
- **Organisms**: múltiplas molecules (header, product card).
- **Templates**: layout-level sem conteúdo real.
- **Pages**: templates com content concreto.

**Crítica prática**: a fronteira atom/molecule/organism é borrada; muitos DS modernos colapsam em "components" + "patterns". Atomic Design é **mental model**, não taxonomy rigorosa.

## Modelos de componentização

### A. Styled components (Material, Chakra, Primer)

Componentes **com estilo baked-in**. Consumer recebe aparência pronta.

- **Prós**: UX-consistent out-of-the-box.
- **Cons**: customização avançada exige override patterns (`sx` prop, theme override). Casos extremos caem em "brigar com o DS".

### B. Headless / Primitives (Radix, Headless UI, Ariakit, Base UI)

Componentes entregam **comportamento + acessibilidade**, **não** estilo. Consumer estiliza como quiser.

- **Prós**: flexibilidade total; ideal para DS customizado sobre primitives.
- **Cons**: cada app recria estilo — defeats reuse se não combinado com tokens/recipes.

### C. Recipe-based (shadcn/ui, tailwindcss-ui)

Headless primitives + **código copiável** estilizado. Não é lib — é **distribuição de templates**.

- **Prós**: zero lock-in de lib; customização trivial (é seu código).
- **Cons**: update manual; divergência entre apps.

### D. Multi-camada (Primer, Polaris, Carbon)

Combina: primer/primitives (tokens) + primer/react (styled) + primer/css (utility classes) + primer/view_components (Rails). Atende múltiplos stacks com 1 fonte de verdade.

**Regra prática**: escolher **um** modelo core por DS; misturar confunde consumers. Recipes funcionam bem como **entry point** (shadcn), DS maduro típico adota **styled** (Primer/Material).

## Distribution

### Monorepo vs polyrepo

**Monorepo** (Primer, Material MUI, Carbon): todos os packages num repo com pnpm workspaces / Turborepo / Nx / Lerna.
- Facil refactor cross-package.
- Versionamento coordenado (Changesets, semantic-release).
- CI complexo mas único.

**Polyrepo**: 1 repo por package.
- Separação estrita.
- Versionamento independente.
- Coordenação custa (N PRs sincronizados).

**Moderno (2024+)**: monorepo é dominante. Turborepo ou pnpm workspaces.

### Package design

```
@company/design-tokens       (camada 2 — JSON + build outputs)
@company/primitives          (camada 3 — Box, Flex, Grid)
@company/react               (camada 4 — Button, Input, Dialog)
@company/patterns            (camada 5 — LoginFlow, DashboardShell)
@company/icons               (foundation — SVG set)
```

Cada package com `peerDependencies` fortes (React, React-DOM) e `dependencies` mínimas.

### Semver + changelog

Design system é **dep crítica** de N apps. Breaking change sem aviso = coordenação pesadelo.

- **major**: breaking API (remoção de prop, rename).
- **minor**: novo componente, nova variante.
- **patch**: bugfix, estilo ajuste dentro da variante.

**Tools**:
- **Changesets** — PR adiciona `.changeset/xxx.md` com tipo de mudança; bot aggregates para release.
- **semantic-release** — commit message-driven.
- **release-please** (Google) — similar.

Cada release gera `CHANGELOG.md` humano-legível.

### Deprecation strategy

Componentes precisam morrer. Sem processo = tech debt eterna.

1. **Deprecation warning** em console no uso do componente velho (com link para substituto).
2. **TypeScript `@deprecated` JSDoc** — linter mostra riscado.
3. **Codemod** (jscodeshift, ts-morph) — ferramenta CLI que transforma código do consumer.
4. **Migration guide** — doc detalhada de antes/depois.
5. **Timeline** — 1 major version deprecated → próxima major removido.

## Theming & multi-brand

### Theme switching (dark/light)

Padrão 2024+: **CSS custom properties** + scope em `html` ou `body`:

```css
:root { --color-fg-default: #1f2328; }
:root[data-theme='dark'] { --color-fg-default: #e6edf3; }
```

Componentes usam `var(--color-fg-default)` — zero JS. Next.js: `next-themes`; Remix: `remix-themes`.

### Multi-brand (1 DS, N marcas)

Pattern Primer-like:
- **Primer (product)** — GitHub main app.
- **Primer Brand** — github.com marketing.
- Compartilham tokens de acessibilidade/motion; **divergem** em tipografia/tom.

Implementação: **brand layer** sobre semantic tokens:
```
primitive: gray-900 (fixed)
  ↓
semantic: color.fg.default (maps to gray-900 in light, gray-50 in dark)
  ↓
brand: overrides específicas (github-brand = gray-900; enterprise = navy-900)
```

### Multi-tenant SaaS com white-label

Cada tenant customiza subset de tokens (logo + primary color + font). Build dinâmico ou runtime injection.

Patterns:
- Runtime CSS vars override per request (server reads tenant config).
- Build-time: 1 build por tenant (escala mal >10 tenants).
- Dinâmico JS: `setProperty('--color-primary', tenant.color)` no mount.

## Governance

Design system sem dono = morre. Modelos comuns:

### Centralized
- Time dedicado "Design Systems Team" owns tudo.
- Contribuições externas via PR process.
- **Prós**: consistência alta.
- **Cons**: bottleneck; não escala muito além de ~50 produtos.

### Federated
- Core team owns primitives + tokens.
- Product teams contribuem componentes "certified" via processo.
- **Prós**: escala; time dedicado não é gargalo.
- **Cons**: precisa RFC process + reviewers treinados.

### Contribution model
- Todos podem propor.
- Review assíncrono com critérios publicados.
- **Prós**: democrático.
- **Cons**: governance frágil; precisa liderança forte.

### Maturity indicators
- **Docs site** ativo (Storybook / docusaurus / zeroheight).
- **Design tokens sync** (Figma ↔ code).
- **Accessibility CI** (axe, Storybook a11y addon).
- **Visual regression tests** (Chromatic, Percy).
- **Metrics**: adoption rate por componente; % coverage design→code; resolution time de issues.

## Multi-app / multi-tenant — o valor real

Quando você tem **N apps compartilhando design system**:

```
apps/
  auth/       — Login, signup
  cms/        — Content management
  lms/        — Learning
  assembly/   — Assembly management
  forum/
  lp/         — Landing page
  ...
packages/
  ui/         — DS: Button, Input, Dialog…
  schemas/    — Zod schemas
  core/       — HTTP client
```

Sem DS:
- Cada app tem `<Button>` local. 6 implementações.
- Dark mode = 6 PRs.
- A11y audit = 6 arquivos pra corrigir.

Com DS `@company/ui`:
- 1 `<Button>`; 6 apps importam.
- Dark mode = 1 PR no DS.
- A11y = 1 correção.

**Pattern Primer** faz isso para GitHub product + github.com brand + Enterprise UI — mesmos tokens base, divergência em camada específica.

## Anti-patterns

1. **"Copiar Material UI e customizar"** — herda opinions Material (paper/elevation); difícil sair. Prefira headless ou começar do zero com tokens.
2. **Componentes com 30 props** — bloat. Primer tem regra "max 7 props por componente; excesso = variante nova ou composição".
3. **Sem design tokens** — hardcoded hex em CSS. Mudança de cor = shotgun surgery ([[../antipatterns/shotgun-surgery|AP-008]]).
4. **Token sprawl** — `color.button.hover.focus.primary.dark` = excesso. Manter taxonomia rasa (2-3 níveis).
5. **Semver desrespeitado** — bump major por feature minor gera fricção em consumers.
6. **Sem Storybook / sandbox** — devs não exploram; design system fica invisível.
7. **Figma e code fora de sync** — designers usam cor que não existe no código. Solução: design tokens como fonte de verdade, Figma consome.
8. **Governance inexistente** — DS vira "time" pet project; sem mandato cross-org.
9. **Breaking change sem codemod** — time de produto pena para migrar; bloqueia adoção.
10. **Acessibilidade como afterthought** — componente lançado → 6 meses depois audit falha. Build into primitives desde dia 1.

## Métricas de sucesso

| Métrica | Sinal |
|---|---|
| % cobertura de componentes do DS em produto novo | > 80% em MVP |
| Tempo médio PR → release | < 1 semana |
| Issues abertas / componentes | < 1.5 |
| Adoption rate por componente (uso / esperado) | > 70% |
| A11y violations em CI | 0 críticas |
| Visual regression changes per release | rastreado |
| Changelog humano-legível | sim (não "deps bumped") |

## Referências — design systems canônicos

| DS | Empresa | Destaque arquitetural |
|---|---|---|
| **Primer** | GitHub | Multi-camada (primitives/react/css/view_components/brand); design tokens em JSON; Octicons |
| **Material 3** | Google | Dynamic color (M3 Expressive); expressive design tokens; cross-platform (Flutter/Android/web) |
| **Fluent 2** | Microsoft | Multi-platform (React, native Windows, native macOS); web components |
| **Carbon** | IBM | Multi-framework (React, Angular, Vue, Svelte, Web Components); enterprise focus |
| **Polaris** | Shopify | Embedded apps para merchants; tokens em JSON |
| **Spectrum** | Adobe | Multi-brand (Creative Cloud + Experience Cloud) |
| **Atlassian Design System** | Atlassian | Enterprise-scale; governance federado |
| **Lightning** | Salesforce | Enterprise; templates + BEM CSS |
| **Base Web** | Uber | Extensibility via overrides; theme heavy |
| **Ant Design** | Origem Alibaba | Open-source; enterprise-enterprise; dominante APAC |
| **Chakra UI** | Open-source | Accessibility + DX; popular em React/Next.js |
| **Radix UI + shadcn/ui** | Open-source | Headless + code distribution; explosivo 2023-24 |

Case study dedicado: [[case-study-primer-github]].

## Relações

- **Cluster**: [[_moc]]
- **Tokens**: [[design-tokens]] (camada crítica)
- **Case study**: [[case-study-primer-github]]
- **Arquitetura geral**: [[../architecture/clean-architecture-layers]] (DS é uma layer separada da app)
- **Anti-patterns relacionados**: [[../antipatterns/shotgun-surgery|AP-008]], [[../antipatterns/duplicate-code|AP-011]]
- **Princípios**: [[../principles/do-dont-checklist]] (DRY, consistency)
- **Stack aplicado**: [[../../20-stacks/typescript/frontend-solidjs]] (paleta imutável + UnoCSS = DS embrionário)

## Fontes

- [Primer Design System](https://primer.style/) (consultada 2026-04-24)
- [Material Design 3](https://m3.material.io/) (consultada 2026-04-24)
- [Fluent 2](https://fluent2.microsoft.design/) (consultada 2026-04-24)
- [Atomic Design — Brad Frost (livro)](https://atomicdesign.bradfrost.com/) (consultada 2026-04-24)
- [Design Systems — Alla Kholmatova (Smashing Magazine)](https://www.smashingmagazine.com/design-systems-book/) (consultada 2026-04-24)
- [Storybook.js](https://storybook.js.org/) (consultada 2026-04-24)
- [Changesets](https://github.com/changesets/changesets) (consultada 2026-04-24)
- [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[design-tokens]]
- [[case-study-primer-github]]
- [[../architecture/clean-architecture-layers]]
- [[../antipatterns/shotgun-surgery]]
- [[../antipatterns/duplicate-code]]
- [[../principles/do-dont-checklist]]
- [[../../20-stacks/typescript/frontend-solidjs]]
