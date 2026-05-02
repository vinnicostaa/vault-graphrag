---
title: Web — Design System
type: spec
tags: [subproject, web, master-sindico, v2, design-system, tokens, unocss, kobalte, identidade-visual, manual-iv, michroma, oklch]
source: Development/web/apps/auth/uno.config.ts (212 linhas) + apps/auth/src/main.css (276 linhas) + Manual Identidade Visual + packages/ui/src/button.tsx
created: 2026-04-23
updated: 2026-04-23
aliases: [design-system, tokens, manual-iv]
---

# Web — Design System

Design system **real** destilado do código em `Development/web/apps/*/uno.config.ts` e `apps/*/src/main.css`, cruzado com o **Manual Identidade Visual — Master Síndico** (PDF em `Development/web/old/docs/` + `90-ingested/legacy-docs/`). Paridade mobile em [[../mobile/ui-catalog]].

> **Paleta é imutável**. Cores em **OKLCH** (CSS vars) — reflete decisão de produto gravada em `main.css`. Mudança requer aprovação explícita.
> PT-BR para copy; EN para identifiers/tokens.

Fontes:
- `Development/web/apps/auth/uno.config.ts` (212 linhas — idêntico entre apps) — tema UnoCSS + shortcuts.
- `Development/web/apps/auth/src/main.css` (276 linhas — idêntico entre apps) — CSS vars tokens (light + dark) + LP blob classes.
- `Development/web/packages/ui/src/button.tsx` — template CVA × Kobalte.
- `Development/web/CLAUDE.md` §UnoCSS convenções (linhas 113-128).

---

## 1. Identidade visual (brand)

### 1.1 Logo + símbolo

Fonte canônica: **Manual IV p.2-4** (arquivo em `web/old/docs/Manual Identidade Visual - Master Síndico.pdf`).

- **Símbolo**: composto por **Brasão + Play + Condomínio + Coroa** em disposição combinada.
- **Wordmark**: "MASTER SÍNDICO" em tipografia **Michroma** (§3).
- **Aplicações válidas**:
  - Positiva (cor sobre fundo claro).
  - Negativa (branco sobre fundo escuro).
  - Monocromática ouro (sobre fundo escuro).
- **Assets disponíveis** (verificado em `apps/*/public/`):
  - `logo-azul.png`, `logo-branco.png`, `logo-dourado.png`
  - `icon-azul.png`, `icon-branco.png`, `icon-dourado.png`
  - `favicon.png`

### 1.2 Usos proibidos

- ❌ Esticar, distorcer, rotacionar logo.
- ❌ Aplicar gradient fora da paleta oficial.
- ❌ Trocar fonte do wordmark.
- ❌ Adicionar sombras decorativas ao logo.
- ❌ Usar ícone sem espaço de respiro mínimo (= altura do "M" do wordmark).

---

## 2. Paleta de cores (OKLCH + CSS vars)

### 2.1 Tokens primários — Brand (imutável)

Valores **exatos** do código (`apps/auth/src/main.css`):

| Token | Light (OKLCH) | Dark (OKLCH) | Aprox. HEX | Uso |
|---|---|---|---|---|
| `--primary` | `oklch(0.715 0.120 84.2)` | idem | **#C49A2A** ≈ #C69C3F (gold CTA) | Botão primário, CTA destaque, foco ring |
| `--primary-foreground` | `oklch(1 0 0)` | idem | `#FFFFFF` | Texto sobre primary |
| `--secondary` | `oklch(0.218 0.055 256.1)` | `oklch(0.274 0.077 256.3)` | **#1A2340** ≈ #071A33 (navy) | Botão secondary, surface dark institutional |
| `--secondary-foreground` | `oklch(0.928 0.006 264.5)` | idem | near-white | Texto sobre navy |
| `--accent` | `oklch(0.871 0.129 82.0)` | idem | **#E8C84A** ≈ #FFCC6A (gold light) | Hover CTA, highlight, selected states |
| `--accent-foreground` | `oklch(0.204 0.033 260.3)` | idem | near-black | Texto sobre accent |

> **Reconciliação Manual IV** (ouro primário `#C69C3F` + navy `#071A33`) vs código (`oklch(0.715 0.120 84.2)` ≈ `#C49A2A` + `oklch(0.218 0.055 256.1)` ≈ `#1A2340`): divergência de ±2 pontos no matiz. OKLCH escolhido por **perceptual uniformity** (dark mode). Registrar D-### em [[../../STATE]] ratificando OKLCH canônico.

### 2.2 Background + Foreground (light/dark)

| Token | Light | Dark | Uso |
|---|---|---|---|
| `--background` | `oklch(0.976 0.003 264.5)` off-white | `oklch(0.183 0.031 263.4)` deep navy | Body background |
| `--foreground` | `oklch(0.204 0.033 260.3)` near-black | `oklch(0.928 0.006 264.5)` near-white | Body text |

### 2.3 Surface + Card + Popover

Surface variants — paleta navy (Manual IV):

| Token | Light | Dark | Uso |
|---|---|---|---|
| `--surface` | `oklch(0.218 0.055 256.1)` navy | `oklch(0.218 0.055 256.1)` | Sidebar, topbar, footer |
| `--surface-foreground` | `oklch(0.928 0.006 264.5)` | idem | Texto sobre surface |
| `--surface-variant` | `oklch(0.274 0.077 256.3)` navy lighter | idem | Hover surface items |
| `--card` | `oklch(1 0 0)` white | `oklch(0.218 0.055 256.1)` navy | Cards de conteúdo |
| `--card-foreground` | `oklch(0.204 0.033 260.3)` | `oklch(0.928 0.006 264.5)` | Texto em card |
| `--popover` | `oklch(1 0 0)` white | `oklch(0.218 0.055 256.1)` | Popover/dropdown |
| `--popover-foreground` | `oklch(0.204 0.033 260.3)` | `oklch(0.928 0.006 264.5)` | Texto em popover |

### 2.4 Muted

| Token | Light | Dark | Uso |
|---|---|---|---|
| `--muted` | `oklch(0.928 0.006 264.5)` | `oklch(0.274 0.077 256.3)` | Background sutil |
| `--muted-foreground` | `oklch(0.551 0.023 264.4)` | `oklch(0.714 0.019 261.3)` | Texto secundário, placeholder |

### 2.5 Borders + Inputs

| Token | Light | Dark | Uso |
|---|---|---|---|
| `--border` | `oklch(0.872 0.009 258.3)` | `oklch(0.274 0.077 256.3)` | Border divisório |
| `--input` | idem `--border` | idem | Input border |
| `--ring` | `oklch(0.715 0.120 84.2)` gold | idem | Focus ring (focus-visible) |

### 2.6 Feedback semântico

| Token | Light | Dark | Hex aprox. | Uso |
|---|---|---|---|---|
| `--destructive` | `oklch(0.568 0.200 26.4)` | `oklch(0.505 0.190 27.5)` | `#C92C2C` | Erro, deletar, risco crítico |
| `--destructive-foreground` | `oklch(1 0 0)` | `oklch(0.936 0.031 17.7)` | — | Texto |
| `--success` | `oklch(0.627 0.170 149.2)` | idem | `#1F8E4E` | Validação positiva, tier Diamante |
| `--success-foreground` | `oklch(1 0 0)` | idem | — | Texto |
| `--warning` | `oklch(0.769 0.165 70.1)` | idem | `#D48A0A` | Pendência, prazo próximo |
| `--warning-foreground` | `oklch(1 0 0)` | idem | — | Texto |

> **Ausente mas necessário** (adicionar M1): `--info` para estados informativos (tooltip, badges informativos). Sugestão: `oklch(0.6 0.150 240)` ≈ `#2974C9`.

### 2.7 Paleta especial LP (hero)

Em `uno.config.ts` (linhas 138-147) — **fora** das CSS vars, para o hero da landing page:

```js
lp: {
  "navy-deep":   "#0a1628",
  "navy-base":   "#0d1f3c",
  "navy-mid":    "#1a3a6e",
  "navy-spread": "#112d5a",
  "gold-soft":   "#8a6200",
  "gold-amber":  "#b07d00",
  "gold-bright": "#d4a017",
  bg:            "#06080f",
}
```

Usadas nas classes `.lp-blob-*` em `main.css` (linhas 155-276) — sistema de blobs com `filter: blur()` + `mix-blend-mode: screen/multiply` para criar atmosfera navy→gold do hero. Uso exclusivo em `apps/lp`.

### 2.8 Radius + Border

- `--radius: 0.5rem` (8px) — default padrão.
- Tailwind/UnoCSS mapeia: `rounded-sm` (2px) · `rounded` (4px) · `rounded-md` (6px) · `rounded-lg` (8px = var) · `rounded-xl` (12px) · `rounded-2xl` (16px) · `rounded-full`.

---

## 3. Tipografia

### 3.1 Fontes (via `presetWebFonts` Google)

```js
// uno.config.ts
fonts: {
  sans: [{ name: "Manrope", weights: ["400","500","600","700","800"], italic: true },
         { name: "sans-serif", provider: "none" }],
  display: [{ name: "Michroma", weights: ["400"] },
            { name: "sans-serif", provider: "none" }],
  mono: ["Fira Code", "Fira Mono:400,700"],
}
```

### 3.2 Aplicação

- **`font-sans` = Manrope** — body, forms, UI geral. Weights 400 (regular) → 800 (extrabold).
- **`font-display` = Michroma** — headlines institucionais + brand wordmark. Weight 400 único. **`letter-spacing: 0.05em` obrigatório** (aplicado em `h1-h6` via CSS global — `main.css` linha 150-153).
- **`font-mono` = Fira Code** — IDs, tokens, código inline (raro em UI produto).

### 3.3 Shortcuts UnoCSS (design system tipográfico)

Definidos em `uno.config.ts` (linhas 180-188):

| Shortcut | Expandido | Uso |
|---|---|---|
| `text-display-h1` | `font-display text-4xl font-normal tracking-tight lg:text-5xl` | H1 hero/landing |
| `text-display-h2` | `font-display text-3xl font-normal tracking-tight` | H2 seções |
| `text-display-h3` | `font-display text-2xl font-normal tracking-tight` | H3 cards/modal |
| `text-body` | `font-sans text-base leading-7` | Parágrafo (16px) |
| `text-small` | `font-sans text-sm font-medium leading-none` | Caption, meta |
| `text-muted` | `font-sans text-sm text-muted-foreground` | Texto secundário |

Adicionais previstos (M1 — FE-backlog):
- `text-display-h4` → 1.5rem (24px)
- `text-large` → 1.125rem (18px) body grande
- `text-xs` já está no preset default

### 3.4 Line-height + Tracking

- Body (`text-body`): `leading-7` (1.75) — conforto em leitura prosa.
- Headings: `tracking-tight` (-0.025em) + Michroma já tem 0.05em — compensa visualmente.
- Códigos/números: `font-mono`.

### 3.5 Tamanhos baseline

```
xs   0.75rem (12px) — legal, meta fine-print
sm   0.875rem (14px) — caption, labels
base 1rem (16px) — body default
lg   1.125rem (18px) — subtítulo
xl   1.25rem (20px) — H4 alternativo
2xl  1.5rem (24px) — H3
3xl  1.875rem (30px) — H2
4xl  2.25rem (36px) — H1
5xl  3rem (48px) — Hero H1 (LP)
```

---

## 4. Spacing, layout e grid

### 4.1 Base: 4px/8px (via Tailwind/UnoCSS default)

```
0.5 = 2px   1 = 4px    2 = 8px    3 = 12px
4   = 16px  5 = 20px   6 = 24px   8 = 32px
10  = 40px  12 = 48px  16 = 64px  20 = 80px
```

Uso preferido: múltiplos de 4 (padrão UnoCSS). Preferência para **8px grid**: gaps/padding `gap-2/4/6/8`, não `gap-3/5/7`.

### 4.2 Layout shortcuts (uno.config.ts linhas 191-199)

| Shortcut | Uso |
|---|---|
| `flex-center` | `flex items-center justify-center` |
| `flex-between` | `flex items-center justify-between` |
| `glass-panel` | `bg-white/10 backdrop-blur-md border border-white/20` — overlays/cards hero |
| `bg-grid` | background `linear-gradient` duplo (rgba 0.08) simulando grid 40×40px |

### 4.3 Container widths

Sugestão (FE-backlog M1 — não ainda no código):

| Shortcut | Max-width | Uso |
|---|---|---|
| `container-sm` | 640px | Form centralizado |
| `container-md` | 800px | Artigo/leitura |
| `container-lg` | 1200px | Dashboard síndico |
| `container-xl` | 1440px | Admin panel |

---

## 5. Breakpoints

UnoCSS default (idênticos Tailwind):

| Breakpoint | Min-width | Target |
|---|---|---|
| (default) | 0 | Mobile |
| `sm` | 640 | Phone landscape |
| `md` | 768 | Tablet portrait |
| `lg` | 1024 | Tablet landscape + laptop |
| `xl` | 1280 | Desktop |
| `2xl` | 1536 | Large desktop |

Mobile-first **obrigatório** (ver [[requirements#3]]).

---

## 6. Componentes catalogados (a evoluir M1-M3)

Atualmente em `packages/ui/src/`:
- ✅ `Button` (`button.tsx` — CVA + Kobalte PolymorphicProps)
- ✅ `cn()` helper (`lib/utils.ts`)

Previsto M1 (FE-backlog, ver [[tasks#M1]]):

### 6.1 Atoms (M1)

| Componente | Kobalte primitive | CVA variants | Estados |
|---|---|---|---|
| **Button** ✅ | `Button.Root` | variant: default/destructive/outline/secondary/ghost/link; size: default/sm/lg/icon | idle, hover, focus-visible, active, disabled, loading |
| **Input** | — (native `<input>` + Kobalte `TextField.Input`) | size: sm/md/lg; state: default/error/success | idle, focus, error, disabled, readOnly |
| **Textarea** | — | size · resize | idle, focus, error |
| **Checkbox** | `Checkbox.Root` + `Indicator` | — | unchecked, checked, indeterminate, disabled |
| **Radio** | `RadioGroup.Root` + `.Item` | — | unchecked, checked, disabled |
| **Switch** | `Switch.Root` | — | off, on, disabled |
| **Badge** | — | variant: default/secondary/outline/destructive/success/warning/info | — |
| **Spinner** | — | size: sm/md/lg | — |
| **Divider** | `Separator.Root` | orientation: horizontal/vertical | — |
| **Avatar** | `Avatar.Root` + `Image` + `Fallback` | size: xs/sm/md/lg/xl | loaded, fallback |
| **Icon** | `<i class="i-lucide-*">` | — | — |

### 6.2 Molecules (M1-M2)

| Componente | Kobalte primitive | Uso |
|---|---|---|
| **FormField** | `TextField.Root` + Label + Input + ErrorMessage + Description | Wrap padrão de input com label, helper, erro |
| **Card** | — | Container de bloco (header, content, footer) |
| **MenuItem** | `DropdownMenu.Item` | Lista clickável |
| **TabStrip** | `Tabs.Root` + List + Trigger + Content | Nav abas |
| **Breadcrumb** | — | Navegação hierárquica |
| **Pagination** | — | Cursor/offset controls |
| **Tooltip** | `Tooltip.Root` + Trigger + Content | Hover/focus info |
| **Kbd** | — | Tecla visual (`⌘K`) |

### 6.3 Organisms (M1-M3)

| Componente | Kobalte primitive | Uso |
|---|---|---|
| **Dialog** (Modal) | `Dialog.Root` + Portal + Overlay + Content + Close | Confirmações, forms complexos |
| **Drawer** | `Dialog.Root` + custom transition | Sidebar mobile, painel lateral |
| **Toast** | `Toast.Root` (Kobalte) + custom host | Feedback ação |
| **Popover** | `Popover.Root` + Trigger + Content | Context menus, filtros |
| **DataTable** | — (custom com `For` + TanStack Table) | Listagens paginadas |
| **Combobox** | `Combobox.Root` | Select com busca |
| **DatePicker** | `DatePicker.Root` | Data única |
| **DateRangePicker** | `DatePicker.Root` + range mode | Filtros dashboard |
| **Select** | `Select.Root` | Dropdown estruturado |
| **UploadDropzone** | — | Upload Mux signed |
| **Stepper** | — | Onboarding multi-step |
| **CommandPalette** | `Combobox.Root` + keyboard shortcut | `⌘K` quick actions |
| **NavSidebar** | — | Navegação principal por persona |
| **TimelineEntryCard** | — | Card de entrada timeline |
| **AnnouncementBanner** | — | Topbar de aviso plataforma |
| **EmptyState** | — | Tela sem dados + CTA |

---

## 7. Shortcuts de botões (design system)

Em `uno.config.ts` linhas 155-177:

```js
// Base comum a todos
["btn-base", "inline-flex items-center justify-center rounded-md font-medium transition-colors duration-200 disabled:opacity-50 disabled:pointer-events-none ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2"]

// Variants
["btn-primary",   "btn-base bg-primary text-primary-foreground hover:bg-accent"]            // gold → gold-light hover
["btn-secondary", "btn-base bg-secondary text-secondary-foreground hover:bg-surface-variant"] // navy → navy-variant hover
["btn-outline",   "btn-base border border-input bg-background hover:bg-accent hover:text-accent-foreground"]
["btn-ghost",     "btn-base hover:bg-accent hover:text-accent-foreground"]
["btn-link",      "btn-base text-primary underline-offset-4 hover:underline"]

// Sizes
["btn-sm",   "h-9 px-3 text-xs"]       // 36px
["btn-md",   "h-10 px-4 py-2"]         // 40px (default)
["btn-lg",   "h-11 px-8 text-lg"]      // 44px (touch target OK)
["btn-icon", "h-10 w-10 p-0"]          // 40×40 (⚠️ abaixo do 44×44 WCAG — revisar M1)
```

> **Ausente**: `btn-destructive`. `packages/ui/src/button.tsx` referencia variant "destructive" mapeado para `btn-destructive`, **mas esse shortcut não está definido** em `uno.config.ts`. Adicionar em FE-M1: `["btn-destructive", "btn-base bg-destructive text-destructive-foreground hover:bg-destructive/90"]`.

---

## 8. Ícones (Lucide)

Prefix `i-` (presetIcons UnoCSS):

```tsx
<i class="i-lucide-home" />
<i class="i-lucide-check text-success" />
<i class="i-lucide-alert-circle text-warning" />
<i class="i-lucide-x text-destructive" />
```

Catalog canônico: **Lucide** (`@iconify-json/lucide` resolvido pelo preset). Outros sets disponíveis (Heroicons `i-heroicons-*`, Carbon `i-carbon-*`) mas **preferência Lucide** para consistência.

**Tamanhos padrão**: `text-sm` (14) · `text-base` (16 default) · `text-lg` (18) · `text-xl` (20) · `text-2xl` (24). Ícones inline herdam `font-size`.

---

## 9. Motion / Animation

### 9.1 Keyframes globais (main.css linhas 103-125 + uno.config.ts preflights linhas 63-82)

```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}
@keyframes slide-in-right {
  from { opacity: 0; transform: translateX(20px); }
  to   { opacity: 1; transform: translateX(0); }
}
@keyframes fadeIn { from { opacity: 0 } to { opacity: 1 } }
@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(10px) }
  to   { opacity: 1; transform: translateY(0) }
}
```

### 9.2 Classes aplicáveis

| Classe | Duração | Uso |
|---|---|---|
| `animate-fade-in` | 0.5s ease-out forwards | Card aparecer |
| `animate-slide-in-right` | 0.4s ease-out forwards | Drawer lateral |
| `fade-in` | 0.3s ease-out forwards | Transição rota |
| `fade-in-up` | 0.4s ease-out forwards | Lista item |

### 9.3 Durations tokens (sugestão M1)

| Token | Valor | Uso |
|---|---|---|
| `duration-fast` | 150ms | Hover, focus |
| `duration-base` | 200ms | Transitions gerais (já em `btn-base`) |
| `duration-slow` | 300ms | Modais abrindo, drawers |
| `duration-slower` | 500ms | Animações de entrada (fade-in) |

### 9.4 Respeito a `prefers-reduced-motion`

Adicionar em `main.css` (M1 — FE-backlog):

```css
@media (prefers-reduced-motion: reduce) {
  * { animation-duration: 0.01ms !important; transition-duration: 0.01ms !important; }
}
```

---

## 10. Dark mode

### 10.1 Mecânica

- **Kobalte `ColorModeProvider`** + `ColorModeScript` injetam lógica de detecção + persistência.
- `ThemeSync` (entry `index.tsx`) escuta `useColorMode()` → alterna classe `dark` em `<html>`.
- CSS vars em `main.css` redefinem-se sob `.dark, [data-kb-theme="dark"]`.
- UnoCSS preset **não precisa** de configuração específica — usa `var(--token)`.

### 10.2 Cobertura

- Sistema default: `colorMode: 'system'` → segue `prefers-color-scheme`.
- Override manual persistido em `localStorage` (Kobalte gerencia). **Não armazenar PII.**
- Todas as cores resolvem via CSS vars → troca automática.

### 10.3 Paleta LP (hero)

LP (`apps/lp`) mantém **paleta fixa dark** (blobs hard-coded) independente de theme switch. Decisão de produto — hero é sempre noturno.

---

## 11. Scrollbar customizada

`main.css` linhas 128-141:

```css
::-webkit-scrollbar { width: 8px; }
::-webkit-scrollbar-track { background: transparent; }
::-webkit-scrollbar-thumb { background: var(--color-border); border-radius: 4px; }
::-webkit-scrollbar-thumb:hover { background: var(--color-muted-foreground); }
```

> Bug conhecido: usa `var(--color-border)` mas CSS vars definidas são `--border` (sem prefix `--color-`). Corrigir M1 FE-backlog: trocar para `var(--border)` e `var(--muted-foreground)`.

---

## 12. Contraste — matriz WCAG AA

Tokens calibrados em mente. Validação dura (a executar M1 — FE-backlog):

| Par | Ratio esperado | WCAG AA (4.5 / 3.0) |
|---|---|---|
| `primary` (gold) sobre `primary-foreground` (white) | ~3.4:1 | ⚠️ texto grande 18px+ only; OU ajustar hue |
| `secondary` (navy) sobre `secondary-foreground` (near-white) | ~11:1 | ✅ passa folgadamente |
| `destructive` sobre `destructive-foreground` (white) | ~4.8:1 | ✅ passa |
| `success` sobre `white` | ~4.9:1 | ✅ passa |
| `warning` sobre `white` | ~3.6:1 | ⚠️ texto grande only |
| `foreground` (near-black) sobre `background` (off-white) | ~19:1 | ✅ folgadamente |

Ações M1:
1. Validar toda combinação com `contrast-ratio` CLI.
2. Ajustar `--warning-foreground` ou `--warning` se falhar em textos pequenos.
3. Documentar em [[requirements#2]] gate CI usando Playwright + `pa11y`.

---

## 13. Kobalte attribute selectors (dark-aware)

Kobalte exporta data-attributes ricos que podem ser usados em UnoCSS variants:

```tsx
<Dialog.Trigger data-kb-theme="dark">  {/* automático via ColorModeProvider */}
<Tabs.Trigger data-selected="true">
<Combobox.Item data-highlighted="true">
<Dialog.Content data-expanded="true">
```

Uno variants (FE-backlog M1):
```js
// uno.config.ts
variants: [
  // matcher para data-state
  (matcher) => {
    if (!matcher.startsWith('kb-open:')) return matcher;
    return { matcher: matcher.slice(8), selector: (s) => `${s}[data-expanded="true"]` };
  },
],
```

---

## 14. Tokens resumo (exportação)

Tokens canônicos para outros consumidores (mobile, Figma sync):

```jsonc
{
  "colors": {
    "primary":           { "light": "oklch(0.715 0.120 84.2)", "dark": "oklch(0.715 0.120 84.2)" },
    "primary-foreground":{ "light": "oklch(1 0 0)",             "dark": "oklch(1 0 0)" },
    "secondary":         { "light": "oklch(0.218 0.055 256.1)", "dark": "oklch(0.274 0.077 256.3)" },
    "accent":            { "light": "oklch(0.871 0.129 82.0)",  "dark": "oklch(0.871 0.129 82.0)" },
    "background":        { "light": "oklch(0.976 0.003 264.5)", "dark": "oklch(0.183 0.031 263.4)" },
    "foreground":        { "light": "oklch(0.204 0.033 260.3)", "dark": "oklch(0.928 0.006 264.5)" },
    "destructive":       { "light": "oklch(0.568 0.200 26.4)",  "dark": "oklch(0.505 0.190 27.5)" },
    "success":           { "light": "oklch(0.627 0.170 149.2)", "dark": "oklch(0.627 0.170 149.2)" },
    "warning":           { "light": "oklch(0.769 0.165 70.1)",  "dark": "oklch(0.769 0.165 70.1)" }
  },
  "fonts": {
    "sans": "Manrope, sans-serif",
    "display": "Michroma, sans-serif",
    "mono": "Fira Code, Fira Mono, monospace"
  },
  "radius": "0.5rem",
  "spacing": { "unit": "4px", "grid": "8px" }
}
```

---

## 15. Roadmap design system

| Sprint | Entrega |
|---|---|
| M1 | Atoms completos (Button ✅, Input, Checkbox, Radio, Switch, Badge, Spinner, Avatar); molecules (FormField, Card, Tab); 1 organism (Dialog). Contraste validado. `btn-destructive` shortcut. `--info` token. |
| M2 | Molecules restantes + organisms: DataTable, Combobox, DatePicker, UploadDropzone, Stepper. CommandPalette. Dark mode validação QA. |
| M3 | Organismos avançados: Assembleia telão, live-vote components. Storybook (opcional — avaliar custo/benefício). Figma sync de tokens. |

---

## 16. Links

- [[README]] — overview web
- [[architecture]] — entry point + ColorModeProvider
- [[patterns]] — CVA + Kobalte + splitProps (template Button)
- [[ui-catalog]] — telas × tokens consumidos
- [[requirements]] — a11y/CWV/contraste gates
- [[../mobile/ui-catalog]] — paridade mobile (tokens espelhados)
- `Development/web/apps/auth/uno.config.ts` — design system UnoCSS fonte
- `Development/web/apps/auth/src/main.css` — CSS vars tokens fonte
- `Development/web/packages/ui/src/button.tsx` — template de componente
- `Development/web/old/docs/Manual Identidade Visual - Master Síndico.pdf` — brand source
