# 00 — DESIGN TOKENS & FUNDAÇÃO VISUAL

> Documento base referenciado por TODOS os outros documentos de feature.
> Stack: SolidJS + UnoCSS + Kobalte/Core + CVA + Manrope + Michroma

---

## PALETA DE CORES (oklch)

Todas as cores estão definidas como CSS custom properties. NUNCA hardcode hex — usar SEMPRE var(--token).

### Light Mode

| Token | oklch | Hex aprox. | Uso |
|---|---|---|---|
| --background | oklch(0.976 0.003 264.5) | #F5F5F7 | Fundo geral |
| --foreground | oklch(0.204 0.033 260.3) | #1A1A2E | Texto corpo |
| --primary | oklch(0.715 0.120 84.2) | #C69C3F | CTA, gold |
| --primary-foreground | oklch(1.000 0.000 000) | #FFFFFF | Texto em gold |
| --secondary | oklch(0.218 0.055 256.1) | #071A33 | Navy base |
| --secondary-foreground | oklch(0.928 0.006 264.5) | #E8E8EC | Texto em navy |
| --accent | oklch(0.871 0.129 82.0) | #FFCC6A | Gold claro |
| --accent-foreground | oklch(0.204 0.033 260.3) | #1A1A2E | Texto em accent |
| --surface | oklch(0.218 0.055 256.1) | #071A33 | Sidebar, topbar |
| --surface-foreground | oklch(0.928 0.006 264.5) | #E8E8EC | Texto em surface |
| --surface-variant | oklch(0.274 0.077 256.3) | #0A274C | Navy médio |
| --muted | oklch(0.928 0.006 264.5) | #E8E8EC | Bg secundários |
| --muted-foreground | oklch(0.551 0.023 264.4) | #6B7280 | Texto secundário |
| --card | oklch(1.000 0.000 000) | #FFFFFF | Cards |
| --card-foreground | oklch(0.204 0.033 260.3) | #1A1A2E | Texto em cards |
| --border | oklch(0.872 0.009 258.3) | #D4D4D8 | Divisórias |
| --input | oklch(0.872 0.009 258.3) | #D4D4D8 | Borda inputs |
| --ring | oklch(0.715 0.120 84.2) | #C69C3F | Focus ring |
| --destructive | oklch(0.568 0.200 26.4) | #DC2626 | Erros |
| --success | oklch(0.627 0.170 149.2) | #16A34A | Confirmação |
| --warning | oklch(0.769 0.165 70.1) | #F59E0B | Alertas |
| --radius | 0.5rem | 8px | Border radius |

### Dark Mode (.dark)

| Token | oklch | Nota |
|---|---|---|
| --background | oklch(0.183 0.031 263.4) | Fundo escuro |
| --foreground | oklch(0.928 0.006 264.5) | Texto claro |
| --card | oklch(0.218 0.055 256.1) | Cards navy |
| --secondary | oklch(0.274 0.077 256.3) | Navy variant |
| --muted | oklch(0.274 0.077 256.3) | Bg muted |
| --muted-foreground | oklch(0.714 0.019 261.3) | Texto muted |
| --border | oklch(0.274 0.077 256.3) | Bordas |

Primary, accent, ring IGUAIS em dark mode — gold é constante da marca.

## REGRAS INEGOCIÁVEIS

1. Superfícies Navy (--surface): Sidebar, Topbar, Footer, overlays premium
2. Gold como acento (--primary/--accent): CTAs, play, link ativo, progress, focus ring, tab ativa, badges
3. Branco/Light (--background/--card): Área de conteúdo, cards, inputs
4. Destructive APENAS para deletar/bloquear/denunciar. Success para confirmações. Warning para travas (90 dias, cota)
5. NUNCA vermelho para "encerrado". NUNCA gradients fora de navy. NUNCA gold+vermelho juntos

## TIPOGRAFIA

Headings: font-family 'Michroma', sans-serif; letter-spacing 0.05em
h1 32px | h2 24px | h3 20px | h4 18px | h5 16px | h6 14px

Body: font-family 'Manrope', sans-serif
16px regular | 14px secundário | 13px metadata | 12px micro | 11px sidebar label (uppercase)
line-height: 1.6 parágrafos | 1.4 interfaces densas | 1.2 labels
Pesos: 400 regular | 500 labels/nav | 600 subtítulos | 700 títulos/CTAs | 800 hero (raro)

## ESPAÇAMENTO

4px (p-1) ícone+texto | 8px (p-2) chips | 12px (p-3) inputs | 16px (p-4) cards/grid | 20px (p-5) seções menores | 24px (p-6) modais | 32px (p-8) seções | 48px (p-12) página | 64px (p-16) hero

Grid: Desktop max-w-7xl px-6 | Tablet px-4 | Mobile px-4
Video Grid: Desktop cols-4 gap-4 | Tablet cols-3 gap-3 | Mobile cols-1 ou cols-2

## SOMBRAS

Level 0 nenhuma | Level 1 shadow-sm (cards) | Level 2 shadow-md (hover/dropdown) | Level 3 shadow-lg (modais) | Level 4 shadow-xl (overlays)
Dark: bordas em vez de sombras

## BORDER RADIUS

Botões rounded-md 8px | Cards rounded-lg 12px | Inputs rounded-md 8px | Avatares rounded-full | Thumbnails rounded-lg | Modais rounded-xl 16px | Tags rounded-full | Bottom Sheet rounded-t-2xl

## BREAKPOINTS

Mobile <768px | Tablet 768-1023px | Desktop ≥1024px | Wide ≥1280px | UltraWide ≥1536px

## CVA VARIANTS (Button)

```ts
export const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-manrope font-semibold transition-all duration-200 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-[var(--ring)] focus-visible:ring-offset-2 disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      variant: {
        primary: "bg-[var(--primary)] text-[var(--primary-foreground)] hover:opacity-90 active:scale-[0.98]",
        secondary: "bg-[var(--secondary)] text-[var(--secondary-foreground)] hover:bg-[var(--surface-variant)]",
        outline: "border border-[var(--border)] bg-transparent text-[var(--foreground)] hover:bg-[var(--muted)]",
        ghost: "bg-transparent text-[var(--foreground)] hover:bg-[var(--muted)]",
        destructive: "bg-[var(--destructive)] text-[var(--destructive-foreground)] hover:opacity-90",
        success: "bg-[var(--success)] text-[var(--success-foreground)] hover:opacity-90",
        link: "text-[var(--primary)] underline-offset-4 hover:underline bg-transparent",
      },
      size: {
        sm: "h-8 px-3 text-xs gap-1.5",
        md: "h-10 px-4 text-sm gap-2",
        lg: "h-11 px-6 text-base gap-2",
        xl: "h-12 px-8 text-base gap-2.5",
        icon: "h-10 w-10",
        "icon-sm": "h-8 w-8",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
);
```

## CVA VARIANTS (Badge)

```ts
export const badgeVariants = cva(
  "inline-flex items-center rounded-full font-manrope font-medium",
  {
    variants: {
      variant: {
        default: "bg-[var(--primary)]/15 text-[var(--primary)] border border-[var(--primary)]/30",
        secondary: "bg-[var(--secondary)] text-[var(--secondary-foreground)]",
        destructive: "bg-[var(--destructive)]/15 text-[var(--destructive)]",
        success: "bg-[var(--success)]/15 text-[var(--success)]",
        warning: "bg-[var(--warning)]/15 text-[var(--warning)]",
        muted: "bg-[var(--muted)] text-[var(--muted-foreground)]",
      },
      size: { sm: "px-2 py-0.5 text-[10px]", md: "px-2.5 py-0.5 text-xs", lg: "px-3 py-1 text-sm" },
    },
    defaultVariants: { variant: "default", size: "md" },
  }
);
```

## STATUS BADGES (Síndico)

```css
.badge-bronze { background: rgba(205,127,50,0.15); color: #CD7F32; border: 1px solid rgba(205,127,50,0.4); }
.badge-prata { background: rgba(192,192,192,0.15); color: #808080; border: 1px solid rgba(192,192,192,0.4); }
.badge-ouro { background: oklch(0.715 0.120 84.2 / 0.15); color: var(--primary); border: 1px solid oklch(0.715 0.120 84.2 / 0.4); }
.badge-diamante { background: linear-gradient(135deg, rgba(185,242,255,0.2), rgba(224,195,252,0.2)); color: #1a1a2e; border: 1px solid rgba(185,242,255,0.5); }
.dark .badge-diamante { color: #E8E8EC; }
```

## ANIMAÇÕES

```css
@keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
@keyframes fadeInUp { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
@keyframes slideInRight { from { opacity: 0; transform: translateX(20px); } to { opacity: 1; transform: translateX(0); } }
@keyframes scaleIn { from { opacity: 0; transform: scale(0.95); } to { opacity: 1; transform: scale(1); } }
@keyframes heartBounce { 0% { transform: scale(1); } 25% { transform: scale(1.3); } 50% { transform: scale(0.9); } 100% { transform: scale(1); } }
.fade-in { animation: fadeIn 0.3s ease-out forwards; }
.fade-in-up { animation: fadeInUp 0.4s ease-out forwards; }
```

## ÍCONES (Iconify + UnoCSS preset-icons)

```html
<div class="i-lucide-home w-5 h-5" />
<div class="i-lucide-play w-6 h-6 text-[var(--primary)]" />
```

Navegação: lucide (home, search, bell, settings, user, menu, x, chevron-*)
Ação: lucide (play, pause, upload, download, share, heart, bookmark, flag)
Conteúdo: lucide (video, file-text, book-open, mic, image, paperclip)
Status: lucide (check-circle, alert-triangle, x-circle, clock, shield)
Gestão: lucide (clipboard-list, vote, bar-chart-2, calendar, file-pdf)
Comércio: lucide (tag, percent, store, map-pin, ticket)

## KOBALTE COMPONENTS

Dialog, Popover, Select, Tabs, Accordion, Tooltip, Switch, Checkbox, RadioGroup, Progress, Separator, ColorMode
