---
title: MOC — UI Design (Design Systems, Tokens, Component Architecture)
type: moc
tags:
  - moc
  - ui-design
  - design-system
  - tokens
  - frontend
created: 2026-04-24
updated: 2026-04-24
aliases:
  - UI Design MOC
  - Design Systems MOC
---

# UI Design

Cluster de knowledge sobre **arquitetura de design systems** — como estruturar, distribuir, governar, versionar design systems que escalam de 1 produto a N produtos, de 1 time a muitos, mantendo consistency, acessibilidade, brand identity, e velocidade de entrega.

> **Regra**: design system **não é** biblioteca de componentes; é **arquitetura** com tokens, composição, versioning, distribution, governance.

## 📚 Notas

- [[design-system-architecture]] — **teoria atemporal**: 5 camadas (foundations → tokens → primitives → components → patterns), Atomic Design, Headless vs Styled vs Recipes vs Multi-camada, Monorepo vs polyrepo, Semver + Changesets, Deprecation strategy, Theming & multi-brand, Governance models (centralized/federated/contribution), Multi-app pattern.
- [[design-tokens]] — **camada fundacional**: W3C Design Tokens Community Group format (DTCG), layered architecture (primitive → semantic → component + brand overlay), tipos de token, Style Dictionary + Tokens Studio, multi-platform output (CSS/JS/iOS/Android), theme switching, runtime multi-tenant, naming conventions, a11y enforcement.
- [[case-study-primer-github]] — **case study concreto**: Primer (GitHub DS), 6 packages (primitives, react, css, view_components, brand, octicons), 9 themes acessibilidade, multi-product (github.com app + marketing + mobile + VS Code ext + Desktop + Enterprise), governance (~15 devs + RFC + adoption metrics), release via Changesets, lições replicáveis.

## 🧭 Decision tree

```
Estou construindo / escolhendo um design system. Preciso:
│
├─ Entender o que é arquitetura de DS?
│   → [[design-system-architecture]]
│
├─ Estruturar tokens (cor, spacing, typography)?
│   → [[design-tokens]]
│
├─ Ver como GitHub resolve multi-product em escala real?
│   → [[case-study-primer-github]]
│
├─ Escolher modelo (Headless vs Styled vs Recipes)?
│   → [[design-system-architecture]] §"Modelos de componentização"
│
├─ Lidar com multi-brand / multi-tenant (white-label)?
│   → [[design-system-architecture]] §"Theming & multi-brand"
│   → [[design-tokens]] §"Runtime per-tenant"
│
├─ Decidir monorepo vs polyrepo?
│   → [[design-system-architecture]] §"Distribution"
│
├─ Versionar DS sem quebrar consumers?
│   → [[design-system-architecture]] §"Semver + changelog" + "Deprecation strategy"
│
└─ Governance (quem decide, como contribuir)?
    → [[design-system-architecture]] §"Governance"
    → [[case-study-primer-github]] §"Governance GitHub"
```

## 🏗️ Design systems canônicos (referência)

| DS | Empresa | Arquitetura | Stack |
|---|---|---|---|
| **Primer** | GitHub | Multi-camada (primitives/react/css/view_components/brand) | React, Ruby, CSS |
| **Material 3** | Google | Dynamic color, tokens cross-platform | Flutter, Android, web |
| **Fluent 2** | Microsoft | Multi-platform web components | React, native Windows/macOS |
| **Carbon** | IBM | Enterprise; multi-framework | React, Angular, Vue, Svelte |
| **Polaris** | Shopify | Embedded apps for merchants | React |
| **Spectrum** | Adobe | Multi-brand (CC + Experience Cloud) | React, CSS |
| **Atlassian DS** | Atlassian | Federated governance | React |
| **Lightning** | Salesforce | Templates + BEM CSS | React, Aura |
| **Base Web** | Uber | Extensibility via overrides | React |
| **Ant Design** | Origem Alibaba | Dominante APAC | React, Vue |
| **Chakra UI** | Open-source | A11y + DX focus | React |
| **Radix + shadcn/ui** | Open-source | Headless + code distribution | React |

## 🔗 Cross-refs com outros clusters

### Arquitetura

- [[../architecture/clean-architecture-layers]] — DS é camada separada da aplicação
- [[../architecture/ddd-strategic-tactical]] — bounded context UI vs domain

### Anti-patterns

- [[../antipatterns/shotgun-surgery|AP-008]] — sem tokens, rebrand vira shotgun surgery
- [[../antipatterns/duplicate-code|AP-011]] — DS elimina duplicação cross-app
- [[../antipatterns/magic-numbers|AP-009]] — hex/px em componente = anti-pattern; usa token

### Princípios

- [[../principles/do-dont-checklist]] — DRY, consistency
- [[../principles/solid]] — Open/Closed (componentes extensíveis via composição + variants, não modification)

### Stack aplicado

- [[../../20-stacks/typescript/frontend-solidjs]] — SolidJS + UnoCSS + Kobalte adopt embrionário de DS patterns (paleta imutável como tokens, Kobalte primitives como headless base)

### Providers relevantes

- Figma (não tem nota provider dedicada ainda)
- Tokens Studio (Figma plugin)
- Storybook (docs/sandbox)

## 📐 Padrões replicáveis para SaaS multi-app

Quando produto tem **N apps compartilhando stack**:

1. **`packages/ui`** — Primer role (componentes React).
2. **`packages/tokens`** — Primer Primitives role (JSON source of truth).
3. **`packages/brand`** separado se marketing tem UX distinto.
4. **Figma sync** via Tokens Studio.
5. **Changesets** + automated release via bot.
6. **Storybook** hospedado (subdomain).
7. **Contribution docs** + RFC template.
8. **CI a11y check** obrigatório (axe-core).
9. **Visual regression** (Chromatic/Percy) recomendado.
10. **Governance clara** — quem owns, como contribui, deprecation timeline.

## 📖 Fontes-chave do cluster

- [Primer Design System](https://primer.style/) (consultada 2026-04-24)
- [Material Design 3](https://m3.material.io/) (consultada 2026-04-24)
- [W3C Design Tokens CG](https://www.w3.org/community/design-tokens/) (consultada 2026-04-24)
- [Style Dictionary](https://styledictionary.com/) (consultada 2026-04-24)
- [Tokens Studio](https://tokens.studio/) (consultada 2026-04-24)
- [Atomic Design — Brad Frost](https://atomicdesign.bradfrost.com/) (consultada 2026-04-24)
- [Design Systems Handbook — DesignBetter.co](https://www.designbetter.co/design-systems-handbook) (consultada 2026-04-24)

## 🧭 Vizinhos no grafo

- [[../_moc]] — knowledge raiz
- [[../architecture/_moc]]
- [[../antipatterns/_moc]]
- [[../principles/_moc]]
- [[../../20-stacks/typescript/frontend-solidjs]]
- [[../../00-meta/STATE]] — D-vault-024

## Regra dura do cluster

Design system tratado como **arquitetura real** — semver, CI, docs, governance, deprecation. Se for tratado como "lib de UI side project", morre em 1 ano.
