---
title: Design Tokens — Master Síndico (Web + Mobile + Theming)
type: concept
tags:
  - master-sindico
  - design-system
  - tokens
  - css
  - flutter
  - oklch
  - theming
project: master-sindico
created: 2026-04-24T00:00:00.000Z
source: >-
  web/apps/*/src/main.css (paleta OKLCH atual) + Material Design 3 + Radix
  Colors + Apple HIG + Shadcn + Tailwind
---

# Design Tokens — Master Síndico

**Fonte única de verdade** para design tokens cross-stack (web SolidJS + mobile Flutter + design Figma). Substitui `03-subprojects/web/design-system.md` e `03-subprojects/mobile/design-system.md` como canônico de tokens (os dois ficam como guidelines de uso).

**Origem OKLCH**: Paleta canônica extraída de `Development/web/apps/*/src/main.css`. Decisão [[../STATE#d-108-—-paleta-oklch-canônica-web-main-css|D-108]] (2026-04-24). OKLCH é perceptualmente uniforme — clareza idêntica em diferentes tons (diferente de HSL/RGB).

**Inspirações de arquitetura**:
- **Material Design 3** (Google) — roles semânticas (`primary`, `on-primary`, `surface-variant`, containers)
- **Radix Colors** (Vercel/Modulz) — escala 12-step (app-bg → subtle bg → UI element → hover → active → border → low-contrast text → high-contrast text)
- **Apple HIG / iOS System Colors** — dynamic colors + dark/light auto + semantic feedback
- **Shadcn/ui** — CSS vars + Tailwind + theming por `data-theme`
- **Tailwind** — escala numérica 50-950 + spacing 0-96
- **GitHub Primer** — foco em acessibilidade (contraste AA/AAA)
- **Atlassian Design System** — tokens enterprise + component tokens (layer 2)

---

## 1. Princípios

1. **OKLCH-first**: toda cor em OKLCH. Variáveis `#hex` apenas como fallback em contexto legacy.
2. **Single source of truth**: tokens definidos em JSON; build gera CSS (web) e Dart (mobile).
3. **Semantic over descriptive**: `--color-danger` não `--color-red-500`.
4. **Theming by data-attribute**: `data-theme="dark"` ou `.dark` controla.
5. **2 layers**:
   - **Layer 1 — Primitives**: paletas, escalas, raw values
   - **Layer 2 — Semantic/Component**: `--button-bg`, `--card-shadow`, `--input-border`
6. **Acessibilidade**: todo par `color + on-color` garante contraste ≥ AA (4.5:1); modos `high-contrast` garantem AAA (7:1).
7. **Motion reduzido**: tokens respeitam `prefers-reduced-motion`.

---

## 2. Paleta canônica (Layer 1 — Primitives)

### 2.1 Brand — Gold CTA + Navy

**Gold (primary)** — extraída de `main.css`:

| Token | OKLCH | Hex fallback | Uso |
|---|---|---|---|
| `--gold-50` | `oklch(0.976 0.020 84.2)` | `#FAEFDC` | backgrounds ultraleves |
| `--gold-100` | `oklch(0.951 0.045 84.2)` | `#F5DFB8` | hover backgrounds |
| `--gold-200` | `oklch(0.915 0.072 84.0)` | `#EDCB8A` | subtle accents |
| `--gold-300` | `oklch(0.871 0.096 83.5)` | `#E3B45F` | decorative |
| `--gold-400` | `oklch(0.810 0.113 83.2)` | `#D5A142` | hover primary |
| `--gold-500` | `oklch(0.715 0.120 84.2)` | `#C69C3F` | **primary CTA** ✨ |
| `--gold-600` | `oklch(0.620 0.112 84.5)` | `#A88435` | pressed primary |
| `--gold-700` | `oklch(0.525 0.098 84.8)` | `#8A6D2A` | borders fortes |
| `--gold-800` | `oklch(0.425 0.080 85.2)` | `#6C5620` | text em gold bg |
| `--gold-900` | `oklch(0.325 0.060 85.8)` | `#4F4018` | — |
| `--gold-950` | `oklch(0.225 0.038 86.2)` | `#332A0F` | — |

**Navy (secondary)** — extraída de `main.css`:

| Token | OKLCH | Hex fallback | Uso |
|---|---|---|---|
| `--navy-50` | `oklch(0.965 0.012 260.0)` | `#EBEDF2` | — |
| `--navy-100` | `oklch(0.928 0.022 260.5)` | `#D8DEE8` | — |
| `--navy-200` | `oklch(0.872 0.032 258.3)` | `#B8C1D2` | borders light |
| `--navy-300` | `oklch(0.780 0.040 257.5)` | `#8B98B4` | — |
| `--navy-400` | `oklch(0.551 0.045 256.9)` | `#4F5B75` | text low-contrast dark |
| `--navy-500` | `oklch(0.408 0.060 256.5)` | `#353F55` | — |
| `--navy-600` | `oklch(0.274 0.077 256.3)` | `#20283A` | **surface-variant** ✨ |
| `--navy-700` | `oklch(0.218 0.055 256.1)` | `#19202E` | **surface/secondary** ✨ |
| `--navy-800` | `oklch(0.204 0.033 260.3)` | `#18202A` | **foreground** ✨ |
| `--navy-900` | `oklch(0.183 0.031 263.4)` | `#15202A` | **background dark** ✨ |
| `--navy-950` | `oklch(0.140 0.025 265.0)` | `#0E1821` | — |

**Accent (gold-light)**: `oklch(0.871 0.129 82.0)` = `#E3B44F` — highlight em light mode.

### 2.2 Neutral / Gray scale (Radix-style 12-step)

| Token | OKLCH | Hex | Uso (Radix semantics) |
|---|---|---|---|
| `--gray-1` | `oklch(0.993 0.002 264.5)` | `#FCFCFD` | app bg |
| `--gray-2` | `oklch(0.981 0.002 264.5)` | `#F9F9FB` | subtle bg |
| `--gray-3` | `oklch(0.953 0.004 264.5)` | `#EFEFF2` | UI element bg |
| `--gray-4` | `oklch(0.928 0.006 264.5)` | `#E7E8EB` | UI element bg hovered |
| `--gray-5` | `oklch(0.908 0.008 264.5)` | `#DEDFE3` | UI element bg active |
| `--gray-6` | `oklch(0.872 0.009 258.3)` | `#D4D5DA` | subtle border / separator ✨ (--border atual) |
| `--gray-7` | `oklch(0.836 0.011 259.0)` | `#C4C5CC` | UI border |
| `--gray-8` | `oklch(0.780 0.014 260.0)` | `#AEB0B9` | UI border hover |
| `--gray-9` | `oklch(0.551 0.023 264.4)` | `#74757D` | solid bg / low-contrast text ✨ (--muted-foreground atual) |
| `--gray-10` | `oklch(0.480 0.020 265.0)` | `#61636B` | solid bg hover |
| `--gray-11` | `oklch(0.390 0.018 264.0)` | `#484A54` | low-contrast text |
| `--gray-12` | `oklch(0.204 0.033 260.3)` | `#18202A` | high-contrast text ✨ (--foreground atual) |

### 2.3 Feedback scales

Cada uma com 11 steps (50-950). Listamos os 3 críticos — completar seguindo padrão Tailwind/OKLCH uniforme.

**Destructive / Error** (red-warm):

```
--error-500: oklch(0.568 0.200 26.4);   /* primary */
--error-600: oklch(0.505 0.190 27.5);   /* dark mode primary */
--error-foreground: oklch(1.000 0 0);
```

**Warning** (amber):

```
--warning-500: oklch(0.769 0.165 70.1);
--warning-600: oklch(0.685 0.158 71.0);
--warning-foreground: oklch(1.000 0 0);
```

**Success** (green):

```
--success-500: oklch(0.627 0.170 149.2);
--success-600: oklch(0.550 0.160 150.0);
--success-foreground: oklch(1.000 0 0);
```

**Info** (blue):

```
--info-500: oklch(0.600 0.180 240.0);
--info-600: oklch(0.530 0.175 242.0);
--info-foreground: oklch(1.000 0 0);
```

### 2.4 OpenType palette (feature colors — futuro)

Reservar slots para sub-produtos específicos se precisar:
- `--connect-me`: não atribuído (usa --primary)
- `--reputation`: `oklch(0.715 0.120 84.2)` (gold — Bronze→Diamante usa gold + ajustes)
- `--assembly`: `oklch(0.568 0.200 26.4)` (red — urgência votação)
- `--compliance`: `oklch(0.627 0.170 149.2)` (green)

---

## 3. Semantic roles (Layer 2 — para uso direto)

Estes são os tokens que componentes consomem. Referenciam Layer 1.

### 3.1 Light mode (default)

```css
:root, [data-theme="light"] {
  /* Background & Foreground */
  --background: var(--gray-1);           /* app bg */
  --foreground: var(--gray-12);          /* default text */
  
  /* Primary */
  --primary: var(--gold-500);
  --primary-foreground: oklch(1.000 0 0); /* white on gold */
  --primary-hover: var(--gold-400);
  --primary-active: var(--gold-600);
  
  /* Secondary */
  --secondary: var(--navy-700);
  --secondary-foreground: var(--gray-2);
  --secondary-hover: var(--navy-600);
  --secondary-active: var(--navy-800);
  
  /* Accent (highlights em light mode) */
  --accent: var(--gold-200);
  --accent-foreground: var(--gray-12);
  
  /* Surface (navy em light também — usado em sidebar/topbar/footer) */
  --surface: var(--navy-700);
  --surface-foreground: var(--gray-2);
  --surface-variant: var(--navy-600);
  
  /* Card & Popover */
  --card: oklch(1 0 0);
  --card-foreground: var(--gray-12);
  --popover: oklch(1 0 0);
  --popover-foreground: var(--gray-12);
  
  /* Muted */
  --muted: var(--gray-4);
  --muted-foreground: var(--gray-9);
  
  /* Borders & Inputs */
  --border: var(--gray-6);
  --input: var(--gray-6);
  --input-bg: oklch(1 0 0);
  --ring: var(--gold-500);          /* focus ring */
  --ring-offset: var(--background);

  /* Input states (Lote 0 2026-04-27 — Figma normalizados, ver §Input states extraídos do Figma) */
  --input-border-active: oklch(0.557 0.243 268.5 / 0.4);   /* azul ativo c/ alpha */
  --input-fill-active: oklch(0.382 0.119 257.4 / 0.1);     /* navy translúcido */
  --input-placeholder: oklch(0.660 0.022 263.0);           /* cinza neutro */

  /* Progress dots (onboarding/wizard step indicators) */
  --progress-active: oklch(1.000 0 0);                     /* branco — dot ativo */
  --progress-inactive: oklch(0.812 0.105 252.1);           /* azul claro — dot pendente */

  /* Texto secundário sutil (cinza neutro reutilizável) */
  --text-secondary: oklch(0.660 0.022 263.0);
  
  /* Feedback */
  --destructive: var(--error-500);
  --destructive-foreground: var(--error-foreground);
  --warning: var(--warning-500);
  --warning-foreground: var(--warning-foreground);
  --success: var(--success-500);
  --success-foreground: var(--success-foreground);
  --info: var(--info-500);
  --info-foreground: var(--info-foreground);
  
  /* Shadow colors */
  --shadow-color: oklch(0.204 0.033 260.3 / 0.1);
  --shadow-color-lg: oklch(0.204 0.033 260.3 / 0.15);
  
  /* Overlay */
  --overlay: oklch(0.204 0.033 260.3 / 0.5);  /* modal backdrop */
  --overlay-light: oklch(0.204 0.033 260.3 / 0.3);
}
```

### 3.2 Dark mode

```css
.dark, [data-theme="dark"], [data-kb-theme="dark"] {
  --background: var(--navy-900);
  --foreground: var(--gray-2);
  
  --primary: var(--gold-500);           /* gold mantém */
  --primary-foreground: oklch(1.000 0 0);
  --primary-hover: var(--gold-400);
  --primary-active: var(--gold-600);
  
  --secondary: var(--navy-600);
  --secondary-foreground: var(--gray-2);
  --secondary-hover: var(--navy-500);
  --secondary-active: var(--navy-700);
  
  --accent: var(--gold-300);
  --accent-foreground: var(--gray-12);
  
  --surface: var(--navy-700);
  --surface-foreground: var(--gray-2);
  --surface-variant: var(--navy-600);
  
  --card: var(--navy-700);
  --card-foreground: var(--gray-2);
  --popover: var(--navy-700);
  --popover-foreground: var(--gray-2);
  
  --muted: var(--navy-600);
  --muted-foreground: oklch(0.714 0.019 261.3);
  
  --border: var(--navy-600);
  --input: var(--navy-600);
  --input-bg: var(--navy-700);
  --ring: var(--gold-500);
  --ring-offset: var(--background);
  
  --destructive: var(--error-600);
  --destructive-foreground: oklch(0.936 0.031 17.7);
  --warning: var(--warning-500);
  --warning-foreground: var(--warning-foreground);
  --success: var(--success-500);
  --success-foreground: var(--success-foreground);
  --info: var(--info-500);
  --info-foreground: var(--info-foreground);
  
  --shadow-color: oklch(0 0 0 / 0.4);
  --shadow-color-lg: oklch(0 0 0 / 0.6);
  
  --overlay: oklch(0 0 0 / 0.7);
  --overlay-light: oklch(0 0 0 / 0.5);
}
```

### 3.3 High-contrast mode (accessibility AAA)

```css
[data-theme="high-contrast"] {
  --background: oklch(1 0 0);           /* pure white */
  --foreground: oklch(0 0 0);           /* pure black */
  
  --primary: oklch(0.5 0.2 84.2);       /* darker gold para contraste */
  --primary-foreground: oklch(1 0 0);
  
  --border: oklch(0 0 0);               /* hard black borders */
  --ring: oklch(0.5 0.2 84.2);
  
  /* ... todas as outras vars ficam mais contrastadas ... */
}

[data-theme="high-contrast-dark"] {
  --background: oklch(0 0 0);           /* pure black */
  --foreground: oklch(1 0 0);           /* pure white */
  --border: oklch(1 0 0);
  /* ... */
}
```

---

## 4. Typography

### 4.1 Font families

```css
--font-sans: 'Manrope', system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', sans-serif;
--font-display: 'Michroma', var(--font-sans);
--font-mono: 'JetBrains Mono', 'SF Mono', Menlo, Monaco, 'Courier New', monospace;
--font-serif: Georgia, 'Times New Roman', serif;  /* raramente usado */
```

### 4.2 Type scale (Material Design 3 compatível)

```css
/* Display — para hero / lp */
--text-display-2xl: 4.5rem;       /* 72px / line 80 */  --line-display-2xl: 1.111;
--text-display-xl:  3.75rem;      /* 60 / 72 */         --line-display-xl:  1.2;
--text-display-lg:  3rem;         /* 48 / 56 */         --line-display-lg:  1.167;
--text-display-md:  2.25rem;      /* 36 / 44 */         --line-display-md:  1.222;
--text-display-sm:  1.875rem;     /* 30 / 40 */         --line-display-sm:  1.333;

/* Headline — titulos de seção */
--text-headline-xl: 2rem;         /* 32 / 40 */         --line-headline-xl: 1.25;
--text-headline-lg: 1.75rem;      /* 28 / 36 */         --line-headline-lg: 1.286;
--text-headline-md: 1.5rem;       /* 24 / 32 */         --line-headline-md: 1.333;
--text-headline-sm: 1.25rem;      /* 20 / 28 */         --line-headline-sm: 1.4;

/* Title — titulos de card, modal */
--text-title-lg:    1.125rem;     /* 18 / 28 */         --line-title-lg:    1.556;
--text-title-md:    1rem;         /* 16 / 24 */         --line-title-md:    1.5;
--text-title-sm:    0.875rem;     /* 14 / 20 */         --line-title-sm:    1.429;

/* Body — texto corrente */
--text-body-lg:     1rem;         /* 16 / 24 */         --line-body-lg:     1.5;
--text-body-md:     0.875rem;     /* 14 / 20 */         --line-body-md:     1.429;
--text-body-sm:     0.75rem;      /* 12 / 16 */         --line-body-sm:     1.333;

/* Label — CTAs, badges, captions */
--text-label-lg:    0.875rem;     /* 14 / 20 */         --line-label-lg:    1.429;
--text-label-md:    0.75rem;      /* 12 / 16 */         --line-label-md:    1.333;
--text-label-sm:    0.6875rem;    /* 11 / 16 */         --line-label-sm:    1.455;
```

### 4.3 Font weights

```css
--weight-thin:       100;
--weight-extralight: 200;
--weight-light:      300;
--weight-regular:    400;   /* default */
--weight-medium:     500;
--weight-semibold:   600;   /* labels, botões */
--weight-bold:       700;   /* titles */
--weight-extrabold:  800;
--weight-black:      900;
```

### 4.4 Letter-spacing / tracking

```css
--tracking-tighter: -0.05em;
--tracking-tight:   -0.025em;
--tracking-normal:  0;
--tracking-wide:    0.025em;
--tracking-wider:   0.05em;
--tracking-widest:  0.1em;    /* ALL CAPS labels */
```

---

## 5. Spacing (Tailwind scale 0-96)

```css
--space-0:    0;
--space-px:   1px;
--space-0-5:  0.125rem;    /* 2px */
--space-1:    0.25rem;     /* 4px */
--space-1-5:  0.375rem;    /* 6px */
--space-2:    0.5rem;      /* 8px */
--space-2-5:  0.625rem;    /* 10px */
--space-3:    0.75rem;     /* 12px */
--space-3-5:  0.875rem;    /* 14px */
--space-4:    1rem;        /* 16px */
--space-5:    1.25rem;     /* 20px */
--space-6:    1.5rem;      /* 24px */
--space-7:    1.75rem;     /* 28px */
--space-8:    2rem;        /* 32px */
--space-10:   2.5rem;      /* 40px */
--space-12:   3rem;        /* 48px */
--space-14:   3.5rem;      /* 56px */
--space-16:   4rem;        /* 64px */
--space-20:   5rem;        /* 80px */
--space-24:   6rem;        /* 96px */
--space-32:   8rem;        /* 128px */
--space-40:   10rem;       /* 160px */
--space-48:   12rem;       /* 192px */
--space-56:   14rem;       /* 224px */
--space-64:   16rem;       /* 256px */
--space-72:   18rem;       /* 288px */
--space-80:   20rem;       /* 320px */
--space-96:   24rem;       /* 384px */
```

---

## 6. Border radius

```css
--radius-none: 0;
--radius-xs:   0.125rem;  /* 2px */
--radius-sm:   0.25rem;   /* 4px */
--radius-md:   0.375rem;  /* 6px */
--radius-lg:   0.5rem;    /* 8px — default ✨ */
--radius-xl:   0.75rem;   /* 12px */
--radius-2xl:  1rem;      /* 16px */
--radius-3xl:  1.5rem;    /* 24px */
--radius-full: 9999px;
```

---

## 7. Shadows

Shadows em OKLCH+alpha — respeitam modo light/dark.

```css
--shadow-xs:   0 1px 2px 0 var(--shadow-color);
--shadow-sm:   0 1px 3px 0 var(--shadow-color), 0 1px 2px -1px var(--shadow-color);
--shadow-md:   0 4px 6px -1px var(--shadow-color), 0 2px 4px -2px var(--shadow-color);
--shadow-lg:   0 10px 15px -3px var(--shadow-color-lg), 0 4px 6px -4px var(--shadow-color-lg);
--shadow-xl:   0 20px 25px -5px var(--shadow-color-lg), 0 8px 10px -6px var(--shadow-color-lg);
--shadow-2xl:  0 25px 50px -12px var(--shadow-color-lg);
--shadow-inner: inset 0 2px 4px 0 var(--shadow-color);
--shadow-none: 0 0 #0000;
```

---

## 8. Transitions

### 8.1 Durations

```css
--duration-instant: 0ms;
--duration-fast:    75ms;
--duration-quick:   100ms;
--duration-normal:  150ms;  /* default ✨ */
--duration-moderate: 200ms;
--duration-slow:    300ms;
--duration-slower:  500ms;
--duration-slowest: 700ms;
--duration-crawl:   1000ms;
```

### 8.2 Easings

```css
--ease-linear:    linear;
--ease-in:        cubic-bezier(0.4, 0, 1, 1);
--ease-out:       cubic-bezier(0, 0, 0.2, 1);          /* default ✨ */
--ease-in-out:    cubic-bezier(0.4, 0, 0.2, 1);
--ease-back-in:   cubic-bezier(0.6, -0.28, 0.735, 0.045);
--ease-back-out:  cubic-bezier(0.175, 0.885, 0.32, 1.275);
--ease-bounce:    cubic-bezier(0.68, -0.55, 0.265, 1.55);
--ease-elastic:   cubic-bezier(0.16, 1.11, 0.3, 1.02);
```

### 8.3 Motion reduzido

```css
@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-fast: 0ms;
    --duration-quick: 0ms;
    --duration-normal: 0ms;
    /* ... desabilita todas as transições */
  }
}
```

---

## 9. Z-index

Ordem explícita de camadas. Evita bugs de z-index inflation.

```css
--z-hide:       -1;
--z-auto:       auto;
--z-base:       0;
--z-raised:     10;    /* cards elevados, avatar focados */
--z-dropdown:   100;   /* menus suspensos */
--z-sticky:     200;   /* navbars sticky */
--z-banner:     300;   /* alertas banner */
--z-overlay:    1000;  /* modal backdrop */
--z-modal:      1100;  /* modal content */
--z-popover:    1200;  /* popover, tooltip menor */
--z-skipLink:   1300;  /* skip-to-content (a11y) */
--z-toast:      1400;  /* notificações flutuantes */
--z-tooltip:    1500;  /* tooltips de máxima prioridade */
--z-max:        9999;
```

---

## 10. Breakpoints

```css
--bp-xs:    480px;   /* phone landscape */
--bp-sm:    640px;   /* tablet portrait */
--bp-md:    768px;   /* tablet landscape */
--bp-lg:    1024px;  /* desktop pequeno */
--bp-xl:    1280px;  /* desktop padrão */
--bp-2xl:   1536px;  /* desktop grande */
--bp-3xl:   1920px;  /* full HD+ */
```

Uso (UnoCSS / media queries):

```css
@media (min-width: 768px) { /* md+ */ }
```

---

## 11. Opacity

```css
--opacity-0:   0;
--opacity-5:   0.05;
--opacity-10:  0.1;
--opacity-20:  0.2;
--opacity-25:  0.25;
--opacity-30:  0.3;
--opacity-40:  0.4;
--opacity-50:  0.5;
--opacity-60:  0.6;
--opacity-70:  0.7;
--opacity-75:  0.75;
--opacity-80:  0.8;
--opacity-90:  0.9;
--opacity-95:  0.95;
--opacity-100: 1;
```

---

## 12. Component tokens (Layer 2.5)

Para consistência entre componentes.

### 12.1 Button

```css
--button-height-xs:  1.5rem;   /* 24px */
--button-height-sm:  2rem;     /* 32px */
--button-height-md:  2.5rem;   /* 40px — default */
--button-height-lg:  3rem;     /* 48px */
--button-height-xl:  3.5rem;   /* 56px */

--button-padding-x-xs: 0.5rem;
--button-padding-x-sm: 0.75rem;
--button-padding-x-md: 1rem;
--button-padding-x-lg: 1.5rem;
--button-padding-x-xl: 2rem;

--button-radius: var(--radius-md);
--button-transition: all var(--duration-normal) var(--ease-out);
```

### 12.2 Input

```css
--input-height-sm: 2rem;      /* 32px */
--input-height-md: 2.5rem;    /* 40px — default */
--input-height-lg: 3rem;      /* 48px */

--input-padding-x: 0.75rem;
--input-radius: var(--radius-md);
--input-border-width: 1px;

--input-focus-ring-width: 3px;
--input-focus-ring-color: oklch(0.715 0.120 84.2 / 0.3);  /* gold/30% */
```

### 12.3 Card

```css
--card-padding-sm: var(--space-4);    /* 16px */
--card-padding-md: var(--space-6);    /* 24px — default */
--card-padding-lg: var(--space-8);    /* 32px */

--card-radius: var(--radius-lg);
--card-shadow: var(--shadow-sm);
--card-border-width: 1px;
```

### 12.4 Icon sizes

```css
--icon-2xs: 0.75rem;   /* 12px */
--icon-xs:  0.875rem;  /* 14px */
--icon-sm:  1rem;      /* 16px */
--icon-md:  1.25rem;   /* 20px — default */
--icon-lg:  1.5rem;    /* 24px */
--icon-xl:  2rem;      /* 32px */
--icon-2xl: 2.5rem;    /* 40px */
--icon-3xl: 3rem;      /* 48px */
```

### 12.5 Touch targets (mobile-first)

```css
--touch-target-min: 44px;   /* Apple HIG minimum */
--touch-target-comfortable: 48px;  /* Material Design recommended */
```

---

## 13. Mobile — Flutter/Dart ThemeData

Equivalente em Dart. Usar package `flutter_theme_material` ou custom via `ThemeExtension`.

### 13.1 Light theme

```dart
class AppColors {
  // Brand
  static const goldPrimary = Color(0xFFC69C3F);  // --gold-500
  static const navyPrimary = Color(0xFF19202E);  // --navy-700
  
  // Neutrals
  static const gray1  = Color(0xFFFCFCFD);
  static const gray6  = Color(0xFFD4D5DA);
  static const gray9  = Color(0xFF74757D);
  static const gray12 = Color(0xFF18202A);
  
  // Feedback
  static const error   = Color(0xFFB83F28);  // --error-500
  static const warning = Color(0xFFDB9B30);
  static const success = Color(0xFF3D9B5A);
  static const info    = Color(0xFF3A7BB8);
}

ThemeData lightTheme = ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.light(
    primary: AppColors.goldPrimary,
    onPrimary: Colors.white,
    secondary: AppColors.navyPrimary,
    onSecondary: AppColors.gray1,
    surface: Colors.white,
    onSurface: AppColors.gray12,
    surfaceVariant: AppColors.gray3,
    outline: AppColors.gray6,
    error: AppColors.error,
    onError: Colors.white,
  ),
  textTheme: _buildTextTheme(AppColors.gray12),
  extensions: [AppSpacing.defaults, AppRadius.defaults],
);
```

### 13.2 Typography

```dart
TextTheme _buildTextTheme(Color onColor) => TextTheme(
  displayLarge:  GoogleFonts.manrope(fontSize: 72, height: 1.111, fontWeight: FontWeight.w700, color: onColor),
  displayMedium: GoogleFonts.manrope(fontSize: 60, height: 1.2,   fontWeight: FontWeight.w700, color: onColor),
  displaySmall:  GoogleFonts.manrope(fontSize: 48, height: 1.167, fontWeight: FontWeight.w700, color: onColor),
  headlineLarge: GoogleFonts.manrope(fontSize: 32, height: 1.25,  fontWeight: FontWeight.w600, color: onColor),
  headlineMedium: GoogleFonts.manrope(fontSize: 28, height: 1.286, fontWeight: FontWeight.w600, color: onColor),
  headlineSmall: GoogleFonts.manrope(fontSize: 24, height: 1.333, fontWeight: FontWeight.w600, color: onColor),
  titleLarge:    GoogleFonts.manrope(fontSize: 18, height: 1.556, fontWeight: FontWeight.w600, color: onColor),
  titleMedium:   GoogleFonts.manrope(fontSize: 16, height: 1.5,   fontWeight: FontWeight.w600, color: onColor),
  titleSmall:    GoogleFonts.manrope(fontSize: 14, height: 1.429, fontWeight: FontWeight.w600, color: onColor),
  bodyLarge:     GoogleFonts.manrope(fontSize: 16, height: 1.5,   fontWeight: FontWeight.w400, color: onColor),
  bodyMedium:    GoogleFonts.manrope(fontSize: 14, height: 1.429, fontWeight: FontWeight.w400, color: onColor),
  bodySmall:     GoogleFonts.manrope(fontSize: 12, height: 1.333, fontWeight: FontWeight.w400, color: onColor),
  labelLarge:    GoogleFonts.manrope(fontSize: 14, height: 1.429, fontWeight: FontWeight.w500, color: onColor),
  labelMedium:   GoogleFonts.manrope(fontSize: 12, height: 1.333, fontWeight: FontWeight.w500, color: onColor),
  labelSmall:    GoogleFonts.manrope(fontSize: 11, height: 1.455, fontWeight: FontWeight.w500, color: onColor),
);
```

### 13.3 ThemeExtension para spacing + radius

```dart
@immutable
class AppSpacing extends ThemeExtension<AppSpacing> {
  final double x0_5, x1, x2, x3, x4, x5, x6, x8, x10, x12, x16, x20, x24;
  
  const AppSpacing({
    this.x0_5 = 2, this.x1 = 4, this.x2 = 8, this.x3 = 12,
    this.x4 = 16, this.x5 = 20, this.x6 = 24, this.x8 = 32,
    this.x10 = 40, this.x12 = 48, this.x16 = 64, this.x20 = 80, this.x24 = 96,
  });
  
  static const defaults = AppSpacing();
  
  @override AppSpacing copyWith(...) => ...;
  @override AppSpacing lerp(...) => ...;
}
```

### 13.4 Dark theme

```dart
ThemeData darkTheme = lightTheme.copyWith(
  brightness: Brightness.dark,
  colorScheme: ColorScheme.dark(
    primary: AppColors.goldPrimary,
    surface: Color(0xFF15202A),  // --navy-900
    onSurface: AppColors.gray1,
    outline: Color(0xFF20283A),  // --navy-600
    // ...
  ),
  // ...
);
```

---

## 14. Source of truth (JSON schema)

Para não duplicar CSS + Dart manualmente, usar build pipeline:

```json
// tokens.json
{
  "$schema": "https://github.com/design-tokens/community-group/blob/main/docs/token-format.md",
  "color": {
    "gold": {
      "500": { "$value": "oklch(0.715 0.120 84.2)", "$type": "color" }
    },
    "navy": {
      "700": { "$value": "oklch(0.218 0.055 256.1)", "$type": "color" }
    }
  },
  "space": {
    "4": { "$value": "1rem", "$type": "dimension" }
  },
  "font": {
    "family": {
      "sans": { "$value": "'Manrope', system-ui, sans-serif", "$type": "fontFamily" }
    }
  }
}
```

**Build tools** (escolher um):
- [Style Dictionary](https://amzn.github.io/style-dictionary/) (Amazon) — gera CSS, Dart, SCSS, JS, XML
- [Specify](https://specifyapp.com) — UI + build
- [Token Transformer](https://github.com/six7/figma-tokens) — Figma ↔ code

**Recomendação**: Style Dictionary + pre-build step no `packages/ui` do monorepo Bun. Gera:
- `packages/ui/dist/tokens.css` (consumido por apps web)
- `app/lib/core/theme/tokens.dart` (consumido por Flutter)

---

## 15. Guidelines de uso

### 15.1 Ordem de preferência

Ao escolher cor/spacing/font em componente:

1. **Semantic tokens** (`--primary`, `--card`, `--space-4`) — preferir sempre
2. **Primitive tokens** (`--gold-500`, `--gray-6`) — só se semantic não servir
3. **Raw values** (`#FF0000`, `16px`) — **proibido** em componente novo; só permitido em styles inline excepcionais com comment justificando

### 15.2 Accessibility

- **Contraste**: todo par `color` + `on-color` garante ≥ 4.5:1 (AA). Validar com [APCA](https://www.myndex.com/APCA/) ou Chrome DevTools.
- **High-contrast mode**: suportar `prefers-contrast: more` via `[data-theme="high-contrast"]`.
- **Reduced motion**: respeitar `prefers-reduced-motion: reduce`.
- **Focus visible**: usar `--ring` sempre; nunca remover outline sem substituir.

### 15.3 Dark mode switch

```html
<html data-theme="dark">
```

Ou via JS/Preference:

```js
// Detect user preference
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
document.documentElement.setAttribute('data-theme', prefersDark ? 'dark' : 'light');
```

### 15.4 Flutter theme switch

```dart
MaterialApp(
  theme: lightTheme,
  darkTheme: darkTheme,
  themeMode: ThemeMode.system,  // ou .light, .dark
)
```

---

## 16. Referências

**Specs canônicas**:
- [Material Design 3 — Color system](https://m3.material.io/styles/color/system/overview)
- [Material Design 3 — Typography](https://m3.material.io/styles/typography/type-scale-tokens)
- [Radix Colors](https://www.radix-ui.com/colors) — escala 12-step
- [Apple Human Interface Guidelines — Color](https://developer.apple.com/design/human-interface-guidelines/color)
- [Shadcn/ui — Theming](https://ui.shadcn.com/docs/theming)
- [Tailwind CSS — Spacing](https://tailwindcss.com/docs/customizing-spacing)
- [GitHub Primer — Design tokens](https://primer.style/design/foundations/color)
- [Atlassian Design System — Tokens](https://atlassian.design/tokens)
- [OKLCH Color Picker](https://oklch.com/) — ferramenta para gerar cores OKLCH
- [Design Tokens Community Group — Format](https://github.com/design-tokens/community-group)

**Ferramentas de build**:
- [Style Dictionary](https://amzn.github.io/style-dictionary/)
- [Specify](https://specifyapp.com)
- [Tokens Studio for Figma](https://tokens.studio/)

**Validação de acessibilidade**:
- [APCA](https://www.myndex.com/APCA/)
- [WebAIM Contrast Checker](https://webaim.org/resources/contrastchecker/)
- [Color Oracle](https://colororacle.org/) — simulador de daltonismo

---

## 17. Rastreabilidade

- **D-108** em [[../STATE#d-108-—-paleta-oklch-canônica-web-main-css|STATE]] — paleta canônica = `main.css` atual
- Complementa [[../03-subprojects/web/design-system]] (guidelines web)
- Complementa [[../03-subprojects/mobile/design-system]] (guidelines mobile)
- Usado por todos os componentes em [[../03-subprojects/web/ui-catalog/_moc]] + [[../03-subprojects/mobile/ui-catalog/_moc]]

## Links

- [[_moc]]
- [[../CLAUDE]]
- [[../STATE#d-108]]
- [[../03-subprojects/web/design-system]]
- [[../03-subprojects/mobile/design-system]]
- [[../03-subprojects/feature-plan-2026-04-24]] (componentes DS Fase 12)


---

## 18. Regras inegociáveis de aplicação

> **Absorvido de** `_chaos/00-DESIGN-TOKENS.md` em **2026-04-25** (Fase B do plano de consolidação).
> Conteúdo abaixo é **delta** sobre o canônico OKLCH D-108: regras semânticas de uso, status badges Bronze→Diamante, keyframes prontos e convenção Iconify.
> **Tradução aplicada**: vocabulário neutro; "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".

### 18.1 Uso semântico (proibições e convenções)

1. **Superfícies Navy** (`--surface`): Sidebar, Topbar, Footer, overlays premium.
2. **Gold como acento** (`--primary` / `--accent`): CTAs, play, link ativo, progress, focus ring, tab ativa, badges de destaque.
3. **Branco / Light** (`--background` / `--card`): área de conteúdo, cards, inputs.
4. **`--destructive`** APENAS para deletar / bloquear / denunciar. **`--success`** para confirmações. **`--warning`** para travas (90 dias, cota, cooldown).
5. **NUNCA** vermelho para "encerrado" (use `--muted-foreground`).
6. **NUNCA** gradients fora do espectro Navy → Gold (LP blob system é exceção controlada).
7. **NUNCA** Gold + Vermelho juntos no mesmo componente — quebra hierarquia visual.
8. **NUNCA** hardcode `#hex` em componente novo — use sempre `var(--token)` semântico (Layer 2) ou `var(--gold-500)` (Layer 1) se semântico não servir.

### 18.2 Status badges — Reputação Bronze→Diamante

Aplicado em síndicos (D-051 / GR-028 — fórmula determinística, imutável via PATCH) e empresas:

```css
.badge-bronze {
  background: rgba(205, 127, 50, 0.15);
  color: #CD7F32;
  border: 1px solid rgba(205, 127, 50, 0.4);
}
.badge-prata {
  background: rgba(192, 192, 192, 0.15);
  color: #808080;
  border: 1px solid rgba(192, 192, 192, 0.4);
}
.badge-ouro {
  background: oklch(0.715 0.120 84.2 / 0.15);
  color: var(--primary);
  border: 1px solid oklch(0.715 0.120 84.2 / 0.4);
}
.badge-diamante {
  background: linear-gradient(
    135deg,
    rgba(185, 242, 255, 0.2),
    rgba(224, 195, 252, 0.2)
  );
  color: oklch(0.204 0.033 260.3);
  border: 1px solid rgba(185, 242, 255, 0.5);
}
.dark .badge-diamante { color: oklch(0.928 0.006 264.5); }
```

Diamante usa o **único gradient permitido fora do navy → gold** porque o tier Diamante é raro e merece destaque cromático.

### 18.3 Keyframes canônicos

Animações reutilizáveis em todo o `web/`. Durações usam tokens da §8.1; respeitar `prefers-reduced-motion` da §8.3.

```css
@keyframes fadeIn       { from { opacity: 0; } to { opacity: 1; } }
@keyframes fadeInUp     {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes slideInRight {
  from { opacity: 0; transform: translateX(20px); }
  to   { opacity: 1; transform: translateX(0); }
}
@keyframes scaleIn      {
  from { opacity: 0; transform: scale(0.95); }
  to   { opacity: 1; transform: scale(1); }
}
@keyframes heartBounce {
  0%   { transform: scale(1); }
  25%  { transform: scale(1.3); }
  50%  { transform: scale(0.9); }
  100% { transform: scale(1); }
}

.fade-in    { animation: fadeIn   var(--duration-slow) ease-out forwards; }
.fade-in-up { animation: fadeInUp var(--duration-moderate) ease-out forwards; }
```

### 18.4 Convenção Iconify (UnoCSS `presetIcons`)

Prefixo `i-` em UnoCSS (`uno.config.ts`):

```html
<div class="i-lucide-home w-5 h-5" />
<div class="i-lucide-play w-6 h-6 text-[var(--primary)]" />
```

Famílias canônicas:

| Categoria | Família + ícones-chave |
|---|---|
| Navegação | `lucide:home`, `search`, `bell`, `settings`, `user`, `menu`, `x`, `chevron-*` |
| Ação | `lucide:play`, `pause`, `upload`, `download`, `share`, `heart`, `bookmark`, `flag` |
| Conteúdo | `lucide:video`, `file-text`, `book-open`, `mic`, `image`, `paperclip` |
| Status | `lucide:check-circle`, `alert-triangle`, `x-circle`, `clock`, `shield` |
| Gestão | `lucide:clipboard-list`, `vote`, `bar-chart-2`, `calendar`, `file-pdf` |
| Comércio | `lucide:tag`, `percent`, `store`, `map-pin`, `ticket` |

Tamanhos canônicos: ver §12.4 (`--icon-2xs` ... `--icon-3xl`).

### 18.5 CVA — Button & Badge canônicos

```ts
// packages/ui/src/button-variants.ts
export const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-manrope font-semibold transition-all duration-200 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[var(--ring)] focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary:     "bg-[var(--primary)] text-[var(--primary-foreground)] hover:opacity-90 active:scale-[0.98]",
        secondary:   "bg-[var(--secondary)] text-[var(--secondary-foreground)] hover:bg-[var(--surface-variant)]",
        outline:     "border border-[var(--border)] bg-transparent text-[var(--foreground)] hover:bg-[var(--muted)]",
        ghost:       "bg-transparent text-[var(--foreground)] hover:bg-[var(--muted)]",
        destructive: "bg-[var(--destructive)] text-[var(--destructive-foreground)] hover:opacity-90",
        success:     "bg-[var(--success)] text-[var(--success-foreground)] hover:opacity-90",
        link:        "text-[var(--primary)] underline-offset-4 hover:underline bg-transparent",
      },
      size: {
        sm:        "h-8 px-3 text-xs gap-1.5",
        md:        "h-10 px-4 text-sm gap-2",
        lg:        "h-11 px-6 text-base gap-2",
        xl:        "h-12 px-8 text-base gap-2.5",
        icon:      "h-10 w-10",
        "icon-sm": "h-8 w-8",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  },
);

export const badgeVariants = cva(
  "inline-flex items-center rounded-full font-manrope font-medium",
  {
    variants: {
      variant: {
        default:     "bg-[var(--primary)]/15 text-[var(--primary)] border border-[var(--primary)]/30",
        secondary:   "bg-[var(--secondary)] text-[var(--secondary-foreground)]",
        destructive: "bg-[var(--destructive)]/15 text-[var(--destructive)]",
        success:     "bg-[var(--success)]/15 text-[var(--success)]",
        warning:     "bg-[var(--warning)]/15 text-[var(--warning)]",
        muted:       "bg-[var(--muted)] text-[var(--muted-foreground)]",
      },
      size: {
        sm: "px-2 py-0.5 text-[10px]",
        md: "px-2.5 py-0.5 text-xs",
        lg: "px-3 py-1 text-sm",
      },
    },
    defaultVariants: { variant: "default", size: "md" },
  },
);
```

### 18.6 Kobalte — primitives canônicos

Lista mínima de primitives Kobalte usados em `web/apps/*/`:

`Dialog`, `Popover`, `Select`, `Tabs`, `Accordion`, `Tooltip`, `Switch`, `Checkbox`, `RadioGroup`, `Progress`, `Separator`, `ColorMode` (theming).

Wrapper-padrão segue `splitProps` + `PolymorphicProps` + CVA (template em `packages/ui/src/button.tsx`). Ver [[../03-subprojects/web/patterns]] §"Kobalte + CVA + splitProps".

### 18.7 Rastreabilidade do delta

- Origem: `_chaos/00-DESIGN-TOKENS.md` (gerado 2026-02-23 — pré-migração).
- Absorvido: 2026-04-25 — Fase B do plano de consolidação.
- Decisões relacionadas: D-051 (reputação determinística), D-108 (paleta OKLCH canônica), D-126 (Morador Pagante removido), D-042 (SolidJS + Kobalte).
- Originais arquivados em `_archive/2026-04-24-chaos-processed/feature-specs-23-fev/00-DESIGN-TOKENS.md`.


---

## §19 Input states extraídos do Figma (Lote 0 — 2026-04-27)

Tokens novos derivados do Figma oficial `Master-sindico-APP---UX` (frame `148:667 LOGIN`), normalizados para OKLCH conforme [feedback design tokens canônicos](feedback_design_tokens_canonicos.md). Vault prevalece sobre Figma em divergências (cor ouro mantida como `#C69C3F`/`oklch(0.715 0.120 84.2)`; tipografia segue Manrope ignorando Poppins do Figma).

### Tokens adicionados

| Token | Valor OKLCH | Hex aprox | Origem Figma | Uso |
|---|---|---|---|---|
| `--input-border-active` | `oklch(0.557 0.243 268.5 / 0.4)` | `#465ff1` @ 40% | `rgba(70,95,241,0.4)` (border de input ativo) | Borda input em estado ativo/focus |
| `--input-fill-active` | `oklch(0.382 0.119 257.4 / 0.1)` | `#144789` @ 10% | `rgba(20,71,137,0.1)` (fill de input ativo) | Fill translúcido de input |
| `--input-placeholder` | `oklch(0.660 0.022 263.0)` | `#9c9aa5` | `#9c9aa5` (texto placeholder/secundário) | Placeholder + texto secundário sutil |
| `--progress-active` | `oklch(1.000 0 0)` | `#FFFFFF` | dot branco ativo (LOGIN tela painel esquerdo) | Dot ativo de progresso (multi-step) |
| `--progress-inactive` | `oklch(0.812 0.105 252.1)` | `#96bfff` | `#96bfff` (dots pendentes) | Dots pendentes de progresso |
| `--text-secondary` | `oklch(0.660 0.022 263.0)` | `#9c9aa5` | mesmo cinza Figma (alias semântico) | Helpers de form, descrições secundárias |

### Decisões aplicadas

- **OKLCH com alpha** (CSS Color Level 4): `oklch(L C H / alpha)` — suportado em todos browsers modernos. Borda e fill usam alpha para sobrepor sobre dark navy mantendo translucência.
- **`--text-secondary` é alias de `--input-placeholder`** — mesma cor servindo duas semânticas. Deliberado: simplifica grafo de tokens, evita drift.
- **`--progress-active` = branco**, **`--progress-inactive` = azul claro Figma** — espelha exatamente o LOGIN do Figma onde dot ativo é branco e demais são azul clarinho. NÃO usa `--primary`/`--accent` (gold) porque o sistema de progress dots do Figma é blue-tonal, separado do CTA gold.

### Shortcuts UnoCSS (consumidores) — em `web/apps/*/uno.config.ts`

```typescript
// Form Inputs
input-base       // h-12 + bg-input-fill-active + border-input-border-active + placeholder:text-input-placeholder + focus:ring-ring
input-label      // text-base font-medium mb-2
input-helper     // text-xs text-text-secondary
input-error      // text-xs text-destructive

// Tabs (segmented control, estilo Figma)
tabs-pill-list   // bg-input-fill-active + p-1 + rounded-lg + flex gap-1
tabs-pill-trigger // text-text-secondary + data-[selected]:bg-primary + data-[selected]:text-primary-foreground

// Progress dots (onboarding wizard)
progress-dot-active     // h-2 w-6 bg-progress-active rounded-full
progress-dot-inactive   // h-2 w-2 bg-progress-inactive rounded-full

// SSO Buttons (Google/Microsoft/Apple)
btn-sso          // h-12 w-28 bg-input-fill-active border-input-border-active
```

### Sincronização cross-app

Os 6 apps (`auth`, `cms`, `lms`, `forum`, `assembly`, `lp`) recebem `main.css` + `uno.config.ts` idênticos via cópia (sincronização manual até DT-WEB-008 mover pra preset compartilhado em `packages/ui`).

### Aplicação esperada

- **`apps/auth` LOGIN/SignIn/SignUp** — usa todos os 6 tokens (Lote 1 imediato)
- **`apps/cms` Onboarding síndico/empresa/parceiro** — usa progress-dot-* + tabs-pill-* + input-* (Lote 2)
- **Demais apps** — input-* + text-secondary genericamente

### Vizinhos

- [[../STATE#D-138 — Lote 0 reconciliação Manual IV vs Figma|D-138]] — decisão registrada
- [[../11-audit/audit-log/2026-04-27-lote-0-design-tokens]] — log detalhado da reconciliação
- [[../03-subprojects/web/_moc#Decisões de destaque|web/_moc]] — decisões web vivas
