---
title: Design Tokens — W3C DTCG format, layered architecture, multi-platform
type: concept
tags:
  - ui-design
  - design-tokens
  - w3c
  - theming
  - css-variables
doc-oficial: https://www.w3.org/community/design-tokens/
doc-consulted: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases:
  - Design Tokens
  - DTCG
  - DS Tokens
---

# Design Tokens

**O que é**: valores atômicos de design (cor, spacing, typography, radius, motion curve, shadow, etc.) armazenados como **dados estruturados** (JSON, YAML), processados por tooling para gerar outputs em múltiplas plataformas (CSS custom properties, JS constants, iOS/Android native, Figma).

> **Mental model**: "tokens são **enums** do design". Ao invés de `color: #0969da` espalhado em N arquivos, `color: var(--accent-fg)` em todos — 1 fonte de verdade. Consequências: rebrand = edit 1 JSON; dark mode = switch theme; a11y compliance = tokens auditáveis.

## Por que tokens importam (além do óbvio)

Sem tokens:
- `#0969da` aparece em 200 lugares → rebrand = 200 PRs ([[../antipatterns/shotgun-surgery|AP-008]]).
- Dark mode exige rewrite de estilos (não tem semântica "foreground vs background").
- Designers usam `#1A8FD3` no Figma; dev implementa `#0969da` — **drift**.
- Acessibilidade: ratios de contraste não-auditáveis automaticamente.

Com tokens:
- 1 JSON define; N plataformas consomem.
- Dark theme = swap de values; zero CSS re-escrito.
- Figma ↔ code sync (designer e dev leem o mesmo).
- A11y CI valida `color.fg.default` vs `color.bg.default` contrast ratio automaticamente.

## W3C Design Tokens Community Group (DTCG) format

Formato oficial em especificação (draft final 2024-2026). Adotado por Style Dictionary, Token Studio, Material Design, Adobe Spectrum, e crescendo.

### Estrutura

```json
{
  "color": {
    "gray": {
      "900": { "$value": "#1f2328", "$type": "color" },
      "50":  { "$value": "#f6f8fa", "$type": "color" }
    },
    "fg": {
      "default": { "$value": "{color.gray.900}", "$type": "color" }
    }
  },
  "spacing": {
    "small": { "$value": "8px", "$type": "dimension" },
    "medium": { "$value": "16px", "$type": "dimension" }
  },
  "typography": {
    "body": {
      "$value": {
        "fontFamily": "{font.family.body}",
        "fontSize": "16px",
        "fontWeight": 400,
        "lineHeight": 1.5
      },
      "$type": "typography"
    }
  }
}
```

Campos chave:
- **`$value`** — valor ou referência `{path.to.token}`.
- **`$type`** — `color`, `dimension`, `typography`, `fontWeight`, `duration`, `cubicBezier`, `shadow`, etc.
- **`$description`** — doc humana.
- **`$extensions`** — metadata vendor-specific.

### Ref resolution

`{color.gray.900}` resolve via DFS. Build tool gera CSS final:
```css
--color-gray-900: #1f2328;
--color-fg-default: var(--color-gray-900);
```

## Layered architecture (camadas de tokens)

**Regra de ouro**: tokens NÃO são flat — são em **camadas**, cada uma com propósito.

```
┌──────────────────────────────────────────────────┐
│ 4. COMPONENT TOKENS (opcional, alta granularidade)│
│    button.primary.fg, button.primary.bg,         │
│    input.focus.border                            │
│    Consumido por: componente específico           │
├──────────────────────────────────────────────────┤
│ 3. SEMANTIC (ALIAS) TOKENS                       │
│    fg.default, fg.muted, bg.canvas,              │
│    border.default, accent.emphasis, danger.fg     │
│    Consumido por: primitives + componentes       │
├──────────────────────────────────────────────────┤
│ 2. PRIMITIVE (BASE / GLOBAL) TOKENS              │
│    gray.50..950, blue.50..950, spacing.1..12     │
│    Não usar direto em componente (em geral)       │
├──────────────────────────────────────────────────┤
│ 1. BRAND OVERLAY (opcional)                       │
│    github-brand, enterprise-brand, white-label    │
│    Substitui subset de camadas 2-3                │
└──────────────────────────────────────────────────┘
```

### Exemplo — dark mode via alias

**Primitive (fixo)**:
```json
{ "gray": { "100": "#f6f8fa", "900": "#1f2328" } }
```

**Semantic (muda por tema)**:
```jsonc
// themes/light.json
{ "fg": { "default": "{gray.900}" }, "bg": { "canvas": "{gray.100}" } }

// themes/dark.json
{ "fg": { "default": "{gray.100}" }, "bg": { "canvas": "{gray.900}" } }
```

**Component tokens** consomem semantic:
```jsonc
{ "button": { "primary": { "fg": "{fg.default}", "bg": "{accent.emphasis}" } } }
```

Switch de tema = trocar semantic layer. Componente token herda automaticamente.

### Quando usar cada camada

- **Primitive**: nunca em componente direto. Paleta exaustiva.
- **Semantic**: **primária** fonte para estilo de componente. 80-90% do consumo.
- **Component**: quando componente precisa token específico que não cabe em semantic (hover states complexos, focus rings específicos). Última opção; evitar bloat.
- **Brand**: multi-tenant ou multi-produto. Rebrand = swap brand layer.

## Tipos de token (DTCG)

| `$type` | Exemplo | Output CSS |
|---|---|---|
| `color` | `"#0969da"` ou `"hsl(212 92% 45%)"` | `--accent-fg: #0969da;` |
| `dimension` | `"16px"` | `--spacing-md: 16px;` |
| `fontFamily` | `["Inter", "sans-serif"]` | `--font-body: Inter, sans-serif;` |
| `fontWeight` | `400` ou `"bold"` | `--weight-regular: 400;` |
| `duration` | `"200ms"` | `--motion-fast: 200ms;` |
| `cubicBezier` | `[0.4, 0, 0.2, 1]` | `--ease-out: cubic-bezier(...);` |
| `number` | `1.5` | `--line-height: 1.5;` |
| `shadow` | `{ offsetX, offsetY, blur, color }` | `--shadow-md: 0 2px 4px ...;` |
| `typography` (composite) | object | Múltiplas vars agrupadas |
| `gradient`, `border`, `transition` | composite | Múltiplas vars |

## Tooling

### Style Dictionary (Amazon, open-source)

Padrão-ouro 2024+. Lê JSON, gera outputs.

```js
// style-dictionary.config.js
export default {
  source: ['tokens/**/*.json'],
  platforms: {
    css: {
      transformGroup: 'css',
      buildPath: 'dist/css/',
      files: [{ destination: 'variables.css', format: 'css/variables' }],
    },
    js: {
      transformGroup: 'js',
      buildPath: 'dist/js/',
      files: [{ destination: 'tokens.ts', format: 'javascript/es6' }],
    },
    ios: {
      transformGroup: 'ios-swift',
      buildPath: 'dist/ios/',
      files: [{ destination: 'Tokens.swift', format: 'ios-swift/class.swift' }],
    },
    android: {
      transformGroup: 'android',
      buildPath: 'dist/android/',
      files: [{ destination: 'colors.xml', format: 'android/colors' }],
    },
  },
}
```

Build gera 4 outputs de 1 fonte. Designer edita JSON; devs de web/iOS/Android consomem.

### Tokens Studio (Figma plugin)

Plugin Figma que sincroniza tokens com GitHub repo:
- Designer edita tokens no Figma (UI amigável).
- Plugin commita JSON no repo.
- CI roda Style Dictionary → gera outputs → deploy.

Sync bidirecional design ↔ code.

### Outros tools

- **Specify** — hosted; SaaS tokens pipeline.
- **Knapsack** — DS platform com tokens.
- **Terrazzo** — alternativa a Style Dictionary (TS-first).
- **Theo** (Salesforce) — deprecated em favor de Style Dictionary.

## Multi-platform output

Mesmo JSON, N outputs:

### Web (CSS custom properties)
```css
:root {
  --color-accent-fg: #0969da;
  --spacing-md: 16px;
}
```

### Web (JS/TS constants)
```ts
export const tokens = {
  color: { accent: { fg: '#0969da' } },
  spacing: { md: '16px' },
} as const
```

### iOS (Swift)
```swift
enum Tokens {
  enum Color {
    static let accentFg = UIColor(hex: "#0969da")
  }
  enum Spacing {
    static let md: CGFloat = 16
  }
}
```

### Android (XML resources)
```xml
<color name="accent_fg">#0969da</color>
<dimen name="spacing_md">16dp</dimen>
```

### Tailwind config
```js
// tailwind.config.js
import tokens from './dist/tokens.js'
export default {
  theme: {
    colors: { accent: { fg: tokens.color.accent.fg } },
    spacing: tokens.spacing,
  }
}
```

## Theme switching (dark/light)

### CSS custom properties (padrão moderno)

```css
:root { --color-fg-default: #1f2328; }
:root[data-theme='dark'] { --color-fg-default: #e6edf3; }
:root[data-theme='high-contrast'] { --color-fg-default: #000; }
```

Toggle no JS:
```ts
document.documentElement.dataset.theme = 'dark'
```

- Zero re-render React.
- Funciona em SSR.
- Suporta N themes.

### next-themes / remix-themes

Libs que gerenciam:
- Persistência em localStorage / cookie.
- Respect `prefers-color-scheme`.
- FOUC prevention (SSR insert style).

### Runtime per-tenant (multi-tenant SaaS)

Server renderiza `<style>` com tokens do tenant:
```tsx
<style dangerouslySetInnerHTML={{ __html: `
  :root {
    --color-accent-fg: ${tenant.primaryColor};
    --font-family-body: ${tenant.fontFamily};
  }
`}} />
```

Ou `setProperty` no mount:
```ts
document.documentElement.style.setProperty('--color-accent-fg', tenant.primary)
```

## Governance & naming

### Naming conventions

Reverso de específico → geral funciona:
- `color.fg.default` (bom)
- `default.fg.color` (ruim — default varia)

Hierarquia comum:
```
<categoria>.<subcategoria>.<role>.<state>
color.accent.fg
color.accent.fg.hover
spacing.component.button.padding-x
```

**Mantenha raso** (max 4 níveis). Token `color.button.primary.hover.focus.disabled.mobile` = code smell.

### Token deprecation

Mesmo processo que componente:
1. `color.fg.muted` → `color.fg.subtle` (rename).
2. Adicionar ambos com mesma ref por 1 major version.
3. `$deprecated: "use color.fg.subtle"` metadata.
4. Codemod disponível.
5. Remover após migration.

### A11y enforcement

CI valida:
- Contrast ratios de pares `fg × bg` ≥ WCAG AA (4.5:1) ou AAA (7:1).
- Focus rings visíveis (`border.focus` vs `bg.canvas`).
- Motion duration ≤ `prefers-reduced-motion` caps.

Libs: `@adobe/leonardo-contrast-colors`, Figma plugin "Stark", CI com `axe-core`.

## Anti-patterns

1. **Tokens flat** (sem semantic layer) — `color.blue-500` direto em componente. Dark mode impossível.
2. **Magic numbers em componente** — `padding: 12px` em vez de `var(--spacing-md)`. Ver [[../antipatterns/magic-numbers|AP-009]].
3. **Token explosion** — 5000 tokens incluindo `color.button-primary-hover-focus-disabled-mobile-small`. Semantic layer rasa + variantes programáticas resolve.
4. **Hex direto no Figma** — designer usa color picker livre; dev fica sem token mapeado. Força Figma a usar library de tokens.
5. **Naming inconsistente** — `color-fg` em um arquivo, `textColor` em outro. Codemod + linter.
6. **Sem versioning dos tokens** — breaking change silencioso quebra tema custom de consumer.
7. **Tokens só em JS** — não CSS vars — perde runtime theme switching.
8. **Ignorar DTCG format** — custom format proprietário; perde ecossistema Style Dictionary, Figma plugins, Tokens Studio.

## Primer tokens (Github) — exemplo canônico

[primer/primitives](https://github.com/primer/primitives) é JSON tokens source of truth para:
- primer/react (React components).
- primer/css (utility CSS classes).
- primer/view_components (Rails).
- primer/figma (design library).

Layers:
- **Base**: `base/*` — primitives (scale.gray.1..9).
- **Function**: `functional/*` — semantic (`fg.default`, `bg.canvas`, `border.muted`).
- **Component**: `component/*` — button/input/select specific.

Themes: light, dark, dark_dimmed, dark_high_contrast, light_high_contrast.

Ver [[case-study-primer-github|case study]] para deep dive.

## Relações

- **Cluster**: [[_moc]]
- **Fundamental de**: [[design-system-architecture]] (camada 2)
- **Case study**: [[case-study-primer-github]]
- **Aplicado**: [[../../20-stacks/typescript/frontend-solidjs]] (paleta imutável como exemplo token)
- **Anti-patterns**: [[../antipatterns/magic-numbers|AP-009]], [[../antipatterns/shotgun-surgery|AP-008]]
- **Acessibilidade**: conecta com [[../security/owasp-top-10]] indiretamente (inclusão é parte de SEC/compliance alguns contextos)

## Fontes

- [W3C Design Tokens Community Group](https://www.w3.org/community/design-tokens/) (consultada 2026-04-24)
- [DTCG Format spec (draft)](https://tr.designtokens.org/format/) (consultada 2026-04-24)
- [Style Dictionary](https://styledictionary.com/) (consultada 2026-04-24)
- [Tokens Studio (Figma plugin)](https://tokens.studio/) (consultada 2026-04-24)
- [primer/primitives](https://github.com/primer/primitives) (consultada 2026-04-24)
- [Material Design Tokens](https://m3.material.io/foundations/design-tokens/overview) (consultada 2026-04-24)
- [Adobe Spectrum Tokens](https://spectrum.adobe.com/page/design-tokens/) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[design-system-architecture]]
- [[case-study-primer-github]]
- [[../antipatterns/magic-numbers]]
- [[../antipatterns/shotgun-surgery]]
- [[../../20-stacks/typescript/frontend-solidjs]]
