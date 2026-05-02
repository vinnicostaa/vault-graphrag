# MASTER SÍNDICO — Frontend Design System & UI Specification
## Guia Definitivo para Implementação Visual (Sprints 1 & 2)

**Versão:** 1.0  
**Stack:** SolidJS · TanStack Router/Query/Form · Kobalte/Core · UnoCSS · Rsbuild  
**Data:** Fevereiro 2026  

---

## 1. FILOSOFIA DE DESIGN

### 1.1 Direção Estética: "Luxury Governance"

O Master Síndico não é rede social, não é site de cursos, não é marketplace. É uma plataforma de **governança condominial aplicada**. A estética deve transmitir:

- **Autoridade institucional** — como um portal bancário premium, não como um app de entretenimento
- **Confiança e sofisticação** — o ouro e o navy comunicam status, seriedade e valor
- **Clareza operacional** — informação densa apresentada com respiração e hierarquia
- **Modernidade contida** — transições suaves (0.3s ease-out), sem fireworks nem efeitos gratuitos

**Tom visual:** Refinado, sóbrio, institucional com toques de luxo. Pense num cruzamento entre um dashboard Bloomberg e a interface do YouTube, temperado pela paleta de um banco private.

### 1.2 Princípios Inegociáveis

1. **Navy é o palco, Ouro é o holofote** — superfícies navy (#071A33 / #0A274C) dominam sidebar, topbar e footer. O ouro (#C69C3F / #FFCC6A) aparece apenas em CTAs primários, badges de status e acentos pontuais.
2. **Conteúdo é rei** — cards, vídeos e dados ocupam 70%+ do viewport. Chrome UI é mínimo.
3. **Feedback é privado** — nunca exibir contadores públicos de likes/comentários para não-donos do conteúdo. A interface é "cega" para terceiros.
4. **Acesso com propósito** — se o plano não permite, a tela nem renderiza. Mostrar paywall elegante, nunca tela quebrada.
5. **Mobile-first, desktop-polished** — todo componente nasce para 375px e escala para 1440px+.

### 1.3 O que NÃO fazer

- **Nunca** usar gradientes roxos, neons, ou estética "crypto/web3"
- **Nunca** usar contadores públicos de engajamento (likes, comments, views para não-donos)
- **Nunca** usar Inter, Roboto, Arial como fonte primária — a marca usa **Michroma** (headings) e **Manrope** (body)
- **Nunca** criar layouts tipo feed infinito estilo Twitter/X — o conteúdo é organizado por categorias e eventos, não por timeline social
- **Nunca** usar ícones coloridos/emoji nos filtros — manter monocromático com acentos gold

---

## 2. DESIGN TOKENS & SISTEMA DE CORES

### 2.1 Paleta Oficial (oklch — já definida no CSS)

```
LIGHT MODE:
┌─────────────────┬──────────────────────────────────────┬──────────────┐
│ Token           │ oklch                                │ Uso          │
├─────────────────┼──────────────────────────────────────┼──────────────┤
│ --background    │ oklch(0.976 0.003 264.5)             │ Fundo geral  │
│ --foreground    │ oklch(0.204 0.033 260.3)             │ Texto corpo  │
│ --primary       │ oklch(0.715 0.120 84.2)  ≈ #C69C3F  │ CTA / Gold   │
│ --secondary     │ oklch(0.218 0.055 256.1) ≈ #071A33  │ Navy base    │
│ --accent        │ oklch(0.871 0.129 82.0)  ≈ #FFCC6A  │ Gold claro   │
│ --surface       │ oklch(0.218 0.055 256.1)             │ Sidebar/Top  │
│ --surface-var   │ oklch(0.274 0.077 256.3) ≈ #0A274C  │ Navy médio   │
│ --muted         │ oklch(0.928 0.006 264.5)             │ Backgrounds  │
│ --muted-fg      │ oklch(0.551 0.023 264.4)             │ Texto sec.   │
│ --card          │ oklch(1.000 0.000 000)               │ Cards        │
│ --border        │ oklch(0.872 0.009 258.3)             │ Divisórias   │
│ --destructive   │ oklch(0.568 0.200 26.4)              │ Erros/Delete │
│ --success       │ oklch(0.627 0.170 149.2)             │ Confirmação  │
│ --warning       │ oklch(0.769 0.165 70.1)              │ Alertas      │
│ --ring          │ oklch(0.715 0.120 84.2)              │ Focus ring   │
│ --radius        │ 0.5rem                               │ Border radius│
└─────────────────┴──────────────────────────────────────┴──────────────┘

DARK MODE (classe .dark):
┌─────────────────┬──────────────────────────────────────┬──────────────┐
│ --background    │ oklch(0.183 0.031 263.4)             │ Fundo escuro │
│ --foreground    │ oklch(0.928 0.006 264.5)             │ Texto claro  │
│ --card          │ oklch(0.218 0.055 256.1)             │ Cards navy   │
│ --secondary     │ oklch(0.274 0.077 256.3)             │ Navy variant │
│ --muted         │ oklch(0.274 0.077 256.3)             │ Bg muted     │
│ --muted-fg      │ oklch(0.714 0.019 261.3)             │ Texto muted  │
│ --border        │ oklch(0.274 0.077 256.3)             │ Bordas       │
└─────────────────┴──────────────────────────────────────┴──────────────┘
```

### 2.2 Regras de Aplicação das Cores

**Superfícies Navy (--surface):**
- Sidebar esquerda (navegação principal)
- Top bar / Header
- Footer
- Overlay de modais
- Background de popovers/dropdowns em contextos premium

**Ouro como Acento (--primary / --accent):**
- Botão CTA primário (fundo gold, texto branco)
- Badge de status do síndico (Bronze/Prata/Ouro/Diamante)
- Ícone de play nos vídeos
- Link ativo na sidebar
- Barra de progresso
- Ring de focus em inputs
- Underline de tab ativa

**Branco/Light (--background / --card):**
- Área de conteúdo principal (light mode)
- Cards de vídeo, cards de perfil, cards de evento
- Inputs e formulários

**Feedback Colors:**
- Destructive (vermelho): apenas para ações destrutivas (deletar, bloquear, denunciar)
- Success (verde): confirmação de voto, upload concluído, cadastro confirmado
- Warning (amarelo-âmbar): trava de 90 dias, limite de cota, aviso de expiração

### 2.3 Tipografia

```
HEADINGS (h1-h6):
  font-family: 'Michroma', sans-serif
  letter-spacing: 0.05em
  text-transform: Nenhum (manter case original)

  h1: 2rem (32px) — Títulos de página
  h2: 1.5rem (24px) — Seções principais
  h3: 1.25rem (20px) — Subseções
  h4: 1.125rem (18px) — Cards títulos
  h5: 1rem (16px) — Labels
  h6: 0.875rem (14px) — Captions

BODY:
  font-family: 'Manrope', sans-serif
  font-size: 1rem (16px) — Texto regular
  font-size: 0.875rem (14px) — Texto secundário, metadata
  font-size: 0.75rem (12px) — Timestamps, counters, micro-text
  line-height: 1.6 — Parágrafos
  line-height: 1.4 — Interfaces densas (tabelas, listas)

PESOS:
  400 — Texto regular
  500 — Labels, navigation items
  600 — Subtítulos, valores destacados
  700 — Títulos, CTAs
  800 — Hero headlines (uso raro)
```

### 2.4 Espaçamento

```
SPACING SCALE (UnoCSS):
  4px  (p-1)  — Padding mínimo entre ícone e texto
  8px  (p-2)  — Gap interno de chips/tags
  12px (p-3)  — Padding de inputs compact
  16px (p-4)  — Padding padrão de cards, gap de grid
  20px (p-5)  — Margem entre seções menores
  24px (p-6)  — Padding de modais, gap de layout
  32px (p-8)  — Margem entre seções principais
  48px (p-12) — Margem de página
  64px (p-16) — Espaço hero

GRID:
  Desktop (>1024px): max-w-7xl (1280px), px-6
  Tablet (768-1023px): px-4
  Mobile (<768px): px-4

VIDEO CARD GRID:
  Desktop: grid-cols-4, gap-4
  Tablet: grid-cols-3, gap-3
  Mobile: grid-cols-1 (full width) ou grid-cols-2 (compact)
```

### 2.5 Sombras & Elevação

```
ELEVATION LEVELS:
  Level 0: Nenhuma sombra (elementos inline)
  Level 1: shadow-sm — Cards em repouso
  Level 2: shadow-md — Cards em hover, dropdowns
  Level 3: shadow-lg — Modais, popovers
  Level 4: shadow-xl — Overlays, menus flutuantes

DARK MODE: Sombras são substituídas por bordas sutis (border-1 border-surface-variant)

TRANSIÇÕES:
  Padrão: transition-all duration-300 ease-out
  Hover de card: scale(1.02) + shadow-md → 200ms
  Route change: fadeInUp 400ms ease-out
  Skeleton loading: pulse 1.5s ease-in-out infinite
```

### 2.6 Border Radius

```
--radius: 0.5rem (8px)

APLICAÇÃO:
  Botões: rounded-md (0.5rem)
  Cards: rounded-lg (0.75rem)
  Inputs: rounded-md (0.5rem)
  Avatares: rounded-full
  Thumbnails de vídeo: rounded-lg
  Modais: rounded-xl (1rem)
  Tags/Chips: rounded-full
  Popovers: rounded-lg
```

---

## 3. LAYOUT PRINCIPAL — SHELL DA APLICAÇÃO

### 3.1 Estrutura Macro (Desktop ≥1024px)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        TOP BAR (64px height)                        │
│  [☰] [Logo MS]         [🔍 Busca...]          [🔔] [Avatar ▼]     │
├──────────────┬──────────────────────────────────────────────────────┤
│              │                                                      │
│   SIDEBAR    │              MAIN CONTENT AREA                       │
│   (240px)    │              (flex-1, scroll-y)                      │
│   Navy bg    │                                                      │
│              │  ┌─────────────────────────────────────────────┐    │
│  [🏠 Home]   │  │  Breadcrumb / Page Title                     │    │
│  [🔍 Busca]  │  │                                               │    │
│  [👤 Perfil] │  │  [ Content Area ]                             │    │
│  [📋 Assemb] │  │                                               │    │
│  [⭐ Avali]  │  │                                               │    │
│  [📊 Gestão] │  │                                               │    │
│  [⚙ Config]  │  └─────────────────────────────────────────────┘    │
│              │                                                      │
│  ─────────── │                                                      │
│  Plano: Pro  │                                                      │
│  Status: 🥇  │                                                      │
│              │                                                      │
├──────────────┴──────────────────────────────────────────────────────┤
│                        FOOTER (opcional)                             │
└─────────────────────────────────────────────────────────────────────┘
```

### 3.2 Estrutura Macro (Mobile <768px)

```
┌─────────────────────────────┐
│  TOP BAR (56px)             │
│  [☰] [Logo] [🔍] [🔔] [👤]│
├─────────────────────────────┤
│                             │
│     MAIN CONTENT            │
│     (full width, scroll-y)  │
│                             │
│                             │
│                             │
│                             │
├─────────────────────────────┤
│  BOTTOM NAV (64px + safe)   │
│  [🏠] [🔍] [+] [📋] [👤]  │
└─────────────────────────────┘
```

### 3.3 Top Bar — Detalhamento

**Desktop:**
```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  [☰ Toggle]  [🛡️ MASTER SÍNDICO]    [🔍 Buscar síndicos,      │
│               (logo + text)           empresas, serviços...]     │
│                                                                  │
│                              [🌙/☀️]  [🔔 •]  [Avatar ▼ Nome]  │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

- Fundo: `var(--surface)` (navy escuro)
- Logo: Brasão Master Síndico em ouro/branco (versão horizontal compact)
- Texto "MASTER SÍNDICO": fonte Michroma, cor `var(--primary)` (gold), tracking largo
- Search bar: fundo `var(--surface-variant)`, borda `var(--border)`, placeholder em muted-foreground, ícone lupa em muted-foreground. Ao focar: ring gold `var(--ring)`
- Toggle theme: ícone lua/sol, transição suave
- Notificação: bell icon com badge vermelho (ponto) se houver pendências de assembleia
- Avatar: circular 32px, borda gold sutil, dropdown ao clicar com: nome, role (Síndico N3), link para perfil, settings, logout
- Height: 64px desktop, 56px mobile
- z-index: 50
- border-bottom: 1px solid `var(--border)` (light mode) ou none (dark mode com sombra sutil)

**Mobile:**
- Logo: apenas o brasão (sem texto "MASTER SÍNDICO")
- Search: ícone que expande para fullscreen search
- Hamburguer: abre drawer de navegação (overlay animado da esquerda)

### 3.4 Sidebar — Detalhamento

**Estilo:** Inspirado no Avalanche (Imagens 3-5), mas adaptado para a paleta Master Síndico

```
SIDEBAR CONTENT (240px, bg: var(--surface)):

  ┌──────────────────────┐
  │  NAVEGAÇÃO PRINCIPAL │
  │                      │
  │  🏠 Início           │ ← gold bg pill quando ativo
  │  🔍 Buscar           │
  │  📹 Vídeos           │
  │  👤 Meu Perfil       │
  │                      │
  │  ── MINHA GESTÃO ──  │ ← label section, Michroma 11px, gold
  │  📋 Assembleias      │   (visível apenas p/ Síndicos N2/N3)
  │  📊 Transparência    │
  │  ⭐ Avaliações       │
  │  📄 Exportar PDF     │
  │                      │
  │  ── CONEXÕES ──      │
  │  📨 Connect Me       │
  │  🏢 Empresas         │
  │                      │
  │  ── CONFIGURAÇÕES ── │
  │  ⚙️ Configurações    │
  │  ❓ Ajuda            │
  │                      │
  │  ─────────────────── │
  │                      │
  │  CARD DO PLANO:      │
  │  ┌────────────────┐  │
  │  │ Síndico Pro    │  │
  │  │ Status: 🥇 Ouro│  │
  │  │ [Gerenciar →]  │  │
  │  └────────────────┘  │
  │                      │
  └──────────────────────┘
```

**Comportamento:**
- Desktop: sidebar fixa, colapsável para 64px (só ícones) via toggle no top bar
- Tablet: sidebar inicia colapsada, expande em hover/click
- Mobile: sidebar não existe. Substituída por bottom navigation + drawer
- Itens de navegação: ícone (20px) + label (Manrope 500, 14px), padding vertical 10px, padding horizontal 12px
- Item ativo: fundo `var(--primary)` com opacity 15%, texto `var(--primary)`, ícone gold, border-left 3px solid gold
- Item hover: fundo `var(--surface-variant)` com opacity 50%
- Texto: `var(--surface-foreground)` (cinza claro)
- Separadores de seção: label em Michroma 10px uppercase, tracking 0.1em, cor `var(--muted-foreground)`, margem top 24px
- Card do plano: fundo `var(--surface-variant)`, border 1px solid rgba(gold, 0.3), border-radius 8px, padding 12px

**Regra de Visibilidade na Sidebar (ABAC):**
- Items de "MINHA GESTÃO" só aparecem se user.role inclui permissão de gestão condominial
- "Connect Me" aparece apenas para roles que possuem essa funcionalidade
- Itens não liberados pelo plano: NÃO mostrar com cadeado. Simplesmente NÃO renderizar.

### 3.5 Bottom Navigation (Mobile)

Inspirado na Imagem 1 (SparkCause) — 5 itens com botão central proeminente:

```
┌───────────────────────────────────────────────┐
│                                               │
│  [🏠]    [🔍]    [⊕]    [📋]    [👤]        │
│  Início  Buscar  Ação   Gestão  Perfil       │
│                                               │
└───────────────────────────────────────────────┘
```

- Fundo: `var(--surface)` (navy)
- Ícone ativo: `var(--primary)` (gold) + label visível
- Ícone inativo: `var(--muted-foreground)` (cinza) + label oculta
- Botão central [⊕]: 48px circle, fundo `var(--primary)` (gold), ícone branco, elevated -12px acima da barra. Abre action sheet (upload vídeo, criar assembleia, etc — conforme role)
- Height: 64px + safe-area-inset-bottom
- Border-top: 1px solid `var(--border)`
- z-index: 50

**Adaptação de itens por role:**
- Morador: [Início] [Buscar] [+Ação] [Benefícios] [Perfil]
- Síndico: [Início] [Buscar] [+Criar] [Gestão] [Perfil]
- Empresa: [Início] [Buscar] [+Upload] [Conteúdo] [Perfil]

---

## 4. COMPONENTES — DESIGN DETALHADO

### 4.1 Video Card (Padrão YouTube — Referência Imagens 4 e 5)

Este é o componente mais reutilizado da plataforma. Aparece na Home, na Busca, no Perfil, nas Assembleias (vídeos de fornecedores).

**Layout Vertical (Grid — Desktop/Tablet):**
```
┌──────────────────────────────┐
│                              │
│      THUMBNAIL (16:9)        │ ← rounded-lg, object-cover
│                              │
│  ▶ (play overlay on hover)   │ ← ícone play centralizado, bg rgba(0,0,0,0.4)
│                    [03:45]   │ ← badge duração, bottom-right, bg black/80, text white, text-xs
│                              │
├──────────────────────────────┤
│ [Avatar 36px] Título do Vídeo│
│               que pode ter   │ ← max 2 linhas, line-clamp-2, Manrope 600, 14px
│               duas linhas    │
│                              │
│              Nome do Canal   │ ← Manrope 400, 13px, color muted-foreground
│              125 mil views • │ ← Manrope 400, 12px, color muted-foreground
│              há 2 dias       │
│                       [⋮]   │ ← menu 3 dots
└──────────────────────────────┘
```

**Layout Horizontal (Lista — Mobile / Sidebar de "Relacionados"):**
```
┌──────────┬───────────────────────┐
│          │ Título do vídeo que   │
│ THUMB    │ pode ter até duas...  │
│ (168x94) │                       │
│  [02:30] │ Canal Name            │
│          │ 50K views • 3h atrás  │
└──────────┴───────────────────────┘
```

**Regras do Card:**
- Thumbnail: aspect-ratio 16/9, rounded-lg, bg `var(--muted)` como placeholder
- Hover: scale(1.03) na thumbnail, overlay play ícone fade-in, shadow-md no card
- Badge de duração: absolute bottom-2 right-2, bg rgba(0,0,0,0.8), text-white, text-xs, px-1.5 py-0.5, rounded-sm
- Avatar do canal: 36px circular, border 2px solid transparent. Se for empresa com plano Pro: border gold sutil
- Título: Manrope 600, 14px, cor `var(--foreground)`, line-clamp-2
- Metadata: Manrope 400, 12-13px, cor `var(--muted-foreground)`, separador " • " (bullet)
- Menu 3 dots: visível apenas em hover (desktop) ou sempre visível (mobile)
- **IMPORTANTE**: Nunca mostrar contador de likes/comments no card. Apenas views (se o usuário for o dono) + timestamp.

**Variações do Card:**
1. **Card de Vídeo Institucional (Empresa):** badge "INSTITUCIONAL" no canto superior esquerdo da thumbnail, fundo gold, texto navy
2. **Card de Vídeo-Currículo (Morador):** badge "CURRÍCULO" no canto superior esquerdo, fundo `var(--accent)`, texto navy
3. **Card de Vídeo de Pauta (Assembleia):** badge "PAUTA #3" com número da pauta, fundo `var(--secondary)`, texto branco
4. **Card com Paywall (preview 25%):** overlay diagonal no thumbnail com texto "Preview 25%", ícone de cadeado, CTA "Assine para ver completo"
5. **Card de Vídeo Travado (90 dias):** grayscale no thumbnail, badge "Atualização em 45 dias"

### 4.2 Video Player Page (Referências: YouTube + Imagens 3 e 5)

**Desktop Layout:**
```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                       │
│          │  PLAYER DE VÍDEO (aspect 16:9, max-height 70vh)      │
│          │  ┌─────────────────────────────────────────────────┐  │
│          │  │                                                 │  │
│          │  │              VIDEO PLAYER                       │  │
│          │  │         (Mux/Cloudflare embarcado)              │  │
│          │  │                                                 │  │
│          │  │  [▶/⏸] [⏮] [Vol] [━━━━━━●━━━] [CC] [⚙] [⛶]  │  │
│          │  └─────────────────────────────────────────────────┘  │
│          │                                                       │
│          │  METADATA DO VÍDEO                                    │
│          │  ┌─────────────────────────────────────────────────┐  │
│          │  │ Título do Vídeo Completo Aqui                   │  │
│          │  │ h2, Michroma, 20px                               │  │
│          │  │                                                 │  │
│          │  │ [Avatar] Canal/Empresa Name  [Seguir] [ConnectMe]│ │
│          │  │          @handle • Status                        │  │
│          │  │                                                 │  │
│          │  │ [👍 Curtir] [↗ Compartilhar] [⚑ Denunciar]     │  │
│          │  │                                                 │  │
│          │  │ Descrição do vídeo...                            │  │
│          │  │ Categorias: #hidráulica #manutenção              │  │
│          │  └─────────────────────────────────────────────────┘  │
│          │                                                       │
│          │  ┌──── CONTEÚDO ABAIXO (2 colunas no desktop) ────┐  │
│          │  │                        │                        │  │
│          │  │  COMENTÁRIOS           │  VÍDEOS RELACIONADOS   │  │
│          │  │  (visível APENAS se    │  (grid vertical)       │  │
│          │  │   user == owner)       │                        │  │
│          │  │                        │  [Video Card H]        │  │
│          │  │  OU                    │  [Video Card H]        │  │
│          │  │                        │  [Video Card H]        │  │
│          │  │  SEÇÃO VAZIA           │  [Video Card H]        │  │
│          │  │  (se user != owner)    │                        │  │
│          │  │                        │                        │  │
│          │  └────────────────────────┴────────────────────────┘  │
└──────────────────────────────────────────────────────────────────┘
```

**Regras do Player:**
- Controles custom: barra de progresso gold (`var(--primary)`), botões brancos
- Qualidade selector: popup com opções (360p, 720p, 1080p)
- Fullscreen: ícone de expandir
- **Paywall (25%):** quando o user é Base e o vídeo é de Empresa, o player corta aos 25% e exibe overlay:
  ```
  ┌─────────────────────────────────────────┐
  │                                         │
  │     🔒 Conteúdo exclusivo               │
  │                                         │
  │     Para assistir ao vídeo completo,    │
  │     atualize seu plano.                 │
  │                                         │
  │     [Conhecer Planos →]                 │
  │                                         │
  │     Fundo: navy com blur do frame       │
  └─────────────────────────────────────────┘
  ```

**Botão Curtir (Feedback Privado):**
- Ícone de like (outline quando não curtiu, filled gold quando curtiu)
- **SEM contador visível para não-donos.** O botão funciona, registra no backend, mas não mostra número.
- Para o dono: exibe "X curtidas" e permite expandir lista de comentários abaixo

**Mobile:**
- Player: full width, aspect-ratio 16:9
- Metadata abaixo do player, coluna única
- Vídeos relacionados: scroll horizontal de cards compactos OU lista vertical

### 4.3 Search Page (Referências: YouTube categories + Imagens 4 e 7)

**Desktop Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  BARRA DE BUSCA (ampliada, full-width)         │
│          │  ┌──────────────────────────────────────────┐  │
│          │  │  🔍 Buscar síndicos, empresas, serviços  │  │
│          │  └──────────────────────────────────────────┘  │
│          │                                                │
│          │  FILTROS (horizontal scrollable pills):         │
│          │  [Todos] [Síndicos] [Empresas] [Comércio]     │
│          │  [+ Categorias ▼]                              │
│          │                                                │
│          │  FILTRO LATERAL (colapsável):                   │
│          │  ┌──────────┐  ┌──────────────────────────┐   │
│          │  │Categorias│  │                           │   │
│          │  │☐ Hidrául.│  │   RESULTADOS (grid)       │   │
│          │  │☐ Elétrica│  │                           │   │
│          │  │☐ Seguros │  │   [Card] [Card] [Card]    │   │
│          │  │...       │  │   [Card] [Card] [Card]    │   │
│          │  │          │  │                           │   │
│          │  │CEP       │  │                           │   │
│          │  │[________]│  │                           │   │
│          │  │          │  │                           │   │
│          │  │Tipo      │  │                           │   │
│          │  │○ Vídeos  │  │                           │   │
│          │  │○ Perfis  │  │                           │   │
│          │  │○ Ambos   │  │                           │   │
│          │  └──────────┘  └──────────────────────────┘   │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Filter Pills (inspirado Imagem 7 — Genaigurus):**
- Horizontal scroll com snap
- Pill ativa: fundo `var(--primary)` (gold), texto white, font-weight 600
- Pill inativa: fundo `var(--muted)`, texto `var(--foreground)`, border 1px solid `var(--border)`
- Pill hover: fundo `var(--accent)` com opacity 20%
- Transição: 200ms ease

**Category Icons Row (inspirado Imagem 4 — Avalanche):**
- Grid horizontal com scroll
- Cada item: ícone 40px em circle + label abaixo
- Categorias do Cap. 6 do Manual (Administração, Jurídico, Contabilidade, Elétrica, Hidráulica, Segurança, etc.)
- Ícone ativo: fundo gold, ícone white
- Ícone inativo: fundo `var(--muted)`, ícone `var(--muted-foreground)`
- Max visível: 8 com setas de navegação (desktop) ou scroll (mobile)

**Cards de Resultado:**
1. **Card de Empresa:** Avatar/logo + nome + categorias (tags) + "Empresa Pro" badge + snippet de descrição
2. **Card de Síndico:** Avatar + nome + status (Bronze/Prata/Ouro/Diamante) + número de condomínios + botão Connect Me (se permitido)
3. **Card de Comércio Local:** Logo + nome + endereço + CEP + "Promoção ativa" badge se houver

**Filtro por CEP (Comércio Local):**
- Input com máscara XXXXX-XXX
- Ao digitar, filtra resultados em real-time
- Exclusivo para busca de Comércio Local / Vizinhança

### 4.4 Profile Page (Referência: Instagram + Imagens 1 e 6)

**Layout do Perfil — Desktop:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  COVER / BANNER (300px height, full-width)     │
│          │  ┌────────────────────────────────────────────┐│
│          │  │        Cover image / gradient navy         ││
│          │  │                                            ││
│          │  │  [Avatar 120px]                            ││
│          │  │  (borda 4px solid gold se Pro)             ││
│          │  └────────────────────────────────────────────┘│
│          │                                                │
│          │  INFORMAÇÕES DO PERFIL                         │
│          │  ┌────────────────────────────────────────────┐│
│          │  │ Nome Completo              [Editar Perfil] ││
│          │  │ @handle • Síndico N3 • ⭐ Status Ouro      ││
│          │  │                                            ││
│          │  │ Bio: Síndico profissional há 8 anos...     ││
│          │  │                                            ││
│          │  │ 📍 São Paulo, SP                           ││
│          │  │ 🏢 3 condomínios • 📹 47 vídeos assistidos ││
│          │  │ 📚 5 módulos concluídos                     ││
│          │  │                                            ││
│          │  │ [Connect Me]  [Compartilhar]               ││
│          │  │  (visível conforme Matriz Funcional)       ││
│          │  └────────────────────────────────────────────┘│
│          │                                                │
│          │  TABS DE CONTEÚDO                              │
│          │  ┌────────────────────────────────────────────┐│
│          │  │ [Vídeos] [Cursos] [Sobre] [Gestão]        ││
│          │  │  ═══════                                   ││
│          │  │                                            ││
│          │  │  Grid de Video Cards (4 cols desktop)      ││
│          │  │  [Card] [Card] [Card] [Card]               ││
│          │  │  [Card] [Card] [Card] [Card]               ││
│          │  └────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────┘
```

**Variações de Perfil por Persona:**

**Perfil do Síndico:**
- Status badge (Bronze/Prata/Ouro/Diamante) — badge metalizado ao lado do nome
- Indicadores públicos: nº condomínios, vídeos assistidos (>70%), módulos concluídos
- Tabs: Vídeos (se N2/N3), Gestão (timeline), Sobre
- Botão Connect Me: visível apenas para quem tem permissão na Matriz Funcional
- Se N1: perfil simplificado, sem vídeos publicados, sem perfil público institucional

**Perfil do Morador Pagante:**
- Vídeo-Currículo fixo (se existir): player embed proeminente no topo do perfil
- Badge "Disponível" se currículo ativo
- Tabs: Currículo (vídeo + ficha), Sobre
- Se morador Base: perfil mínimo, sem currículo

**Perfil da Empresa:**
- Logo da empresa como avatar (quadrado com rounded-lg)
- Categorias técnicas (até 5 tags)
- Badge de plano: "Plus" ou "Pro"
- Vídeo institucional fixo (se existir e se Pro)
- Tabs: Vídeos, Cursos (se Pro), Sobre
- **Sem dados de contato visíveis** — sem telefone, email, site, WhatsApp

**Perfil do Comércio Local (Vizinhança):**
- Logo + nome + endereço + CEP
- Promoção do dia ativa (se houver)
- Tabs: Promoções, Sobre

**Status Badges (Síndico) — Design:**
```
  🥉 Bronze  — bg: #CD7F32 com opacity 15%, texto: #CD7F32, border: 1px solid #CD7F32
  🥈 Prata   — bg: #C0C0C0 com opacity 15%, texto: #808080, border: 1px solid #C0C0C0
  🥇 Ouro    — bg: var(--primary) com opacity 15%, texto: var(--primary), border: 1px solid var(--primary)
  💎 Diamante — bg: linear-gradient(135deg, #B9F2FF, #E0C3FC), texto: #1a1a2e, border: 1px solid #B9F2FF
```

### 4.5 Authentication Pages (Onboarding Stripe-like — Sprint 1)

**Fluxo Visual:**

```
PASSO 1: URL Parametrizada → Landing de Cadastro
PASSO 2: Formulário → Validação Email
PASSO 3: Confirmação Email → Definição de Senha
PASSO 4: Conta Ativa → Redirect para Home
```

**Layout de Auth (Desktop):**
```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  ┌────────────────────┐  ┌────────────────────────────┐ │
│  │                    │  │                            │ │
│  │  PAINEL VISUAL     │  │  FORMULÁRIO DE AUTH        │ │
│  │  (50% width)       │  │  (50% width)              │ │
│  │                    │  │                            │ │
│  │  Fundo: Navy       │  │  [🛡️ Logo Master Síndico] │ │
│  │  gradient com      │  │                            │ │
│  │  pattern sutil     │  │  Criar sua conta           │ │
│  │  de prédios/city   │  │  Síndico Profissional Pro  │ │
│  │                    │  │                            │ │
│  │  Frase de impacto: │  │  [Nome completo]           │ │
│  │  "Governança       │  │  [Email]                   │ │
│  │   começa com       │  │  [CPF/CNPJ]               │ │
│  │   método."         │  │                            │ │
│  │                    │  │  [Criar Conta →]           │ │
│  │  — Master Síndico  │  │  botão gold, full-width    │ │
│  │                    │  │                            │ │
│  │                    │  │  Já tem conta? [Entrar]    │ │
│  │                    │  │                            │ │
│  └────────────────────┘  └────────────────────────────┘ │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Mobile Auth:**
- Painel visual vira header compacto (logo + frase, 120px height)
- Formulário ocupa o resto da tela
- Full-width, sem split

**Componentes de Input:**
- Label: Manrope 500, 14px, cor `var(--foreground)`, mb-1.5
- Input: h-11 (44px), px-3, bg `var(--card)`, border 1px `var(--border)`, rounded-md
- Focus: ring-2 ring-`var(--ring)` (gold), border-`var(--ring)`
- Error: border-`var(--destructive)`, mensagem abaixo em 12px destructive
- Placeholder: cor `var(--muted-foreground)`

**OTP Field (verificação de email — usando @corvu/otp-field):**
```
┌─────────────────────────────────────┐
│                                     │
│  Confirme seu email                 │
│  Enviamos um código para            │
│  joao@email.com                     │
│                                     │
│  ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐ ┌──┐   │
│  │ 4│ │ 2│ │ 8│ │  │ │  │ │  │   │
│  └──┘ └──┘ └──┘ └──┘ └──┘ └──┘   │
│                                     │
│  Não recebeu? [Reenviar código]     │
│                                     │
│  [Confirmar →]                      │
│                                     │
└─────────────────────────────────────┘
```
- Cada campo: 48x56px, font-size 24px bold, text-center
- Fundo: `var(--card)`, border 2px `var(--border)`
- Preenchido: border-`var(--primary)` (gold)
- Auto-focus no próximo campo ao digitar

**Tela de Definição de Senha:**
```
┌─────────────────────────────────────┐
│                                     │
│  Defina sua senha                   │
│                                     │
│  [Nova senha        ] 👁            │
│  [Confirmar senha   ] 👁            │
│                                     │
│  Requisitos:                        │
│  ✅ Mínimo 8 caracteres            │
│  ✅ Uma letra maiúscula            │
│  ❌ Um número                       │
│  ❌ Um caractere especial           │
│                                     │
│  [Ativar Conta →]                   │
│                                     │
└─────────────────────────────────────┘
```
- Requisitos: checklist com ícones ✅ (success green) e ❌ (muted), atualiza em real-time

### 4.6 Home Page — Feed (Referência: YouTube Home + Imagens 4 e 5)

**Desktop Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  CATEGORY ICONS ROW (scroll horizontal)         │
│          │  [👤Para Você] [⭐Popular] [📰Notícias]       │
│          │  [🔧Manutenção] [⚡Elétrica] [💧Hidráulica]   │
│          │  [🔒Segurança] [📋Jurídico] [➡️]              │
│          │                                                │
│          │  ─── Recomendados para você ───                │
│          │                                                │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │                                                │
│          │  ─── Vídeos de Empresas ───                    │
│          │                                                │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │                                                │
│          │  ─── Assembleias Recentes ───                  │
│          │  (visível se user tem condomínio vinculado)     │
│          │                                                │
│          │  [Assembly Card] [Assembly Card]               │
│          │                                                │
│          │  ─── Comércio Local ───                        │
│          │  (se user tem CEP cadastrado)                   │
│          │                                                │
│          │  [Promo Card] [Promo Card] [Promo Card]       │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Category Icons Row (inspirado Avalanche — Imagem 4):**
- Cada item: ícone dentro de quadrado 56x56px rounded-lg + label 12px abaixo
- Ícones: line-style, 24px, cor `var(--muted-foreground)`
- Ativo: fundo gold, ícone white
- Container: overflow-x auto, snap-x, hide scrollbar, gap-3
- Setas: visíveis nos extremos em desktop (chevron-left/right)

**Seções da Home:**
- Cada seção: título h3 (Michroma 18px, gold) + link "Ver todos →" (Manrope 14px, gold)
- Grid de cards: 4 colunas desktop, 3 tablet, 2 mobile (ou 1 se card horizontal)
- Entre seções: separator visual (border-top 1px `var(--border)`, my-8)

### 4.7 Settings Page (Referência: Meta/Facebook Settings — Sprint 1)

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  SETTINGS LEFT NAV    │  SETTINGS CONTENT      │
│          │  (200px)              │                        │
│          │                       │                        │
│          │  [Conta]              │  CONTA                 │
│          │  [Segurança]          │                        │
│          │  [Notificações]       │  Nome: João Silva      │
│          │  [Privacidade]        │  [Editar]              │
│          │  [Aparência]          │                        │
│          │  [Plano]              │  Email: joao@...       │
│          │                       │  [Alterar]             │
│          │                       │                        │
│          │                       │  Telefone: (11) 9...   │
│          │                       │  [Alterar]             │
│          │                       │                        │
│          │                       │  ─── Zona de Perigo ── │
│          │                       │  [Desativar conta]     │
│          │                       │  (texto destructive)   │
│          │                       │                        │
└──────────────────────────────────────────────────────────┘
```

**Settings Items:**
- Cada item: row com label (Manrope 500, 14px), valor (Manrope 400, 14px, muted-foreground), botão "Editar" (link gold)
- Seções separadas por heading (Michroma 14px uppercase, gold)
- Toggle switches para notificações: gold quando ativo, `var(--muted)` quando inativo
- Seletor de tema: 3 opções (Claro/Escuro/Sistema) com radio visual

---

## 5. TELAS DA SPRINT 2 — GESTÃO CONDOMINIAL

### 5.1 Assembly List Page (Lista de Assembleias)

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  ASSEMBLEIAS E EVENTOS         [+ Nova →]     │
│          │                                                │
│          │  FILTROS: [Todas] [Agendadas] [Abertas]       │
│          │           [Encerradas]                          │
│          │                                                │
│          │  ┌────────────────────────────────────────┐   │
│          │  │  📋 Assembleia Ordinária 2026/1        │   │
│          │  │  Status: 🟢 ABERTA                     │   │
│          │  │  Data: 15/03/2026 • 5 pautas           │   │
│          │  │  Votação até: 20/03/2026               │   │
│          │  │                                [→]     │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
│          │  ┌────────────────────────────────────────┐   │
│          │  │  📋 Manutenção Elevadores              │   │
│          │  │  Status: ⏳ AGENDADA                    │   │
│          │  │  Data: 01/04/2026 • 3 pautas           │   │
│          │  │                                [→]     │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
│          │  ┌────────────────────────────────────────┐   │
│          │  │  📋 Escolha de Prestador - Fachada     │   │
│          │  │  Status: 🔴 ENCERRADA                  │   │
│          │  │  Data: 10/01/2026 • Resultado: ✅      │   │
│          │  │                                [→]     │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Assembly Card:**
- Fundo: `var(--card)`, border 1px `var(--border)`, rounded-lg, p-4
- Hover: border-`var(--primary)` sutil, shadow-sm
- Status badges: 
  - ABERTA: bg success/10, text success, border success
  - AGENDADA: bg warning/10, text warning, border warning  
  - ENCERRADA: bg muted, text muted-foreground

### 5.2 Assembly Detail / Creation Wizard (Síndico N2/N3)

**Wizard Steps (top):**
```
  [1. Dados Gerais] ─── [2. Pautas] ─── [3. Votação] ─── [4. Revisão]
       ●═══════════════○═══════════════○═══════════════○
       ativo           próximo         próximo         próximo
```

- Step ativo: circle gold filled, linha gold
- Step futuro: circle outline `var(--border)`, linha `var(--border)`
- Step concluído: circle gold com ✓ branco

**Step 2 — Pautas (Área mais complexa):**
```
┌────────────────────────────────────────────────────────┐
│  Pautas da Assembleia                    [+ Nova Pauta]│
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  PAUTA 1: Contratação de empresa para fachada    │  │
│  │                                                  │  │
│  │  Descrição: Lorem ipsum dolor sit amet...        │  │
│  │                                                  │  │
│  │  📹 Vídeo Explicativo do Síndico:               │  │
│  │  ┌────────────────────┐                          │  │
│  │  │ [Upload vídeo]     │  ou  [Gravar da câmera]  │  │
│  │  │ drag & drop zone   │                          │  │
│  │  └────────────────────┘                          │  │
│  │                                                  │  │
│  │  🎵 Áudio Explicativo:                          │  │
│  │  [Upload áudio] ou [Gravar]                      │  │
│  │                                                  │  │
│  │  📎 Vídeos de Fornecedores (anexos):             │  │
│  │  [🔍 Buscar no acervo de empresas]               │  │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │  │
│  │  │Empresa A │ │Empresa B │ │Empresa C │        │  │
│  │  │ [vídeo]  │ │ [vídeo]  │ │ [vídeo]  │        │  │
│  │  └──────────┘ └──────────┘ └──────────┘        │  │
│  │                                                  │  │
│  │  [⬆️ Mover] [⬇️ Mover] [🗑 Remover]            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  [+ Adicionar Pauta]                                   │
│                                                        │
│  [← Voltar]                      [Próximo Passo →]    │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Upload Zone:**
- Drag & drop: border dashed 2px `var(--border)`, rounded-lg, p-8, text-center
- Hover com arquivo: border-`var(--primary)`, bg `var(--primary)/5`
- Ícone cloud-upload centralizado, 48px, cor `var(--muted-foreground)`
- Texto: "Arraste ou clique para enviar" (Manrope 14px)

### 5.3 Voting Interface (Morador — dentro da Assembleia)

**Layout da Votação:**
```
┌────────────────────────────────────────────────────────┐
│  ASSEMBLEIA: Ordinária 2026/1                          │
│  ⏰ Votação aberta até 20/03/2026 às 23:59            │
│                                                        │
│  PAUTA 3: Escolha do Fornecedor para Fachada          │
│                                                        │
│  📹 [Player: Vídeo explicativo do Síndico]            │
│                                                        │
│  Opções de voto:                                       │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  ○  Empresa Alpha Fachadas                       │  │
│  │     📹 [Ver vídeo da empresa]                    │  │
│  │     Proposta: R$ 45.000 • Prazo: 60 dias         │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  ○  Beta Recuperação Predial                     │  │
│  │     📹 [Ver vídeo da empresa]                    │  │
│  │     Proposta: R$ 52.000 • Prazo: 45 dias         │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  ┌──────────────────────────────────────────────────┐  │
│  │  ○  Gamma Engenharia                             │  │
│  │     📹 [Ver vídeo da empresa]                    │  │
│  │     Proposta: R$ 48.500 • Prazo: 50 dias         │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│                        [Confirmar Voto →]              │
│                                                        │
│  ⚠️ Seu voto é vinculado à sua unidade. Após          │
│  confirmar, não será possível alterar.                  │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Após votar:**
```
  ┌──────────────────────────────────────┐
  │  ✅ Seu voto foi registrado!         │
  │                                      │
  │  Pauta: Escolha do Fornecedor        │
  │  Sua escolha: Empresa Alpha          │
  │  Unidade: Apt 302 - Bloco A          │
  │                                      │
  │  [← Voltar às pautas]               │
  └──────────────────────────────────────┘
```

- Feedback: Toast de sucesso (solid-sonner) + tela de confirmação
- Radio option selecionada: border-left 4px gold, bg `var(--primary)/5`

### 5.4 Transparency Dashboard (Síndico N3)

**Layout:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  PAINEL DE TRANSPARÊNCIA                       │
│          │                                                │
│          │  ┌─────────────────────────────────────────┐  │
│          │  │  PLANO DE GESTÃO                        │  │
│          │  │                                         │  │
│          │  │  ┌─── O que prometi ────────────────┐   │  │
│          │  │  │ 1. Reforma da fachada     [80%]  │   │  │
│          │  │  │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │   │  │
│          │  │  │ 2. Troca dos elevadores   [40%]  │   │  │
│          │  │  │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │   │  │
│          │  │  │ 3. Paisagismo da entrada  [100%] │   │  │
│          │  │  │ ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │   │  │
│          │  │  └──────────────────────────────────┘   │  │
│          │  │                                         │  │
│          │  │  Execução geral: ████████░░ 73%         │  │
│          │  │                                         │  │
│          │  └─────────────────────────────────────────┘  │
│          │                                                │
│          │  TIMELINE DA GESTÃO                            │
│          │  ┌─────────────────────────────────────────┐  │
│          │  │  ● Mar/2026 — Início da obra fachada    │  │
│          │  │  │  Contratada: Alpha Fachadas           │  │
│          │  │  │  Valor: R$ 45.000                     │  │
│          │  │  │  [📹 Vídeo] [📄 Edital]              │  │
│          │  │  │                                       │  │
│          │  │  ● Fev/2026 — Assembleia Ordinária       │  │
│          │  │  │  5 pautas • 78% participação          │  │
│          │  │  │  [Ver detalhes]                       │  │
│          │  │  │                                       │  │
│          │  │  ● Jan/2026 — Manutenção preventiva      │  │
│          │  │     Elevadores - Inspeção periódica      │  │
│          │  └─────────────────────────────────────────┘  │
│          │                                                │
│          │  [📄 Exportar Gestão em PDF]                   │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Progress Bars:**
- Track: bg `var(--muted)`, h-2, rounded-full
- Fill: bg `var(--primary)` (gold), animated width
- 100%: bg `var(--success)` (verde)
- Label: porcentagem ao lado direito, Manrope 600, 14px

**Timeline:**
- Linha vertical: border-left 2px solid `var(--border)`, ml-4
- Dots: 12px circle, bg `var(--primary)` (gold), border 3px solid `var(--card)`
- Cada item: pl-8 relative, com connector na linha
- Hover: bg `var(--muted)` com rounded-lg

### 5.5 Rating / Evaluation (Morador avaliando Síndico)

```
┌────────────────────────────────────────────────────────┐
│  AVALIAR GESTÃO DO SÍNDICO                             │
│  João Silva • Condomínio Villa Park                    │
│                                                        │
│  Pergunta 1 de 5:                                      │
│  "Como você avalia a transparência das decisões        │
│   tomadas neste período?"                              │
│                                                        │
│     ☆  ☆  ☆  ☆  ☆                                    │
│     1  2  3  4  5                                      │
│                                                        │
│  ─────────────────────────────                         │
│                                                        │
│  Pergunta 2 de 5:                                      │
│  "A comunicação sobre as atividades do condomínio      │
│   foi clara e tempestiva?"                             │
│                                                        │
│     ★  ★  ★  ☆  ☆                                    │
│     1  2  3  4  5                                      │
│                                                        │
│  ─────────────────────────────                         │
│                                                        │
│  [← Anterior]              [Próxima Pergunta →]       │
│                                                        │
│  ████████░░░░░░░░░ 2/5 perguntas                      │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Stars:**
- Estrela vazia: outline, cor `var(--muted-foreground)`
- Estrela preenchida: filled, cor `var(--primary)` (gold), com animação scale(1.2) ao clicar
- Hover: estrela + todas anteriores ficam gold (preview)
- Tamanho: 32px cada, gap-2

**Nota Final (para o Síndico):**
- Card com nota consolidada: número grande (Michroma 48px, gold), "/5"
- Breakdown por pergunta: barra horizontal por item
- **Privacidade**: morador vê "Avaliação enviada ✅". Não vê avaliações de outros.

### 5.6 Connect Me — Formulário de Contato (Sprint 1)

```
┌────────────────────────────────────────────────────────┐
│  CONNECT ME                                            │
│  Entrar em contato com João Silva (Síndico)            │
│                                                        │
│  ⚠️ Você possui 3 de 4 contatos disponíveis este ano  │
│                                                        │
│  [Seu nome]                                            │
│  [Seu email de contato]                                │
│  [Telefone (opcional)]                                 │
│                                                        │
│  Assunto:                                              │
│  [▼ Selecione a categoria]                             │
│                                                        │
│  Mensagem:                                             │
│  ┌──────────────────────────────────────────────────┐  │
│  │                                                  │  │
│  │  Textarea (max 500 caracteres)                   │  │
│  │                                                  │  │
│  │                                          450/500 │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  ☑ Concordo que este formulário será enviado por       │
│    email ao destinatário. Não há chat ou reply          │
│    dentro da plataforma.                               │
│                                                        │
│  [Enviar Formulário →]                                 │
│                                                        │
│  ℹ️ Este formulário não constitui chat. O contato      │
│  será feito fora da plataforma pelo destinatário.      │
│                                                        │
└────────────────────────────────────────────────────────┘
```

- Cota visual: progress bar mostrando "X de Y contatos utilizados"
- Após envio: Toast success + redirect para perfil do destinatário
- Se cota esgotada: formulário inteiro desabilitado, mensagem: "Você atingiu o limite anual de contatos."

---

## 6. ESTADOS, LOADING & FEEDBACK

### 6.1 Skeleton Loading

```
CARD SKELETON:
┌──────────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │ ← bg muted, pulse animation
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░ │
├──────────────────────────────┤
│ ○ ░░░░░░░░░░░░░░░░          │
│   ░░░░░░░░░                  │
└──────────────────────────────┘

Cor: var(--muted), animação: pulse 1.5s ease-in-out infinite
Rounded: mesmo do componente real
```

### 6.2 Empty States

```
┌────────────────────────────────────────┐
│                                        │
│        🔍 (ícone 64px, muted)          │
│                                        │
│     Nenhum resultado encontrado        │
│     (Michroma 18px)                    │
│                                        │
│     Tente ajustar seus filtros ou      │
│     buscar por outros termos.          │
│     (Manrope 14px, muted-foreground)   │
│                                        │
│     [Limpar Filtros]                   │
│                                        │
└────────────────────────────────────────┘
```

### 6.3 Error States

```
┌────────────────────────────────────────┐
│                                        │
│        ⚠️ (ícone 64px, destructive)    │
│                                        │
│     Algo deu errado                    │
│                                        │
│     Não foi possível carregar...       │
│                                        │
│     [Tentar Novamente]                 │
│                                        │
└────────────────────────────────────────┘
```

### 6.4 Toast Notifications (solid-sonner)

- **Success:** borda-left 4px `var(--success)`, ícone ✅
- **Error:** borda-left 4px `var(--destructive)`, ícone ❌
- **Warning:** borda-left 4px `var(--warning)`, ícone ⚠️
- **Info:** borda-left 4px `var(--primary)`, ícone ℹ️
- Posição: top-right
- Duração: 4s (auto-dismiss)
- Animation: slide-in-right

### 6.5 Paywall / Access Denied States

```
TELA BLOQUEADA (role sem permissão):
┌────────────────────────────────────────┐
│                                        │
│        🔒 (ícone 64px, gold)           │
│                                        │
│     Funcionalidade Exclusiva           │
│     (Michroma 20px)                    │
│                                        │
│     O acesso a esta área requer o      │
│     plano Síndico Pro.                 │
│     (Manrope 14px)                     │
│                                        │
│     [Conhecer Planos →]               │
│     (botão gold, primary)              │
│                                        │
└────────────────────────────────────────┘
```

### 6.6 Trava de 90 Dias (Upload Temporal)

```
UPLOAD BLOQUEADO:
┌────────────────────────────────────────┐
│                                        │
│        ⏰ (ícone 48px, warning)         │
│                                        │
│     Vídeo atualizado recentemente      │
│                                        │
│     Seu vídeo institucional foi        │
│     enviado em 15/12/2025.             │
│     Próxima atualização disponível     │
│     em: 15/03/2026 (em 26 dias)       │
│                                        │
│     ████████████░░░░░ 71%              │
│     (progress bar de tempo restante)   │
│                                        │
└────────────────────────────────────────┘
```

---

## 7. ÍCONES & ASSETS

### 7.1 Biblioteca de Ícones

**Usar:** `@iconify/json` com preset UnoCSS Icons

**Família padrão:** `lucide` (linestyle, 24px stroke, consistente com a estética refinada)

**Mapeamento de ícones por funcionalidade:**
```
Home:           i-lucide-home
Buscar:         i-lucide-search
Vídeos:         i-lucide-play-circle
Perfil:         i-lucide-user
Assembleias:    i-lucide-clipboard-list
Transparência:  i-lucide-bar-chart-3
Avaliações:     i-lucide-star
Settings:       i-lucide-settings
Connect Me:     i-lucide-mail
Notificações:   i-lucide-bell
Upload:         i-lucide-upload-cloud
Play:           i-lucide-play
Pause:          i-lucide-pause
Volume:         i-lucide-volume-2
Fullscreen:     i-lucide-maximize
Like:           i-lucide-heart
Share:          i-lucide-share-2
Report:         i-lucide-flag
Menu:           i-lucide-menu
Close:          i-lucide-x
Chevron:        i-lucide-chevron-right
Filter:         i-lucide-sliders-horizontal
Calendar:       i-lucide-calendar
PDF:            i-lucide-file-text
Lock:           i-lucide-lock
Check:          i-lucide-check
Warning:        i-lucide-alert-triangle
Moon:           i-lucide-moon
Sun:            i-lucide-sun
Logout:         i-lucide-log-out
```

### 7.2 Logo

- **Header Desktop:** Brasão + "MASTER SÍNDICO" (horizontal, gold sobre navy)
- **Header Mobile:** Apenas Brasão (compact, 32px)
- **Favicon:** Brasão simplificado, fundo navy, brasão gold
- **Loading Splash:** Brasão centralizado com pulse animation

---

## 8. ANIMAÇÕES & MICRO-INTERAÇÕES

### 8.1 Transições de Rota (TanStack Router)

```css
.fade-in-up {
  animation: fadeInUp 0.4s ease-out forwards;
}

@keyframes fadeInUp {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
```

Cada página principal entra com `fade-in-up`. Componentes dentro da página usam `animation-delay` escalonado (0ms, 50ms, 100ms, 150ms) para efeito de cascata.

### 8.2 Hover States

- **Cards:** scale(1.02) + shadow-md → 200ms ease
- **Botões:** brightness(1.1) → 150ms
- **Links/Nav:** color transition → 150ms
- **Thumbnails:** overlay play icon fade-in → 200ms
- **Avatar:** ring-2 ring-gold → 150ms

### 8.3 Loading Button

```
ESTADO IDLE:    [Criar Assembleia →]
ESTADO LOADING: [⟳ Criando...]  (spinner gold, texto muted, pointer-events none)
ESTADO SUCCESS: [✓ Criado!]     (bg success, flash 500ms, volta ao idle)
```

---

## 9. RESPONSIVIDADE — BREAKPOINTS

```
MOBILE:    <768px   — 1 coluna, bottom nav, drawer sidebar
TABLET:    768-1023 — 2-3 colunas, sidebar colapsada
DESKTOP:   ≥1024    — 3-4 colunas, sidebar expandida
WIDE:      ≥1440    — max-width container, mais espaço lateral
```

**Regras Mobile-Specific:**
- Touch targets: mínimo 44x44px
- Swipe gestures: drawer sidebar (swipe right to open), dismiss toasts (swipe right)
- Safe area: padding-bottom para bottom nav
- Inputs: font-size 16px mínimo (evitar zoom no iOS)
- Modais: fullscreen em mobile (sheet de baixo para cima)

---

## 10. DARK MODE SPECIFICS

O dark mode é o modo "nativo" da marca (navy). O light mode é a adaptação para uso diurno.

**Dark Mode (padrão sugerido):**
- Background: navy profundo
- Cards: navy mais claro (`var(--card)`)
- Texto: branco/cinza claro
- Bordas: navy variant (`var(--border)`)
- Sombras: quase inexistentes (usar bordas sutis em vez disso)
- Gold continua gold (sem alteração)

**Light Mode:**
- Background: off-white (`var(--background)`)
- Cards: branco puro
- Texto: navy escuro
- Bordas: cinza suave
- Sombras: shadow-sm padrão
- Gold continua gold (sem alteração)

**Regra de ouro:** O gold e os feedback colors (destructive, success, warning) NÃO mudam entre temas. Apenas backgrounds, foregrounds e bordas se adaptam.

---

## 11. MAPA DE TELAS — SPRINT 1 & 2

### Sprint 1 — Fundação, Identidade e Acesso
```
1.  /login                    → Tela de Login (split layout)
2.  /cadastro                 → Cadastro Inteligente (parametrizado por URL)
3.  /verificar-email          → OTP Field (confirmação)
4.  /definir-senha            → Definição de senha com requisitos
5.  /                         → Home (feed estilo YouTube)
6.  /buscar                   → Busca com filtros e categorias
7.  /perfil/:id               → Perfil público (variável por role)
8.  /perfil/editar            → Edição de perfil
9.  /video/:id                → Player de vídeo (com paywall condicional)
10. /connect-me/:targetId     → Formulário Connect Me
11. /configuracoes            → Settings (estilo Meta)
12. /configuracoes/conta      → Dados da conta
13. /configuracoes/seguranca  → Alterar senha
14. /configuracoes/notif      → Toggle de notificações
15. /configuracoes/aparencia  → Tema claro/escuro/auto
16. /configuracoes/plano      → Detalhes do plano atual
```

### Sprint 2 — Motor de Gestão Condominial
```
17. /assembleias                 → Lista de assembleias
18. /assembleias/criar           → Wizard de criação (4 steps)
19. /assembleias/:id             → Detalhe da assembleia (pautas, vídeos)
20. /assembleias/:id/votar/:pId  → Interface de votação por pauta
21. /transparencia               → Dashboard de transparência do síndico
22. /transparencia/timeline      → Timeline completa da gestão
23. /transparencia/plano         → Plano de reeleição (edição)
24. /avaliacao/:sindicoId        → Formulário de avaliação (morador → síndico)
25. /avaliacao/resultado         → Resultado consolidado (visível só p/ síndico)
```

---

## 12. PADRÕES DE COMPONENTES (Kobalte/Core)

### Componentes Kobalte a utilizar:
```
Dialog       → Modais (confirmação de voto, connect me, upload)
Popover      → Menu do avatar, tooltips expandidos
DropdownMenu → Menu 3 dots nos cards, filtros
Select       → Selects de categoria, assunto
Tabs         → Tabs de perfil, tabs de assembleia
Accordion    → FAQ, detalhes de pauta expandível
Separator    → Divisores entre seções
Toast        → Via solid-sonner (já configurado)
Toggle       → Switches de notificações
RadioGroup   → Votação, seleção de tema
Checkbox     → Concordância de termos, filtros múltiplos
Progress     → Barras de execução, progresso de curso
```

### CVA (class-variance-authority) — Button Variants:
```typescript
const buttonVariants = cva(
  "inline-flex items-center justify-center rounded-md font-medium transition-all duration-200 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring",
  {
    variants: {
      variant: {
        primary:     "bg-primary text-primary-foreground hover:brightness-110",
        secondary:   "bg-secondary text-secondary-foreground hover:bg-secondary/80",
        outline:     "border border-border bg-transparent hover:bg-muted",
        ghost:       "hover:bg-muted hover:text-foreground",
        destructive: "bg-destructive text-destructive-foreground hover:brightness-110",
        link:        "text-primary underline-offset-4 hover:underline",
      },
      size: {
        sm: "h-8 px-3 text-sm",
        md: "h-10 px-4 text-sm",
        lg: "h-11 px-6 text-base",
        icon: "h-9 w-9",
      },
    },
    defaultVariants: { variant: "primary", size: "md" },
  }
)
```

---

## 13. CONCLUSÃO — CHECKLIST DE QUALIDADE

Antes de considerar qualquer tela "pronta", verificar:

- [ ] Paleta de cores respeitada integralmente (oklch tokens)
- [ ] Michroma em headings, Manrope em body — sem exceções
- [ ] Nenhum contador público de likes/comments para não-donos
- [ ] Paywall implementado onde a Matriz Funcional exige
- [ ] Responsivo testado em 375px, 768px, 1024px, 1440px
- [ ] Dark mode e light mode funcionando
- [ ] Skeleton loading em toda tela que faz fetch
- [ ] Empty states para listas vazias
- [ ] Error states para falhas de rede
- [ ] Focus ring gold em todos os inputs (acessibilidade)
- [ ] Touch targets ≥44px em mobile
- [ ] Animations: fadeInUp em rotas, hover states em cards, transitions em botões
- [ ] Toast feedback em toda ação do usuário (sucesso/erro)
- [ ] Sidebar items condicionais por role (ABAC)
- [ ] Logo Master Síndico no header (brasão gold)
- [ ] Scrollbar customizada (8px, cor border, thumb rounded)

---

---

## ADDENDUM A — INSIGHTS DE NOVAS REFERÊNCIAS VISUAIS (Batch 2)

### Referências analisadas:
- **VoiceLabs** (Landing Page, Dashboard, Explore/Library, Mobile) — Imagens 3-9
- **Genaigurus** (Profile com Badges, Leaderboard, Web App completo, Mobile Responsive, Search, Feedback) — Imagens 1-2

---

## A.1 SIDEBAR — REFINAMENTO BASEADO NO VOICELABS

A sidebar do VoiceLabs é provavelmente a melhor referência que temos para o Master Síndico. Ela resolve vários problemas que precisamos:

### A.1.1 Estrutura Detalhada da Sidebar (VoiceLabs como base)

O VoiceLabs organiza a sidebar assim:
1. **Logo + Toggle** no topo
2. **Account Switcher** (avatar + nome + dropdown chevron)
3. **Navegação primária** (Home, Voices)
4. **Seção agrupada "PLAYGROUND"** com label uppercase cinza
5. **Seção agrupada "PRODUCTS"** com label uppercase cinza
6. **Widget de uso de crédito** (circular progress, 80%, "Upgrade plan")

**Adaptação para Master Síndico:**

```
┌──────────────────────────────┐
│  [🛡️ MASTER SÍNDICO] [◫]   │ ← Logo + toggle collapse
│                              │
│  ┌────────────────────────┐  │
│  │ 👤 João Silva      ⌄  │  │ ← Account info com dropdown
│  │    Síndico Pro • 🥇   │  │    (trocar condomínio se múltiplos)
│  └────────────────────────┘  │
│                              │
│  ■ Início                    │ ← Item ativo: pill bg gold/15%
│  ♪ Vídeos                    │    com borda esquerda gold 3px
│  🔍 Buscar                   │    e texto gold
│                              │
│  MINHA GESTÃO ─────────────  │ ← Label: Michroma 10px uppercase
│  📋 Assembleias              │    letter-spacing 0.1em
│  📊 Transparência            │    cor muted-foreground
│  ⭐ Avaliações               │    margin-top 24px
│  📄 Exportar PDF             │
│                              │
│  CONEXÕES ──────────────────  │
│  📨 Connect Me          2/4  │ ← Badge numérico à direita
│  🏢 Empresas                 │    mostrando uso de cota
│  🏪 Vizinhança               │
│                              │
│  CONFIGURAÇÕES ─────────────  │
│  ⚙️ Configurações            │
│  ❓ Ajuda                    │
│                              │
│  ┌────────────────────────┐  │
│  │  ┌──┐                 │  │ ← Widget de Cota (inspirado
│  │  │73│ Uso do Plano     │  │    no "Credit Uses" do VoiceLabs)
│  │  │% │                  │  │
│  │  └──┘                  │  │    Circular progress ring gold
│  │  4 de 12 vídeos/mês   │  │    Texto: "X de Y vídeos/mês"
│  │  [Gerenciar Plano →]  │  │    Link gold para upgrade
│  └────────────────────────┘  │
│                              │
└──────────────────────────────┘
```

### A.1.2 Widget de Cota na Sidebar (NOVO — inspirado VoiceLabs "Credit Uses")

Este é um componente crucial que o VoiceLabs implementa brilhantemente. No canto inferior esquerdo da sidebar, há um widget compacto mostrando uso de créditos com um progress ring circular.

**Para o Master Síndico, adaptar para mostrar:**

**Para Empresas (Plus/Pro):**
```
┌────────────────────────────┐
│  ┌──────┐                  │
│  │ 67%  │  Uso do Plano    │
│  │  ◔   │                  │
│  └──────┘                  │
│  8 de 12 vídeos/mês       │
│  87 de 150 armazenados    │
│                            │
│  Dismiss   Gerenciar →     │
└────────────────────────────┘
```

**Para Síndicos (N2/N3):**
```
┌────────────────────────────┐
│  ┌──────┐                  │
│  │ 75%  │  Vídeos do Mês   │
│  │  ◔   │                  │
│  └──────┘                  │
│  3 de 4 vídeos publicados  │
│                            │
│  Dismiss   Ver Plano →     │
└────────────────────────────┘
```

**Para Moradores Pagantes:**
```
┌────────────────────────────┐
│  ┌──────┐                  │
│  │ 50%  │  Connect Me      │
│  │  ◔   │                  │
│  └──────┘                  │
│  2 de 4 contatos/ano      │
│                            │
│  Dismiss                   │
└────────────────────────────┘
```

**Componente técnico do Progress Ring:**
```
SVG circle com:
  - Track: stroke var(--surface-variant), stroke-width 3
  - Fill: stroke var(--primary) (gold), stroke-width 3
  - stroke-dasharray calculado: circumference * percentage
  - Transição: stroke-dashoffset 600ms ease-out
  - Tamanho: 48x48px
  - Texto central: porcentagem em Manrope 700, 14px
```

**Comportamento:**
- Dismissável com "X" (volta a aparecer no próximo login)
- Se uso > 80%: ring muda para `var(--warning)` (âmbar)
- Se uso = 100%: ring muda para `var(--destructive)` (vermelho), texto "Limite atingido"
- Link "Gerenciar →" redireciona para `/configuracoes/plano`

### A.1.3 Account Switcher na Sidebar (inspirado VoiceLabs)

O VoiceLabs tem um seletor de conta elegante no topo da sidebar. Para Master Síndico, adaptar para **seletor de condomínio** (síndicos que gerenciam múltiplos condomínios):

```
┌────────────────────────────┐
│  👤 João Silva           ⌄ │
│     Síndico Pro • 🥇 Ouro │
└────────────────────────────┘
         │
         ▼ (dropdown ao clicar no chevron)
┌────────────────────────────┐
│  SEUS CONDOMÍNIOS:         │
│                            │
│  ● Cond. Villa Park       │ ← ativo (dot gold)
│  ○ Cond. Jardim Aurora    │ ← inativo (dot muted)
│  ○ Cond. Solar das Flores │
│                            │
│  [+ Vincular condomínio]  │
└────────────────────────────┘
```

**Isso é relevante para:**
- Síndicos N3 que gerenciam múltiplos condomínios
- O contexto da sidebar muda conforme o condomínio selecionado (assembleias, moradores, avaliações)
- Indicador no header: "Villa Park" como breadcrumb contextual

---

## A.2 HOME PAGE — REFINAMENTO BASEADO NO VOICELABS + GENAIGURUS

### A.2.1 Saudação Personalizada (VoiceLabs "Good Morning, Jhon")

O VoiceLabs abre a home com "My Workspace" + "Good Morning, Jhon". Isso cria sensação de pertencimento. Adaptar:

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  Meu Painel                                                  │
│  Bom dia, João 👋                                           │
│                                             (Manrope 14px,  │
│  (Michroma 24px, var(--foreground))         muted-foreground)│
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

- "Bom dia" / "Boa tarde" / "Boa noite" dinâmico baseado no horário
- Nome truncado se muito longo
- Opcional: "Você tem 2 assembleias abertas" como subtexto contextual

### A.2.2 Quick Action Cards (VoiceLabs Grid 2x3)

O VoiceLabs usa cards grandes em grid 2x3 com ícones ilustrados para ações rápidas. Adaptar para Master Síndico como **cards de atalho na Home**:

**Desktop (grid 3x2 ou 6 colunas):**
```
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  ┌────────┐ │ │  ┌────────┐ │ │  ┌────────┐ │
│  │  📹    │ │ │  │  🔍    │ │ │  │  📋    │ │
│  └────────┘ │ │  └────────┘ │ │  └────────┘ │
│  Meus       │ │  Buscar     │ │  Assembleias│
│  Vídeos     │ │  Empresas   │ │             │
└─────────────┘ └─────────────┘ └─────────────┘
┌─────────────┐ ┌─────────────┐ ┌─────────────┐
│  ┌────────┐ │ │  ┌────────┐ │ │  ┌────────┐ │
│  │  📨    │ │ │  │  🏪    │ │ │  │  ⭐    │ │
│  └────────┘ │ │  └────────┘ │ │  └────────┘ │
│  Connect Me │ │  Vizinhança │ │  Avaliações │
└─────────────┘ └─────────────┘ └─────────────┘
```

**Especificações dos Quick Action Cards:**
- Tamanho: flex, min-height 100px, aspect-ratio ~1.2
- Fundo: `var(--card)` com borda 1px `var(--border)`
- Hover: borda `var(--primary)` + shadow-sm + scale(1.02)
- Ícone: 32px, centralizado no card, dentro de um circle 56x56 bg `var(--muted)`
- Ícone hover: circle bg `var(--primary)/10`, ícone cor `var(--primary)` (gold)
- Label: Manrope 600, 14px, centralizado abaixo do ícone, mt-3
- **Mobile:** grid 2x3 com cards menores (similar ao VoiceLabs mobile - Image 6)
- **ABAC:** Cards de funcionalidades não disponíveis no plano: NÃO renderizar. O grid se adapta.

**Variação Mobile (inspirada VoiceLabs Mobile - Image 6):**
- Cards com fundo `var(--surface-variant)` (navy variant no dark mode)
- Grid 2 colunas
- Ícone ilustrado centralizado no card
- Label abaixo
- Rounded-xl para cards maiores
- Gap-3

### A.2.3 Seção "Latest Blog / Featured Content" (Genaigurus)

O Genaigurus tem uma seção de "Latest blog" com cards em carousel horizontal, thumbnails grandes. Adaptar como **"Destaques"** na Home:

```
── Destaques ───────────────────────────── [Ver todos →]

┌──────────────────┐ ┌──────────────────┐ ┌──────────────┐
│                  │ │                  │ │              │
│    THUMBNAIL     │ │    THUMBNAIL     │ │   THUMBNAIL  │
│    (16:9 large)  │ │    (16:9 large)  │ │   (parcial)  │
│                  │ │                  │ │              │
│  Título do vídeo │ │  Título do vídeo │ │  Título...   │
│  em destaque     │ │  em destaque     │ │              │
└──────────────────┘ └──────────────────┘ └──────────────┘
         ←    scroll horizontal com snap    →
```

**Especificações:**
- Container: overflow-x auto, scroll-snap-type x mandatory, hide scrollbar
- Cada card: min-width 300px (desktop), min-width 260px (mobile)
- scroll-snap-align: start
- Thumbnail: aspect-ratio 16/9, rounded-t-xl, object-cover
- Card body: padding 12px, bg `var(--card)`
- Título: Manrope 600, 16px, line-clamp-2
- Meta: canal + timestamp, 13px muted
- Navegação: setas chevron nos extremos (desktop), dots indicator (mobile)
- Este carousel mostra vídeos curados/fixados pelo MasterSíndico admin

### A.2.4 Seção "Popular YouTube Videos" → "Vídeos Populares" (Genaigurus)

Logo abaixo dos destaques, o Genaigurus exibe uma seção de vídeos populares em grid padrão (como YouTube). Manter o mesmo padrão:

```
── Vídeos Populares ────────────────────── [Ver todos →]

[Video Card] [Video Card] [Video Card] [Video Card]
(grid 4 cols desktop, 2 cols mobile — padrão já definido na seção 4.1)
```

Nada muda do que já definimos, mas reforça que a Home é multi-seção vertical com seções temáticas.

### A.2.5 Seção "Categories" como Pills Horizontais (Genaigurus)

O Genaigurus mostra categorias como pills com a primeira destacada ("All Healthcare" em pill filled, demais em outline). Já definimos isso mas agora com mais detalhe visual:

```
── Categorias ──────────────────────────── [Ver todas →]

[Todos]  [Administração]  [Jurídico]  [Hidráulica]  [Elétrica]  [→]
 ═══════
 (gold     (outline         (outline     ...
  filled)   border)          border)
```

**Especificações refinadas:**
- Pill ativa: `bg-primary text-primary-foreground font-semibold rounded-full px-4 py-2`
- Pill inativa: `bg-transparent text-foreground border border-border rounded-full px-4 py-2`
- Pill hover (inativa): `bg-muted`
- Transição: background-color 150ms ease, color 150ms ease
- Gap entre pills: 8px
- Scroll horizontal com hide-scrollbar em mobile
- No desktop: se caber, mostrar todas. Se não couber, setas de navegação.

### A.2.6 Seção "Articles Based on Your Interest" → "Recomendados para Você" (Genaigurus)

O Genaigurus tem uma seção inferior com artigos baseados em interesse, exibidos como cards horizontais com avatar, título, data e ícones de interação. Adaptar para mostrar **vídeos personalizados baseados nas categorias de interesse do síndico/morador:**

```
── Recomendados para você ──────────────── [Ver todos →]

┌──────────────────────────────────────────────────────┐
│  [Avatar] Alex Santos               Set 15, 2026     │
│           Manutenção Preventiva de Fachadas:          │
│           Como evitar patologias estruturais          │
│                                                      │
│  [👍 Curtir]  [📋 Copiar]  [↗ Compartilhar]         │
└──────────────────────────────────────────────────────┘
```

Este é um formato alternativo de exibição para conteúdo tipo "artigo do fórum" ou vídeo em formato de lista horizontal. Útil para quando houver fórum (Sprint 4), mas a estrutura visual pode ser preparada agora.

---

## A.3 PROFILE PAGE — SISTEMA DE BADGES/STATUS (Genaigurus)

### A.3.1 Tab de Badges no Perfil (Genaigurus "My Profile > Badges")

O Genaigurus implementa uma aba "Badges" no perfil com:
- Grid de badges conquistados (coloridos, vibrantes)
- Badges não conquistados (cinza, outline, locked)
- Progresso em cada badge

**Adaptação para Master Síndico — Tab "Status" no Perfil do Síndico:**

Em vez de "badges" genéricos, usamos a lógica de Status do Cap. 8 do Manual:

```
┌──────────────────────────────────────────────────────────────┐
│  TABS: [Sobre] [Vídeos] [Gestão] [Status]                   │
│                                    ═══════                   │
│                                                              │
│  SEU STATUS ATUAL                                            │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │     🥉          🥈          🥇          💎            │  │
│  │   Bronze       Prata        Ouro      Diamante        │  │
│  │   (cinza)     (cinza)    (ATIVO ✓)   (locked)         │  │
│  │                             │                          │  │
│  │   ━━━━━━━━━━━━━━━━━━━━━━━━━●━━━━━━━━━━━░░░░░░░░░░░   │  │
│  │                                                        │  │
│  │   Seu status: Ouro                                     │  │
│  │   Próximo nível: Diamante                              │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  CRITÉRIOS DE EVOLUÇÃO                                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │                                                        │  │
│  │  📹 Vídeos assistidos (>70%)                           │  │
│  │  ████████████████████████████░░░ 47/60                 │  │
│  │  Assistiu 47 vídeos com mais de 70% de retenção        │  │
│  │                                                        │  │
│  │  🏢 Condomínios vinculados                             │  │
│  │  ████████████████░░░░░░░░░░░░░░ 3/5                   │  │
│  │  3 condomínios ativos no seu perfil                    │  │
│  │                                                        │  │
│  │  📚 Módulos de capacitação                             │  │
│  │  ██████████████████████████████ 5/5  ✅                │  │
│  │  Todos os módulos disponíveis concluídos               │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
│  ℹ️ Seu status é calculado automaticamente pela plataforma. │
│  Ele reflete seu engajamento com conteúdo qualificado,       │
│  não substitui a avaliação dos moradores.                    │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Design dos Status Badges (visual tipo medal/shield):**

```
BRONZE:
  ┌──────────┐
  │   🛡️     │  Circle 64px
  │  Bronze  │  bg: linear-gradient(135deg, #CD7F32 0%, #8B5E34 100%)
  │          │  border: 2px solid #CD7F32
  └──────────┘  shadow: 0 0 12px rgba(205,127,50,0.3)
                Ícone: shield com "B" centralizado

PRATA:
  Circle 64px
  bg: linear-gradient(135deg, #E8E8E8 0%, #A0A0A0 100%)
  border: 2px solid #C0C0C0
  shadow: 0 0 12px rgba(192,192,192,0.3)

OURO:
  Circle 64px
  bg: linear-gradient(135deg, #FFCC6A 0%, #C69C3F 100%)
  border: 2px solid var(--primary)
  shadow: 0 0 16px rgba(198,156,63,0.4)

DIAMANTE:
  Circle 64px
  bg: linear-gradient(135deg, #B9F2FF 0%, #E0C3FC 50%, #B9F2FF 100%)
  border: 2px solid #B9F2FF
  shadow: 0 0 20px rgba(185,242,255,0.3)
  animation: shimmer 3s ease-in-out infinite (brilho sutil)
```

**Status inativo (não alcançado):**
- Mesma shape mas com: opacity 0.3, grayscale(1), sem shadow
- Ícone de cadeado 16px no canto inferior direito

**Progress bar entre status:**
- Track: bg `var(--muted)`, h-2, rounded-full, full width
- Segmentos: 4 seções iguais separadas por dots (representando Bronze→Prata→Ouro→Diamante)
- Fill: gradient gold, animado da esquerda para a posição atual
- Dots ativos: 12px circle gold filled
- Dots inativos: 12px circle muted outline

### A.3.2 Profile Header Refinado (Genaigurus + VoiceLabs)

O Genaigurus mostra um perfil mobile com avatar centralizado, nome, handle, stats em row, e tabs abaixo. Combinar com o que já definimos:

**Mobile Profile Header:**
```
┌─────────────────────────────────────┐
│                                     │
│      ┌──────────┐                   │
│      │          │                   │
│      │  AVATAR  │  80px circular    │
│      │          │  border 3px gold  │
│      └──────────┘  se plano Pro     │
│                                     │
│      João Silva                     │  Michroma 20px
│      @joaosilva • 🥇 Ouro          │  Manrope 14px muted
│                                     │
│  ┌────────┐ ┌────────┐ ┌────────┐  │
│  │   3    │ │   47   │ │   5    │  │  Stats em row
│  │ Cond.  │ │ Vídeos │ │Módulos │  │  Número: Manrope 700 18px
│  └────────┘ └────────┘ └────────┘  │  Label: Manrope 400 12px muted
│                                     │
│  [Editar Perfil]  [Compartilhar]    │
│                                     │
│  [Sobre] [Vídeos] [Gestão] [Status] │
│   ═════                              │
│                                     │
└─────────────────────────────────────┘
```

**Stats Row:**
- 3 colunas iguais, text-center
- Separadas por border-right 1px `var(--border)` (exceto última)
- Número: Manrope 700, 18px, `var(--foreground)`
- Label: Manrope 400, 12px, `var(--muted-foreground)`
- **IMPORTANTE**: Estes stats são públicos conforme Cap. 8 do Manual (condomínios, vídeos >70%, módulos). NÃO são likes/comments.

---

## A.4 SIDEBAR — BADGES DE NOTIFICAÇÃO E "NEW" (VoiceLabs)

### A.4.1 Badge "New" em Items da Sidebar

O VoiceLabs mostra um badge "New" ao lado de itens como "Voice Changer" e "Dubbing". Útil para Master Síndico quando:
- Uma nova assembleia foi criada
- Há votação aberta pendente
- Nova funcionalidade foi liberada no plano

```
  📋 Assembleias        ● 2    ← badge numérico circular
  📊 Transparência
  ⭐ Avaliações          New   ← badge "Novo" para feature nova
```

**Badge numérico:**
- Circle 20px, bg `var(--destructive)`, text white, font-size 11px, font-weight 700
- Posição: à direita do label, margin-left auto
- Animação: scale(1) → scale(1.1) → scale(1) pulse sutil ao aparecer

**Badge "Novo":**
- Pill shape, bg `var(--primary)`, text `var(--primary-foreground)`, font-size 10px
- px-1.5 py-0.5, rounded-full, font-weight 600
- Desaparece após o usuário clicar no item

### A.4.2 Sidebar Sections com Separadores Visuais (VoiceLabs)

O VoiceLabs usa labels uppercase cinza para separar seções. Reforçar o padrão:

```css
.sidebar-section-label {
  font-family: 'Michroma', sans-serif;
  font-size: 10px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  padding: 0 12px;
  margin-top: 24px;
  margin-bottom: 8px;
}
```

---

## A.5 SEARCH PAGE — REFINAMENTO BASEADO NO GENAIGURUS

### A.5.1 Search com "Browse by Subjects" (Genaigurus Mobile)

O Genaigurus mobile mostra uma tela de busca com:
1. Barra de busca no topo
2. "Search now & previously" com chips de pesquisas recentes
3. "Browse questions by subjects" com tags de categorias

**Adaptação para Master Síndico — Tela de Busca (Mobile):**

```
┌─────────────────────────────────────┐
│  [←]  [🔍 Buscar...]        [Ads]  │
│                                     │
│  PESQUISAS RECENTES                 │
│  [✕ hidráulica] [✕ fachada]        │
│  [✕ síndico são paulo]             │
│                                     │
│  ── BUSCAR POR CATEGORIA ────────   │
│                                     │
│  [Administração]  [Jurídico]        │
│  [Hidráulica]     [Elétrica]        │
│  [Seguros]        [Engenharia]      │
│  [Segurança]      [Climatização]    │
│  [Manutenção]     [Sustentabilidade]│
│                                     │
│  ── HISTÓRICO DE BUSCA ──────────   │
│                                     │
│  🕐 empresa de elevadores          │
│  🕐 reforma fachada preço          │
│  🕐 síndico profissional bh        │
│                                     │
└─────────────────────────────────────┘
```

**Tags de categoria:**
- Cada tag: pill outline, border `var(--border)`, rounded-full, px-3 py-1.5
- Texto: Manrope 500, 13px, `var(--foreground)`
- Hover: bg `var(--muted)`, border `var(--primary)`
- Ao clicar: navega para `/buscar?categoria=hidraulica`
- Grid: flex wrap, gap-2

**Pesquisas recentes:**
- Chip com "✕" para remover
- bg `var(--surface-variant)`, text `var(--surface-foreground)`, rounded-full
- "✕" icon: 14px, muted-foreground, hover destructive
- Armazenado em localStorage (máximo 10 items)

**Histórico de busca:**
- Lista vertical com ícone 🕐 (clock), texto Manrope 400 14px
- Swipe-to-delete em mobile
- Hover: bg `var(--muted)` com rounded-md

### A.5.2 Resultados com Tabs (Genaigurus)

Após digitar e buscar, resultados organizados em tabs:

```
┌─────────────────────────────────────────────────────┐
│  🔍 "empresa hidráulica"                   [✕]     │
│                                                     │
│  [Todos]  [Empresas]  [Síndicos]  [Vídeos]  [CEP] │
│   ═══════                                           │
│                                                     │
│  3 empresas encontradas                             │
│                                                     │
│  ┌───────────────────────────────────────────────┐  │
│  │  [Logo] Hidra Solutions                       │  │
│  │         Empresa Pro • ⭐ 4.8                  │  │
│  │         #hidráulica #detecção-vazamentos      │  │
│  │         São Paulo, SP                         │  │
│  │                              [Ver Perfil →]   │  │
│  └───────────────────────────────────────────────┘  │
│                                                     │
│  ┌───────────────────────────────────────────────┐  │
│  │  [Logo] AquaFix Engenharia                    │  │
│  │         Empresa Plus                          │  │
│  │         #hidráulica #bombas #saneamento       │  │
│  │         Rio de Janeiro, RJ                    │  │
│  │                              [Ver Perfil →]   │  │
│  └───────────────────────────────────────────────┘  │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Result Card (tipo lista):**
- Fundo: `var(--card)`, border 1px `var(--border)`, rounded-lg
- Hover: border `var(--primary)/30`, shadow-sm
- Logo/Avatar: 48px, rounded-lg (empresa) ou rounded-full (síndico)
- Nome: Manrope 600, 16px
- Badge de plano: pill, eg "Empresa Pro" → bg `var(--primary)/10`, text `var(--primary)`
- Tags de categoria: pills menores, bg `var(--muted)`, text `var(--muted-foreground)`, font-size 12px
- Localização: ícone pin + texto, 13px muted
- CTA: "Ver Perfil →" link gold, text-sm

---

## A.6 FEEDBACK/REPORT FORM (Genaigurus "Tell us the problem")

### A.6.1 Formulário de Denúncia / Feedback

O Genaigurus tem uma tela "Send us some feedback" e "Tell us the problem" com input de descrição e botão de envio. Adaptar para o **botão de denúncia** do Master Síndico (moderação reativa):

```
┌────────────────────────────────────────────────────────┐
│  ← Denunciar Conteúdo                                  │
│                                                        │
│  Qual o problema?                                      │
│                                                        │
│  ○ Conteúdo comercial/propaganda                       │
│  ○ Dados de contato no vídeo (tel, email, WhatsApp)   │
│  ○ Conteúdo ofensivo ou inadequado                     │
│  ○ Informação técnica incorreta/perigosa               │
│  ○ Spam ou conteúdo repetitivo                         │
│  ○ Outro                                               │
│                                                        │
│  Descreva o problema (opcional):                       │
│  ┌──────────────────────────────────────────────────┐  │
│  │                                                  │  │
│  │  Textarea, max 300 chars                         │  │
│  │                                          120/300 │  │
│  └──────────────────────────────────────────────────┘  │
│                                                        │
│  [Enviar Denúncia]                                     │
│                                                        │
│  ℹ️ Sua denúncia será analisada pelo administrador     │
│  da plataforma. O conteúdo pode ser removido se        │
│  violar as diretrizes.                                 │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Design:**
- Modal (Dialog Kobalte) em desktop, sheet de baixo para cima em mobile
- Radio options: cada uma em um card minimal (p-3, rounded-md, hover bg muted)
- Radio selecionado: border-left 3px gold, bg `var(--primary)/5`
- Textarea: aparece com slide-down quando "Outro" é selecionado ou sempre visível
- Botão: full-width, variant primary
- Após envio: Toast success + fechar modal

---

## A.7 LANDING PAGE (VoiceLabs — Referência para Futuro)

Embora as Sprints 1 e 2 foquem na aplicação autenticada, vale registrar o padrão da landing page para quando for necessário (pode ser usada no onboarding parametrizado):

### A.7.1 Estrutura da Landing (VoiceLabs como modelo)

O VoiceLabs tem uma landing com:
1. **Header fixo**: Logo + nav links + "Log in" (outline) + "Sign up" (filled green)
2. **Hero section**: Subtitle pequeno (badge/label), headline grande, descrição, 2 CTAs
3. **Feature showcase**: Cards com ilustrações
4. **Trust logos**: Marquee horizontal de logos
5. **How it works**: Steps 1-2-3 com ícones
6. **Footer**: Multi-column links

**Adaptação para Master Síndico (esboço para cadastro parametrizado):**

```
┌──────────────────────────────────────────────────────────┐
│  [🛡️ MASTER SÍNDICO]        [Sobre] [Planos] [Entrar]  │
│                                                          │
│  ── HERO ─────────────────────────────────────────────── │
│                                                          │
│  Governança condominial                                  │  (small label gold)
│                                                          │
│  Decisões melhores.                                      │  (Michroma 40px)
│  Gestão com lastro.                                      │
│                                                          │
│  A plataforma que transforma conhecimento técnico        │  (Manrope 18px muted)
│  em critério de contratação.                             │
│                                                          │
│  [Criar Conta Grátis]    [Conhecer Planos]              │  (gold filled + outline)
│                                                          │
│  ── TRUST LOGOS ──────────────────────────────────────── │
│  [Logo1] [Logo2] [Logo3] [Logo4] [Logo5]  (scroll)     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

- Hero CTA primário: bg gold, text white, px-8 py-3, rounded-md, text-lg
- Hero CTA secundário: bg transparent, border gold, text gold, px-8 py-3
- Background: navy profundo com subtle radial gradient gold no canto superior direito (glow sutil)
- Trust logos: grayscale por padrão, cor no hover (opcional)

---

## A.8 DASHBOARD APP SHELL — REFINAMENTO FINAL (VoiceLabs)

### A.8.1 Header App (Top Bar) Refinado

Combinando VoiceLabs (Imagens 4, 8, 9) com o que já definimos. O VoiceLabs tem:
- Breadcrumb à esquerda ("Home", "Voice > Explore")
- Botão "Upgrade" com ícone de diamante
- Ícone de notificação (bell)
- Avatar

**Top Bar Master Síndico — Versão Refinada:**

```
┌──────────────────────────────────────────────────────────────────┐
│                                                                  │
│  [◫ Toggle]  Início > Assembleias      [🔍]  [🌙]  [🛡️ Pro]  │
│              (breadcrumb dinâmico)             toggle  [🔔 ●]   │
│                                                theme   [👤 ⌄]   │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

**Detalhe do botão "Pro" (inspirado VoiceLabs "Upgrade"):**
- Se o usuário NÃO está no plano máximo:
  ```
  [⬆ Upgrade]  — outline gold, text gold, rounded-full, px-3 py-1
  ```
- Se o usuário JÁ está no plano máximo:
  ```
  [🛡️ Pro]  — badge gold, bg gold/10, text gold, rounded-full, px-3 py-1
  ```
- Hover: bg gold/20
- Clicar: redireciona para `/configuracoes/plano`

**Breadcrumb:**
- Formato: "Seção > Subseção"
- Separador: ">" em muted-foreground
- Último item: font-weight 600, cor foreground
- Items anteriores: cor muted-foreground, hover text gold, cursor pointer (navegável)
- Font: Manrope 14px
- Visível apenas em desktop. Mobile: apenas título da página atual.

### A.8.2 Content Area — Spacing e Container

O VoiceLabs usa um content area com:
- Padding top generoso (32px após o header de contexto)
- Seções claramente separadas com headlines em uppercase

**Padrão de Content Area:**
```css
.main-content {
  padding: 24px 24px 48px;   /* desktop */
  padding: 16px 16px 80px;   /* mobile (80px para bottom nav) */
  max-width: 1280px;
  margin: 0 auto;
}

.section-headline {
  font-family: 'Michroma', sans-serif;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  margin-bottom: 16px;
}

.section-title {
  font-family: 'Michroma', sans-serif;
  font-size: 18px;
  color: var(--foreground);
  margin-bottom: 16px;
}
```

### A.8.3 Cards de Ação "Coloridos" (VoiceLabs "Create or Clone a Voice")

O VoiceLabs tem cards na parte inferior do dashboard com backgrounds coloridos vibrantes (gradiente rosa, verde, laranja). Isso é interessante para cards de destaque/CTA na Home:

**Adaptação: "Cards de Destaque" na Home (uso pontual):**

```
── COMECE AQUI ──────────────────────────────────────

┌──────────────────┐ ┌──────────────────┐ ┌──────────────────┐
│ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │ │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │ │ ▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓ │
│ Gradient gold    │ │ Gradient navy    │ │ Gradient navy    │
│                  │ │                  │ │ + gold           │
│ Complete seu     │ │ Explore          │ │ Crie sua         │
│ Perfil           │ │ Empresas         │ │ 1ª Assembleia    │
│                  │ │                  │ │                  │
│ Preencha todos   │ │ Encontre os      │ │ Comece a         │
│ os dados para    │ │ melhores         │ │ documentar sua   │
│ ativar sua conta │ │ fornecedores     │ │ gestão agora     │
│                  │ │                  │ │                  │
│ [Completar →]    │ │ [Buscar →]       │ │ [Criar →]        │
└──────────────────┘ └──────────────────┘ └──────────────────┘
```

**Design dos cards coloridos:**
- Card 1: bg `linear-gradient(135deg, var(--primary) 0%, var(--accent) 100%)`, text white
- Card 2: bg `var(--secondary)`, text `var(--secondary-foreground)`
- Card 3: bg `linear-gradient(135deg, var(--secondary) 0%, var(--surface-variant) 100%)`, acento gold no border
- Rounded-xl, p-5, min-height 180px
- Hover: brightness(1.1) + shadow-lg
- CTA: botão outline white, rounded-md
- **Uso:** apenas na home, para onboarding de novos usuários. Desaparece após completar as ações.

---

## A.9 EXPLORE PAGE / LIBRARY (VoiceLabs "Voices > Explore")

### A.9.1 Tabs Internas em Páginas de Conteúdo

O VoiceLabs Explore (Image 8) mostra:
- Breadcrumb: "Voice > Explore"
- Tabs: "My Voice" | "Explore" | "Default Voice"
- Filtros: "Filters" + "Trending" no canto direito
- Conteúdo organizado por seções: "Trending Voices", "Curated Language Collections", "Weekly Spotlight"

**Adaptação para Master Síndico — Página de Vídeos (/videos):**

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Vídeos                                         │
│          │                                                │
│          │  [Meus Vídeos] [Explorar] [Salvos]             │
│          │                 ═════════                       │
│          │                                     [⚙ Filtros]│
│          │                                                │
│          │  ── TENDÊNCIAS DA SEMANA ──────────            │
│          │  [Video Card] [Video Card] [Video Card] [→]   │
│          │  (carousel horizontal)                         │
│          │                                                │
│          │  ── POR CATEGORIA ────────────────             │
│          │  [Hidráulica ▼]                                │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │                                                │
│          │  [Elétrica ▼]                                  │
│          │  [Video Card] [Video Card] [Video Card] [VC]  │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Tabs internas:**
- Tab ativa: text `var(--foreground)`, font-weight 600, underline 2px gold
- Tab inativa: text `var(--muted-foreground)`, font-weight 400
- Gap: 24px entre tabs
- Animação: underline slide com transition 200ms

**Seções colapsáveis por categoria:**
- Heading com chevron (▼/▶) — clicar colapsa/expande
- Default: primeiras 2 categorias expandidas, demais colapsadas
- Transição: max-height + opacity, 300ms ease

---

## A.10 SUGESTÕES ADICIONAIS BASEADAS NA ANÁLISE

### A.10.1 Newsletter / Subscribe Widget (Genaigurus)

O Genaigurus tem um widget "Joining our newsletter" na sidebar com input de email e botão Subscribe. Para Master Síndico, isso pode ser adaptado para:

**Widget "Comunicados Oficiais" na sidebar (para moradores):**
```
┌────────────────────────────┐
│  📩 Comunicados            │
│                            │
│  Receba atualizações da    │
│  sua gestão condominial    │
│                            │
│  [Suas notificações: ON]   │
│  (toggle switch)           │
│                            │
└────────────────────────────┘
```

Simples toggle para ativar/desativar recebimento de comunicados do MasterSíndico admin. Não é newsletter comercial.

### A.10.2 Player de Vídeo Landscape (Genaigurus)

A Imagem 1 mostra um player de vídeo em landscape mode com:
- Timing Progress bar na parte inferior
- Botões: backend, forward skip
- Collapse button
- Progress bar com thumb arrastável

**Refinamento do Player Master Síndico:**

```
┌──────────────────────────────────────────────────────┐
│                                                      │
│                    VIDEO CONTENT                     │
│                                                      │
│                                                      │
│  ┌──────────────────────────────────────────────┐   │
│  │  [⏮ 10s]  [▶/⏸]  [⏭ 10s]                   │   │
│  │                                              │   │
│  │  0:45 ━━━━━━━━━●━━━━━━━━━━━━━━━━━━━ 3:30   │   │
│  │                (thumb gold, 14px circle)      │   │
│  │                                              │   │
│  │  [🔈 Vol]  [CC]  [⚙ 720p]  [PiP]  [⛶ Full] │   │
│  └──────────────────────────────────────────────┘   │
│                                                      │
└──────────────────────────────────────────────────────┘
```

**Controls specifics:**
- Progress bar track: bg `var(--muted)`, h-1 (expande para h-1.5 em hover)
- Progress bar fill: bg `var(--primary)` (gold)
- Thumb: 14px circle gold, visible only on hover/touch
- Buffer bar: bg `var(--muted-foreground)` com opacity 30%
- Skip buttons: 10 segundos forward/backward com ícone circular
- Volume: slider horizontal, gold fill
- Settings gear: dropdown com qualidade (Auto, 1080p, 720p, 480p, 360p)
- PiP (Picture in Picture): ícone de mini player
- Fullscreen: ícone de expandir
- Controles: fade-in on hover/touch, fade-out após 3s de inatividade
- Mobile: controles always visible durante primeiro toque, fade após 3s

### A.10.3 Social Links no Footer da Sidebar (Genaigurus)

O Genaigurus mostra ícones de redes sociais (Facebook, YouTube, X) na base da sidebar. Para Master Síndico, posicionar no **footer da aplicação** (não na sidebar):

```
Footer (discreto, bg var(--surface)):
  © 2026 Master Síndico
  [Termos] [Privacidade] [Suporte]
  [YouTube] [Instagram] (ícones 20px, muted-foreground)
```

---

## A.11 MAPA DE COMPONENTES ATUALIZADO

Com base nas novas referências, componentes adicionais a implementar:

```
NOVOS COMPONENTES (adicionados neste addendum):

ProgressRing          → Circular SVG progress (cota de uso na sidebar)
AccountSwitcher       → Dropdown de seleção de condomínio (síndicos)
QuickActionCard       → Cards de atalho na home (grid 2x3 / 3x2)
StatusBadge           → Badge metalizado Bronze/Prata/Ouro/Diamante
StatusProgressBar     → Barra segmentada de progresso de status
SearchHistoryChip     → Chip de pesquisa recente com dismiss
CategoryTag           → Pill de categoria para busca
BreadcrumbNav         → Navegação contextual no top bar
PlanBadge             → Badge do plano atual (Pro, Plus, N1, N2, N3)
UpgradePill           → Botão "Upgrade" no top bar
SidebarQuotaWidget    → Widget de uso de cota (bottom sidebar)
HighlightCard         → Card colorido gradient para onboarding CTA
SectionHeadline       → Label de seção uppercase (Michroma 10px)
VideoPlayerControls   → Controles customizados do player
ReportDialog          → Modal de denúncia com radio options
```

---

*Addendum A incorporado ao Design System v1.1*  
*Próximo passo: enviar mais referências visuais ou iniciar implementação via Claude Code.*

---

## ADDENDUM A — REFINAMENTOS BASEADOS EM REFERÊNCIAS ADICIONAIS

### A.1 Sidebar Refinada — Padrão VoiceLabs (Referências: Imagens VoiceLabs Dashboard)

A sidebar do VoiceLabs trouxe um padrão de organização superior ao que tínhamos definido inicialmente. A estrutura usa **agrupamento semântico com labels uppercase** e um **widget de uso de recursos** fixo no rodapé da sidebar. Vamos incorporar ambos.

**Nova Estrutura da Sidebar (revisada):**

```
┌──────────────────────────────┐
│  [🛡️ MASTER SÍNDICO]        │ ← Logo horizontal, Michroma, gold
│  [□ Toggle collapse]         │ ← Ícone de colapsar sidebar
│                              │
│  ┌────────────────────────┐  │
│  │ 👤 João Silva      ⌄  │  │ ← Avatar 32px + nome + dropdown arrow
│  │    Síndico Pro · 🥇   │  │ ← Role + status badge inline
│  └────────────────────────┘  │
│                              │
│  ███ Home                    │ ← Item ativo: fundo gold/10, borda-left 3px gold
│  ░░░ Vídeos                  │    texto gold, ícone gold
│                              │
│  MINHA GESTÃO ─────────────  │ ← Label: Michroma 10px, uppercase, letter-spacing 0.12em
│  ░░░ Assembleias             │    cor: var(--muted-foreground), margin-top 24px
│  ░░░ Transparência           │    margin-bottom 8px
│  ░░░ Avaliações              │
│                              │
│  DESCOBRIR ────────────────  │
│  ░░░ Buscar                  │
│  ░░░ Empresas                │
│  ░░░ Comércio Local          │
│                              │
│  CONEXÕES ─────────────────  │
│  ░░░ Connect Me              │
│                              │
│  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─   │ ← Separador tracejado sutil
│  ░░░ Configurações           │
│  ░░░ Ajuda                   │
│                              │
│  ┌────────────────────────┐  │ ← WIDGET DE USO (fixo no bottom)
│  │                        │  │    Inspirado no "Credit Uses" do VoiceLabs
│  │  ◐ 75%                │  │ ← Circular progress ring, stroke gold
│  │                        │  │
│  │  Uso do Plano          │  │ ← Manrope 600, 13px
│  │  3 de 4 vídeos/mês    │  │ ← Manrope 400, 12px, muted-foreground
│  │  2 de 4 Connect Me/ano│  │
│  │                        │  │
│  │  [Dismiss] [Gerenciar] │  │ ← Links: ghost e gold
│  └────────────────────────┘  │
│                              │
└──────────────────────────────┘
```

**Detalhes do Widget de Uso (Credit Uses):**
- Posição: fixo no bottom da sidebar, acima do padding inferior
- Background: `var(--surface-variant)` com border 1px `var(--border)`
- Border-radius: 12px
- Padding: 16px
- Circular progress: 40px de diâmetro, stroke-width 3px, cor `var(--primary)` (gold) para o preenchido, `var(--border)` para o track
- Percentagem: centralizada dentro do ring, Manrope 700, 14px
- Dismissível: botão X no canto superior direito, 16px, ghost
- "Gerenciar": link que redireciona para `/configuracoes/plano`
- O widget deve ser **condicional** — só aparece quando o uso está acima de 50% do limite do plano. Abaixo disso, a sidebar mostra apenas a navegação limpa.
- Quando uso atinge 90%+: o ring muda de gold para `var(--warning)` (âmbar) como alerta visual

**Comportamento do User Account Switcher:**
- O dropdown (⌄) ao lado do nome abre um popover com:
  - Foto de perfil maior (64px)
  - Nome completo
  - Email
  - Role e Plano atual
  - Status do Síndico (badge)
  - Separador
  - "Ver meu perfil" → link
  - "Trocar conta" → se houver múltiplas contas (futuro)
  - "Sair" → logout
- Transição: popover slide-down 200ms ease, com backdrop blur sutil

**Badge "Novo" em itens da sidebar:**
- Inspirado no VoiceLabs que usa a pill "New" ao lado de itens
- No Master Síndico: usar uma pill `var(--primary)` com texto "Novo" (Manrope 600, 10px, uppercase) ao lado de funcionalidades recém-lançadas
- A pill é removida automaticamente após o primeiro acesso do usuário àquela rota

### A.2 Home Dashboard Aprimorada — Greeting + Action Cards (Referência: VoiceLabs Home)

O VoiceLabs mostra uma abordagem de home dashboard que combina **saudação personalizada** + **grid de ações rápidas** + **conteúdo recente**. Isso é superior ao feed puro estilo YouTube que definimos antes. A home do Master Síndico deve ser um **hybrid**: dashboard operacional no topo + feed de conteúdo abaixo.

**Nova Home — Layout Desktop:**

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  ┌──────────────────────────────────────────────────┐  │
│          │  │  Meu Espaço                                      │  │
│          │  │  Bom dia, João                      [❓ Ajuda]  │  │
│          │  │  h1 Michroma 24px                                │  │
│          │  │                                                  │  │
│          │  │  AÇÕES RÁPIDAS (grid 2x3 desktop, 2x2 mobile)   │  │
│          │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐           │  │
│          │  │  │ 📋      │ │ 📹      │ │ 🔍      │           │  │
│          │  │  │ Criar   │ │ Upload  │ │ Buscar  │           │  │
│          │  │  │Assembleia│ │ Vídeo  │ │Empresas │           │  │
│          │  │  └─────────┘ └─────────┘ └─────────┘           │  │
│          │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐           │  │
│          │  │  │ 📊      │ │ 📨      │ │ 🎟️      │           │  │
│          │  │  │Transpar.│ │Connect  │ │Benefíc. │           │  │
│          │  │  │         │ │ Me      │ │         │           │  │
│          │  │  └─────────┘ └─────────┘ └─────────┘           │  │
│          │  │                                                  │  │
│          │  └──────────────────────────────────────────────────┘  │
│          │                                                        │
│          │  ── ASSEMBLEIAS ATIVAS ────────────── [Ver todas →]   │
│          │  ┌──────────────────┐ ┌──────────────────┐            │
│          │  │ 📋 Ordinária 26/1│ │ 📋 Manutenção    │            │
│          │  │ 🟢 ABERTA        │ │ ⏳ AGENDADA      │            │
│          │  │ 3 pautas pendentes│ │ Para: 01/04     │            │
│          │  │ Votação: 5 dias  │ │                  │            │
│          │  └──────────────────┘ └──────────────────┘            │
│          │                                                        │
│          │  ── CATEGORY ICONS ──────────────────────────────────  │
│          │  [👤Para Você] [⭐Popular] [🔧Manutenção] [⚡Elétrica]│
│          │  [💧Hidráulica] [🔒Segurança] [📋Jurídico] [→]      │
│          │                                                        │
│          │  ── VÍDEOS RECOMENDADOS ─────────────── [Ver todos →]  │
│          │  [Video Card] [Video Card] [Video Card] [Video Card]  │
│          │                                                        │
│          │  ── COMÉRCIO LOCAL ──────────────────── [Ver todos →]  │
│          │  [Promo Card] [Promo Card] [Promo Card]               │
│          │                                                        │
└──────────────────────────────────────────────────────────────────┘
```

**Action Cards (Quick Actions Grid):**

Inspirados diretamente nos cards do VoiceLabs (Text to Speech, Audiobook, Image to Video, etc.), mas adaptados para o contexto Master Síndico.

**Design de cada Action Card:**
```
┌──────────────────────────┐
│                          │
│     📋                   │  ← Ícone 32px, cor var(--primary) gold
│                          │
│  Criar Assembleia        │  ← Manrope 600, 13px, var(--foreground)
│                          │
└──────────────────────────┘
```

- Dimensão: 120x100px (desktop) ou flex-1 min-w-[100px] (responsive)
- Background: `var(--card)` (branco em light / navy-card em dark)
- Border: 1px solid `var(--border)`
- Border-radius: 12px (rounded-xl)
- Hover: border-color `var(--primary)`, shadow-sm, scale(1.02), transition 200ms
- Active (pressed): scale(0.98)
- Ícone: centralizado no topo, 32px, cor `var(--primary)` por default
- Label: centralizado abaixo, Manrope 600, 13px
- Gap entre ícone e label: 8px

**Action Cards variam por role:**

| Síndico N3 | Síndico N2 | Síndico N1 | Morador Pg | Morador Base | Empresa Pro |
|---|---|---|---|---|---|
| Criar Assembleia | Criar Assembleia | Buscar Empresas | Buscar Empresas | Buscar Empresas | Upload Vídeo |
| Upload Vídeo | Upload Vídeo | Assistir Vídeos | Meu Currículo | Buscar Síndicos | Criar Curso |
| Buscar Empresas | Buscar Empresas | Fórum (S4) | Connect Me | Ver Benefícios | Meus Vídeos |
| Transparência | Connect Me | Ver Benefícios | Ver Benefícios | — | Connect Me |
| Connect Me | Ver Benefícios | — | — | — | Ver Currículos |
| Ver Benefícios | — | — | — | — | Ver Benefícios |

Cards de funcionalidades não disponíveis no plano: **NÃO renderizar**. Sem cadeado, sem upsell no grid de ações.

**Greeting personalizado:**
- Label superior: "Meu Espaço" → Manrope 400, 13px, cor `var(--muted-foreground)`
- Greeting: "Bom dia, João" → Michroma 24px (h1), cor `var(--foreground)`
- Lógica de saudação: 
  - 05:00 - 11:59 → "Bom dia"
  - 12:00 - 17:59 → "Boa tarde"
  - 18:00 - 04:59 → "Boa noite"
- Direita do greeting: botão "Ajuda" (ghost, ícone `i-lucide-help-circle`)

**Seção "Assembleias Ativas":**
- Só aparece se o usuário tem condomínio vinculado E existem assembleias com status ABERTA ou AGENDADA
- Cards compactos: 280px width, horizontal scroll em mobile
- Badge de urgência: se votação expira em <48h, badge pulsante `var(--warning)`
- Se nenhuma assembleia ativa: seção completamente oculta (não mostrar empty state)

### A.3 Profile com Badges e Status Detalhado (Referência: Genaigurus Profile + Badge System)

O Genaigurus mostra um sistema de **perfil com tabs (Posts/Badges)** e uma **tela de detalhe de badge com progresso e milestones**. Isso mapeia perfeitamente para o sistema de Status do Síndico (Bronze/Prata/Ouro/Diamante).

**Profile Page — Tab "Badges" (apenas para Síndicos):**

```
┌──────────────────────────────────────────────────────────────────┐
│  PERFIL DO SÍNDICO                                                │
│                                                                   │
│  TABS: [Vídeos] [Gestão] [Status] [Sobre]                       │
│                  ═══════                                          │
│                                                                   │
│  ┌──────────────────────────────────────────────────────────┐    │
│  │  SEU STATUS ATUAL                                         │    │
│  │                                                           │    │
│  │       🥇                                                  │    │
│  │      OURO                                                 │    │
│  │                                                           │    │
│  │  Conquiste mais 15 módulos para alcançar 💎 Diamante      │    │
│  │  ██████████████████░░░░░░░░ 68%                          │    │
│  │                                                           │    │
│  └──────────────────────────────────────────────────────────┘    │
│                                                                   │
│  CRITÉRIOS DO STATUS                                              │
│                                                                   │
│  ┌────────────────────────────────────────────────────┐          │
│  │  📹 Vídeos Assistidos (>70%)                       │          │
│  │                                                    │          │
│  │  Meta Ouro: 50 vídeos     ✅ Atingido (47/50)     │          │
│  │  Meta Diamante: 100 vídeos  ░░ 47/100             │          │
│  │  ██████████████████████████████████████░░░░ 47%   │          │
│  └────────────────────────────────────────────────────┘          │
│                                                                   │
│  ┌────────────────────────────────────────────────────┐          │
│  │  🏢 Condomínios Vinculados                         │          │
│  │                                                    │          │
│  │  Meta Ouro: 3 condomínios   ✅ Atingido (3/3)     │          │
│  │  Meta Diamante: 5 condomínios  ░░ 3/5             │          │
│  │  ██████████████████████████████░░░░░░░░░░░ 60%    │          │
│  └────────────────────────────────────────────────────┘          │
│                                                                   │
│  ┌────────────────────────────────────────────────────┐          │
│  │  📚 Módulos Concluídos                             │          │
│  │                                                    │          │
│  │  Meta Ouro: 5 módulos       ✅ Atingido (5/5)     │          │
│  │  Meta Diamante: 20 módulos   ░░ 5/20              │          │
│  │  ████████████░░░░░░░░░░░░░░░░░░░░░░░░░░░░ 25%    │          │
│  └────────────────────────────────────────────────────┘          │
│                                                                   │
│  JORNADA DE STATUS                                                │
│                                                                   │
│  🥉 Bronze ──── 🥈 Prata ──── 🥇 Ouro ──── 💎 Diamante        │
│     ✅            ✅            ●(atual)      ○(próximo)         │
│                                                                   │
│  Step ✅: circle filled verde, linha cheia                        │
│  Step ●: circle filled gold (atual), pulsante                     │
│  Step ○: circle outline cinza, linha tracejada                    │
│                                                                   │
└──────────────────────────────────────────────────────────────────┘
```

**Design dos Status Badges (versão expandida):**

```
BRONZE:
  Container: bg gradient(135deg, #CD7F32 15%, #8B5E23 100%) com opacity 10%
  Border: 1px solid rgba(#CD7F32, 0.3)
  Ícone: medalha bronze 48px (custom SVG ou emoji 🥉)
  Label: "BRONZE" Michroma 14px, #CD7F32
  Glow (hover): box-shadow 0 0 20px rgba(#CD7F32, 0.15)

PRATA:
  Container: bg gradient(135deg, #C0C0C0 15%, #808080 100%) com opacity 10%
  Border: 1px solid rgba(#C0C0C0, 0.3)
  Ícone: 🥈 48px
  Label: "PRATA" Michroma 14px, #A0A0A0
  Glow: 0 0 20px rgba(#C0C0C0, 0.15)

OURO:
  Container: bg gradient(135deg, var(--primary) 15%, #A67C00 100%) com opacity 10%
  Border: 1px solid rgba(var(--primary), 0.3)
  Ícone: 🥇 48px
  Label: "OURO" Michroma 14px, var(--primary)
  Glow: 0 0 20px rgba(var(--primary), 0.2)

DIAMANTE:
  Container: bg gradient(135deg, #B9F2FF 10%, #E0C3FC 50%, #B9F2FF 100%) com opacity 10%
  Border: 1px solid rgba(#B9F2FF, 0.4)
  Ícone: 💎 48px (ou custom SVG com animação shimmer)
  Label: "DIAMANTE" Michroma 14px, gradient text (#B9F2FF → #E0C3FC)
  Glow: 0 0 24px rgba(#B9F2FF, 0.2)
  Efeito especial: shimmer animation sutil no container (keyframe que move um highlight linear da esquerda para a direita a cada 3s)
```

**Shimmer Animation para Diamante:**
```css
@keyframes shimmer {
  0% { background-position: -200% center; }
  100% { background-position: 200% center; }
}

.status-diamante {
  background: linear-gradient(
    90deg,
    transparent 0%,
    rgba(185, 242, 255, 0.05) 50%,
    transparent 100%
  );
  background-size: 200% 100%;
  animation: shimmer 3s ease-in-out infinite;
}
```

**Badge Inline (versão compacta para uso em cards, sidebar, etc.):**
```
┌─────────────────┐
│ 🥇 Ouro         │  ← 24px height, rounded-full, px-2.5
└─────────────────┘     bg: color/10, border: 1px solid color/30
                        font: Manrope 600, 11px, uppercase
```

### A.4 Breadcrumb Navigation (Referência: VoiceLabs)

O VoiceLabs usa breadcrumbs simples (Voice > Explore) para navegação em profundidade. Essencial para as telas de assembleia e transparência.

```
FORMATO:
  Início  /  Assembleias  /  Ordinária 2026/1  /  Pauta 3

DESIGN:
  Cor dos links: var(--muted-foreground), hover: var(--foreground)
  Separador: "/" em var(--muted-foreground), px-1.5
  Item atual (último): var(--foreground), font-weight 600, sem link
  Font: Manrope 400, 13px
  Posição: top do content area, abaixo do header, mb-4
```

**Quando usar breadcrumbs:**
- Assembleias > Detalhe > Pauta > Votação (3-4 níveis)
- Transparência > Timeline > Evento específico
- Configurações > Seção específica
- Buscar > Perfil da empresa > Vídeo

**Quando NÃO usar:**
- Home (nível 0)
- Páginas de primeiro nível (Buscar, Vídeos, Connect Me)

### A.5 Seção "Assembleias Ativas" — Cards Compactos com Urgência

Inspirado nos cards de "Latest from the Library" do VoiceLabs, mas com dados de assembleia.

**Assembly Preview Card (usado na Home):**
```
┌──────────────────────────────────────────┐
│                                          │
│  📋  Assembleia Ordinária 2026/1         │ ← Ícone + título, Manrope 600, 14px
│                                          │
│  🟢 Aberta  •  5 pautas  •  Votação     │ ← Status badge + metadata inline
│                    até 20/03             │    Manrope 400, 12px, muted-foreground
│                                          │
│  ⚡ Votação expira em 2 dias            │ ← Badge de urgência (se <48h)
│                                          │    bg warning/10, text warning
│                                          │    animação pulse sutil
└──────────────────────────────────────────┘

DIMENSÕES:
  Min-width: 280px
  Padding: 16px
  Background: var(--card)
  Border: 1px solid var(--border)
  Border-radius: 12px
  Hover: border-color var(--primary), cursor pointer
```

**Regra de urgência visual:**
- Votação expira em >7 dias: nenhum badge especial
- Votação expira em 3-7 dias: badge "Votação em X dias" — cor `var(--muted-foreground)`, sem animação
- Votação expira em <48h: badge "⚡ Expira em X horas" — cor `var(--warning)`, pulse animation
- Votação encerrada: badge "Encerrada" — cor `var(--muted-foreground)`, grayscale no card

### A.6 Search Page — Browse by Categories (Referência: Genaigurus Search)

O Genaigurus mostra uma search page com "Browse questions by subjects" usando uma **grid de category pills grandes** + **histórico de busca**. Isso é valioso para a busca do Master Síndico.

**Search Page Refinada:**
```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  ┌──────────────────────────────────────────────────┐  │
│          │  │  🔍 Buscar síndicos, empresas, serviços...      │  │
│          │  │     input full-width, h-12, text-base            │  │
│          │  └──────────────────────────────────────────────────┘  │
│          │                                                        │
│          │  ANTES DE DIGITAR (estado inicial da busca):            │
│          │                                                        │
│          │  Buscar por categorias                                  │
│          │  ┌────────────────────────────────────────────────┐    │
│          │  │                                                │    │
│          │  │  [🏛️ Administração]  [⚖️ Jurídico]            │    │
│          │  │  [📊 Contabilidade]  [👥 RH e Pessoas]        │    │
│          │  │  [📚 Cursos]         [🎤 Oratória]            │    │
│          │  │  [🛡️ Seguros]       [🔍 Laudos e Vistorias]  │    │
│          │  │  [🦠 Saúde Ambiental][⚡ Elétrica e Energia]  │    │
│          │  │  [💧 Hidráulica]     [📏 Medição Consumo]     │    │
│          │  │  [🔥 Gás]           [🛗 Elevadores]           │    │
│          │  │  [❄️ Climatização]   [🧯 Incêndio]            │    │
│          │  │  [🔒 Segurança]      [🧹 Limpeza Especial]   │    │
│          │  │  [👷 Terceirização]  [🌿 Jardins]             │    │
│          │  │  [🏊 Lazer]          [🏗️ Arquitetura]         │    │
│          │  │  [🔧 Engenharia]     [🧱 Construção]          │    │
│          │  │  [🏢 Fachadas]       [🅿️ Garagens]            │    │
│          │  │  [💻 Tecnologia]     [♻️ Sustentabilidade]    │    │
│          │  │  [🛒 Mercado Cond.]  [📦 Materiais]           │    │
│          │  │  [🏪 Comércio Local]                           │    │
│          │  │                                                │    │
│          │  └────────────────────────────────────────────────┘    │
│          │                                                        │
│          │  Buscas recentes                      [Limpar]        │
│          │  ┌────────────────────────────────────────────────┐    │
│          │  │  🕐 empresa elétrica zona sul                  │    │
│          │  │  🕐 síndico profissional são paulo             │    │
│          │  │  🕐 impermeabilização fachada                   │    │
│          │  └────────────────────────────────────────────────┘    │
│          │                                                        │
└──────────────────────────────────────────────────────────────────┘
```

**Category Pill (Browse Grid):**
```
┌──────────────────────┐
│  ⚡ Elétrica e       │
│     Energia          │
└──────────────────────┘

DESIGN:
  Width: auto (fit-content), min-width: 140px
  Height: auto
  Padding: 12px 16px
  Background: var(--muted)  (light) / var(--surface-variant) (dark)
  Border: 1px solid var(--border)
  Border-radius: 12px
  Font: Manrope 500, 13px
  Ícone: 18px, cor var(--primary), mr-2
  
  Hover: bg var(--primary)/8, border-color var(--primary)/30
  Active: bg var(--primary)/15, scale(0.98)
```

**Layout da grid de categorias:** `grid-cols-2` (mobile), `grid-cols-3` (tablet), `grid-cols-4` (desktop), gap-3

**Buscas Recentes:**
- Armazenadas em localStorage (máximo 10 itens)
- Cada item: ícone relógio (🕐 / `i-lucide-clock`), texto da busca, botão X para remover individual
- "Limpar": remove todas de uma vez
- Click no item: preenche o input de busca e dispara a busca

**Após digitar (estado de resultados):**
- Transição: a grade de categorias e buscas recentes faz fadeOut e é substituída pela lista de resultados com fadeIn
- Filtros horizontais aparecem: [Todos] [Síndicos] [Empresas] [Comércio] + [Filtros avançados ▼]
- Debounce de 300ms antes de disparar a query

### A.7 Video Player Mobile — Landscape Mode (Referência: Genaigurus Video Player)

O Genaigurus mostra um player de vídeo em landscape no mobile com controles sobrepostos. Importante para a experiência de consumo de vídeo no Master Síndico.

**Mobile Portrait (vídeo no topo):**
```
┌─────────────────────────────┐
│  [←] Título do vídeo  [⋮]  │ ← Sticky header com back button
├─────────────────────────────┤
│                             │
│     VIDEO PLAYER 16:9       │ ← Aspect ratio fixo, full-width
│                             │
│  [▶/⏸] [━━━━●━━━] [⛶]    │ ← Controles inline
├─────────────────────────────┤
│                             │
│  Título do Vídeo Completo   │
│  Canal/Empresa • 2h atrás   │
│                             │
│  [👍] [↗ Compartilhar]     │
│                             │
│  ── Descrição ──            │
│  Texto expansível...        │
│  [Ver mais ▼]               │
│                             │
│  ── Vídeos Relacionados ──  │
│  [Card H] [Card H] [Card H]│ ← Scroll horizontal
│                             │
├─────────────────────────────┤
│  BOTTOM NAV                 │
└─────────────────────────────┘
```

**Mobile Landscape (fullscreen):**
```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│                                                              │
│                     VIDEO PLAYER                             │
│                   (fullscreen 100vw x 100vh)                 │
│                                                              │
│  Backend ←  │  ● HD  │  Frente →                            │
│                                                              │
│  [⏮] [▶/⏸] [⏭]                                             │
│                                                              │
│  7:00 ━━━━━━━━━━━━━━━━━━━●━━━━━━━━━━━ 45:26                │
│                                                              │
│  [🔊] [CC] [⚙️ Qualidade] [⏩ Velocidade] [⛶ Sair FS]     │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Controles do player customizado:**
- Barra de progresso: height 4px (repouso) → 8px (hover/touch), cor `var(--primary)` (gold)
- Thumb da barra: 16px circle, bg white, shadow-md, visível apenas em hover/touch
- Buffered indicator: cor `var(--primary)` com opacity 30%
- Tempo: Manrope 500, 12px, white
- Botões de controle: white, 24px, opacity 0.9, hover opacity 1
- Background dos controles: gradient transparent → rgba(0,0,0,0.7) (bottom gradient)
- Auto-hide: controles somem após 3s sem interação, reaparecem ao toque/mouse move
- Transição dos controles: fadeIn/fadeOut 300ms

**Paywall overlay no player (para Base assistindo empresa):**
- Ativado quando o vídeo atinge 25% do tempo total
- Player pausa automaticamente
- Overlay: fundo `var(--secondary)` com 85% opacity + backdrop-blur 8px
- Conteúdo centralizado:
  ```
  🔒
  
  Conteúdo exclusivo para assinantes
  
  Assine para assistir ao vídeo completo e
  acessar todo o acervo técnico da plataforma.
  
  [Conhecer Planos →]  (botão gold, primary)
  [Continuar Navegando] (link ghost)
  ```
- O frame do vídeo fica congelado e desfocado (filter: blur(8px)) atrás do overlay

### A.8 Explore/Voices Page Pattern → Catálogo de Empresas (Referência: VoiceLabs Voices)

O VoiceLabs Explore page (Imagem 8) mostra um padrão de **tabs horizontais** + **trending items** + **curated collections** + **weekly spotlight**. Isso mapeia bem para o catálogo de empresas do Master Síndico.

**Catálogo de Empresas — Tela "Buscar Empresas":**
```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  EMPRESAS DO MERCADO CONDOMINIAL                       │
│          │                                                        │
│          │  TABS: [Todas] [Por Categoria] [Por Região]            │
│          │        ══════                                          │
│          │                                                        │
│          │  ┌─── DESTAQUES ──────────────── [Ver todos →] ───┐   │
│          │  │                                                 │   │
│          │  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────┐ │   │
│          │  │  │ Logo    │ │ Logo    │ │ Logo    │ │ ... │ │   │
│          │  │  │ Alpha   │ │ Beta    │ │ Gamma   │ │     │ │   │
│          │  │  │ Eng.    │ │ Seg.    │ │ Elét.   │ │     │ │   │
│          │  │  │         │ │         │ │         │ │     │ │   │
│          │  │  │ Pro ●   │ │ Plus ●  │ │ Pro ●   │ │     │ │   │
│          │  │  │ [tag]   │ │ [tag]   │ │ [tag]   │ │     │ │   │
│          │  │  │ [tag]   │ │ [tag]   │ │ [tag]   │ │     │ │   │
│          │  │  └─────────┘ └─────────┘ └─────────┘ └─────┘ │   │
│          │  │                                                 │   │
│          │  └─────────────────────────────────────────────────┘   │
│          │                                                        │
│          │  ┌─── POR CATEGORIA ──────────────────────────────┐   │
│          │  │                                                 │   │
│          │  │  HIDRÁULICA E SANEAMENTO                        │   │
│          │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐         │   │
│          │  │  │Card  │ │Card  │ │Card  │ │Card  │  [→]    │   │
│          │  │  └──────┘ └──────┘ └──────┘ └──────┘         │   │
│          │  │                                                 │   │
│          │  │  ELÉTRICA, ENERGIA E SPDA                       │   │
│          │  │  ┌──────┐ ┌──────┐ ┌──────┐ ┌──────┐         │   │
│          │  │  │Card  │ │Card  │ │Card  │ │Card  │  [→]    │   │
│          │  │  └──────┘ └──────┘ └──────┘ └──────┘         │   │
│          │  │                                                 │   │
│          │  └─────────────────────────────────────────────────┘   │
│          │                                                        │
└──────────────────────────────────────────────────────────────────┘
```

**Company Card (Card de Empresa):**
```
┌──────────────────────────────┐
│                              │
│  [Logo 48px]                 │  ← Logo quadrado, rounded-lg, object-cover
│                              │    fallback: iniciais da empresa em bg gold
│  Alpha Engenharia            │  ← Manrope 600, 15px
│  Pro ●                       │  ← Badge plano: pill gold (Pro) ou silver (Plus)
│                              │
│  [Hidráulica] [Saneamento]  │  ← Tags de categoria: rounded-full, bg muted
│  [Detecção de Vazamentos]   │    font 11px, max 3 visíveis + "+2"
│                              │
│  "Especialistas em detecção  │  ← Bio snippet: Manrope 400, 13px, muted-foreground
│   não destrutiva de..."      │    line-clamp-2
│                              │
│  📹 24 vídeos                │  ← Metadata: Manrope 400, 12px, muted-foreground
│                              │
└──────────────────────────────┘

DIMENSÕES:
  Width: flex (grid-cols-4 desktop, 3 tablet, 2 mobile)
  Padding: 16px
  Background: var(--card)
  Border: 1px solid var(--border)
  Border-radius: 12px
  Hover: border-color var(--primary)/50, shadow-sm
  Click: navega para /perfil/:empresaId
```

**IMPORTANTE — Empresa card NÃO mostra:**
- Telefone, email, site, WhatsApp, QR code (proibido pela governança de conteúdo, Cap. 7)
- Número de likes, seguidores, rating (métricas são privadas)
- Botão "Contratar" ou "Orçamento" (contato é via Connect Me, não marketplace)

### A.9 Saudação + Dashboard Adaptativo por Role (Refinamento)

**Home do Morador Base:**
```
  Bom dia, Maria
  
  AÇÕES: [Buscar Empresas] [Buscar Síndicos] [Ver Benefícios]
  
  ASSEMBLEIA DO MEU CONDOMÍNIO (se vinculada):
  [Card da assembleia ativa com link para votação]
  
  VÍDEOS RECOMENDADOS (com paywall de 25%)
  COMÉRCIO LOCAL (por CEP)
```

**Home do Síndico N3:**
```
  Bom dia, João
  
  AÇÕES: [Criar Assembleia] [Upload Vídeo] [Buscar Empresas] 
         [Transparência] [Connect Me] [Ver Benefícios]
  
  ASSEMBLEIAS ATIVAS (gerenciáveis)
  AVALIAÇÃO PENDENTE (se há avaliação aberta)
  VÍDEOS RECOMENDADOS
  COMÉRCIO LOCAL
```

**Home da Empresa Pro:**
```
  Bom dia, Alpha Engenharia
  
  AÇÕES: [Upload Vídeo] [Criar Curso] [Meus Vídeos] 
         [Connect Me] [Ver Currículos] [Ver Benefícios]
  
  MÉTRICAS PRIVADAS (card):
    - 12 vídeos publicados este mês (de 12)
    - Último vídeo: "Detecção de vazamentos" - há 3 dias
  
  VÍDEOS RECOMENDADOS
  COMÉRCIO LOCAL
```

### A.10 Newsletter/Subscribe Widget → Comunicados MasterSíndico

Inspirado no widget de newsletter da sidebar do Genaigurus. No Master Síndico, adaptamos para "Comunicados Oficiais".

**Widget na Home (abaixo dos action cards):**
```
┌──────────────────────────────────────────────────┐
│  📢 COMUNICADO OFICIAL                            │
│                                                    │
│  "Manutenção programada da plataforma no          │
│   dia 25/03/2026 das 02:00 às 06:00"             │
│                                                    │
│  Publicado por MasterSíndico • 14/03/2026         │
│                                                    │
│  [Ver detalhes →]                                  │
│                                                    │
└──────────────────────────────────────────────────┘

DESIGN:
  Background: var(--primary)/5 (gold tint sutil)
  Border-left: 4px solid var(--primary)
  Border-radius: 0 12px 12px 0
  Padding: 16px
  Dismissível: X no canto, marca como lido
```

### A.11 Feedback/Report Form (Referência: Genaigurus "Tell us the problem")

Para o botão de denúncia previsto no escopo (moderação reativa):

**Modal de Denúncia:**
```
┌──────────────────────────────────────────────────┐
│  ⚑ Denunciar Conteúdo                     [✕]  │
│                                                    │
│  O que há de errado com este conteúdo?            │
│                                                    │
│  ○ Conteúdo promocional/comercial                 │
│  ○ Dados de contato expostos                      │
│  ○ Conteúdo ofensivo ou inadequado                │
│  ○ Informação técnica incorreta ou perigosa       │
│  ○ Spam ou conteúdo repetitivo                    │
│  ○ Outro                                          │
│                                                    │
│  Descreva o problema (opcional):                  │
│  ┌──────────────────────────────────────────────┐│
│  │                                              ││
│  │  Textarea, max 300 chars                     ││
│  │                                     250/300  ││
│  └──────────────────────────────────────────────┘│
│                                                    │
│  [Cancelar]              [Enviar Denúncia]        │
│   ghost                   destructive variant      │
│                                                    │
└──────────────────────────────────────────────────┘

DESIGN:
  Modal: 480px max-width, rounded-xl, bg var(--card)
  Overlay: rgba(0,0,0,0.5) + backdrop-blur 4px
  Radio options: 44px height each (touch friendly)
  Botão enviar: variant destructive (vermelho), para comunicar gravidade
```

### A.12 Empty State com Ilustração Contextual

Inspirado nos empty states do VoiceLabs e Genaigurus que usam ilustrações sutis:

**Empty State Personalizado por Tela:**

| Tela | Ícone/Ilustração | Título | Descrição | CTA |
|------|-------------------|--------|-----------|-----|
| Home sem vídeos | `i-lucide-play-circle` 64px | "Nenhum vídeo ainda" | "Os vídeos aparecerão aqui conforme empresas e síndicos publicarem conteúdo." | — |
| Assembleias vazia | `i-lucide-clipboard` 64px | "Nenhuma assembleia" | "Crie sua primeira assembleia para organizar votações e pautas." | [Criar Assembleia →] |
| Busca sem resultado | `i-lucide-search-x` 64px | "Nenhum resultado" | "Tente ajustar seus filtros ou buscar por outros termos." | [Limpar Filtros] |
| Connect Me sem cota | `i-lucide-mail-x` 64px | "Cota esgotada" | "Você utilizou todos os contatos disponíveis este ano." | [Ver meu plano] |
| Timeline vazia | `i-lucide-timeline` 64px | "Histórico em branco" | "Registre suas primeiras ações para começar sua linha do tempo." | [Registrar Ação →] |
| Transparência vazia | `i-lucide-bar-chart` 64px | "Plano não definido" | "Defina seu plano de gestão para acompanhar a execução." | [Criar Plano →] |
| Votação encerrada | `i-lucide-check-circle` 64px | "Votação encerrada" | "O período de votação desta pauta já foi encerrado." | [Ver Resultado] |

**Design consistente de Empty State:**
```
  Container: flex-col items-center justify-center, min-h-[300px], text-center
  Ícone: 64px, cor var(--muted-foreground), opacity 0.5
  Título: Michroma 18px, var(--foreground), mt-4
  Descrição: Manrope 400, 14px, var(--muted-foreground), mt-2, max-w-[320px]
  CTA: botão primary (gold) se houver ação, mt-6
```

### A.13 Micro-Interações Refinadas (inspiradas nos detalhes VoiceLabs + Genaigurus)

**1. Hover em Video Card — Thumbnail Preview:**
- Ao fazer hover por >500ms no thumbnail, exibir uma prévia estática dos primeiros frames (se disponível via API Mux)
- Barra de progresso fina (2px) na base do thumbnail indicando duração
- Transição: opacity 0 → 1 em 200ms

**2. Like Animation:**
- Click no ❤️: o ícone escala de 1 → 1.3 → 1 com spring animation (200ms)
- Fill color: transição de outline (transparent) para filled `var(--primary)` (gold)
- Particles: 4-6 micro-partículas gold explodem do centro do ícone (CSS only, pseudo-elements)
- Se o user é o dono: o contador incrementa com animação de slideUp no número

**3. Vote Confirmation:**
- Ao confirmar voto: 
  - Botão faz loading spinner (500ms)
  - Toda a interface da pauta faz um sutil pulse verde
  - Confetti animation leve (3-4 partículas gold caindo, 1s)
  - Toast de sucesso aparece
  - Card da pauta transiciona para estado "Votado" com ícone ✅

**4. Scroll-triggered Section Headers:**
- Os títulos de seção na Home (VÍDEOS RECOMENDADOS, COMÉRCIO LOCAL) fazem fadeInUp quando entram no viewport
- Intersection Observer com threshold 0.1
- Cada seção tem animation-delay escalonado (+100ms)

**5. Sidebar Collapse Animation:**
- Colapsar: width 240px → 64px em 300ms ease-in-out
- Labels fazem fadeOut em 100ms (antes do width mudar)
- Ícones recentram-se no espaço restante
- Tooltip aparece ao hover nos ícones quando colapsada
- Widget de uso faz slideDown e desaparece
- Expandir: inverso, labels fazem fadeIn após width atingir 200px

**6. Page Transition (TanStack Router):**
- Saída da página atual: fadeOut 150ms
- Entrada da nova página: fadeInUp 300ms com delay de 50ms
- Skeleton loading aparece durante a transição se há data fetching

### A.14 Topbar Refinada com Upgrade Button (Referência: VoiceLabs)

O VoiceLabs mostra um botão "⚡ Upgrade" proeminente no topbar. Para o Master Síndico, isso é relevante para upsell.

**Topbar — Desktop (versão final refinada):**
```
┌──────────────────────────────────────────────────────────────────────┐
│                                                                      │
│  [□ Toggle]  [🛡️ MASTER SÍNDICO]                                   │
│               gold, Michroma                                         │
│                                                                      │
│              [🔍 Buscar síndicos, empresas, serviços...]            │
│               width: 400px, centered                                 │
│                                                                      │
│              [⚡ Upgrade]  [🌙]  [🔔 ●]  [👤 João ⌄]              │
│               gold outline   ghost  relative   dropdown              │
│               btn, visible                                           │
│               only if not                                            │
│               max plan                                               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

**Botão Upgrade:**
- Visível apenas se o usuário NÃO está no plano máximo da sua role
  - Morador Base → mostra "Upgrade" (para Morador Pg)
  - Síndico N1 → mostra (para N2)
  - Síndico N2 → mostra (para N3)
  - Síndico N3 → oculto
  - Empresa Plus → mostra (para Pro)
  - Empresa Pro → oculto
- Design: border 1px `var(--primary)`, bg transparent, text `var(--primary)`, rounded-full, px-4, h-8
- Ícone: ⚡ ou `i-lucide-zap`, 14px, mr-1
- Hover: bg `var(--primary)/10`
- Click: navega para `/configuracoes/plano`

**Notification Bell (🔔):**
- Badge vermelho (ponto 8px, bg `var(--destructive)`) quando há:
  - Assembleia com votação aberta
  - Avaliação pendente
  - Comunicado oficial não lido
- Click: popover com lista de notificações (max 5 recentes)
- Cada notificação: ícone + texto + timestamp + indicador de não-lido (dot gold)
- "Ver todas" no rodapé do popover → navega para página de notificações (futuro)

---

## ADDENDUM B — MAPEAMENTO VISUAL DE REFERÊNCIAS CONSOLIDADO

| Referência | O que extraímos | Onde aplicamos |
|---|---|---|
| **VoiceLabs Dashboard** | Sidebar com seções agrupadas (labels uppercase), widget "Credit Uses", greeting "Good Morning", action cards grid 2x3, botão "Upgrade" no topbar, breadcrumbs | Shell da aplicação (sidebar, topbar), Home page (greeting + action cards), navegação profunda |
| **VoiceLabs Voices/Explore** | Tabs horizontais + trending + curated collections por categoria + filter buttons | Catálogo de empresas, busca por categoria, organização de conteúdo por seção |
| **VoiceLabs Landing** | Dual CTA (outline + filled), hero typography, feature cards com descrição, footer multi-column | Telas de auth (split layout), paywall overlays, eventual landing page institucional |
| **VoiceLabs Mobile** | Grid 2x2 de action cards, hamburger menu, greeting adaptado | Home mobile, action cards responsivos |
| **Genaigurus Profile** | Tabs Posts/Badges, badge system com earned/unearned, profile header com cover | Perfil do síndico (tabs Vídeos/Status/Sobre), sistema de status Badge |
| **Genaigurus Badge Detail** | Progress bar para próximo nível, milestones listados, visual de medalha | Tela de Status do Síndico (Bronze→Diamante com progresso e critérios) |
| **Genaigurus Leaderboard** | Top 3 podium, lista ranqueada com pontos | **NÃO USAR publicamente** (viola princípio "sem ranking"). Adaptar apenas para painel admin MasterSíndico |
| **Genaigurus Search** | Browse by subjects (grid de pills grandes), search history com clock icon | Search page estado inicial (antes de digitar), categorias como grid clicável |
| **Genaigurus Video Player** | Landscape mode com timing bar, controles overlay, back/forward | Player mobile landscape, controles custom do player |
| **Genaigurus Home Web** | Sidebar com newsletter/subscribe, Latest blog carousel, Popular videos + articles, Categories pills | Home feed sections, widget de comunicados, organização de conteúdo por tipo |
| **Genaigurus Mobile Home** | Cards de vídeo com badge de destaque, scroll horizontal de featured, bottom feedback form | Home mobile sections, report/denúncia modal |
| **Genaigurus Feedback** | "Tell us the problem" form com seleção de tipo + textarea | Modal de denúncia de conteúdo |
| **Avalanche** (batch anterior) | Video cards grid 4 colunas, sidebar com folders, category icons row, video player com sidebar de relacionados | Home feed grid, sidebar colapsável, category navigation, video page layout |
| **SparkCause** (batch anterior) | Bottom navigation com botão central [+], dark theme gold/âmbar, category pills scroll | Bottom nav mobile, acentos gold, filter pills |
| **Desofy** (batch anterior) | Sidebar com icon+label, feed com composer e filter tabs, profile card | Sidebar items, profile layout |

---

## ADDENDUM C — DECISÕES DE DESIGN CONTROVERSAS E JUSTIFICATIVAS

### C.1 Por que NÃO usar leaderboard público

O Genaigurus mostra um leaderboard bonito com Top 3 podium. Seria tentador replicar para síndicos. **Mas não vamos fazer isso.** Razão: o Manual Técnico (Cap. 8.5) é explícito — "Não é ranking. Não compara síndicos entre si. Não gera competição pública." O sistema de Status é **individual** (cada síndico vs sua própria trajetória), não **comparativo**.

O leaderboard pode existir APENAS no Painel Admin MasterSíndico (Sprint 4) para analytics internas.

### C.2 Por que o Home é hybrid (dashboard + feed) e não puro YouTube

O YouTube é 100% feed de vídeos. O Master Síndico é uma plataforma de **governança** que também tem vídeos. Se a Home fosse só vídeos, o síndico entraria na plataforma e veria conteúdo educacional antes de ver que tem uma votação pendente ou uma assembleia aberta. Isso inverte a prioridade.

A solução: greeting + action cards + assembleias ativas NO TOPO (zona de governança), seguido de category icons + feed de vídeos ABAIXO (zona de conteúdo). O usuário resolve o operacional antes de consumir conteúdo.

### C.3 Por que não mostrar cadeado em funcionalidades bloqueadas

Muitas plataformas mostram features com cadeado (🔒) e texto "Upgrade para desbloquear". No Master Síndico, funcionalidades não liberadas **simplesmente não existem na interface**. Razão: o princípio "Acesso com Propósito" (Cap. 4.2.1 do Manual) diz que o acesso existe porque há um evento ou contexto. Se não há permissão, não há contexto, portanto não há razão para exibir.

Exceção: o paywall do vídeo (preview 25%) mostra overlay porque o conteúdo já foi parcialmente consumido — há contexto de continuidade.

### C.4 Por que a sidebar mostra widget de uso apenas acima de 50%

Se o widget estivesse sempre visível, seria ruído visual constante. Mostrar "2 de 12 vídeos usados" quando o usuário está longe do limite não acrescenta informação útil. O widget aparece apenas quando o uso passa de 50%, criando awareness progressiva sem alarme prematuro.

### C.5 Por que o botão "Upgrade" está no topbar e não na sidebar

Posicionamento no topbar (direita, ao lado de notificações) é mais consistente com o padrão SaaS (VoiceLabs, Notion, Figma). Na sidebar, competiria com a navegação e pareceria "ad". No topbar, é contextual — o usuário vê ao lado de suas ações, como uma opção natural.

---

*Addendum A atualizado em Fevereiro 2026.*

---

## ADDENDUM B — INSIGHTS DA REFERÊNCIA "FLAVOR" (Batch 3)

### Referências analisadas (10 imagens):
- **Flavor** — Plataforma de receitas e food content com dark mode premium.
  - Image 1 (d3e7a2): **Design System completo** — tipografia, cores, ícones, componentes, grid, espaçamento, botões, inputs, badges, cards
  - Image 2 (c7a98e): **Home Feed desktop** — sidebar + grid de cards + seções + filtros
  - Image 3 (516cd2): **Profile page** com cover + avatar + stats + tabs + grid de conteúdo + menu 3-dots
  - Image 4 (644cec): **Settings completas** — 6 sub-pages: Account, Notifications, Appearance, Privacy, About, cada uma detalhada
  - Image 5 (a63e68): **Detail page (Recipe)** — layout com hero image + sidebar info + conteúdo + ingredientes + reviews
  - Image 6 (3df7d9): **Mobile responsive completo** — home, profile, detail, bottom sheet, bottom nav, swipe patterns
  - Image 7 (7e4ef8): **Search page** — busca com filtros, categorias em grid, resultados com tabs
  - Image 8 (e9b1d1): **Saved/Bookmarks page** — lista de itens salvos com cards horizontais + filtros
  - Image 9 (a96d6d): **Onboarding/Auth flow** — stepper, formulários, welcome screen
  - Image 10 (71979e): **Mockup 3D** — apresentação do app em dispositivos reais (iPhone + MacBook)

### Por que esta referência é a mais aplicável até agora:

1. **Dark theme com accent warm** — usa tons quentes (laranja/âmbar dourado) sobre base escura, mapeamento quase 1:1 com nosso Navy+Gold
2. **Settings extremamente detalhadas** — 6 sub-páginas com toggles, forms, sessions, aparência. Melhor modelo para nossas configurações
3. **Profile com tabs, stats row, cover** — exatamente o que precisamos para perfis de Síndico/Empresa
4. **Card patterns sofisticados** — variações para grid, lista horizontal, carousel, com badges, bookmarks, timestamps
5. **É plataforma de conteúdo, não rede social** — mesmo posicionamento filosófico: "conteúdo com propósito, não palco"
6. **Design System documentado** — a Image 1 mostra tipografia, spacing, componentes, cores, botões, inputs, badges tudo detalhado
7. **Formulários e inputs polidos** — select com busca, textarea com counter, checkbox groups, radio cards
8. **Mobile patterns completos** — bottom sheet, pull-to-refresh, swipe actions, bottom nav, tudo exemplificado

---

## B.1 DESIGN SYSTEM BASE — LIÇÕES DO FLAVOR (Image 1: d3e7a2)

A Image 1 é um **design system sheet** completo com todos os componentes do Flavor. Vou extrair os patterns visuais mais relevantes.

### B.1.1 Hierarquia Tipográfica Documentada

O Flavor documenta a tipografia assim:
- Display: título grande, semibold, usado apenas em hero/landing
- Heading 1: para títulos de página
- Heading 2: para títulos de seção
- Heading 3: para subtítulos de card/section
- Body: para texto corrido
- Caption: para metadata, timestamps
- Overline: para labels de seção (uppercase, tracking wide)

**Confirmação para Master Síndico (já definido, agora reforçado):**

```
Display:     Michroma 32px, letter-spacing 0.05em   → Títulos de página hero
Heading 1:   Michroma 24px, letter-spacing 0.05em   → Títulos de seção principal
Heading 2:   Michroma 18px, letter-spacing 0.05em   → Subtítulos/Seções
Heading 3:   Michroma 14px, letter-spacing 0.05em   → Labels de cards/grupos
Body:        Manrope 16px, weight 400, line-height 1.6  → Texto corrido
Body Small:  Manrope 14px, weight 400, line-height 1.5  → Descrições secundárias
Caption:     Manrope 13px, weight 400, muted-foreground → Metadata, timestamps
Micro:       Manrope 12px, weight 500                   → Badges, pills, counters
Overline:    Michroma 10px, uppercase, tracking 0.12em   → Section labels
```

### B.1.2 Botões Documentados (Flavor Button System)

O design system do Flavor mostra variantes de botão com states (default, hover, active, disabled, loading). Reforçar nosso sistema com mais precisão:

```
PRIMARY BUTTON (Gold CTA):
  Default:    bg var(--primary), text var(--primary-foreground), rounded-md
  Hover:      brightness(1.1), shadow-sm
  Active:     brightness(0.95), scale(0.98)
  Disabled:   opacity 0.5, cursor not-allowed, no hover effect
  Loading:    text hidden, spinner 16px centralized, pointer-events none

  Sizes:
    sm:  h-8  px-3  text-13px  gap-1.5  icon-14px
    md:  h-10 px-4  text-14px  gap-2    icon-16px
    lg:  h-11 px-6  text-15px  gap-2.5  icon-18px
    xl:  h-12 px-8  text-16px  gap-3    icon-20px

SECONDARY BUTTON (Navy):
  Default:    bg var(--secondary), text var(--secondary-foreground)
  Hover:      bg var(--surface-variant)

OUTLINE BUTTON:
  Default:    bg transparent, border 1px var(--border), text var(--foreground)
  Hover:      bg var(--muted), border var(--muted-foreground)

GHOST BUTTON:
  Default:    bg transparent, text var(--foreground)
  Hover:      bg var(--muted)

DESTRUCTIVE BUTTON:
  Default:    bg var(--destructive), text var(--destructive-foreground)
  Hover:      brightness(1.1)

LINK BUTTON:
  Default:    bg transparent, text var(--primary), underline-offset-4
  Hover:      underline, brightness(1.1)

ICON BUTTON:
  Default:    w-9 h-9, rounded-md, bg transparent, icon var(--muted-foreground)
  Hover:      bg var(--muted), icon var(--foreground)
```

**Todos os botões compartilham:**
```css
.btn-base {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  font-family: 'Manrope', sans-serif;
  font-weight: 600;
  border-radius: var(--radius); /* 0.5rem */
  transition: all 200ms ease;
  cursor: pointer;
  white-space: nowrap;
  user-select: none;
}

.btn-base:focus-visible {
  outline: none;
  ring: 2px solid var(--ring); /* gold */
  ring-offset: 2px;
}
```

### B.1.3 Input Fields Documentados (Flavor)

O Flavor mostra inputs com estados: default, hover, focus, error, disabled, filled.

```
INPUT STATES:

Default:
  bg: var(--card)
  border: 1px solid var(--border)
  text: var(--foreground)
  placeholder: var(--muted-foreground)
  h: 44px (h-11)
  px: 12px
  rounded: var(--radius)
  font: Manrope 400 15px

Hover:
  border: 1px solid var(--muted-foreground)

Focus:
  border: 1px solid var(--ring) (gold)
  ring: 2px solid var(--ring)
  ring-offset: 1px

Error:
  border: 1px solid var(--destructive)
  ring: 2px solid var(--destructive)
  + mensagem de erro abaixo: Manrope 400 13px, var(--destructive)
  + ícone ⚠️ dentro do input à direita, 16px, var(--destructive)

Disabled:
  opacity: 0.5
  cursor: not-allowed
  bg: var(--muted)

Filled (com valor):
  text: var(--foreground), font-weight 500

Label:
  font: Manrope 500 14px
  color: var(--foreground)
  margin-bottom: 6px

Helper text:
  font: Manrope 400 13px
  color: var(--muted-foreground)
  margin-top: 4px
```

### B.1.4 Badge System (Flavor)

O Flavor documenta badges com variantes de cor e tamanho:

```
BADGE VARIANTS:

Default:     bg var(--muted), text var(--muted-foreground)
Primary:     bg var(--primary)/10, text var(--primary)
Secondary:   bg var(--secondary), text var(--secondary-foreground)
Destructive: bg var(--destructive)/10, text var(--destructive)
Success:     bg var(--success)/10, text var(--success)
Warning:     bg var(--warning)/10, text var(--warning)
Outline:     bg transparent, border 1px var(--border), text var(--foreground)

Tamanhos:
  sm:  h-5  px-1.5  text-11px  rounded-sm
  md:  h-6  px-2    text-12px  rounded-md
  lg:  h-7  px-2.5  text-13px  rounded-md

Todos:
  font-family: 'Manrope'
  font-weight: 600
  display: inline-flex
  align-items: center
  white-space: nowrap
```

**Aplicação específica no Master Síndico:**
```
"Empresa Pro"    → badge primary (gold)
"Síndico N2"     → badge primary (gold)
"Morador Base"   → badge default (muted)
"🟢 Aberta"      → badge success
"🔴 Encerrada"   → badge destructive
"⏳ Agendada"    → badge warning
"Novo"           → badge primary sm
"3/4 vídeos"     → badge outline
"Hidráulica"     → badge default sm (category tag)
"🥇 Ouro"       → badge custom com bg gradient gold
```

---

## B.2 SETTINGS PAGE — REDESIGN COMPLETO (Image 4: 644cec)

A Image 4 do Flavor é a referência mais detalhada de Settings que analisamos. Mostra 6 sub-páginas completas: Account, Notifications, Appearance, Privacy, About, e uma com sessões ativas. Vou reconstruir todo o padrão.

### B.2.1 Estrutura Geral das Settings

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  Configurações                                     │
│          │                                                    │
│          │  ┌─────────────┬──────────────────────────────┐   │
│          │  │ SETTINGS    │                              │   │
│          │  │ NAV (left)  │  SETTINGS CONTENT (right)    │   │
│          │  │             │                              │   │
│          │  │ [👤 Conta]  │  ┌────────────────────────┐  │   │
│          │  │ [🔔 Notif.] │  │  Conteúdo dinâmico    │  │   │
│          │  │ [🎨 Aparên.]│  │  baseado na tab ativa  │  │   │
│          │  │ [🔒 Privac.]│  │                        │  │   │
│          │  │ [🛡️ Segur.] │  │                        │  │   │
│          │  │ [📋 Plano]  │  │                        │  │   │
│          │  │ [ℹ️ Sobre]  │  │                        │  │   │
│          │  │             │  └────────────────────────┘  │   │
│          │  └─────────────┴──────────────────────────────┘   │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

**Settings Nav (Desktop):**
- Width: 220px fixo
- Cada item: ícone 20px + label Manrope 500 14px
- Item ativo: bg `var(--muted)`, text `var(--foreground)`, font-weight 600, rounded-md
- Item hover: bg `var(--muted)/50`
- Padding vertical: 10px, horizontal: 12px
- Gap entre items: 2px
- Border-right: nenhuma borda (dark mode), 1px `var(--border)` (light mode)

**Mobile:** Settings Nav vira lista vertical de cards navegáveis (iOS-style list). Cada item é um row com ícone + label + chevron-right. Ao clicar, navega para sub-página com back button `[← Configurações]`.

```css
/* Settings Nav Item */
.settings-nav-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 12px;
  border-radius: var(--radius);
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  cursor: pointer;
  transition: all 150ms ease;
}

.settings-nav-item:hover {
  background: oklch(from var(--muted) l c h / 0.5);
  color: var(--foreground);
}

.settings-nav-item.active {
  background: var(--muted);
  color: var(--foreground);
  font-weight: 600;
}
```

### B.2.2 Settings > Conta (Account)

O Flavor mostra Account settings com avatar editável, campos agrupados em "cards" com rows, e seção de perigo no final.

```
┌──────────────────────────────────────────────────────────┐
│  Conta                                                    │
│                                                          │
│  Gerencie suas informações pessoais e dados da conta.    │
│  (Manrope 14px, muted-foreground)                        │
│                                                          │
│  FOTO DE PERFIL ──────────────────────────────────────── │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                                                    │  │
│  │  ┌──────────┐                                      │  │
│  │  │          │  Foto de perfil                      │  │
│  │  │  AVATAR  │  JPG, PNG ou GIF. Máximo 5MB.       │  │
│  │  │  80px    │                                      │  │
│  │  │          │  [Alterar foto]   [Remover]          │  │
│  │  └──────────┘  (link gold)      (link muted)       │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  INFORMAÇÕES PESSOAIS ─────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Nome completo                                     │  │
│  │  João Carlos Silva                        [Editar] │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Email                                             │  │
│  │  joao.silva@email.com          [Alterar] [✓ Verif.]│  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Telefone                                          │  │
│  │  (11) 99876-5432                          [Editar] │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  CPF                                               │  │
│  │  •••.•••.456-78                              [🔒]  │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Bio                                               │  │
│  │  Síndico profissional há 8 anos...        [Editar] │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ENDEREÇO ────────────────────────────────────────────── │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  CEP                                               │  │
│  │  01310-100                                [Editar] │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Cidade / Estado                                   │  │
│  │  São Paulo / SP                           [Editar] │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  DADOS PROFISSIONAIS (condicional por role) ───────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Condomínios vinculados (Síndicos)                 │  │
│  │  3 condomínios ativos                     [Gerir →]│  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Categorias de atuação (Empresas, máx. 5)         │  │
│  │  Hidráulica, Elétrica, Manutenção Predial [Editar] │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  CNPJ (Empresas)                                   │  │
│  │  12.345.678/0001-90                          [🔒]  │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ZONA DE PERIGO ────────────────────────────────────── │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Desativar conta                                   │  │
│  │  Sua conta será desativada e seus dados            │  │
│  │  preservados por 90 dias.        [Desativar conta] │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**CSS para Settings Group (card agrupado com rows):**

```css
.settings-group {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  overflow: hidden;
  margin-bottom: 24px;
}

.settings-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  border-bottom: 1px solid var(--border);
  transition: background-color 150ms ease;
  min-height: 56px;
}

.settings-row:last-child {
  border-bottom: none;
}

.settings-row:hover {
  background: var(--muted);
}

.settings-row-label {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 500;
  color: var(--muted-foreground);
  margin-bottom: 2px;
}

.settings-row-value {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 500;
  color: var(--foreground);
}

.settings-row-action {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--primary);
  cursor: pointer;
  transition: opacity 150ms;
  flex-shrink: 0;
  margin-left: 16px;
}

.settings-row-action:hover {
  opacity: 0.8;
}
```

**Section Headers:**
```css
.settings-section-header {
  font-family: 'Michroma', sans-serif;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  padding: 0 4px;
  margin-top: 32px;
  margin-bottom: 12px;
}
```

**Avatar Upload area:**
- Avatar: 80px circular, border 3px solid `var(--border)`
- Se plano Pro: border 3px solid `var(--primary)` (gold ring)
- Hover no avatar: overlay com ícone câmera, bg rgba(0,0,0,0.5), rounded-full
- Botões lado a lado: "Alterar foto" (link gold) + "Remover" (link muted, aparece só quando tem foto)
- File picker: aceita jpg, png, gif, max 5MB
- Preview: mostra nova imagem no avatar imediatamente antes de salvar (local preview)
- Save: auto-save ao selecionar arquivo, com toast "Foto atualizada ✅"

**"Zona de Perigo":**
- Seção separada com: border 1px `var(--destructive)/20`
- Label da seção: cor `var(--destructive)`
- Botão "Desativar conta": variant destructive outline → border `var(--destructive)`, text `var(--destructive)`, bg transparent
- Hover: bg `var(--destructive)/10`
- Ao clicar: Dialog confirmação → input para digitar "DESATIVAR" → botão confirm destructive
- Nunca aparece para admin MasterSíndico

**Inline Edit Flow (ao clicar "Editar"):**
- O valor se transforma em input editável in-place
- Aparece: input preenchido + [Salvar] (gold) + [Cancelar] (ghost)
- Escape: cancela edição
- Enter: salva
- Transição: fade de text → input, 150ms
- Após salvar: toast success "Dados atualizados ✅"

### B.2.3 Settings > Notificações

O Flavor mostra toggles de notificação agrupados por tipo, com descrições. Cada toggle tem label + description + switch à direita.

```
┌──────────────────────────────────────────────────────────┐
│  Notificações                                            │
│                                                          │
│  Gerencie como e quando você recebe comunicados.         │
│                                                          │
│  EMAIL ────────────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Comunicados oficiais                     [====●] │  │
│  │  Receba comunicados do administrador               │  │
│  │  da plataforma Master Síndico.                     │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Atualizações de assembleia               [====●] │  │
│  │  Novas assembleias, votações abertas e             │  │
│  │  resultados do seu condomínio.                     │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Resumo semanal                           [●====] │  │
│  │  Relatório semanal com atividades do               │  │
│  │  seu condomínio.                                   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  PUSH (MOBILE) ────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Votações abertas                         [====●] │  │
│  │  Notificação quando votação for aberta.            │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Lives ao vivo                            [====●] │  │
│  │  Aviso quando uma transmissão iniciar.             │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Connect Me recebido                      [====●] │  │
│  │  Quando alguém enviar formulário para você.        │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ℹ️ A plataforma não envia notificações sobre           │
│  interações sociais (curtidas, comentários).             │
│  Apenas comunicados institucionais e ações relevantes.   │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Toggle Switch detalhado (Kobalte Toggle):**
```
ON state:
  Track: 44px × 24px, rounded-full
  Track bg: var(--primary) (gold)
  Thumb: 20px circle, bg white
  Thumb position: translateX(20px) — direita
  Transition: background-color 200ms cubic-bezier(0.4, 0, 0.2, 1),
              transform 200ms cubic-bezier(0.4, 0, 0.2, 1)

OFF state:
  Track bg: var(--muted)
  Thumb position: translateX(0) — esquerda

Disabled:
  Track: opacity 0.5
  Thumb: bg var(--muted-foreground)
  cursor: not-allowed

Focus:
  ring: 2px solid var(--ring) (gold)
  ring-offset: 2px
```

**Toggle Row Layout:**
```css
.toggle-row {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  padding: 16px 20px;
  gap: 16px;
}

.toggle-row-content {
  flex: 1;
  min-width: 0;
}

.toggle-row-label {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 2px;
}

.toggle-row-description {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  line-height: 1.4;
  max-width: 85%; /* para não competir com toggle */
}

.toggle-row-switch {
  flex-shrink: 0;
  margin-top: 2px; /* alinha com primeira linha do label */
}
```

### B.2.4 Settings > Aparência (Theme Selection)

O Flavor mostra seleção de tema com **cards preview visuais** — cada card contém um mini mockup da interface naquele tema. Este é o pattern mais elegante para seleção de tema.

```
┌──────────────────────────────────────────────────────────┐
│  Aparência                                               │
│                                                          │
│  Personalize a aparência da plataforma.                  │
│                                                          │
│  TEMA ─────────────────────────────────────────────────  │
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │  │
│  │ │░░░░░░░░░░│ │  │ │▓▓▓▓▓▓▓▓▓▓│ │  │ │░░░▓▓▓▓▓▓│ │  │
│  │ │░ ☀️     ░│ │  │ │▓ 🌙     ▓│ │  │ │░░░ 💻 ▓▓│ │  │
│  │ │░ Preview ░│ │  │ │▓ Preview ▓│ │  │ │░░░     ▓▓│ │  │
│  │ │░░░░░░░░░░│ │  │ │▓▓▓▓▓▓▓▓▓▓│ │  │ │░░░▓▓▓▓▓▓│ │  │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │  │
│  │              │  │              │  │              │  │
│  │   Claro      │  │   Escuro     │  │   Sistema    │  │
│  │   ○          │  │   ●          │  │   ○          │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  TAMANHO DO TEXTO ────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  A─────────────────●───────────────A              │  │
│  │  Pequeno          Normal          Grande           │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  IDIOMA ──────────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Idioma da interface                               │  │
│  │  Português (Brasil)                          [▼]   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Theme Preview Card specs:**
- Card container: 160px × 140px, rounded-lg, border 2px solid `var(--border)`, p-2, cursor pointer
- Card selecionado: border 2px solid `var(--primary)` (gold), shadow `0 0 0 1px var(--primary)`
- Preview interno: rounded-md, h-80px, overflow hidden
  - Light: bg `#f4f4f5`, mini sidebar `#e4e4e7`, mini content area `#fafafa`
  - Dark: bg `#1a1b26`, mini sidebar `#16171f`, mini content area `#1e1f2b`
  - Auto: split diagonal — metade light, metade dark
- Radio circle: 16px, border 2px, centralizado abaixo
  - Selected: border gold, inner dot 8px gold
  - Unselected: border muted-foreground
- Label: Manrope 500, 14px, text-center, mt-2
- Hover (não selecionado): border `var(--muted-foreground)`, shadow-sm
- Transição: border-color 150ms, box-shadow 150ms

**Font Size Slider:**
- Track: bg `var(--muted)`, h-2, rounded-full, full width
- Thumb: 20px circle, bg `var(--primary)` (gold), border 2px white, shadow-sm
- 3 stops: Pequeno (14px), Normal (16px default), Grande (18px)
- Labels abaixo: Manrope 400, 12px, muted-foreground
- "A" small e "A" large nos extremos para indicar visualmente
- Preview: textos da página ajustam live via CSS custom property `--font-size-base`

### B.2.5 Settings > Privacidade

```
┌──────────────────────────────────────────────────────────┐
│  Privacidade                                             │
│                                                          │
│  Controle quem pode ver suas informações.                │
│                                                          │
│  VISIBILIDADE DO PERFIL ──────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Perfil visível na busca                  [====●] │  │
│  │  Seu perfil aparece nos resultados conforme        │  │
│  │  as regras do seu plano.                           │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Mostrar última atividade                 [●====] │  │
│  │  Exibir quando você esteve ativo pela              │  │
│  │  última vez.                                       │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Exibir condomínios no perfil             [====●] │  │
│  │  Lista de condomínios vinculados visível            │  │
│  │  no seu perfil público.                            │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  DADOS E HISTÓRICO ──────────────────────────────────── │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Histórico de buscas                               │  │
│  │  23 pesquisas salvas           [Limpar Histórico]  │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Dados de uso                                      │  │
│  │  A plataforma coleta dados anônimos                │  │
│  │  para melhorar a experiência.     [Saiba mais →]   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ℹ️ Suas métricas de engajamento (curtidas,             │
│  visualizações) nunca são exibidas publicamente.         │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

### B.2.6 Settings > Segurança

```
┌──────────────────────────────────────────────────────────┐
│  Segurança                                               │
│                                                          │
│  SENHA ────────────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Alterar senha                                     │  │
│  │  Última alteração: 15/01/2026    [Alterar senha →] │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  SESSÕES ATIVAS ──────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  💻 Chrome • Windows 11                            │  │
│  │     São Paulo, SP • Ativo agora        [Este ✓]   │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  📱 Safari • iPhone 15                             │  │
│  │     São Paulo, SP • Há 2 horas    [Encerrar ✕]    │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  💻 Firefox • macOS                                │  │
│  │     Rio de Janeiro, RJ • Há 3 dias [Encerrar ✕]   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  [Encerrar todas as outras sessões]                      │
│  (botão destructive outline, full-width)                 │
│                                                          │
│  AUTENTICAÇÃO ────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Verificação em duas etapas               [●====] │  │
│  │  Camada extra de segurança via email.              │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Session Row spec:**
- Device icon: 💻 ou 📱, 20px, var(--muted-foreground)
- Texto principal: Manrope 500, 15px → "Browser • OS"
- Texto secundário: Manrope 400, 13px, muted-foreground → "Local • Tempo"
- Badge "Este ✓": pill, bg `var(--success)/10`, text `var(--success)`, rounded-full, font-size 12px
- Botão "Encerrar ✕": text `var(--destructive)`, hover bg `var(--destructive)/10`, rounded-md, px-2 py-1
- Ao encerrar: Dialog confirmação → remover com slide-out left + fade

### B.2.7 Settings > Plano (Específico Master Síndico)

```
┌──────────────────────────────────────────────────────────┐
│  Seu Plano                                               │
│                                                          │
│  PLANO ATUAL ─────────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │                                                    │  │
│  │  🛡️ Síndico Pro                                   │  │
│  │  (Michroma 20px, var(--primary))                   │  │
│  │                                                    │  │
│  │  Status: 🥇 Ouro                                  │  │
│  │  Membro desde: Janeiro 2025                        │  │
│  │  Próxima cobrança: R$ 89,90 em 15/03/2026         │  │
│  │                                                    │  │
│  │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │  │
│  │                                                    │  │
│  │  Recursos do seu plano:                            │  │
│  │                                                    │  │
│  │  ✅ Perfil institucional público                   │  │
│  │  ✅ Publicação de até 4 vídeos/mês                │  │
│  │  ✅ Connect Me ativo                               │  │
│  │  ✅ Cursos certificados                            │  │
│  │  ✅ Ferramentas de gestão completas                │  │
│  │  ✅ Link de evento compartilhável                  │  │
│  │  ✅ Avaliação objetiva                             │  │
│  │  ✅ Exportação PDF timeline                        │  │
│  │                                                    │  │
│  │  [Gerenciar Assinatura]    [Ver Todos os Planos]  │  │
│  │  (outline)                  (gold filled)          │  │
│  │                                                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  USO DO PERÍODO ──────────────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Vídeos publicados                                 │  │
│  │  ████████████████████████░░░░░░░░░ 3 de 4          │  │
│  │  Renova em: 01/03/2026                             │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Connect Me enviados                               │  │
│  │  ██████████░░░░░░░░░░░░░░░░░░░░░░ 2 de 8/ano     │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  Vídeo institucional                               │  │
│  │  Próxima atualização: 15/04/2026 (56 dias)        │  │
│  │  ██████████████████░░░░░░░░░░░░░░ 38% do ciclo    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  HISTÓRICO DE PAGAMENTOS ─────────────────────────────  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  15/02/2026  R$ 89,90   Pago ✅   [Recibo ↓]     │  │
│  │  15/01/2026  R$ 89,90   Pago ✅   [Recibo ↓]     │  │
│  │  15/12/2025  R$ 89,90   Pago ✅   [Recibo ↓]     │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Usage Progress Bars:**
```css
.usage-progress {
  width: 100%;
  height: 8px;
  background: var(--muted);
  border-radius: 4px;
  overflow: hidden;
  margin: 8px 0;
}

.usage-progress-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 600ms ease-out;
}

/* Cor dinâmica baseada em porcentagem */
.usage-progress-fill[data-usage="low"]    { background: var(--primary); }   /* <50% → gold */
.usage-progress-fill[data-usage="medium"] { background: var(--warning); }   /* 50-80% → âmbar */
.usage-progress-fill[data-usage="high"]   { background: var(--destructive); } /* >80% → vermelho */
.usage-progress-fill[data-usage="full"]   { background: var(--destructive); animation: pulse 1s infinite; }
```

---

## B.3 HOME FEED — CARDS REFINADOS (Image 2: c7a98e)

### B.3.1 Card de Conteúdo com Bookmark (Flavor Recipe Card)

O Flavor mostra cards de receita com: thumbnail grande, ícone de bookmark overlay, título, metadata, e tags de categoria. Adaptar:

```
┌──────────────────────────────────┐
│                                  │
│      THUMBNAIL (16:9)            │
│                                  │
│  [🔖]                   [03:45] │  ← bookmark top-left
│                                  │     duration bottom-right
│                                  │
├──────────────────────────────────┤
│                                  │
│  Título do Vídeo Instrucional    │  ← Manrope 600, 15px
│  sobre Manutenção de Fachadas    │    line-clamp-2
│                                  │
│  [Avatar 28px] HidraFix Eng.     │  ← Manrope 400, 13px, muted
│                há 3 dias         │
│                                  │
│  [Hidráulica] [Manutenção]       │  ← category pills tiny
│                                  │
└──────────────────────────────────┘
```

**Bookmark Button (overlay na thumbnail):**
- Posição: absolute, top-8px, left-8px (ou right-8px — decisão de design)
- Container: 32px circle, bg `rgba(0,0,0,0.4)`, backdrop-filter `blur(4px)`
- Ícone: bookmark outline, 16px, white
- Desktop: visível apenas em hover no card (opacity 0 → 1, transition 200ms)
- Mobile: sempre visível (sem hover em touch)
- Estado salvo: ícone bookmark filled, cor `var(--primary)` (gold), bg `rgba(0,0,0,0.6)`
- Hover no botão: scale(1.1)
- Click: toggles save state + toast "Vídeo salvo" ou "Removido dos salvos"
- Não usa animação de "coração explodindo" — o Master Síndico é sóbrio

**Duration Badge (overlay na thumbnail):**
- Posição: absolute, bottom-8px, right-8px
- Container: px-1.5 py-0.5, bg `rgba(0,0,0,0.75)`, rounded-sm
- Texto: Manrope 600, 12px, white

**Category Pills no Card:**
- Tags minúsculas na parte inferior do card body
- bg `var(--muted)`, text `var(--muted-foreground)`, font-size 11px
- rounded-full, px-2 py-0.5, font-weight 500
- Máximo 2 visíveis + "+N" se houver mais
- "+N" badge: bg `var(--muted)`, text `var(--muted-foreground)`, font-weight 600
- Clicar numa tag: navega para `/buscar?categoria=hidraulica`
- Gap: 4px entre tags

### B.3.2 Card Horizontal (Flavor "Saved" List View)

O Flavor mostra itens em lista com thumbnail lateral pequena. Perfeito para: resultados de busca (modo lista), vídeos salvos, vídeos relacionados no player, pautas de assembleia.

```
┌──────────────────────────────────────────────────────────┐
│  ┌──────────┐                                            │
│  │          │  Manutenção Preventiva de Fachadas:        │
│  │  THUMB   │  Guia Completo para Síndicos               │
│  │  120×80  │                                            │
│  │  rounded │  HidraFix Engenharia • há 5 dias           │
│  │  [02:30] │  [Hidráulica] [Fachada]                    │
│  └──────────┘                                       [⋮]  │
└──────────────────────────────────────────────────────────┘
```

**Specs:**
- Container: flex, items-center, p-2, rounded-lg, hover bg `var(--muted)`, cursor pointer, transition 150ms
- Thumbnail: 120×80px fixed, rounded-md, object-cover, flex-shrink-0
- Duration: absolute bottom-4px right-4px do thumb, px-1 py-0.5, bg black/75, rounded-sm, 11px white
- Content: flex-1, pl-3, min-width-0 (para truncation funcionar)
- Título: Manrope 600, 14px, line-clamp-2, `var(--foreground)`
- Meta: Manrope 400, 12px, `var(--muted-foreground)`, mt-1
- Tags: mt-1.5, flex, gap-1
- Menu ⋮: 24px icon, `var(--muted-foreground)`, flex-shrink-0, ml-2
  - Desktop: visible on hover
  - Mobile: always visible
- Gap entre cards em lista: 2px (compacto) ou 8px (espaçoso)

### B.3.3 Página de Itens Salvos (/salvos) — NOVA ROTA

Baseado no Flavor "Saved/Bookmarks" pattern:

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Itens Salvos                          12 itens│
│          │                                                │
│          │  [Todos] [Vídeos] [Empresas] [Assembleias]    │
│          │   ═════                                        │
│          │                                                │
│          │  ┌─ Card Horizontal ──────────────────────┐   │
│          │  │ [Thumb] Título do vídeo salvo...       │   │
│          │  │         Canal • 3 dias • [tags]   [⋮]  │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
│          │  ┌─ Card Horizontal ──────────────────────┐   │
│          │  │ [Thumb] Outro vídeo salvo...           │   │
│          │  │         Canal • 1 semana • [tags] [⋮]  │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
│          │  ┌─ Card Horizontal ──────────────────────┐   │
│          │  │ [Logo]  Empresa XYZ (Perfil salvo)     │   │
│          │  │         Empresa Pro • Hidráulica  [⋮]  │   │
│          │  └────────────────────────────────────────┘   │
│          │                                                │
│          │  ──────────────────────────────────────────── │
│          │  EMPTY STATE (se nenhum item salvo):           │
│          │                                                │
│          │        🔖 (64px, muted)                       │
│          │  Nenhum item salvo ainda                       │
│          │  Salve vídeos e perfis para                    │
│          │  acessá-los rapidamente.                       │
│          │  [Explorar vídeos →]                           │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Sidebar item novo:**
```
  🔖 Salvos              ← adicionar à navegação principal
```

**Menu ⋮ (dropdown Kobalte):**
```
┌──────────────────────┐
│  ▶️ Assistir / Ver    │  ← navega para o conteúdo
│  📋 Copiar link      │  ← copia URL para clipboard
│  🗑 Remover dos salvos│  ← text destructive, remove com undo toast
└──────────────────────┘
```

**Undo Toast (ao remover):**
```
┌────────────────────────────────────────────┐
│  🗑 "Item removido dos salvos"   [Desfazer]│  ← link gold
└────────────────────────────────────────────┘
  auto-dismiss em 5s, se clicar "Desfazer" restaura o item
```

---

## B.4 PROFILE PAGE REFINADA (Image 3: 516cd2)

### B.4.1 Profile Header com Cover Image (Flavor)

O Flavor mostra um profile com cover image grande, avatar sobreposto, e informações abaixo. Este é o padrão Instagram que já definimos, mas o Flavor adiciona detalhes importantes:

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  ████████████████████████████████████████████████████████│
│  ██                                                    ██│
│  ██              COVER IMAGE                           ██│ ← 240px height desktop
│  ██              (banner ou gradient navy)              ██│   180px mobile
│  ██                                                    ██│   gradient default se
│  ████████████████████████████████████████████████████████│   sem imagem
│                                                          │
│           ┌──────────────┐                               │
│           │              │  ← Avatar 96px                │
│           │    AVATAR    │     border 4px var(--card)     │
│           │              │     margin-top -48px           │
│           └──────────────┘     (sobrepõe o cover)        │
│                                                          │
│           João Carlos Silva                              │ Michroma 20px
│           @joaosilva                                     │ Manrope 14px muted
│           Síndico Pro • 🥇 Ouro                         │ badge primary + badge gold
│                                                          │
│           Síndico profissional há 8 anos,                │ Manrope 14px, line-clamp-2
│           especializado em condomínios residenciais...   │ muted-foreground
│                                                          │
│  ┌────────┐  ┌────────┐  ┌────────┐                    │
│  │   3    │  │  47    │  │   5    │                    │ Stats row
│  │ Cond.  │  │ Vídeos │  │Módulos │                    │ separados por border-right
│  └────────┘  └────────┘  └────────┘                    │
│                                                          │
│  [Editar Perfil]  [Compartilhar]  [⋮]                   │ Buttons row
│                                                          │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                                          │
│  [📹 Vídeos] [📊 Gestão] [🏅 Status] [ℹ️ Sobre]       │ Tabs com ícones
│   ═══════════                                            │
│                                                          │
│  ── Conteúdo da tab ativa ────────────────────────────  │
│                                                          │
│  [⊞ Grid]  [≡ Lista]              Ordenar: [Recentes ▼] │ View toggle + sort
│                                                          │
│  [Card] [Card] [Card] [Card]                            │
│  [Card] [Card] [Card] [Card]                            │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Cover Image specs:**
- Height: 240px desktop, 180px tablet, 160px mobile
- Default (sem upload): gradient `linear-gradient(135deg, var(--secondary) 0%, var(--surface-variant) 100%)`
- Com upload: object-cover, full-width
- Overlay sutil: `linear-gradient(to bottom, transparent 60%, var(--background) 100%)` — fade no bottom para fundir com o content
- Botão "Editar cover" (owner only): absolute top-12px right-12px, icon-button, bg black/40, backdrop-blur, ícone câmera branco
- Hover: bg black/60

**Avatar overlap specs:**
- Tamanho: 96px desktop, 80px mobile
- Border: 4px solid `var(--card)` (cria gap visual entre cover e avatar)
- Border adicional (Pro): ring 2px solid `var(--primary)` ao redor
- Position: margin-top -48px (metade do avatar, faz overlap no cover)
- Forma: rounded-full

**Stats Row:**
- 3 colunas iguais, text-center
- Separadas por border-right 1px `var(--border)` (exceto última)
- Número: Manrope 700, 18px, `var(--foreground)`
- Label: Manrope 400, 12px, `var(--muted-foreground)`
- Para Síndico: Condomínios | Vídeos (>70%) | Módulos (Cap. 8 Manual)
- Para Empresa: Vídeos | Cursos | Categoria principal
- Para Morador: Condomínio | Contatos restantes | N/A
- **NUNCA**: likes, followers, comments — lembrando: "engajamento não é palco"

**View Toggle (Grid/Lista):**
```
┌──────┬──────┐
│  ⊞   │  ≡  │
│ grid │ list │
└──────┴──────┘
```
- Container: inline-flex, border 1px `var(--border)`, rounded-md, overflow hidden
- Button ativo: bg `var(--muted)`, icon `var(--foreground)`
- Button inativo: bg transparent, icon `var(--muted-foreground)`
- Cada button: 36×32px, flex items-center justify-center
- Transição do conteúdo ao trocar view: fade-out 100ms → layout swap → fade-in 150ms

**Profile Tab com Animated Underline:**
```jsx
// Conceito: div absoluta que desliza acompanhando a tab ativa
<div class="tab-container relative">
  <button>Vídeos</button>
  <button>Gestão</button>
  <button>Status</button>
  <button>Sobre</button>
  
  <!-- Underline que desliza -->
  <div class="tab-indicator absolute bottom-0 h-0.5 bg-primary rounded-full"
       style="left: {activeTabLeft}px; width: {activeTabWidth}px; transition: all 250ms ease" />
</div>
```

---

## B.5 DETAIL PAGE — LAYOUT COM INFO SIDEBAR (Image 5: a63e68)

### B.5.1 Assembleia Detail (Adaptado do Flavor Recipe Detail)

O Flavor mostra um detail page com: hero image no topo, seção de ingredientes em sidebar, instruções passo-a-passo no content principal, e reviews no final. Mapear para Assembleia:

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ← Voltar para Assembleias                         │
│          │                                                    │
│          │  ┌─────────────────────────┬───────────────────┐  │
│          │  │                         │                   │  │
│          │  │  CONTEÚDO (flex-1)      │  INFO (320px)     │  │
│          │  │                         │                   │  │
│          │  │  Assembleia Ordinária   │  DETALHES         │  │
│          │  │  2026/1                 │                   │  │
│          │  │  (Michroma 24px)        │  Status:          │  │
│          │  │                         │  ┌──────────┐    │  │
│          │  │  Edital de convocação   │  │🟢 Aberta │    │  │
│          │  │  para assembleia geral  │  └──────────┘    │  │
│          │  │  ordinária...           │                   │  │
│          │  │  (Manrope 15px)         │  Data:           │  │
│          │  │                         │  15/03/2026      │  │
│          │  │  📹 Vídeo do Síndico:  │  19:00           │  │
│          │  │  ┌──────────────────┐  │                   │  │
│          │  │  │  [Player embed]  │  │  Votação até:    │  │
│          │  │  │  aspect-16/9     │  │  20/03/2026      │  │
│          │  │  │  max-w-full      │  │  23:59           │  │
│          │  │  └──────────────────┘  │                   │  │
│          │  │                         │  Pautas: 5       │  │
│          │  │  ━━━━━━━━━━━━━━━━━━━━  │                   │  │
│          │  │                         │  Participação:   │  │
│          │  │  PAUTAS                 │  ████████░░ 78%  │  │
│          │  │                         │                   │  │
│          │  │  ▶ Pauta 1: Contrat...│  ──────────────── │  │
│          │  │  ▼ Pauta 2: Orçament..│                   │  │
│          │  │    [conteúdo expandido] │  AÇÕES           │  │
│          │  │    descrição, vídeos,   │                   │  │
│          │  │    fornecedores...      │  [🗳️ Votar →]   │  │
│          │  │  ▶ Pauta 3: Segurança.│  [📄 Edital PDF] │  │
│          │  │  ▶ Pauta 4: Reforma...│  [🔗 Link temp.] │  │
│          │  │  ▶ Pauta 5: AOB       │  [↗ Compartilhar]│  │
│          │  │                         │                   │  │
│          │  └─────────────────────────┴───────────────────┘  │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

**Info Sidebar (direita):**
- Width: 320px desktop
- bg: `var(--card)`, border-left 1px `var(--border)`, p-6
- position: sticky, top: 80px (fica fixo ao scrollar conteúdo)
- Seções separadas por border-top 1px `var(--border)`, pt-4, mt-4
- Labels: Manrope 500, 13px, muted-foreground
- Valores: Manrope 600, 15px, foreground
- CTAs: botões full-width, stacked, gap-2
- CTA primário (Votar): bg `var(--primary)` (gold), icon left, font-weight 600
- CTAs secundários: variant outline

**Mobile:** Info sidebar colapsa para um card sticky no topo com expansão:
```
┌─────────────────────────────────────┐
│  Assembleia Ordinária 2026/1        │
│  🟢 Aberta • 15/03/2026 • 5 pautas │
│  [Ver detalhes ▼]      [🗳️ Votar] │
└─────────────────────────────────────┘
```
- Sticky top: 56px (abaixo do header mobile)
- Expandir: mostra todos os detalhes + ações
- Shadow-md na borda inferior quando sticky

### B.5.2 Accordion de Pautas (Flavor Step Pattern)

O Flavor mostra passos de receita em formato expandível. Adaptar para pautas:

```
COLLAPSED:
┌────────────────────────────────────────────────────────┐
│  ▶ Pauta 1                                        [▼] │
│    Contratação empresa de manutenção de fachada        │
│    (Manrope 400, 13px, muted-foreground, line-clamp-1) │
└────────────────────────────────────────────────────────┘

EXPANDED:
┌────────────────────────────────────────────────────────┐
│  ▼ Pauta 2                                        [▲] │
│    Orçamento para impermeabilização                    │
│                                                        │
│  ────────────────────────────────────────────────────  │
│                                                        │
│  Descrição:                                            │
│  O condomínio necessita de serviço de                  │
│  impermeabilização na cobertura do bloco A.            │
│  Foram solicitados 3 orçamentos de empresas...         │
│                                                        │
│  📹 Vídeo explicativo do Síndico:                     │
│  ┌─────────────────────────────────────────────────┐  │
│  │  [Mini Player — 16:9 aspect ratio]              │  │
│  │  aspect-ratio: 16/9, max-height: 240px          │  │
│  │  rounded-lg, overflow hidden                    │  │
│  └─────────────────────────────────────────────────┘  │
│                                                        │
│  📎 Propostas de fornecedores:                        │
│                                                        │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐        │
│  │ [Thumb]    │ │ [Thumb]    │ │ [Thumb]    │        │
│  │ ImperTech  │ │ AquaSeal   │ │ TectPro    │        │
│  │ [▶ Vídeo] │ │ [▶ Vídeo] │ │ [▶ Vídeo] │        │
│  └────────────┘ └────────────┘ └────────────┘        │
│                                                        │
│  Tem votação? [🗳️ Votar nesta pauta →]               │
│                                                        │
└────────────────────────────────────────────────────────┘
```

**Accordion specs:**
```css
.accordion-trigger {
  width: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px;
  cursor: pointer;
  transition: background-color 150ms;
}

.accordion-trigger:hover {
  background: var(--muted);
}

.accordion-trigger[data-state="open"] {
  border-bottom: 1px solid var(--border);
}

.accordion-chevron {
  width: 20px;
  height: 20px;
  color: var(--muted-foreground);
  transition: transform 200ms ease;
}

.accordion-trigger[data-state="open"] .accordion-chevron {
  transform: rotate(180deg);
}

.accordion-content {
  padding: 16px 20px 20px;
  animation: slideDown 200ms ease;
}

@keyframes slideDown {
  from { opacity: 0; max-height: 0; }
  to { opacity: 1; max-height: 2000px; }
}

.accordion-item {
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  margin-bottom: 8px;
  overflow: hidden;
}

.accordion-item[data-state="open"] {
  border-color: oklch(from var(--primary) l c h / 0.3);
}
```

**Pauta Number Badge:**
- "Pauta 1" → pill, bg `var(--primary)/10`, text `var(--primary)`, font-size 12px, font-weight 600, rounded-full, mr-2

**Fornecedor Mini Cards:**
- Grid 3 cols (desktop), 2 cols (tablet), horizontal scroll (mobile)
- Cada card: 140px width, rounded-lg, border 1px `var(--border)`, overflow hidden
- Thumb: aspect-16/9, object-cover
- Nome empresa: Manrope 600, 13px, p-2
- Botão "▶ Vídeo": link gold, 12px, p-2

---

## B.6 SEARCH PAGE REFINADA (Image 7: 7e4ef8)

### B.6.1 Search Desktop com Categorias em Grid

O Flavor mostra uma search page com campo de busca proeminente, categorias em grid de cards, e resultados filtráveis. Adaptar:

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Buscar                                         │
│          │                                                │
│          │  ┌──────────────────────────────────────────┐  │
│          │  │  🔍  Buscar empresas, síndicos, vídeos...│  │
│          │  └──────────────────────────────────────────┘  │
│          │                                                │
│          │  BUSCAR POR CATEGORIA ────────────────────── │
│          │                                                │
│          │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────┐ │
│          │  │  🔧     │ │  ⚡    │ │  🔒     │ │ ⚖️ │ │
│          │  │Hidrául. │ │Elétrica │ │Seguranç.│ │Juríd│ │
│          │  └─────────┘ └─────────┘ └─────────┘ └────┘ │
│          │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌────┐ │
│          │  │  🏗️     │ │  🛡️    │ │  📊     │ │ 🏠 │ │
│          │  │Engenhar.│ │Seguros  │ │Contabil.│ │Refor│ │
│          │  └─────────┘ └─────────┘ └─────────┘ └────┘ │
│          │                                                │
│          │  [Ver todas as 31 categorias →]                │
│          │                                                │
│          │  PESQUISAS RECENTES ──────────────────────── │
│          │                                                │
│          │  🕐 empresa hidráulica             [✕]        │
│          │  🕐 síndico profissional sp        [✕]        │
│          │  🕐 reforma fachada                [✕]        │
│          │                                                │
│          │  [Limpar histórico]                            │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Category Grid Cards:**
- Grid: 4 cols desktop, 3 tablet, 2 mobile
- Cada card: aspect-square (ou 1.2), min-height 80px, rounded-lg
- bg: `var(--card)`, border 1px `var(--border)`
- Ícone: 28px, centralizado, `var(--muted-foreground)`
- Label: Manrope 500, 13px, text-center, mt-2, line-clamp-1
- Hover: border `var(--primary)`, bg `var(--primary)/5`, ícone `var(--primary)`
- Click: navega para `/buscar?categoria=hidraulica`
- Transição: border-color 150ms, background 150ms

**Search Input (proeminente):**
- Height: 48px (maior que inputs normais)
- Ícone 🔍: 20px, `var(--muted-foreground)`, pl-4
- Placeholder: "Buscar empresas, síndicos, vídeos..."
- Font: Manrope 400, 16px
- Focus: ring-2 gold, border gold
- Clear button "✕": aparece quando tem texto, absolute right-12px

**Após pesquisar — resultados com tabs:**
```
  [Todos]  [Empresas]  [Síndicos]  [Vídeos]  [Comércio]
   ═══════

  3 empresas encontradas

  ┌────────────────────────────────────────────────────┐
  │  [Logo] Hidra Solutions                            │
  │         Empresa Pro • ⭐ 4.8                       │
  │         #hidráulica #detecção-vazamentos            │
  │         São Paulo, SP                [Ver Perfil →]│
  └────────────────────────────────────────────────────┘
```

---

## B.7 MOBILE PATTERNS (Image 6: 3df7d9)

### B.7.1 Bottom Sheet

O Flavor usa bottom sheets extensivamente no mobile. Implementar como padrão para: menus contextuais, filtros, confirmações, forms curtos.

```
┌─────────────────────────────────────┐
│                                     │  ← Overlay: bg rgba(0,0,0,0.5)
│                                     │     tap para fechar
│                                     │
│                                     │
│  ┌─────────────────────────────┐    │
│  │  ━━━━━━━━ (handle)         │    │  ← 40×4px, bg muted/40, mx-auto
│  │                             │    │     mt-8px, mb-16px
│  │  Opções do vídeo            │    │  ← Michroma 14px, px-4
│  │                             │    │
│  │  ┌─────────────────────┐   │    │
│  │  │ ▶️  Assistir        │   │    │  ← h-48px, flex items-center
│  │  ├─────────────────────┤   │    │     gap-3, px-4
│  │  │ 🔖  Salvar          │   │    │     hover bg muted
│  │  ├─────────────────────┤   │    │     icon 20px muted-foreground
│  │  │ 📋  Copiar link     │   │    │     label Manrope 500 15px
│  │  ├─────────────────────┤   │    │
│  │  │ ↗️  Compartilhar    │   │    │
│  │  ├── separator ────────┤   │    │  ← border-top 1px border, my-1
│  │  │ ⚑  Denunciar       │   │    │  ← text destructive
│  │  └─────────────────────┘   │    │
│  │                             │    │
│  │  [Cancelar]                 │    │  ← ghost button, full-width
│  │                             │    │     text muted-foreground
│  └─────────────────────────────┘    │
│           (safe-area-bottom)        │
└─────────────────────────────────────┘
```

**Bottom Sheet CSS/Animation:**
```css
.bottom-sheet-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  z-index: 50;
  animation: fadeIn 200ms ease;
}

.bottom-sheet-content {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: var(--card);
  border-radius: 1rem 1rem 0 0;
  padding-bottom: env(safe-area-inset-bottom, 16px);
  z-index: 51;
  animation: slideUp 300ms cubic-bezier(0.32, 0.72, 0, 1);
  max-height: 85vh;
  overflow-y: auto;
}

@keyframes slideUp {
  from { transform: translateY(100%); }
  to { transform: translateY(0); }
}

.bottom-sheet-handle {
  width: 40px;
  height: 4px;
  background: var(--muted-foreground);
  opacity: 0.3;
  border-radius: 2px;
  margin: 8px auto 16px;
}
```

**Dismiss:**
- Tap no overlay
- Swipe down no handle (>50px threshold)
- Botão "Cancelar"
- Animation out: slideDown 200ms

### B.7.2 Pull-to-Refresh

```
IDLE:       (hidden)
PULLING:    ↓ seta + "Puxe para atualizar" (12px muted)
THRESHOLD:  ↺ ícone + "Solte para atualizar" (12px primary)
LOADING:    ⟳ spinner gold 24px + "Atualizando..." (12px muted)
DONE:       ✓ check gold, fade-out 500ms
```

- Threshold: 60px de pull distance
- Indicator: acima do primeiro item, centralizado
- Spinner: SVG circle, stroke `var(--primary)`, animate rotate 1s linear infinite
- Bounce-back: spring animation 200ms ao soltar

### B.7.3 Swipe Actions em Cards de Lista

```
  ←── SWIPE LEFT (remover/denunciar) ──→
  ┌──────────────────────────────────────────┐
  │  [Card content visível]    🗑 [REMOVER]  │
  │                            bg destructive │
  └──────────────────────────────────────────┘

  ←── SWIPE RIGHT (salvar) ──→
  ┌──────────────────────────────────────────┐
  │ 🔖 [SALVAR]       [Card content visível] │
  │ bg primary (gold)                         │
  └──────────────────────────────────────────┘
```

- Swipe threshold: 80px para confirmar ação
- Snap-back se < threshold
- Background revela: cor sólida + ícone + label centralizado
- Haptic feedback: navigator.vibrate(10) no threshold

---

## B.8 ONBOARDING REFINADO (Image 9: a96d6d)

### B.8.1 Stepper Visual Melhorado

```
     ●━━━━━━━━●━━━━━━━━○━━━━━━━━○
     1        2        3        4
   Dados    Email    Senha    Pronto
```

**Step Indicators:**
```
Completed:  28px circle
            bg var(--primary), icon ✓ branco 14px
            shadow: 0 0 0 4px oklch(from var(--primary) l c h / 0.15)

Current:    28px circle
            bg var(--primary), border 3px solid var(--card)
            box-shadow: 0 0 0 2px var(--primary)
            animation: pulse-ring 2s ease infinite
            
Upcoming:   28px circle
            bg var(--muted), border 2px var(--border)

Connector (done):    h-0.5, bg var(--primary)
Connector (pending): h-0.5, bg var(--border)
```

```css
@keyframes pulse-ring {
  0%, 100% { box-shadow: 0 0 0 2px var(--primary); }
  50% { box-shadow: 0 0 0 6px oklch(from var(--primary) l c h / 0.15); }
}
```

### B.8.2 Welcome Screen Pós-Cadastro (/bem-vindo)

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│                    🛡️ (Brasão 80px animado)              │
│                       fade-in + scale                    │
│                                                          │
│              Bem-vindo ao Master Síndico                 │  Michroma 24px
│                                                          │
│              Sua conta Síndico Pro está ativa.            │  Manrope 16px muted
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  Próximos passos:                                  │  │
│  │                                                    │  │
│  │  ○ Complete seu perfil                             │  │  Circle 24px outline
│  │    Adicione foto, bio e dados profissionais        │  │  Manrope 500 15px
│  │                                                    │  │  Description 13px muted
│  │  ○ Vincule seu primeiro condomínio                 │  │
│  │    Para ativar ferramentas de gestão               │  │
│  │                                                    │  │
│  │  ○ Explore vídeos técnicos                         │  │
│  │    Capacite-se antes de contratar                  │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│              [Ir para o Painel →]                        │  Gold button large
│              (w-full max-w-xs, h-12)                     │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Animação de entrada:**
1. Brasão: scale(0) → scale(1), opacity 0 → 1, 600ms cubic-bezier spring (delay 0ms)
2. Título: opacity 0 → 1, translateY(10px) → 0, 400ms ease (delay 300ms)
3. Subtítulo: mesmo, delay 400ms
4. Checklist: fade-in staggered, delay 500ms + 100ms cada item
5. Botão: opacity 0 → 1, delay 800ms

---

## B.9 FORM COMPONENTS REFINADOS (Flavor Design System)

### B.9.1 Select/Dropdown com Busca Integrada

```
  Categoria de serviço
  ┌──────────────────────────────────────┐
  │  Hidráulica e Saneamento         ▼  │  ← trigger
  └──────────────────────────────────────┘
           │
           ▼
  ┌──────────────────────────────────────┐
  │  🔍 Buscar categoria...             │  ← search input (sticky top)
  ├──────────────────────────────────────┤
  │  ✓ Hidráulica e Saneamento          │  ← selected: text gold, ✓ left
  │    Elétrica, Energia e SPDA         │  ← hover: bg muted
  │    Segurança e Controle de Acesso   │
  │    Jurídico e Direito Imobiliário   │
  │    Engenharia                       │
  │    Manutenção de Fachadas           │
  │    Climatização e Ventilação        │
  └──────────────────────────────────────┘
```

**Specs (Kobalte Select):**
- Trigger: h-11 (44px), px-3, bg `var(--card)`, border 1px `var(--border)`, rounded-md, flex items-center justify-between
- Chevron: 16px, `var(--muted-foreground)`, transition rotate 180° on open
- Content: bg `var(--card)`, border 1px `var(--border)`, rounded-lg, shadow-lg, mt-1, p-1
- Search: px-3 py-2, border-bottom 1px `var(--border)`, sticky top, bg `var(--card)`
- Item: px-3 py-2, rounded-md, Manrope 400 14px, cursor pointer
- Item hover: bg `var(--muted)`
- Item selected: text `var(--primary)`, font-weight 600, ✓ ícone gold 16px
- Max-height dropdown: 240px, overflow-y auto, scrollbar customizada
- Animation: scale(0.95) opacity(0) → scale(1) opacity(1), 150ms ease

### B.9.2 Textarea com Character Counter

```
  Mensagem *
  ┌──────────────────────────────────────────────────────┐
  │                                                      │
  │  Conteúdo digitado pelo usuário aqui...              │
  │                                                      │
  │                                                      │
  │                                            245/500   │  ← counter
  └──────────────────────────────────────────────────────┘
  Descreva o motivo do seu contato com detalhes.           ← helper text
```

- Min-height: 120px, resize vertical (resize-y)
- bg: `var(--card)`, border 1px `var(--border)`, rounded-md, p-3
- Focus: ring-2 `var(--ring)`, border `var(--ring)`
- Font: Manrope 400, 15px, line-height 1.6
- Counter: absolute bottom-8px right-12px, Manrope 400, 12px
  - Normal (<80%): `var(--muted-foreground)`
  - Warning (80-99%): `var(--warning)`
  - Limit (100%): `var(--destructive)`, input bloqueado
- Helper text: Manrope 400, 13px, `var(--muted-foreground)`, mt-1

---

## B.10 ROTAS ATUALIZADAS

```
NOVAS ROTAS (adicionadas neste addendum):

26. /salvos                     → Itens salvos
27. /configuracoes/seguranca    → Sessões, 2FA, senha
28. /configuracoes/privacidade  → Visibilidade, histórico
29. /configuracoes/sobre        → Info da plataforma
30. /bem-vindo                  → Welcome screen pós-cadastro

TOTAL SPRINTS 1-2: 30 rotas
```

---

## B.11 COMPONENTES ADICIONADOS

```
NOVOS COMPONENTES (Addendum B):

SettingsGroup           → Card agrupado com rows separadas por border
SettingsRow             → Row: label + value + action link
SettingsToggleRow       → Row com toggle switch integrado
ThemePreviewCard        → Card com mini-preview do tema (light/dark/auto)
FontSizeSlider          → Slider com 3 stops
SessionRow              → Row de sessão ativa (device, local, ações)
BookmarkButton          → Botão salvar overlay na thumbnail
CardHorizontal          → Card em lista (thumb lateral + info)
BottomSheet             → Modal mobile slide-up
PullToRefresh           → Pull-to-refresh indicator
SwipeAction             → Ações em swipe esq/dir
AccordionPauta          → Accordion expandível para pautas
StepperVisual           → Stepper com progress line animada
WelcomeScreen           → Tela boas-vindas pós-cadastro
SelectSearchable        → Dropdown com busca
TextareaWithCounter     → Textarea com contador
GridListToggle          → Toggle grid/lista
ProfileTabsAnimated     → Tabs com underline deslizante
PlanUsageCard           → Card uso do plano (settings)
PaymentHistoryTable     → Tabela pagamentos
InfoSidebar             → Sidebar de detalhes (assembleia)
CoverImage              → Banner de cover no perfil
InlineEditField         → Campo com edição in-place
CategoryGridCard        → Card de categoria com ícone (busca)
UndoToast               → Toast com botão desfazer
```

---

*Addendum B incorporado ao Design System v1.2*

---

## ADDENDUM C — INSIGHTS DE CROSSFADER, AVALANCHE E AI CONTENT APP (Batch 4)

### Referências analisadas (10 imagens, 4 projetos distintos):

- **Crossfader** (3 imagens) — Plataforma LMS de cursos para DJs. Dark theme, orange accent. Mostra:
  - Login page centralizada (Image 7/61c940)
  - Course listing com paywall modal "Way to unlock" (Image 2/6dac63)
  - Course detail com lesson plan, info sidebar, tabs, player, rating (Image 6/61c940)

- **Avalanche** (1 imagem) — Plataforma de video-sharing estilo YouTube redesenhado. Dark theme, cyan accent. Mostra:
  - Home feed com sidebar de account, subscriptions, folders
  - Category icon grid (For you, Popular now, News & Media, Music, Games, Sport & Fitness)
  - Video player page com tabs (All videos, Similar videos, This author)
  - Player controls completos com engagement bar

- **AI Content App / Genaigurus** (1 imagem longa) — App mobile completo para criação de conteúdo. Mostra:
  - Landscape video player com labels (Backend, Font, Timing Progress bar, Make the collapse)
  - Profile page com Posts tab e Badges tab (grid de badges earned/locked)
  - Web app responsivo com sidebar, blog cards, video cards, article cards
  - Mobile: home page, search page com subjects/history, featured articles, feedback form

- **VoiceLabs** (2 imagens — já analisadas, reforço de detalhes):
  - Dashboard completo com quick action cards e credit widget (Image 4)
  - Scope of Work (Image 8) — estrutura de deliverables (útil como referência metodológica)

- **UX Research Process** (1 imagem) — Double diamond diagram (Discover → Problem → Define → Develop → Solution → Deliver). Referência metodológica, não visual direta.

### Relevância para o Master Síndico:

Este batch é crucial porque traz pela primeira vez **referências diretas de LMS** (Crossfader) que servem para a Sprint 4 mas também introduzem padrões de paywall, lesson lists e course info sidebar que são diretamente aplicáveis já na Sprint 1-2 (paywall de vídeos 25%, lista de pautas numeradas, sidebar de info de assembleia). O Avalanche reforça o padrão YouTube que você quer para a home/feed. E o app de AI Content traz o melhor pattern de **badges grid** para o status do síndico.

---

## C.1 PAYWALL MODAL — PATTERN DE UPGRADE/DESBLOQUEIO (Crossfader Image 2)

O Crossfader mostra um modal "Way to unlock this course" com duas opções lado a lado (individual vs package). Este pattern é **diretamente aplicável** ao Master Síndico em dois cenários:

**Cenário 1 — Paywall de vídeo (25% preview para Base):**
Quando um Morador Base atinge 25% de um vídeo de empresa, o player pausa e aparece o overlay de paywall.

**Cenário 2 — Upgrade de plano:**
Quando um usuário tenta acessar funcionalidade do plano superior.

### C.1.1 Paywall de Vídeo (25% Preview)

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ██████████████████████████████████████████████████████████  │
│  ██                                                      ██  │
│  ██              VÍDEO PAUSADO                           ██  │
│  ██              (thumbnail blur 8px)                    ██  │
│  ██                                                      ██  │
│  ██         ┌──────────────────────────────────┐         ██  │
│  ██         │                                  │         ██  │
│  ██         │     🔒                           │         ██  │
│  ██         │                                  │         ██  │
│  ██         │  Continue assistindo com         │         ██  │
│  ██         │  acesso completo                 │         ██  │
│  ██         │                                  │         ██  │
│  ██         │  Este vídeo é exclusivo para     │         ██  │
│  ██         │  assinantes. Assine para         │         ██  │
│  ██         │  assistir na íntegra.            │         ██  │
│  ██         │                                  │         ██  │
│  ██         │  [Ver Planos →]                  │         ██  │
│  ██         │  (gold button, large)            │         ██  │
│  ██         │                                  │         ██  │
│  ██         │  Já é assinante? [Fazer login]   │         ██  │
│  ██         │                                  │         ██  │
│  ██         └──────────────────────────────────┘         ██  │
│  ██                                                      ██  │
│  ██████████████████████████████████████████████████████████  │
│                                                              │
│  ─── 25% ●──────────────────────────────────────────── 100% │
│  0:45                                                  3:00  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Overlay specs:**
- Posição: absolute, inset 0, sobre o player
- Background: `rgba(0, 0, 0, 0.85)` com `backdrop-filter: blur(8px)` sobre o último frame visível
- Card interno: bg `var(--card)`, rounded-xl, p-8, max-width 400px, centralizado (flex items-center justify-center)
- Shadow: `0 25px 50px -12px rgba(0, 0, 0, 0.5)`
- Lock icon: 48px, `var(--primary)` (gold), mb-4
- Título: Michroma 18px, text-center, mb-2
- Descrição: Manrope 14px, `var(--muted-foreground)`, text-center, mb-6, max-width 280px
- CTA: gold button, full-width, h-11, text-15px
- Login link: Manrope 14px, `var(--primary)`, text-center, mt-3

**Progress bar abaixo:**
- Track: bg `var(--muted)`, h-1
- Fill até 25%: bg `var(--primary)` (gold)
- Após 25%: gradiente fade do gold para `var(--destructive)/50` indicando bloqueio
- O progresso para de avançar no 25%

**Animação de entrada:**
- Overlay: fade-in 400ms
- Card: scale(0.9) opacity(0) → scale(1) opacity(1), 300ms ease, delay 200ms
- Lock icon: bounce sutil (translateY(-4px) → 0), 400ms, delay 400ms

### C.1.2 Modal de Upgrade de Plano (Crossfader "Unlock" pattern)

Quando usuário tenta acessar funcionalidade restrita (ex: Morador Base tenta usar Connect Me):

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  ┌─── BACKDROP (Dialog overlay) ─────────────────────────┐  │
│  │                                                        │  │
│  │  ┌────────────────────────────────────────────────┐   │  │
│  │  │                                          [✕]   │   │  │
│  │  │  🔒 Funcionalidade exclusiva                   │   │  │
│  │  │  (Michroma 20px)                               │   │  │
│  │  │                                                │   │  │
│  │  │  O Connect Me está disponível a partir         │   │  │
│  │  │  do plano Morador Pagante.                     │   │  │
│  │  │  (Manrope 14px, muted-foreground)              │   │  │
│  │  │                                                │   │  │
│  │  │  ┌─────────────────┐ ┌─────────────────┐     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  │  SEU PLANO      │ │  RECOMENDADO    │     │   │  │
│  │  │  │  ATUAL          │ │  ⭐             │     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  │  Membro Base    │ │  Morador        │     │   │  │
│  │  │  │  (Michroma 16px)│ │  Pagante        │     │   │  │
│  │  │  │                 │ │  (Michroma 16px)│     │   │  │
│  │  │  │  Grátis         │ │  R$ 19,90/mês   │     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  │  ────────────── │ │  ────────────── │     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  │  ✅ Busca geral │ │  ✅ Tudo do Base│     │   │  │
│  │  │  │  ✅ Preview 25% │ │  ✅ Connect Me  │     │   │  │
│  │  │  │  ✅ Votação     │ │  ✅ Currículo   │     │   │  │
│  │  │  │  ✅ Assembleia  │ │  ✅ Perfil visív│     │   │  │
│  │  │  │  ✅ Avaliação   │ │  ✅ 4 contatos  │     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  │  Plano atual    │ │  [Assinar →]    │     │   │  │
│  │  │  │  (ghost, desab.)│ │  (gold, large)  │     │   │  │
│  │  │  │                 │ │                 │     │   │  │
│  │  │  └─────────────────┘ └─────────────────┘     │   │  │
│  │  │                                                │   │  │
│  │  │  [Ver todos os planos →]                       │   │  │
│  │  │  (link gold, text-center)                      │   │  │
│  │  │                                                │   │  │
│  │  └────────────────────────────────────────────────┘   │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Plan Comparison Cards (Crossfader pattern adaptado):**
- Container modal: bg `var(--card)`, rounded-xl, p-6, max-width 640px
- Dois cards lado a lado: flex, gap-4
- Card "Plano Atual": border 1px `var(--border)`, rounded-lg, p-5, opacity 0.7
- Card "Recomendado": border 2px `var(--primary)` (gold), rounded-lg, p-5, shadow `0 0 20px oklch(from var(--primary) l c h / 0.15)`
- Badge "RECOMENDADO": absolute top -12px right 16px, pill bg `var(--primary)`, text white, px-3 py-1, font-size 11px, Michroma uppercase
- Preço: Manrope 700, 28px (Crossfader usa fonte grande para preço — impactante)
- Preço riscado (se houver desconto): text-decoration line-through, `var(--muted-foreground)`, mr-2
- Feature list: cada item com ✅ `var(--success)` 16px + Manrope 400, 14px
- Features ausentes no plano atual: ✕ `var(--muted-foreground)` + texto line-through
- CTA "Assinar": bg `var(--primary)`, full-width, h-11
- CTA "Plano atual": ghost button, disabled, cursor not-allowed
- Mobile: cards empilham vertical (flex-col)

**Quando exibir este modal:**
- Morador Base tenta usar Connect Me → modal sugere Morador Pagante
- Morador tenta ver Fórum → modal sugere plano com acesso
- Síndico N1 tenta publicar vídeo → modal sugere Síndico N2
- Empresa Plus tenta criar curso → modal sugere Empresa Pro
- Regra: o modal sempre compara o plano atual vs. o mínimo necessário para a funcionalidade

---

## C.2 COURSE/LMS DETAIL PAGE — PATTERN COMPLETO (Crossfader Image 6)

O Crossfader mostra a melhor referência de LMS detail page que recebemos. Layout: breadcrumb → header com título/rating → player → tabs → lesson list. Sidebar: info card + related courses. Adaptar para Sprint 4 (cursos) mas documentar agora porque o **padrão de layout se aplica a assembleia detail também**.

### C.2.1 Layout Principal — Curso Detail

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ← Todos os cursos / Manutenção de Fachadas       │ ← breadcrumb
│          │                                                    │
│          │  ┌─────────────────────────────┬───────────────┐  │
│          │  │                             │               │  │
│          │  │  CONTEÚDO (flex-1)          │ INFO (300px)  │  │
│          │  │                             │               │  │
│          │  │  Manutenção Preventiva      │ ┌───────────┐│  │
│          │  │  de Fachadas: Guia          │ │ COURSE    ││  │
│          │  │  Completo para Síndicos     │ │ INFO CARD ││  │
│          │  │  (Michroma 22px)            │ │           ││  │
│          │  │                             │ │ 📚 Nível: ││  │
│          │  │  HidraFix Engenharia        │ │ Iniciante ││  │
│          │  │  "Aprenda os critérios..."  │ │           ││  │
│          │  │  (Manrope 14px, muted)      │ │ 📖 Aulas: ││  │
│          │  │                             │ │ 14        ││  │
│          │  │  4.8 ⭐⭐⭐⭐⭐ (127)        │ │           ││  │
│          │  │  (star rating + count)      │ │ ⏱ Duração:││  │
│          │  │                             │ │ 7h        ││  │
│          │  │  ┌───────────────────────┐  │ │           ││  │
│          │  │  │                       │  │ │───────────││  │
│          │  │  │    VIDEO PLAYER       │  │ │           ││  │
│          │  │  │    aspect-16/9        │  │ │ Acesso:   ││  │
│          │  │  │    [▶ play center]    │  │ │ Sínd. N2+ ││  │
│          │  │  │                       │  │ │ Emp. Pro  ││  │
│          │  │  │  ▶ 0:05/1:10         │  │ │           ││  │
│          │  │  │  ━━━━━●━━━━━━━━━━━━━ │  │ │[Inscrever]││  │
│          │  │  │          🔈 ⛶         │  │ │ (gold)    ││  │
│          │  │  └───────────────────────┘  │ │           ││  │
│          │  │                             │ └───────────┘│  │
│          │  │  [Visão geral] [Aulas] ... │               │  │
│          │  │                ══════       │ ┌───────────┐│  │
│          │  │                             │ │ CURSOS    ││  │
│          │  │  Este curso contém 14 aulas:│ │RELACIONAD.││  │
│          │  │                             │ │           ││  │
│          │  │  ┌─────────────────────┐   │ │ [thumb]   ││  │
│          │  │  │ 01  Introdução ao   │   │ │ Título    ││  │
│          │  │  │     curso  5 min    │   │ │ ⭐ 4.9    ││  │
│          │  │  ├─────────────────────┤   │ │           ││  │
│          │  │  │ 02  Tipos de        │   │ │ [thumb]   ││  │
│          │  │  │     fachada  8 min  │   │ │ Título    ││  │
│          │  │  ├─────────────────────┤   │ │ ⭐ 4.7    ││  │
│          │  │  │ 03  Normas e        │   │ │           ││  │
│          │  │  │     legislação 12min│   │ └───────────┘│  │
│          │  │  └─────────────────────┘   │               │  │
│          │  │                             │               │  │
│          │  └─────────────────────────────┴───────────────┘  │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

### C.2.2 Course Info Card (Sidebar — Crossfader)

O Crossfader mostra uma info card compacta com ícones + dados + CTAs. Adaptar para cursos e assembleias:

```
┌──────────────────────────────────┐
│                                  │
│  📚  Nível: Iniciante           │  ← icon 18px muted + label 13px muted
│                                  │     + value 15px 500 foreground
│  📖  Aulas: 14                  │
│                                  │
│  ⏱  Duração: 7h                │
│                                  │
│  🏢  Empresa: HidraFix Eng.    │
│                                  │
│  📅  Criado em: Jan 2026        │
│                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                  │
│  Acesso disponível para:         │  ← Manrope 13px, muted
│                                  │
│  [Síndico N2+] [Empresa Pro]    │  ← badges primary
│  [Marketing]                     │
│                                  │
│  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━  │
│                                  │
│  [Inscrever-se neste curso →]   │  ← gold button, full-width
│                                  │
│  [📋 Compartilhar curso]       │  ← outline button
│                                  │
└──────────────────────────────────┘
```

**CSS da Info Card:**
```css
.course-info-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 24px;
  position: sticky;
  top: 80px;
}

.course-info-row {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 8px 0;
}

.course-info-icon {
  width: 18px;
  height: 18px;
  color: var(--muted-foreground);
  flex-shrink: 0;
}

.course-info-label {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
}

.course-info-value {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 500;
  color: var(--foreground);
  margin-left: auto;
}
```

### C.2.3 Lesson List / Aula List (Crossfader Pattern)

O Crossfader mostra uma lista de aulas numerada com: número sequencial em circle, título, duração, e badge "Free lesson" para preview grátis. Este pattern se aplica a:
- **Cursos** (Sprint 4): lista de aulas
- **Assembleias** (Sprint 2): lista de pautas
- **Módulos de capacitação**: lista de vídeos no módulo

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  Este curso contém 14 aulas:                             │
│  (Manrope 600, 16px)                                     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  ┌────┐                                            │  │
│  │  │ 01 │  Introdução ao curso          5 min        │  │
│  │  └────┘  de fachadas                  [Grátis]     │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  ┌────┐                                            │  │
│  │  │ 02 │  Tipos de fachada e           8 min        │  │
│  │  └────┘  seus revestimentos           🔒           │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  ┌────┐                                            │  │
│  │  │ 03 │  Normas e legislação          12 min       │  │
│  │  └────┘  aplicável                    🔒           │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  ┌────┐                                            │  │
│  │  │ 04 │  Inspeção visual:             15 min       │  │
│  │  └────┘  o que observar               🔒           │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  ┌────┐                             ▶ Assistindo  │  │
│  │  │ 05 │  Plano de manutenção         10 min        │  │
│  │  └────┘  preventiva                  ████░░ 60%    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Number Circle (Crossfader):**
```css
.lesson-number {
  width: 36px;
  height: 36px;
  border-radius: 8px; /* rounded-lg, não full circle como Crossfader usa */
  background: var(--muted);
  color: var(--muted-foreground);
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

/* Lesson em progresso */
.lesson-number.current {
  background: var(--primary);
  color: var(--primary-foreground);
}

/* Lesson concluída */
.lesson-number.completed {
  background: var(--success);
  color: var(--success-foreground);
}

/* Lesson bloqueada */
.lesson-number.locked {
  background: var(--muted);
  color: var(--muted-foreground);
  opacity: 0.5;
}
```

**Lesson Row:**
```css
.lesson-row {
  display: flex;
  align-items: center;
  gap: 16px;
  padding: 14px 20px;
  border-bottom: 1px solid var(--border);
  cursor: pointer;
  transition: background-color 150ms;
}

.lesson-row:hover {
  background: var(--muted);
}

.lesson-row:last-child {
  border-bottom: none;
}

.lesson-title {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 500;
  color: var(--foreground);
  flex: 1;
  min-width: 0;
}

.lesson-title.locked {
  color: var(--muted-foreground);
}

.lesson-duration {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  flex-shrink: 0;
  white-space: nowrap;
}

.lesson-badge-free {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 600;
  color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.1);
  padding: 2px 8px;
  border-radius: 9999px;
}

.lesson-lock-icon {
  width: 16px;
  height: 16px;
  color: var(--muted-foreground);
  opacity: 0.5;
}
```

**Progress Bar Inline (para aula em andamento):**
- Track: h-2, bg `var(--muted)`, rounded-full, max-width 80px
- Fill: bg `var(--primary)`, rounded-full
- Percentual: Manrope 12px, muted-foreground, ml-1

### C.2.4 Course Tabs (Crossfader)

O Crossfader mostra 6 tabs: Overview, Lessons plan, What's included, Equipment, About the tutor, Testimonials.

**Adaptar para Master Síndico (Curso Detail):**
```
[Visão Geral]  [Aulas]  [Material]  [Empresa]  [Avaliações]
                ══════
```

**Adaptar para Master Síndico (Assembleia Detail) — já definido, reforço:**
```
[Pautas]  [Votação]  [Edital]  [Fornecedores]
 ══════
```

**Tab Component refinado (confirmação com Crossfader + Flavor + Genaigurus):**
```css
.content-tabs {
  display: flex;
  gap: 0;
  border-bottom: 1px solid var(--border);
  overflow-x: auto;
  scrollbar-width: none;
  -webkit-overflow-scrolling: touch;
}

.content-tabs::-webkit-scrollbar {
  display: none;
}

.content-tab {
  position: relative;
  padding: 12px 20px;
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  white-space: nowrap;
  cursor: pointer;
  transition: color 200ms ease;
  border-bottom: 2px solid transparent;
  margin-bottom: -1px; /* sobrepõe border do container */
}

.content-tab:hover {
  color: var(--foreground);
}

.content-tab.active {
  color: var(--primary); /* Crossfader usa orange, nós usamos gold */
  font-weight: 600;
  border-bottom-color: var(--primary);
}
```

### C.2.5 Star Rating Component (Crossfader)

O Crossfader mostra rating com estrelas cheias em laranja. Adaptar para dourado:

```
  4.8 ⭐⭐⭐⭐⭐ (127 avaliações)
```

**Specs:**
- Stars: 5 ícones de 16px
- Star filled: `var(--primary)` (gold)
- Star empty: `var(--muted)` com stroke `var(--border)`
- Star half: clip-path para metade gold, metade muted
- Número: Manrope 700, 15px, `var(--foreground)`, mr-1
- Count: Manrope 400, 13px, `var(--muted-foreground)`, entre parênteses
- Hover (se interativo): star pulsa scale(1.1), cursor pointer
- Read-only mode: cursor default, sem hover effect
- Para avaliação do síndico: usar este mesmo componente

**CSS:**
```css
.star-rating {
  display: inline-flex;
  align-items: center;
  gap: 2px;
}

.star-icon {
  width: 16px;
  height: 16px;
  transition: transform 150ms;
}

.star-icon.filled {
  color: var(--primary);
}

.star-icon.empty {
  color: var(--muted);
}

.star-icon.interactive:hover {
  transform: scale(1.2);
}

.star-rating-value {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 700;
  color: var(--foreground);
  margin-right: 4px;
}

.star-rating-count {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  margin-left: 4px;
}
```

---

## C.3 LOGIN PAGE — PATTERN CENTRALIZADO (Crossfader Image 7)

O Crossfader mostra uma login page extremamente limpa: background escuro, card centralizado com logo no topo, email, password com toggle visibility, CTA accent, social auth, e link para criar conta.

### C.3.1 Login Page Refinada

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  [← Voltar ao site]                                          │  ← top-left, link muted
│                                                              │
│                                                              │
│                                                              │
│           ┌────────────────────────────────────┐             │
│           │                                    │             │
│           │  🛡️ (Brasão Master Síndico 48px)  │             │
│           │                                    │             │
│           │  Entrar na sua conta               │             │  Michroma 20px
│           │  Digite seu email e senha para     │             │  Manrope 14px muted
│           │  acessar a plataforma.             │             │
│           │                                    │             │
│           │  Email                             │             │
│           │  ┌──────────────────────────────┐  │             │
│           │  │  joao@email.com              │  │             │
│           │  └──────────────────────────────┘  │             │
│           │                                    │             │
│           │  Senha                    [Esqueceu?]│            │  "Esqueceu?" = link gold
│           │  ┌──────────────────────────────┐  │             │
│           │  │  ••••••••••          [👁]    │  │             │  toggle eye icon
│           │  └──────────────────────────────┘  │             │
│           │                                    │             │
│           │  [        Entrar         ]         │             │  gold button, full-width
│           │                                    │             │
│           │  ─── ou ───                        │             │  divider com texto
│           │                                    │             │
│           │  [G  Continuar com Google  ]       │             │  outline button, full-width
│           │                                    │             │
│           │  Não tem conta? [Criar conta]      │             │  muted + link gold
│           │                                    │             │
│           └────────────────────────────────────┘             │
│                                                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Login Card specs (refinamento do Crossfader):**
- Card: bg `var(--card)`, border 1px `var(--border)`, rounded-xl, p-8
- Max-width: 420px, mx-auto
- Brasão: mb-6, centered
- Shadow: `0 25px 50px -12px rgba(0, 0, 0, 0.25)` (sutil no dark mode)
- Inputs: full-width, h-11, stack vertical com gap-4
- Password toggle (👁): icon button 20px, absolute right-12px, `var(--muted-foreground)`, hover `var(--foreground)`, toggle entre eye e eye-off
- "Esqueceu?" link: absolute right-0 do label, Manrope 13px, `var(--primary)`
- CTA "Entrar": bg `var(--primary)`, text white, full-width, h-11, mt-2
- Divider "ou": flex items-center gap-3, linha 1px `var(--border)` de cada lado, texto Manrope 12px muted
- Google button: bg `var(--secondary)`, border 1px `var(--border)`, text `var(--foreground)`, full-width, h-11
- Google icon: 18px, mr-2
- Footer link: Manrope 14px, text-center, mt-4

**"← Voltar ao site":**
- Position: absolute top-6 left-6 (ou fixed)
- Manrope 14px, `var(--muted-foreground)`
- Ícone ← 16px, mr-1
- Hover: `var(--foreground)`
- Apenas visível se o usuário veio de uma landing page (não logado)

**Background da auth page:**
- bg `var(--background)` solid
- Opção premium: gradient radial sutil no centro com `oklch(from var(--primary) l c h / 0.03)` criando um glow gold quase imperceptível atrás do card
- Mobile: sem glow (performance), card full-width com px-4

### C.3.2 Cadastro Page (Adaptar login para signup)

O cadastro no Master Síndico é "Stripe-like" (parametrizado via URL). A estrutura visual é similar ao login mas com mais campos e o stepper:

```
┌────────────────────────────────────────┐
│                                        │
│  🛡️                                  │
│                                        │
│  Criar sua conta                       │  Michroma 20px
│  Tipo: Síndico Pro                     │  Badge gold mostrando plano detectado
│                                        │
│  ●━━━━━━━━○━━━━━━━━○━━━━━━━━○         │  Stepper (step 1 de 4)
│  1        2        3        4          │
│                                        │
│  Nome completo *                       │
│  ┌──────────────────────────────────┐  │
│  │                                  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Email *                               │
│  ┌──────────────────────────────────┐  │
│  │                                  │  │
│  └──────────────────────────────────┘  │
│                                        │
│  Telefone                              │
│  ┌──────────────────────────────────┐  │
│  │  (  )                            │  │  masked input
│  └──────────────────────────────────┘  │
│                                        │
│  [          Continuar →          ]     │  gold, full-width
│                                        │
│  Já tem conta? [Entrar]               │
│                                        │
└────────────────────────────────────────┘
```

**Badge de plano detectado:**
- Aparece entre o título e o stepper
- pill bg `var(--primary)/10`, text `var(--primary)`, icon 🛡️, Manrope 600 13px
- "Tipo: Síndico Pro" — detectado da URL (?role=sindico&plan=pro)
- Se URL não tiver params: badge não aparece, field de seleção de tipo visível

---

## C.4 VIDEO PLATFORM FEED — AVALANCHE PATTERNS (Image 5)

### C.4.1 Category Icon Grid (Avalanche)

O Avalanche mostra categorias como um grid de ícones quadrados com labels, similar ao YouTube redesign. Este pattern é superior aos pills de categoria para a home page porque ocupa mais espaço visual e é mais explorável.

```
DESKTOP (horizontal scroll with arrows):

[← ] [🏗️ Todos] [⚡ Hidráulica] [🔧 Elétrica] [🔒 Segurança] [⚖️ Jurídico] [🏠 Reformas] [🛡️ Seguros] [📊 Contabil.] [ →]

MOBILE (grid 2 rows):

[🏗️ Todos  ] [⚡ Hidráulica] [🔧 Elétrica ] [🔒 Segurança]
[⚖️ Jurídico] [🏠 Reformas ] [🛡️ Seguros  ] [📊 Contabil.]
                                            ← scroll horizontal →
```

**Avalanche mostra dois padrões de categorias:**

**Padrão A — Icon Chips (horizontal scroll, preferido para home):**
```css
.category-chip {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  background: var(--muted);
  border: 1px solid var(--border);
  border-radius: 9999px; /* pill */
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 500;
  color: var(--foreground);
  white-space: nowrap;
  cursor: pointer;
  transition: all 150ms;
  flex-shrink: 0;
}

.category-chip:hover {
  background: var(--muted);
  border-color: var(--muted-foreground);
}

.category-chip.active {
  background: var(--primary);
  border-color: var(--primary);
  color: var(--primary-foreground);
  font-weight: 600;
}

.category-chip .icon {
  width: 16px;
  height: 16px;
}
```

**Padrão B — Icon Cards (grid, para página de busca/explorar):**

O Avalanche mostra cards quadrados com ícone + label (For you, Popular now, News & Media, etc). Adaptar para a página de busca como já definido no Addendum B, mas refinando:

```css
.category-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 16px 12px;
  min-height: 80px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  cursor: pointer;
  transition: all 150ms;
}

.category-card:hover {
  border-color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.05);
}

.category-card:hover .category-card-icon {
  color: var(--primary);
  transform: scale(1.1);
}

.category-card-icon {
  width: 28px;
  height: 28px;
  color: var(--muted-foreground);
  transition: all 200ms;
}

.category-card-label {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 500;
  color: var(--muted-foreground);
  text-align: center;
  line-height: 1.2;
}
```

### C.4.2 Video Player Page Layout (Avalanche)

O Avalanche mostra o player page com: player full-width, info do channel abaixo, tabs para conteúdo relacionado, e engagement bar. O layout confirma e reforça o que já definimos, mas adiciona detalhes:

**Channel Info Bar (abaixo do player):**
```
┌────────────────────────────────────────────────────────────┐
│                                                            │
│  [Avatar 40px]  HidraFix Engenharia                       │
│                 Empresa Pro • 127 vídeos                   │
│                                                            │
│                        [Connect Me →]  [🔖 Salvar]  [↗]  │
│                                                            │
└────────────────────────────────────────────────────────────┘
```

**Specs:**
- Container: flex, items-center, justify-between, py-4, border-bottom 1px `var(--border)`
- Avatar: 40px rounded-full, border 2px `var(--border)` (ou gold se Pro)
- Nome: Manrope 600, 16px
- Subtitle: Manrope 400, 13px, muted-foreground
- Verified badge: ✓ icon 14px em circle 18px bg `var(--primary)`, text white (apenas para perfis institucionais verificados)
- Actions (direita): flex gap-2
- "Connect Me →": botão primary sm (só aparece se usuário logado tem permissão para contatar)
- "Salvar": icon button outline, bookmark icon
- "↗": icon button outline, share icon

**Tabs abaixo do player (Avalanche "All videos / Similar videos / This author"):**

Adaptar para Master Síndico:
```
[Relacionados]  [Mesmo canal]  [Mesma categoria]
 ═══════════════
```

- São filtros de conteúdo na lista de "vídeos relacionados" abaixo do player
- Grid de cards horizontais (CardHorizontal) com scroll ou lista vertical

### C.4.3 Sidebar Left — Subscriptions/Channels Pattern (Avalanche)

O Avalanche mostra na sidebar: "MY CHANNELS", "Subscriptions" com lista de canais, e "Favourite videos" com folder management. Adaptar:

**Para Síndicos — "Meus Condomínios" na sidebar (já definido como AccountSwitcher, agora expandir):**

No contexto de sidebar expandida, abaixo do account switcher, listar condomínios como o Avalanche lista channels:

```
MEUS CONDOMÍNIOS ──────────

[🏢] Cond. Vila Mariana   ● (ativo)
[🏢] Ed. Paulista Center
[🏢] Res. Jardim Europa

[+ Vincular condomínio]
```

- Cada item: flex, items-center, gap-2, py-1.5, px-2
- Ícone: 🏢 16px, muted-foreground
- Nome: Manrope 400, 13px, truncate
- Dot ativo: 6px circle, `var(--success)`, ml-auto
- Hover: bg `var(--muted)`, rounded-md
- Click: muda contexto do condomínio (AccountSwitcher)
- "+ Vincular": text `var(--primary)`, 13px, mt-1
- Colapsável: section header com chevron, começa expandida

---

## C.5 BADGES GRID — PROFILE PATTERN (AI Content App, Image 1)

### C.5.1 Badges Tab no Profile (Genaigurus pattern)

A Image 1 mostra uma tela mobile "My Profile (Badges)" com uma grid de badges, mostrando badges earned (coloridos) e badges locked (cinza/opaco). Este é o **melhor pattern para o sistema de status do Síndico**.

```
┌──────────────────────────────────────────────────────────┐
│  [Posts]  [Vídeos]  [Gestão]  [Status]  [Conquistas]    │
│                                          ═══════════      │
│                                                          │
│  CONQUISTAS ──────────────── 12 de 24 desbloqueadas     │
│                                                          │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │ 🏆     │ │ 📹     │ │ 📚     │ │ 🔒     │           │
│  │        │ │        │ │        │ │ (gray) │           │
│  │Primeiro│ │ 10     │ │ 5      │ │ ???    │           │
│  │Vídeo   │ │ Vídeos │ │Módulos │ │        │           │
│  │ [✓]    │ │ [✓]    │ │ [✓]    │ │ [🔒]   │           │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│  ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│  │ 🏅     │ │ 🏢     │ │ 🔒     │ │ 🔒     │           │
│  │        │ │        │ │ (gray) │ │ (gray) │           │
│  │Bronze  │ │ 3      │ │ ???    │ │ ???    │           │
│  │Status  │ │ Cond.  │ │        │ │        │           │
│  │ [✓]    │ │ [✓]    │ │ [🔒]   │ │ [🔒]   │           │
│  └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                          │
│  ── PRÓXIMA CONQUISTA ──                                 │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │ 🥈 Status Prata                                   │  │
│  │                                                    │  │
│  │ Assista 30 vídeos (>70%) e vincule 3 condomínios  │  │
│  │                                                    │  │
│  │ ████████████████████████░░░░░░░░░ 72%              │  │
│  │                                                    │  │
│  │ 📹 22/30 vídeos    🏢 3/3 condomínios ✅          │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Badge Card specs:**
```css
.badge-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 6px;
  padding: 16px 8px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  text-align: center;
  transition: all 200ms;
  aspect-ratio: 1;
  min-height: 100px;
}

/* Badge desbloqueado */
.badge-card.earned {
  border-color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.05);
}

.badge-card.earned .badge-icon {
  opacity: 1;
}

/* Badge bloqueado */
.badge-card.locked {
  opacity: 0.4;
  filter: grayscale(1);
}

.badge-card.locked .badge-icon {
  opacity: 0.3;
}

.badge-icon {
  font-size: 32px;
  line-height: 1;
}

.badge-label {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 600;
  color: var(--foreground);
  line-height: 1.2;
}

.badge-check {
  width: 18px;
  height: 18px;
  border-radius: 9999px;
}

.badge-check.earned {
  background: var(--success);
  color: white;
  /* ícone ✓ 12px */
}

.badge-check.locked {
  background: var(--muted);
  color: var(--muted-foreground);
  /* ícone 🔒 10px */
}
```

**Grid layout:**
- Desktop: 4 cols (grid-template-columns: repeat(4, 1fr))
- Tablet: 3 cols
- Mobile: 3 cols (cards menores, min-height 80px)
- Gap: 12px

**"Próxima Conquista" Card:**
- bg `var(--card)`, border 1px `var(--primary)/30`, rounded-xl, p-5
- Ícone grande: 40px, mb-2
- Título: Manrope 600, 16px
- Descrição: Manrope 400, 14px, muted-foreground
- Progress bar: h-3, bg muted, rounded-full, fill `var(--primary)`, animation width 600ms ease-out
- Sub-critérios: flex gap-4, cada um com ícone 16px + contagem + check se completo

**Lista de Conquistas do Master Síndico:**
```
Tier 1 (fácil):
🏆 Primeiro Acesso     — Fez login pela primeira vez
📸 Perfil Completo     — Preencheu foto, bio e dados
📹 Primeiro Vídeo      — Assistiu primeiro vídeo >70%
🏢 Primeiro Condomínio — Vinculou primeiro condomínio

Tier 2 (médio):
📹 10 Vídeos           — Assistiu 10 vídeos >70%
📚 Primeiro Módulo     — Completou primeiro módulo de capacitação
🗳️ Primeira Votação    — Participou de votação na assembleia
📊 Primeira Avaliação  — Avaliou a gestão do síndico

Tier 3 (avançado):
🥉 Status Bronze       — Atingiu status Bronze
🥈 Status Prata        — Atingiu status Prata
🥇 Status Ouro         — Atingiu status Ouro
💎 Status Diamante     — Atingiu status Diamante
📹 50 Vídeos           — Assistiu 50 vídeos >70%
📚 5 Módulos           — Completou 5 módulos
🏢 3 Condomínios       — Vinculou 3+ condomínios

Tier 4 (elite):
📹 100 Vídeos          — Assistiu 100 vídeos >70%
📚 Todos os Módulos    — Completou todos os módulos disponíveis
🏆 Certificado         — Obteve certificado em curso
🔐 Conquistas Ocultas  — ??? (desbloqueiam por ações específicas)
```

**Nota importante:** O sistema de conquistas (badges/achievements) é um complemento ao sistema de Status (Bronze/Prata/Ouro/Diamante) já definido no Capítulo 8 do Manual. As conquistas são mais granulares e visíveis apenas no perfil do síndico como motivação pessoal. O Status é o indicador público principal. Conquistas ≠ gamificação agressiva — são marcos de jornada, não competição.

---

## C.6 FEEDBACK/DENÚNCIA FORM — MOBILE PATTERN (AI Content App, Image 1)

A Image 1 mostra uma tela mobile "Send us some feedback!" com:
- Campo "Tell us the problem" (descrição do issue)
- Campo "Describe your issue or idea" (textarea)
- Indicação visual de que é para "Providing the feedback about the app"

### C.6.1 Formulário de Denúncia Refinado (Mobile-First)

Já definimos o ReportDialog no Addendum A, mas agora temos referência visual para refinar a versão mobile:

```
MOBILE (Bottom Sheet):
┌─────────────────────────────────────┐
│  ━━━━━━━━ (handle)                  │
│                                     │
│  Denunciar conteúdo                 │  Michroma 16px
│                                     │
│  Nos ajude a manter a qualidade     │  Manrope 14px muted
│  dos conteúdos da plataforma.       │
│                                     │
│  Qual o problema? *                 │  Label 14px 500
│                                     │
│  ○ Conteúdo comercial/propaganda    │  Radio cards
│  ○ Dados de contato no vídeo        │
│  ○ Conteúdo ofensivo               │
│  ○ Informação técnica incorreta     │
│  ○ Spam ou repetitivo               │
│  ● Outro                            │  ← selected: gold dot, border gold
│                                     │
│  Descreva o problema (opcional)     │  Slide-down se "Outro"
│  ┌─────────────────────────────┐   │
│  │                             │   │
│  │                     120/300 │   │
│  └─────────────────────────────┘   │
│                                     │
│  [Enviar denúncia]                  │  gold button full-width
│                                     │
│  ℹ️ Denúncias são analisadas       │  info callout
│  pelo administrador da plataforma.  │
│                                     │
└─────────────────────────────────────┘
```

**Radio Card (refinado com Crossfader + AI App):**
```css
.radio-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 16px;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  margin-bottom: 8px;
  cursor: pointer;
  transition: all 150ms;
}

.radio-card:hover {
  background: var(--muted);
}

.radio-card.selected {
  border-color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.05);
  border-left: 3px solid var(--primary);
  padding-left: 13px; /* compensar border extra */
}

.radio-dot {
  width: 18px;
  height: 18px;
  border-radius: 9999px;
  border: 2px solid var(--border);
  flex-shrink: 0;
  transition: all 150ms;
  display: flex;
  align-items: center;
  justify-content: center;
}

.radio-card.selected .radio-dot {
  border-color: var(--primary);
}

.radio-card.selected .radio-dot::after {
  content: '';
  width: 8px;
  height: 8px;
  border-radius: 9999px;
  background: var(--primary);
}

.radio-label {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--foreground);
}
```

**Info Callout (bottom note):**
```css
.info-callout {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 12px 16px;
  background: oklch(from var(--primary) l c h / 0.05);
  border: 1px solid oklch(from var(--primary) l c h / 0.15);
  border-radius: 0.5rem;
  margin-top: 16px;
}

.info-callout-icon {
  width: 16px;
  height: 16px;
  color: var(--primary);
  flex-shrink: 0;
  margin-top: 1px;
}

.info-callout-text {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  line-height: 1.4;
}
```

---

## C.7 WEB APP RESPONSIVE — SIDEBAR + BLOG/VIDEO GRID (AI Content App, Image 1)

A Image 1 mostra o web app responsivo com sidebar completa e content area com múltiplas seções de conteúdo. Detalhes que confirmam e refinam nosso layout:

### C.7.1 Sidebar Newsletter/CTA Widget (AI Content App)

A sidebar do web app mostra um widget "Join our newsletter" com input de email e botão Subscribe. No Master Síndico, já definimos que isso vira um toggle de comunicados. Mas o pattern visual é útil:

```
SIDEBAR BOTTOM:

┌─────────────────────────┐
│  📩 Comunicados         │
│                         │
│  Receba atualizações    │
│  sobre seu condomínio   │
│  e a plataforma.        │
│                         │
│  ┌─────────────────┐   │
│  │ Notificações:   │   │
│  │ [====● ON]      │   │
│  └─────────────────┘   │
│                         │
│  ─────────────────────  │
│                         │
│  [𝕏] [📸] [▶]         │  ← social links (se tiver)
│                         │
└─────────────────────────┘
```

- Container: bg `var(--surface-variant)`, rounded-lg, p-4, mt-auto (fica no bottom da sidebar)
- Título: Manrope 600, 14px
- Descrição: Manrope 400, 12px, muted-foreground
- Toggle: inline, com label "Notificações" ao lado
- Social links: icon buttons 20px, gap-3, muted-foreground, hover foreground

### C.7.2 "Latest Blog" → "Vídeos em Destaque" Section Layout

A Image 1 mostra um grid de blog cards com thumbnails grandes na parte superior da content area. Mapear:

```
── VÍDEOS EM DESTAQUE ────────────────── [Ver todos →]

┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│ [THUMB]  │ │ [THUMB]  │ │ [THUMB]  │ │ [THUMB]  │  ← 4 cols large
│ 16:9     │ │ 16:9     │ │ 16:9     │ │ 16:9     │     thumbnails
│          │ │          │ │          │ │          │
│ Título   │ │ Título   │ │ Título   │ │ Título   │
│ do vídeo │ │ do vídeo │ │ do vídeo │ │ do vídeo │
└──────────┘ └──────────┘ └──────────┘ └──────────┘

── VÍDEOS POPULARES ──────────────────── [Ver todos →]

[Card H] [Card H] [Card H]
(3 cols de cards horizontais menores)
```

**Section com "Ver todos →":**
```css
.section-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.section-title {
  font-family: 'Michroma', sans-serif;
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
}

.section-link {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 500;
  color: var(--primary);
  display: flex;
  align-items: center;
  gap: 4px;
  transition: opacity 150ms;
}

.section-link:hover {
  opacity: 0.8;
}

.section-link .arrow {
  width: 14px;
  height: 14px;
  transition: transform 150ms;
}

.section-link:hover .arrow {
  transform: translateX(2px);
}
```

### C.7.3 Mobile Home Pattern (AI Content App)

A Image 1 mostra a home mobile com:
1. Banner carousel no topo (featured content)
2. Section "Popular youtube videos" com cards horizontais
3. Section "Popular articles" com cards menores
4. Category pills no bottom

**Mobile Home Order (top to bottom):**
```
[Header: Logo + Search + Avatar]
[Featured Banner Carousel - full width, aspect 16/9]
[Category Pills: scroll horizontal]
[Vídeos em Destaque: cards 2 cols]
[Seção "Assembleia Ativa" (se houver): card highlight]
[Vídeos Populares: cards 2 cols ou scroll horizontal]
[Vizinhança/Cupons: horizontal scroll cards]
[Bottom Nav: 5 items]
```

---

## C.8 SCOPE OF WORK — REFERÊNCIA METODOLÓGICA (VoiceLabs Image 8)

A Image 8 mostra a divisão de deliverables do projeto VoiceLabs. Embora não seja uma UI reference direta, é útil como checklist para validar que nosso design system cobre todos os aspectos:

```
✅ Product UX strategy & information architecture  → Documento atual
✅ User flows (voice generation, selection, usage)  → Mapear para nossos user flows
✅ Wireframes for landing page and web app          → Layouts detalhados neste doc
✅ Responsive design (desktop, tablet, mobile)      → Breakpoints definidos
✅ High-fidelity UI for all pages                   → Specs detalhados por tela
✅ Component-based UI for scalability               → Design tokens + components
✅ Visual hierarchy and layout system               → Tipografia + spacing
✅ CTA placement and messaging hierarchy            → CTAs gold definidos
✅ Trust signals and credibility design              → Status badges, plan badges
✅ Product storytelling (discovery → activation)     → Onboarding flow
✅ Design system components                         → Lista completa de components
✅ Responsive layouts                               → Mobile-first patterns
```

---

## C.9 LANDSCAPE VIDEO PLAYER (AI Content App, Image 1)

A Image 1 mostra um video player mobile em **landscape mode** com anotações detalhadas. Padrão essencial para vídeos institucionais, pautas de assembleia, e LMS.

### C.9.1 Player Landscape Mobile

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  [Backend]                                        [Font]     │  ← top overlay info
│                      Landscape View                          │
│                                                              │
│                     ┌──────────┐                             │
│                     │   ⊙      │                             │  ← center play/pause
│                     │  replay  │                             │     (large 64px button)
│                     └──────────┘                             │
│                                                              │
│  ← 10s    ▶ / ⏸    10s →                                   │  ← skip controls
│                                                              │
│  7:00 ━━━━━━━━━━━●━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 45:29   │  ← progress bar
│                                                              │
│  [Collapse ↓]                                                │  ← minimize/exit
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Landscape Player specs:**
- Full-screen mode no mobile quando rotaciona para landscape
- Background: pure black
- Controls: overlay transparente, appear on tap, auto-hide after 3s
- Play/Pause center: 64px circle, bg `rgba(0,0,0,0.5)`, icon white 28px
- Skip buttons: 40px circles, bg `rgba(0,0,0,0.3)`, icon white 20px, posicionados ±120px do centro
- Progress bar: bottom 48px, full-width com padding 16px lateral
  - Track: h-3 (mais grosso que portrait para facilitar touch), rounded-full, bg white/20
  - Fill: bg `var(--primary)` (gold)
  - Thumb: 16px circle gold, sempre visível em mobile (sem hover)
  - Buffered: bg white/10
- Time: left=currentTime, right=totalDuration, Manrope 600, 13px, white
- "Collapse" button: bottom-right ou top-right, icon minimize, text "Sair de tela cheia"
- Orientation lock: sugerir landscape automaticamente quando player abre (via Screen Orientation API)

**Portrait mode (embedded):**
- Player fica dentro do content flow, aspect-16/9
- Tap nos controls: overlay aparece
- Botão fullscreen: expande para landscape

**Paywall indicator no progress bar:**
- Se Morador Base: mark at 25% position
- Mark: thin vertical line (2px × 12px), `var(--destructive)`, absolute no 25%
- Tooltip: "Preview até aqui" ao hover
- Após 25%: progress bar fill muda de gold para `var(--destructive)/50` e overlay de paywall aparece

---

## C.10 ROTAS E COMPONENTES ATUALIZADOS

### Novas Rotas (Sprint 4 preview + complementos Sprint 1-2):
```
31. /cursos                        → Lista de cursos (Sprint 4)
32. /cursos/:id                    → Detail do curso com aulas (Sprint 4)
33. /cursos/:id/aula/:aulaId       → Player de aula (Sprint 4)
34. /conquistas                    → Página de conquistas/badges do perfil
```

### Novos Componentes:
```
PaywallOverlay          → Overlay de bloqueio 25% no video player
UpgradeModal            → Modal comparação de planos (current vs recommended)
PlanComparisonCard      → Card de plano com features list e CTA
LessonList              → Lista numerada de aulas/pautas com progress
LessonRow               → Row individual com number, title, duration, status
StarRating              → Componente de estrelas (read-only e interactive)
LoginCard               → Card centralizado de login
SignupCard              → Card de cadastro com stepper
CategoryChip            → Pill de categoria com ícone (horizontal scroll)
CategoryCard            → Card de categoria ícone+label (grid busca)
BadgeCard               → Card de conquista (earned/locked)
BadgeGrid               → Grid de conquistas
NextAchievementCard     → Card da próxima conquista com progress
RadioCard               → Radio option como card (para forms)
InfoCallout             → Callout informativo com ícone e texto
ChannelInfoBar          → Bar com avatar, nome, plano e actions (abaixo player)
LandscapePlayer         → Player otimizado para landscape mobile
PaywallProgressMark     → Marcador de 25% no progress bar
SectionHeader           → Header com título e "Ver todos →"
SidebarCTAWidget        → Widget de comunicados/newsletter no sidebar bottom
CondominiumList         → Lista de condomínios na sidebar
```

### Componentes Totais no Design System:
**Batch 1 (doc original):** ~35 componentes
**Addendum A (VoiceLabs/Genaigurus):** +15 componentes
**Addendum B (Flavor):** +24 componentes
**Addendum C (Crossfader/Avalanche/AI App):** +18 componentes
**Total: ~92 componentes especificados**

---

*Addendum C incorporado ao Design System v1.3*

---

## ADDENDUM D — CHECKOUT, PRICING, FÓRUM, FILTROS E MOBILE NAV (Batch 5)

### Referências analisadas (10 imagens, 6 projetos):

- **Crossfader** (3 imagens novas):
  - Image 1 (583e1b): **AI Assistant / Chat interface** — "Welcome to Nudge" com suggestion chips, input com mic/send, new chat, histórico
  - Image 7 (437385): **Checkout page completa** — billing details form com campos side-by-side, order summary sidebar sticky, payment methods (card + Google Pay), coupon code input, Terms/Privacy links
  - Image 8 (425630): **Subscription Plans page** — stat highlight cards, "What's Included" checklist, bonus content list, benefits grid 2×2, 3-tier pricing cards (Lowest price / Most popular / Special offer)

- **SparkCause** (2 imagens):
  - Image 3 (50c105): **Social feed mobile** — stories row (avatars horizontais), category pills, post card com avatar+nome+local+tempo+imagem+engagement(❤️ 212, 💬 20, ↩️ Reply)+texto, bottom nav 5 items com center FAB (+)
  - Image 5 (45163c): **Design process cards** — numbered cards (01-05) com ícone+título+descrição, project timeline horizontal (Exploration→Definition→Design→Presentation) com sub-tarefas em cascata, mobile app view perspective

- **Humanified** (1 imagem):
  - Image 4 (50b4a0): **Photo/content platform** — sidebar (Feed/Discover/Settings), photo grid com badge "Original", hashtag detail page (#EnjoyThePlanet) com grid+info sidebar, "Humanifiers to follow" suggestions list com avatar+name+follow button, "Trending Cause" cards, registration modal com ilustração

- **LoveNest** (1 imagem):
  - Image 6 (45033f): **Profile cards grid** — 2-col grid de cards com foto grande+nome+idade+status dot+horário+descrição, tabs (Discover/Let's chat/Liked you), availability slot card. Purple accent.

- **Design Process / Workspace** (1 imagem):
  - Image 2 (516cd2): **Search by Filters mobile** — sliders com labels e ranges (Price $10-$1000+, Distance 5min-30+, Number of people 2-10+), Double Diamond model

- **Genaigurus** (2 imagens — views adicionais do app já analisado)

### Por que este batch é crucial:

1. **Checkout + Pricing** → Embora o Master Síndico use Mercado Pago como gateway, precisamos de telas de checkout e planos de assinatura para quando o usuário fizer upgrade. O Crossfader tem a referência mais completa e polida.
2. **Forum/Feed cards** → SparkCause mostra o melhor padrão de post card para o fórum (Sprint 4). A estrutura avatar+content+engagement é exatamente o que precisamos.
3. **Stories row** → Pode ser adaptado para "Assembleias Ativas" ou "Síndicos em Destaque" na home.
4. **Slider filters** → Para busca avançada com filtros de faixa (distância por CEP, etc).
5. **Bottom nav mobile com FAB** → Pattern fundamental que ainda não tínhamos detalhado.
6. **Profile cards grid** → Padrão para o Banco de Talentos (vídeo-currículos visíveis para empresas).
7. **Follow suggestions** → "Empresas recomendadas" ou "Síndicos da sua região".

---

## D.1 CHECKOUT / PAGAMENTO (Crossfader Image 7)

O Crossfader mostra uma checkout page completa com: formulário de billing, order summary sidebar, métodos de pagamento, e coupon input. Embora o Master Síndico use Mercado Pago (redirect ou embedded), precisamos da UI para:
- Página pré-checkout (resumo do plano selecionado antes de redirecionar)
- Confirmação pós-pagamento
- Gerenciamento de assinatura

### D.1.1 Página de Checkout / Confirmação de Plano

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  Complete sua assinatura                            │
│          │  (Michroma 22px, "assinatura" em var(--primary))   │
│          │                                                    │
│          │  ┌────────────────────────────┬────────────────┐  │
│          │  │                            │                │  │
│          │  │  FORMULÁRIO (flex-1)       │ RESUMO (300px) │  │
│          │  │                            │                │  │
│          │  │  DADOS DE COBRANÇA ─────  │ ┌────────────┐ │  │
│          │  │                            │ │            │ │  │
│          │  │  ┌────────────────────────┐│ │ Resumo do  │ │  │
│          │  │  │                        ││ │ pedido     │ │  │
│          │  │  │  Seus dados de         ││ │            │ │  │
│          │  │  │  cobrança.             ││ │ Síndico Pro│ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  │  Nome        Sobrenome ││ │ R$ 89,90   │ │  │
│          │  │  │  ┌──────┐  ┌──────┐   ││ │ /mês       │ │  │
│          │  │  │  │ João │  │Silva │   ││ │            │ │  │
│          │  │  │  └──────┘  └──────┘   ││ │────────────│ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  │  + Dados opcionais     ││ │ Subtotal:  │ │  │
│          │  │  │  (link gold, collapsibl)│ │ R$ 89,90  │ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  │  CPF/CNPJ              ││ │ Desconto:  │ │  │
│          │  │  │  ┌────────────────────┐││ │ - R$ 0,00 │ │  │
│          │  │  │  │ 123.456.789-00     │││ │            │ │  │
│          │  │  │  └────────────────────┘││ │────────────│ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  └────────────────────────┘│ │ Total:     │ │  │
│          │  │                            │ │ R$ 89,90   │ │  │
│          │  │  MÉTODO DE PAGAMENTO ────  │ │ (Manrope   │ │  │
│          │  │                            │ │  700 24px) │ │  │
│          │  │  ┌────────────────────────┐│ │            │ │  │
│          │  │  │                        ││ │ ┌────────┐ │ │  │
│          │  │  │  Aceitamos cartão de   ││ │ │MASTER10│ │ │  │
│          │  │  │  crédito e Mercado Pago││ │ │ ✕ Aplic│ │ │  │
│          │  │  │                        ││ │ └────────┘ │ │  │
│          │  │  │  ● [💳] [VISA] [MC]   ││ │ (coupon)   │ │  │
│          │  │  │  ○ [MP] Mercado Pago   ││ │            │ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  │  Número do cartão      ││ │ Ao clicar  │ │  │
│          │  │  │  ┌────────────────────┐││ │ você       │ │  │
│          │  │  │  │ 0000 0000 0000 0000│││ │ concorda   │ │  │
│          │  │  │  └────────────────────┘││ │ com os     │ │  │
│          │  │  │                        ││ │ Termos     │ │  │
│          │  │  │  Validade     CVV      ││ │            │ │  │
│          │  │  │  ┌────────┐ ┌──────┐  ││ │[Assinar →] │ │  │
│          │  │  │  │ MM/AA  │ │ 123  │  ││ │ (gold btn) │ │  │
│          │  │  │  └────────┘ └──────┘  ││ │ full-width │ │  │
│          │  │  │                        ││ │            │ │  │
│          │  │  │  ○ [G Pay] Google Pay  ││ └────────────┘ │  │
│          │  │  │                        ││                │  │
│          │  │  └────────────────────────┘│                │  │
│          │  │                            │                │  │
│          │  └────────────────────────────┴────────────────┘  │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

**Order Summary Sidebar (Crossfader pattern):**
```css
.order-summary {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 24px;
  position: sticky;
  top: 80px;
  width: 300px;
  flex-shrink: 0;
}

.order-summary-title {
  font-family: 'Michroma', sans-serif;
  font-size: 16px;
  letter-spacing: 0.05em;
  margin-bottom: 20px;
}

.order-summary-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 0;
  border-bottom: 1px solid var(--border);
}

.order-summary-item-name {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--foreground);
}

.order-summary-item-price {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
}

.order-summary-total {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  padding-top: 16px;
  margin-top: 8px;
}

.order-summary-total-label {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
}

.order-summary-total-value {
  font-family: 'Manrope', sans-serif;
  font-size: 24px;
  font-weight: 700;
  color: var(--foreground);
}
```

**Coupon Code Input (Crossfader pattern):**
```
┌────────────────────────────────────┐
│  MASTER10               ✕  [Aplicar] │
└────────────────────────────────────┘
```

- Container: flex, items-center, border 1px `var(--primary)/40`, rounded-lg, p-1, mt-4
- Input: bg transparent, border none, flex-1, Manrope 500, 14px, uppercase tracking wide, pl-3
- "✕" clear: icon 16px, `var(--muted-foreground)`, hover `var(--foreground)`, cursor pointer
- "Aplicar": text button, `var(--primary)`, font-weight 600, px-3
- Quando cupom válido: border `var(--success)`, background `var(--success)/5`
- Badge "Aplicado ✅": substitui botão Aplicar, text `var(--success)`
- Quando inválido: border `var(--destructive)`, shake animation 300ms, mensagem "Cupom inválido" abaixo

**Payment Method Radio Cards (Crossfader):**
```css
.payment-method-card {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 14px 16px;
  border: 1px solid var(--border);
  border-radius: 0.5rem;
  cursor: pointer;
  transition: all 150ms;
  margin-bottom: 8px;
}

.payment-method-card:hover {
  background: var(--muted);
}

.payment-method-card.selected {
  border-color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.05);
}

.payment-method-radio {
  width: 18px;
  height: 18px;
  border: 2px solid var(--border);
  border-radius: 9999px;
  flex-shrink: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 150ms;
}

.payment-method-card.selected .payment-method-radio {
  border-color: var(--primary);
}

.payment-method-card.selected .payment-method-radio::after {
  content: '';
  width: 8px;
  height: 8px;
  border-radius: 9999px;
  background: var(--primary);
}

.payment-brand-icons {
  display: flex;
  gap: 8px;
  align-items: center;
}

.payment-brand-icon {
  height: 24px;
  width: auto;
  border-radius: 4px;
  border: 1px solid var(--border);
  padding: 2px 4px;
  background: white; /* brand icons sempre com fundo branco */
}
```

**Card Fields (inline grid):**
```
Nome        Sobrenome
┌──────┐   ┌──────┐        ← grid-cols-2 gap-4

Número do cartão
┌──────────────────┐        ← full-width, masked "0000 0000 0000 0000"

Validade    CVV
┌────────┐ ┌──────┐        ← grid-cols-2 gap-4, masked "MM/AA", "123"
```

- Card number mask: auto-space every 4 digits, max 16 digits
- Expiry mask: auto-insert "/" after 2 digits, max "MM/AA"
- CVV: max 3-4 digits, type password (dots), toggle visibility
- Card brand detection: ao digitar primeiros dígitos, ícone VISA/MC/ELO aparece dentro do input à direita
- Auto-focus: ao preencher um campo, foco avança para o próximo

**Terms footer:**
```
Ao clicar abaixo, você concorda com os "Termos de Uso"
e confirma que leu a "Política de Privacidade".
```
- Manrope 13px, `var(--muted-foreground)`, text-center, mt-4
- Links: `var(--primary)`, hover underline
- Posicionado acima do CTA final

**Mobile checkout:** Form e summary empilham verticalmente. Summary vira sticky bottom bar:
```
┌──────────────────────────────────────┐
│  Síndico Pro • R$ 89,90/mês         │
│                          [Assinar →] │  ← sticky bottom, shadow-up, h-16
└──────────────────────────────────────┘
```

---

## D.2 SUBSCRIPTION / PRICING PAGE (Crossfader Image 8)

A Image 8 é a referência mais completa de pricing page que recebemos. Mostra: header com stats, checklist de inclusões, lista de bônus, grid de benefícios, e 3 cards de pricing com badges diferentes. Adaptar para a página "Ver Todos os Planos" do Master Síndico.

### D.2.1 Pricing Page Completa

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ← Voltar                                          │
│          │                                                    │
│          │  Planos Master Síndico                             │  Michroma 24px
│          │  Escolha o plano ideal para sua jornada.           │  Manrope 15px muted
│          │                                                    │
│          │  ── STAT HIGHLIGHTS ────────────────────────────  │
│          │                                                    │
│          │  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│          │  │  31       │  │  50+     │  │  ∞       │       │
│          │  │ Categorias│  │ Empresas │  │ Vídeos   │       │
│          │  │ técnicas  │  │ ativas   │  │ técnicos │       │
│          │  │     [📂]  │  │     [🏢]  │  │     [📹]  │       │
│          │  └──────────┘  └──────────┘  └──────────┘       │
│          │                                                    │
│          │  ── PLANOS ────────────────────────────────────── │
│          │                                                    │
│          │  ┌────────────┐ ┌────────────┐ ┌────────────┐    │
│          │  │            │ │ ⭐ POPULAR │ │ 💎 COMPLETO│    │
│          │  │            │ │            │ │            │    │
│          │  │  Membro    │ │  Síndico   │ │  Síndico   │    │
│          │  │  Base      │ │  N2 Plus   │ │  N3 Pro    │    │
│          │  │            │ │            │ │            │    │
│          │  │  Grátis    │ │ R$ 49,90   │ │ R$ 89,90   │    │
│          │  │            │ │ /mês       │ │ /mês       │    │
│          │  │            │ │            │ │            │    │
│          │  │ ──────────│ │ ──────────│ │ ──────────│    │
│          │  │            │ │            │ │            │    │
│          │  │ ✅ Busca   │ │ ✅ Tudo do │ │ ✅ Tudo do │    │
│          │  │    geral   │ │    Base    │ │    Plus    │    │
│          │  │ ✅ Preview │ │ ✅ Perfil  │ │ ✅ Link    │    │
│          │  │    25%     │ │    público │ │    evento  │    │
│          │  │ ✅ Votação │ │ ✅ 4 víds  │ │ ✅ Avaliação│   │
│          │  │ ✅ Assemb. │ │    /mês    │ │    objetiva│    │
│          │  │ ✅ Avaliação││ ✅ Connect │ │ ✅ Plano    │    │
│          │  │            │ │    Me      │ │    reeleição│   │
│          │  │ ✕ Perfil  │ │ ✅ Cursos  │ │ ✅ Export   │    │
│          │  │    público │ │    certif. │ │    PDF     │    │
│          │  │ ✕ Connect │ │ ✅ Fórum   │ │ ✅ Fórum   │    │
│          │  │    Me      │ │ ✅ Lives   │ │ ✅ Lives   │    │
│          │  │ ✕ Fórum   │ │            │ │            │    │
│          │  │ ✕ Lives   │ │            │ │            │    │
│          │  │            │ │            │ │            │    │
│          │  │ Plano atual│ │ [Assinar] │ │ [Assinar] │    │
│          │  │ (disabled) │ │ (outline)  │ │ (gold ★)  │    │
│          │  │            │ │            │ │            │    │
│          │  └────────────┘ └────────────┘ └────────────┘    │
│          │                                                    │
│          │  ── PERGUNTAS FREQUENTES ────────────────────── │
│          │                                                    │
│          │  ▶ Posso trocar de plano a qualquer momento?      │
│          │  ▶ Como funciona o cancelamento?                   │
│          │  ▶ Meus dados são mantidos se eu fizer downgrade? │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

**Stat Highlight Cards (Crossfader pattern):**
```css
.stat-highlight {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 20px 24px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  min-width: 0;
}

.stat-highlight-number {
  font-family: 'Manrope', sans-serif;
  font-size: 28px;
  font-weight: 800;
  color: var(--foreground);
  line-height: 1;
}

.stat-highlight-label {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  margin-top: 2px;
}

.stat-highlight-icon {
  width: 40px;
  height: 40px;
  border-radius: 0.75rem;
  background: oklch(from var(--primary) l c h / 0.1);
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--primary);
  flex-shrink: 0;
}
```

**Pricing Card (3-tier, Crossfader pattern):**
```css
.pricing-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 28px 24px;
  display: flex;
  flex-direction: column;
  position: relative;
  transition: all 200ms;
}

.pricing-card:hover {
  border-color: var(--muted-foreground);
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.15);
}

/* Card destacado (recommended/popular) */
.pricing-card.featured {
  border: 2px solid var(--primary);
  box-shadow: 0 0 30px oklch(from var(--primary) l c h / 0.1);
}

/* Badge no topo (Crossfader: "Most popular", "Special offer") */
.pricing-badge {
  position: absolute;
  top: -12px;
  left: 50%;
  transform: translateX(-50%);
  padding: 4px 16px;
  border-radius: 9999px;
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 700;
  white-space: nowrap;
}

.pricing-badge.popular {
  background: var(--primary);
  color: var(--primary-foreground);
}

.pricing-badge.special {
  background: var(--success);
  color: var(--success-foreground);
}

.pricing-plan-name {
  font-family: 'Michroma', sans-serif;
  font-size: 16px;
  letter-spacing: 0.05em;
  margin-bottom: 4px;
}

.pricing-price {
  font-family: 'Manrope', sans-serif;
  font-size: 36px;
  font-weight: 800;
  color: var(--foreground);
  line-height: 1;
  margin: 16px 0 4px;
}

.pricing-price-period {
  font-size: 14px;
  font-weight: 400;
  color: var(--muted-foreground);
}

.pricing-price-original {
  font-size: 16px;
  font-weight: 400;
  color: var(--muted-foreground);
  text-decoration: line-through;
  margin-right: 8px;
}

.pricing-save-badge {
  display: inline-block;
  padding: 2px 8px;
  border-radius: 4px;
  background: var(--success);
  color: var(--success-foreground);
  font-size: 12px;
  font-weight: 700;
  margin-left: 8px;
}

.pricing-features {
  flex: 1;
  margin: 20px 0;
}

.pricing-feature {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 6px 0;
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
}

.pricing-feature-icon.included {
  color: var(--success);
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  margin-top: 2px;
}

.pricing-feature-icon.excluded {
  color: var(--muted-foreground);
  opacity: 0.4;
}

.pricing-feature-text.excluded {
  color: var(--muted-foreground);
  text-decoration: line-through;
  opacity: 0.5;
}

.pricing-cta {
  margin-top: auto;
  width: 100%;
}
```

**Pricing grid layout:**
- Desktop: grid-cols-3, gap-6, max-width 960px, mx-auto
- Tablet: grid-cols-3, gap-4 (cards mais compactos)
- Mobile: grid-cols-1, gap-4 (stack vertical), featured card primeiro com visual "recomendado"

**FAQ Accordion (abaixo dos pricing cards):**
- Reusa AccordionPauta component (já definido)
- Perguntas hardcoded sobre planos, cancelamento, upgrade, downgrade

### D.2.2 Planos Específicos do Master Síndico (Mapeamento completo)

Baseado na Matriz Funcional, os pricing cards devem cobrir:

**Grupo Síndico (3 cards):**
```
[Síndico N1 "Aprender"]  [Síndico N2 "Atuar" ⭐]  [Síndico N3 "Consolidar" 💎]
```

**Grupo Empresa (3 cards):**
```
[Empresa Plus "Demonstrar"]  [Empresa Pro "Influenciar" ⭐]  [Marketing "Escalar"]
```

**Grupo Morador (2 cards):**
```
[Membro Base "Explorar"]  [Morador Pagante "Participar" ⭐]
```

Cada grupo é exibido em tabs na pricing page:
```
[Síndico]  [Empresa]  [Morador]  [Vizinhança]
 ═════════
```

---

## D.3 FÓRUM / FEED CARDS (SparkCause Image 3)

A Image 3 mostra o **melhor pattern de social feed card** que recebemos. Avatar + nome + local + timestamp no header, imagem, engagement row, texto de preview, e botão reply. Adaptar para o Fórum do Master Síndico (Sprint 4).

### D.3.1 Forum Post Card

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [Avatar 36px]  João Carlos Silva            [⋮]        │
│                 Síndico Pro • Cond. Vila Mariana         │
│                 há 3 horas                               │
│                                                          │
│  [Hidráulica]  [Manutenção Preventiva]                   │  ← category tags
│                                                          │
│  Alguém já teve experiência com                          │
│  impermeabilização de laje com manta                     │
│  asfáltica em condomínios grandes?                       │
│  Estou com orçamentos muito dispares                     │
│  e queria entender melhor o critério...                  │
│  [Ver mais]                                              │
│                                                          │
│  ┌──────────────────────────────────────────────────┐   │  ← imagem anexa (opcional)
│  │                                                  │   │     rounded-lg, max-h 300px
│  │  [Foto do problema / documento]                  │   │     object-cover
│  │                                                  │   │
│  └──────────────────────────────────────────────────┘   │
│                                                          │
│  ────────────────────────────────────────────────────── │
│                                                          │
│  👍 Curtir    💬 Responder    🔖 Salvar                 │  ← engagement bar
│                                                          │
│  VISIBILIDADE DAS MÉTRICAS (regra Master Síndico):      │
│  • Se DONO do post: "👍 12 curtidas • 💬 5 respostas"  │
│  • Se NÃO dono: botões sem contadores (interface cega)  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Forum Post Card CSS:**
```css
.forum-post-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 20px;
  margin-bottom: 12px;
  transition: all 150ms;
}

.forum-post-card:hover {
  border-color: var(--muted-foreground);
}

/* Header */
.forum-post-header {
  display: flex;
  align-items: flex-start;
  gap: 12px;
  margin-bottom: 12px;
}

.forum-post-avatar {
  width: 36px;
  height: 36px;
  border-radius: 9999px;
  object-fit: cover;
  flex-shrink: 0;
  border: 2px solid var(--border);
}

.forum-post-avatar.pro {
  border-color: var(--primary); /* gold ring para Pro */
}

.forum-post-author-name {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 600;
  color: var(--foreground);
}

.forum-post-author-meta {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: var(--muted-foreground);
  display: flex;
  align-items: center;
  gap: 4px;
}

.forum-post-menu {
  margin-left: auto;
  color: var(--muted-foreground);
  cursor: pointer;
}

/* Tags */
.forum-post-tags {
  display: flex;
  gap: 6px;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

/* Body */
.forum-post-body {
  font-family: 'Manrope', sans-serif;
  font-size: 15px;
  font-weight: 400;
  color: var(--foreground);
  line-height: 1.6;
  margin-bottom: 12px;
}

.forum-post-body.truncated {
  display: -webkit-box;
  -webkit-line-clamp: 4;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.forum-post-read-more {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--primary);
  cursor: pointer;
}

/* Image attachment */
.forum-post-image {
  width: 100%;
  max-height: 300px;
  object-fit: cover;
  border-radius: 0.5rem;
  margin-bottom: 12px;
}

/* Engagement bar */
.forum-post-engagement {
  display: flex;
  align-items: center;
  gap: 0;
  padding-top: 12px;
  border-top: 1px solid var(--border);
}

.forum-engagement-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 16px;
  border-radius: var(--radius);
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  cursor: pointer;
  transition: all 150ms;
}

.forum-engagement-btn:hover {
  background: var(--muted);
  color: var(--foreground);
}

.forum-engagement-btn.liked {
  color: var(--primary);
}

.forum-engagement-btn .icon {
  width: 18px;
  height: 18px;
}

/* Métricas visíveis apenas para dono */
.forum-engagement-count {
  font-weight: 600;
}
```

### D.3.2 Forum Reply / Comentário (Indentado)

```
    ┌──────────────────────────────────────────────────┐
    │  [Avatar 28px]  Maria Santos                      │
    │                 Empresa Pro • HidraFix Eng.       │
    │                 há 1 hora                         │
    │                                                    │
    │  Recomendo consultar a NBR 9575                   │
    │  antes de fechar orçamento...                     │
    │                                                    │
    │  👍 Curtir    ↩️ Responder                        │
    └──────────────────────────────────────────────────┘
```

- Indentação: margin-left 48px (alinha com o body do post parent)
- Avatar menor: 28px
- Sem imagem attachment (respostas são texto-only)
- Border-left: 2px solid `var(--border)`, padding-left 16px (visual thread)
- Nesting máximo: 2 níveis (post → reply → reply-to-reply)
- Após 2 níveis: "Ver mais respostas →" link

### D.3.3 Forum Page Layout

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  Fórum                                              │
│          │                                                    │
│          │  ┌────────────────────────────────┬──────────┐    │
│          │  │ 🔍 Buscar no fórum...          │[+ Novo] │    │
│          │  └────────────────────────────────┴──────────┘    │
│          │                                                    │
│          │  [Todos] [Hidráulica] [Elétrica] [Seguros] [...]  │
│          │   ═══════                                          │
│          │                                                    │
│          │  Ordenar: [Recentes ▼]                            │
│          │                                                    │
│          │  ┌─ Forum Post Card ──────────────────────────┐   │
│          │  │ [Avatar] Nome • Tipo • Tempo    [⋮]        │   │
│          │  │ [Tags]                                      │   │
│          │  │ Texto do post...                            │   │
│          │  │ [Imagem opcional]                            │   │
│          │  │ ─── 👍 Curtir  💬 Resp.  🔖 Salvar ──── │   │
│          │  └────────────────────────────────────────────┘   │
│          │                                                    │
│          │  ┌─ Forum Post Card ──────────────────────────┐   │
│          │  │ ...                                         │   │
│          │  └────────────────────────────────────────────┘   │
│          │                                                    │
└──────────────────────────────────────────────────────────────┘
```

**"+ Novo Tópico" button:**
- Desktop: botão à direita da search bar, primary (gold), icon "+" left
- Mobile: FAB (Floating Action Button) no bottom-right, 56px circle, bg `var(--primary)`, icon "+" white 24px, shadow-lg

---

## D.4 STORIES ROW / HIGHLIGHTS (SparkCause Image 3)

A Image 3 mostra uma **stories row** com avatares circulares horizontais, cada um com nome abaixo. No Master Síndico, NÃO temos stories (não é rede social). Mas o padrão visual é perfeito para:

### D.4.1 "Assembleias Ativas" — Home Row

```
── ASSEMBLEIAS ATIVAS ──────────────────── [Ver todas →]

[🏢 Cond. Vila]  [🏢 Ed. Paulist]  [🏢 Res. Jardim]
 "Ord. 2026/1"    "Ext. Manutenção"  "Ord. 2025/4"
  🟢 Aberta        🟢 Votando         ⏳ Agendada
```

**Specs:**
- Container: horizontal scroll, py-2, gap-4
- Cada item: flex-col, items-center, width 80px, cursor pointer
- Ícone/Avatar: 56px circle, bg `var(--surface-variant)`, border 2px solid `var(--border)`
  - Se assembleia aberta: border 2px solid `var(--success)`, ring glow animation sutil
  - Se votando: border 2px solid `var(--primary)` (gold), pulse
  - Se agendada: border 2px solid `var(--border)` (default)
  - Ícone dentro: 🏢 28px ou initial letter do condomínio
- Nome: Manrope 500, 12px, text-center, line-clamp-1, max-width 80px
- Status: badge tiny (8px dot + text 10px)
- Hover: scale(1.05), transition 150ms

### D.4.2 "Síndicos em Destaque" — Home Row (para Empresas)

Para empresas vendo a home, uma row similar mostrando síndicos com status elevado:

```
── SÍNDICOS EM DESTAQUE ──────────────── [Ver mais →]

[📸 Avatar]  [📸 Avatar]  [📸 Avatar]  [📸 Avatar]
 João C.      Maria S.     Pedro F.     Ana L.
 🥇 Ouro      💎 Diamante  🥈 Prata     🥇 Ouro
```

- Mesma estrutura visual, avatares reais
- Border color mapeada ao status: Bronze=`#CD7F32`, Prata=`#C0C0C0`, Ouro=`var(--primary)`, Diamante=gradient `linear-gradient(135deg, #b9f2ff, #e0e0ff)`

---

## D.5 BOTTOM NAVIGATION MOBILE COM FAB (SparkCause Image 3)

A Image 3 mostra o melhor padrão de **bottom nav mobile com center FAB** que recebemos. 5 itens, sendo o central um FAB circular proeminente (+).

### D.5.1 Bottom Nav — Mobile

```
┌──────────────────────────────────────────────────────────┐
│                                                          │
│  [🏠]     [🔍]     [+]      [🔖]     [👤]             │
│  Home    Buscar          Salvos   Perfil                │
│           ═══════                                        │
│                    ┌────┐                                │
│                    │ +  │  ← FAB: 52px circle            │
│                    │    │     bg var(--primary)           │
│                    └────┘     icon + white 24px           │
│                               shadow-lg                   │
│                               elevated -12px              │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Bottom Nav CSS:**
```css
.bottom-nav {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  height: calc(56px + env(safe-area-inset-bottom, 0px));
  padding-bottom: env(safe-area-inset-bottom, 0px);
  background: var(--card);
  border-top: 1px solid var(--border);
  display: flex;
  align-items: center;
  justify-content: space-around;
  z-index: 40;
  /* Blur backdrop para conteúdo por trás */
  backdrop-filter: blur(12px);
  background: oklch(from var(--card) l c h / 0.9);
}

.bottom-nav-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 2px;
  padding: 6px 0;
  min-width: 48px;
  cursor: pointer;
  transition: color 150ms;
}

.bottom-nav-item .icon {
  width: 22px;
  height: 22px;
  color: var(--muted-foreground);
  transition: color 150ms;
}

.bottom-nav-item .label {
  font-family: 'Manrope', sans-serif;
  font-size: 10px;
  font-weight: 500;
  color: var(--muted-foreground);
}

.bottom-nav-item.active .icon {
  color: var(--primary);
}

.bottom-nav-item.active .label {
  color: var(--primary);
  font-weight: 600;
}

/* FAB central */
.bottom-nav-fab {
  width: 52px;
  height: 52px;
  border-radius: 9999px;
  background: var(--primary);
  color: var(--primary-foreground);
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: -20px; /* elevação acima da nav */
  box-shadow: 0 4px 12px oklch(from var(--primary) l c h / 0.3);
  transition: all 200ms;
  border: 3px solid var(--card); /* cria gap visual */
}

.bottom-nav-fab:active {
  transform: scale(0.92);
}

.bottom-nav-fab .icon {
  width: 24px;
  height: 24px;
  color: white;
}
```

**O que o FAB faz por persona:**
- **Síndico N2/N3:** Abre menu "Criar" → [📹 Upload vídeo] [📋 Nova assembleia] [📝 Novo post fórum]
- **Empresa Pro:** Abre menu "Criar" → [📹 Upload vídeo] [📚 Novo curso] [📝 Novo post fórum]
- **Morador:** Sem FAB (item central vira link direto para funcionalidade principal, ex: "Assembleia" com ícone 🏛)
- **Morador Base:** Nav de 4 items sem FAB: Home, Buscar, Assembleia, Perfil

**Menu do FAB (ao tocar):**
- Bottom sheet com opções de criação
- Animação: FAB rotaciona 45° (+ vira ×), items surgem com stagger

**Visibility:**
- Bottom nav só aparece em mobile (max-width 768px)
- Desktop usa sidebar (já definida)
- Ao scrollar para baixo: bottom nav faz slide-down 300ms (hide)
- Ao scrollar para cima: slide-up 300ms (show)

---

## D.6 PROFILE CARDS GRID — BANCO DE TALENTOS (LoveNest Image 6)

A Image 6, embora seja de um app de dating, mostra o **melhor padrão de profile card grid** que recebemos. Cards com foto grande, nome sobreposto, indicador de status, e info adicional. Adaptar para o Banco de Talentos (vídeo-currículos) do Sprint 4.

### D.6.1 Talent/Currículo Card

```
┌──────────────┐
│              │
│  [FOTO]      │  ← aspect 3:4, object-cover
│  portrait    │     rounded-lg, overflow hidden
│              │
│  📹 90s      │  ← badge: duração do vídeo-currículo
│              │     absolute bottom-left
│  Ana Santos  │
│  Designer    │  ← overlay gradient bottom
│  São Paulo   │     absolute bottom
│              │
│  🟢 online  │  ← status dot (optional)
│              │
└──────────────┘
```

**Talent Card CSS:**
```css
.talent-card {
  position: relative;
  border-radius: 0.75rem;
  overflow: hidden;
  cursor: pointer;
  aspect-ratio: 3/4;
  background: var(--muted);
  transition: all 200ms;
}

.talent-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.2);
}

.talent-card-image {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

/* Gradient overlay no bottom */
.talent-card-overlay {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 50%;
  background: linear-gradient(to top, rgba(0,0,0,0.85) 0%, transparent 100%);
  padding: 16px;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
}

.talent-card-name {
  font-family: 'Manrope', sans-serif;
  font-size: 16px;
  font-weight: 700;
  color: white;
  line-height: 1.2;
}

.talent-card-role {
  font-family: 'Manrope', sans-serif;
  font-size: 13px;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.8);
  margin-top: 2px;
}

.talent-card-location {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 400;
  color: rgba(255, 255, 255, 0.6);
  margin-top: 2px;
}

/* Badge duração do vídeo */
.talent-card-duration {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 3px 8px;
  border-radius: 4px;
  background: rgba(0, 0, 0, 0.7);
  backdrop-filter: blur(4px);
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 600;
  color: white;
  display: flex;
  align-items: center;
  gap: 4px;
}

/* Indicador de vídeo (vs sem vídeo) */
.talent-card-has-video {
  position: absolute;
  top: 10px;
  left: 10px;
  width: 28px;
  height: 28px;
  border-radius: 9999px;
  background: var(--primary);
  display: flex;
  align-items: center;
  justify-content: center;
}

.talent-card-has-video .icon {
  width: 14px;
  height: 14px;
  color: white;
}
```

**Grid layout:**
- Desktop: grid-cols-4, gap-4
- Tablet: grid-cols-3
- Mobile: grid-cols-2, gap-3

**Banco de Talentos Page:**
```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Banco de Talentos                              │
│          │                                                │
│          │  ┌────────────────────────────────────────┐    │
│          │  │ 🔍 Buscar por nome, área, região...   │    │
│          │  └────────────────────────────────────────┘    │
│          │                                                │
│          │  Filtros: [Área ▼] [Região ▼] [Ordenar ▼]    │
│          │                                                │
│          │  47 profissionais encontrados                   │
│          │                                                │
│          │  ┌────┐ ┌────┐ ┌────┐ ┌────┐                │
│          │  │    │ │    │ │    │ │    │                │
│          │  │ 📸 │ │ 📸 │ │ 📸 │ │ 📸 │                │
│          │  │    │ │    │ │    │ │    │                │
│          │  │Nome│ │Nome│ │Nome│ │Nome│                │
│          │  │Área│ │Área│ │Área│ │Área│                │
│          │  └────┘ └────┘ └────┘ └────┘                │
│          │                                                │
│          │  ┌────┐ ┌────┐ ┌────┐ ┌────┐                │
│          │  │... │ │... │ │... │ │... │                │
│          │  └────┘ └────┘ └────┘ └────┘                │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

---

## D.7 SLIDER FILTERS (Design Process Image 2)

A Image 2 mostra filtros por **range sliders** com labels e valores min/max. Útil para busca avançada no Master Síndico.

### D.7.1 Range Slider Filter Component

Adaptar para filtros de busca:
- **Distância (por CEP):** 1km — 50km+
- **Período:** Últimos 7 dias — Último ano
- **Duração do vídeo:** 1 min — 60 min+

```
  Distância máxima
  ○━━━━━━━━━━━━━━━━━●━━━━━━━━━━━━ ○
  1km              15km           50km+
```

**Slider CSS:**
```css
.range-slider-container {
  padding: 8px 0;
}

.range-slider-label {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--foreground);
  margin-bottom: 12px;
}

.range-slider-track {
  width: 100%;
  height: 4px;
  background: var(--muted);
  border-radius: 2px;
  position: relative;
  cursor: pointer;
}

.range-slider-fill {
  height: 100%;
  background: var(--primary);
  border-radius: 2px;
  position: absolute;
  left: 0;
}

.range-slider-thumb {
  width: 20px;
  height: 20px;
  border-radius: 9999px;
  background: var(--primary);
  border: 3px solid var(--card);
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.2);
  position: absolute;
  top: 50%;
  transform: translate(-50%, -50%);
  cursor: grab;
  transition: box-shadow 150ms;
  z-index: 1;
}

.range-slider-thumb:active {
  cursor: grabbing;
  box-shadow: 0 0 0 6px oklch(from var(--primary) l c h / 0.15);
  transform: translate(-50%, -50%) scale(1.1);
}

.range-slider-thumb:focus-visible {
  outline: none;
  box-shadow: 0 0 0 4px var(--ring);
}

.range-slider-values {
  display: flex;
  justify-content: space-between;
  margin-top: 8px;
}

.range-slider-value {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 400;
  color: var(--muted-foreground);
}

.range-slider-current {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 700;
  color: var(--primary);
  text-align: center;
  margin-top: 4px;
}
```

**Tooltip acima do thumb (on drag):**
- Aparece ao iniciar drag
- bg `var(--secondary)`, text white, px-2 py-1, rounded-md, font-size 12px bold
- Arrow pointing down: CSS triangle
- Posição: acima do thumb, translateY(-100% - 8px)
- Mostra valor atual: "15km"
- Transition: opacity 150ms

---

## D.8 FOLLOW/SUGGESTION LIST — "RECOMENDADOS" (Humanified Image 4)

A Image 4 mostra "Humanifiers to follow" com avatar + nome + descrição + botão follow (+). Adaptar para "Empresas Recomendadas" na sidebar ou home.

### D.8.1 Suggestion Row Component

```
┌──────────────────────────────────────────────────────────┐
│  EMPRESAS RECOMENDADAS ──────────────── [Ver mais →]     │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  [Logo 36px]  HidraFix Engenharia         [Ver →] │  │
│  │               Empresa Pro • Hidráulica             │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  [Logo 36px]  EletroPred Soluções         [Ver →] │  │
│  │               Empresa Plus • Elétrica              │  │
│  ├────────────────────────────────────────────────────┤  │
│  │  [Logo 36px]  Segurmax Condominial        [Ver →] │  │
│  │               Empresa Pro • Segurança              │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Suggestion Row CSS:**
```css
.suggestion-row {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 10px 16px;
  border-bottom: 1px solid var(--border);
  transition: background-color 150ms;
}

.suggestion-row:last-child {
  border-bottom: none;
}

.suggestion-row:hover {
  background: var(--muted);
}

.suggestion-avatar {
  width: 36px;
  height: 36px;
  border-radius: 0.5rem; /* squared-rounded para empresas */
  object-fit: cover;
  flex-shrink: 0;
  background: var(--muted);
}

.suggestion-info {
  flex: 1;
  min-width: 0;
}

.suggestion-name {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.suggestion-meta {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 400;
  color: var(--muted-foreground);
}

.suggestion-action {
  flex-shrink: 0;
}
```

**Onde exibir:**
- Home sidebar (desktop): widget "Empresas Recomendadas"
- Home mobile: seção horizontal scroll de cards pequenos
- Busca: seção "Sugeridos para você" acima dos resultados
- Algoritmo: baseado nas categorias do condomínio do síndico (ex: se o condomínio teve problema hidráulico recente → sugerir empresas de hidráulica)

---

## D.9 PHOTO GRID COM BADGES (Humanified Image 4)

A Image 4 mostra um grid de fotos com badge "Original" sobre a primeira imagem, e ícones de compartilhamento em cada foto. Adaptar para galeria de fotos/vídeos no perfil de empresa.

### D.9.1 Media Grid (Perfil de Empresa)

```
┌──────────────────────────────────────────────────────────┐
│  Vídeos (12)                     [⊞ Grid] [≡ Lista]     │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │          │ │          │ │          │                │
│  │ [THUMB]  │ │ [THUMB]  │ │ [THUMB]  │                │
│  │ 16:9     │ │          │ │          │                │
│  │          │ │          │ │          │                │
│  │ [Instit.]│ │ [02:30]  │ │ [04:15]  │                │
│  │ [📌 Fixo]│ │          │ │          │                │
│  └──────────┘ └──────────┘ └──────────┘                │
│  Vídeo instit.  Manutenção   Detecção de                │
│                 de fachadas   vazamentos                  │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │ ...      │ │ ...      │ │ ...      │                │
│  └──────────┘ └──────────┘ └──────────┘                │
│                                                          │
└──────────────────────────────────────────────────────────┘
```

**Badge "Institucional" / "Fixo":**
- Posição: absolute top-8px left-8px
- bg `var(--primary)`, text white, font-size 11px, font-weight 600, px-2 py-0.5, rounded-sm
- Para vídeo institucional: "📌 Institucional"
- Para vídeo-currículo (morador): "📌 Currículo"
- Indica que é o vídeo fixo no perfil (trava trimestral de 90 dias)

---

## D.10 AI ASSISTANT / HELP INTERFACE (Crossfader Image 1)

O Crossfader mostra "Nudge" — um assistente AI com interface de chat, suggestion chips, input com mic/send. Embora o Master Síndico não tenha AI integrado agora, o pattern é útil para uma futura página de **Ajuda / FAQ interativo** ou **Central de Suporte**.

### D.10.1 Help Center (Conceito futuro, documentar agora)

```
┌──────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                │
│          │  Central de Ajuda                               │
│          │                                                │
│          │  ┌────────────────────────────────────────┐    │
│          │  │                                        │    │
│          │  │        🛡️ (brasão 48px)               │    │
│          │  │                                        │    │
│          │  │  Como podemos ajudar?                  │    │
│          │  │  (Michroma 20px, "ajudar" em gold)     │    │
│          │  │                                        │    │
│          │  │  ┌────────────┐  ┌────────────┐       │    │
│          │  │  │ 🏢 Sobre   │  │ 💳 Planos  │       │    │  ← Suggestion chips
│          │  │  │ assembleias│  │ e cobrança │       │    │    (Crossfader pattern)
│          │  │  └────────────┘  └────────────┘       │    │
│          │  │  ┌────────────┐  ┌────────────┐       │    │
│          │  │  │ 📹 Vídeos  │  │ 👤 Meu     │       │    │
│          │  │  │ e uploads  │  │ perfil     │       │    │
│          │  │  └────────────┘  └────────────┘       │    │
│          │  │                                        │    │
│          │  │  ┌──────────────────────────────────┐  │    │
│          │  │  │ 🔍 Buscar dúvida...              │  │    │
│          │  │  └──────────────────────────────────┘  │    │
│          │  │                                        │    │
│          │  └────────────────────────────────────────┘    │
│          │                                                │
│          │  ── PERGUNTAS FREQUENTES ────────────────────  │
│          │                                                │
│          │  ▶ Como funciona a votação em assembleia?      │
│          │  ▶ Como faço upload do meu vídeo institucional?│
│          │  ▶ O que é o Connect Me?                       │
│          │  ▶ Como altero meu plano?                      │
│          │                                                │
└──────────────────────────────────────────────────────────┘
```

**Suggestion Chips (Crossfader "Ask about a DJ technique"):**
```css
.suggestion-chip {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 10px 16px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--foreground);
  cursor: pointer;
  transition: all 150ms;
  white-space: nowrap;
}

.suggestion-chip:hover {
  border-color: var(--primary);
  background: oklch(from var(--primary) l c h / 0.05);
}

.suggestion-chip .emoji {
  font-size: 18px;
  line-height: 1;
}
```

---

## D.11 SIDEBAR CTA WIDGET — GIFT/PROMO (Crossfader Images 1, 7, 8)

Todas as 3 imagens do Crossfader mostram um **widget promocional no bottom da sidebar**: "Gift a DJ course!" com imagem de presente, descrição, e CTA. Adaptar para Master Síndico.

### D.11.1 Sidebar Promo Widget

```
┌─────────────────────────────┐
│  ┌──────────────────────┐   │
│  │  [Imagem/Ilustração] │   │
│  │  de promoção         │   │
│  └──────────────────────┘   │
│                             │
│  Indique um síndico!        │  ← Manrope 600, 14px
│                             │
│  Convide colegas para a     │  ← Manrope 400, 12px, muted
│  plataforma e ganhe          │
│  benefícios.                │
│                             │
│  [Indicar →]                │  ← sm button, gold
│                             │
└─────────────────────────────┘
```

**CSS:**
```css
.sidebar-promo {
  background: var(--surface-variant);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 16px;
  margin-top: auto; /* push to bottom */
}

.sidebar-promo-image {
  width: 100%;
  aspect-ratio: 16/10;
  border-radius: 0.5rem;
  object-fit: cover;
  margin-bottom: 12px;
}

.sidebar-promo-title {
  font-family: 'Manrope', sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 4px;
}

.sidebar-promo-description {
  font-family: 'Manrope', sans-serif;
  font-size: 12px;
  font-weight: 400;
  color: var(--muted-foreground);
  line-height: 1.4;
  margin-bottom: 12px;
}
```

**Variações do widget por contexto:**
- Pré-upgrade: "Conheça o plano Pro" com preview de features
- Indicação: "Indique um síndico" com código de referral
- Evento ativo: "Assembleia aberta!" com countdown e CTA "Votar →"
- Sazonal: banners rotativos definidos pelo admin MasterSíndico

---

## D.12 ROTAS E COMPONENTES ATUALIZADOS

### Novas Rotas:
```
35. /checkout/:planId              → Checkout / Confirmação de assinatura
36. /checkout/sucesso              → Página de confirmação pós-pagamento
37. /planos                        → Pricing page com todos os planos
38. /forum                         → Listagem de tópicos do fórum (Sprint 4)
39. /forum/:postId                 → Detalhe do post com replies (Sprint 4)
40. /forum/novo                    → Criar novo tópico (Sprint 4)
41. /talentos                      → Banco de talentos / currículos (Sprint 4)
42. /ajuda                         → Central de ajuda / FAQ
```

### Novos Componentes:
```
OrderSummaryCard         → Resumo do pedido (sidebar checkout)
CouponCodeInput          → Input de cupom com validação
PaymentMethodCard        → Radio card de método de pagamento (CC/MP/GPay)
CardNumberInput          → Input masked para número de cartão com brand detect
PricingCard              → Card de plano com features, preço, CTA
PricingGrid              → Grid responsivo de 3 pricing cards
StatHighlightCard        → Card de estatística (número + label + ícone)
ForumPostCard            → Post do fórum (avatar + content + engagement)
ForumReply               → Reply indentado com thread visual
ForumEngagementBar       → Bar de curtir/responder/salvar (privacy-aware)
StoriesRow               → Row horizontal de itens circulares (assembleias/síndicos)
BottomNav                → Navegação mobile fixa com 5 items
BottomNavFab             → FAB central da bottom nav
TalentCard               → Card de currículo com foto portrait + overlay info
TalentGrid               → Grid de talent cards
RangeSlider              → Slider de faixa com thumb draggable
SuggestionRow            → Row de empresa/síndico recomendado
SuggestionChip           → Chip de sugestão com ícone (help center)
SidebarPromoWidget       → Widget promocional no bottom da sidebar
MediaGridBadge           → Badge de "Institucional" / "Fixo" em thumbnails
```

### Totais Acumulados:
**Rotas:** 42 mapeadas (Sprints 1-2 completas + Sprint 3-4 preview)
**Componentes:** ~115 especificados
**Linhas:** ~8.000+

---

*Addendum D incorporado ao Design System v1.4*
*5 projetos analisados neste batch: Crossfader (checkout/pricing/AI assistant), SparkCause (feed/stories/bottom nav/FAB), Humanified (grid/suggestions), LoveNest (profile cards), Design Process (slider filters).*
*Próximo passo: enviar mais referências ou iniciar implementação via Claude Code.*
