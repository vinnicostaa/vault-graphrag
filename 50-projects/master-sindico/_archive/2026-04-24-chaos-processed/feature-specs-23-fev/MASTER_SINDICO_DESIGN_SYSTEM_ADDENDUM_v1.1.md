# MASTER SÍNDICO — Frontend Design System ADDENDUM v1.1
## Novos Padrões Extraídos das Referências VoiceLabs & Genaigurus

**Documento complementar ao:** MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md
**Data:** Fevereiro 2026

---

## ANÁLISE DETALHADA DAS NOVAS REFERÊNCIAS

### O que foi analisado nesta rodada:

1. **Genaigurus (2 imagens):** App social/educacional com perfil (posts + badges), player de vídeo landscape, criação de conteúdo com AI, web app com sidebar + blog + vídeos + artigos + categorias, mobile responsive com home/search/feedback, leaderboard com podium e badges de nível (Silver com progress bar e requisitos), player mobile com upvote/downvote/comment
2. **VoiceLabs (8 imagens):** Landing page clean com hero + dual CTA, dashboard desktop com sidebar (user switcher, seções agrupadas, New badges, Credit Uses widget circular 80%), grid de action cards 2x3 com ícones ilustrados, explore page com tabs + trending cards + curated collections, mobile com grid 2x2, scope of work metodológico

---

## A. NOVOS COMPONENTES IDENTIFICADOS

### A.1 QUOTA/USAGE WIDGET (Extraído do VoiceLabs "Credit Uses")

**Referência:** VoiceLabs mostra um widget fixo no bottom da sidebar com um progress circular mostrando "80% — Credit Uses" + "Your team has used 80% of available space. Need more?" + links "Dismiss" e "Upgrade plan".

**Adaptação para Master Síndico:**

Este padrão é PERFEITO para exibir quotas de uso que existem na plataforma. Cada persona tem limites diferentes conforme a Matriz Funcional:

- **Morador Pagante:** 4 Connect Me por ano (quota anual)
- **Morador Base:** 2 Connect Me por ano
- **Empresa Plus:** 8 vídeos/mês (máx. 2/semana), 100 vídeos armazenados
- **Empresa Pro:** 12 vídeos/mês (máx. 3/semana), 150 vídeos armazenados
- **Síndico N2:** 4 vídeos/mês
- **Trava de 90 dias:** Progresso temporal do vídeo institucional/currículo

**Design do Widget — Sidebar Bottom (Desktop):**

    ┌────────────────────────────────┐
    │                                │
    │  ┌────┐  Uso do Plano          │
    │  │ 75%│  3 de 4 Connect Me     │
    │  │ ◕  │  utilizados este ano   │
    │  └────┘                        │
    │                                │
    │  [Gerenciar Plano →]           │
    │                                │
    │  ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
    │                                │
    │  📹 Vídeos: 6/8 este mês      │
    │  ━━━━━━━━━━━━━━━━━━━━░░░░░░  │
    │                                │
    └────────────────────────────────┘

**Especificação do Circular Progress:**
- Tamanho: 48x48px
- Stroke width: 4px
- Track: var(--surface-variant) (navy claro)
- Fill: var(--primary) (gold) — animação de arco no mount
- Quando >80%: fill muda para var(--warning) (amber)
- Quando 100%: fill muda para var(--destructive) (vermelho)
- Texto central: porcentagem em Manrope 700, 14px
- Container: fundo var(--surface-variant), rounded-lg, p-3, border 1px solid rgba(gold, 0.15)

**Variações do Widget por Persona:**

Para Morador Base/Pagante:

    ┌────────────────────────────────┐
    │  ┌────┐  Connect Me            │
    │  │ 50%│  2 de 4 restantes      │
    │  │ ◕  │  Renova em Jan/2027    │
    │  └────┘                        │
    │  [Ver Detalhes →]              │
    └────────────────────────────────┘

Para Empresa Plus/Pro:

    ┌────────────────────────────────┐
    │  ┌────┐  Publicações           │
    │  │ 75%│  6 de 8 vídeos         │
    │  │ ◕  │  este mês              │
    │  └────┘                        │
    │  Armazenamento: 67/100 vídeos  │
    │  ━━━━━━━━━━━━━━━━━━░░░░░░░░░  │
    │  [Upgrade para Pro →]          │
    └────────────────────────────────┘

Para Síndico N3 (Trava de 90 dias):

    ┌────────────────────────────────┐
    │  ┌────┐  Vídeo Institucional   │
    │  │ 71%│  Próxima atualização   │
    │  │ ◕  │  em 26 dias            │
    │  └────┘                        │
    │  Enviado: 15/12/2025           │
    │  Disponível: 15/03/2026        │
    └────────────────────────────────┘

**Comportamento de Dismiss:**
- Ícone X no canto superior direito (8px, muted-foreground)
- Ao clicar: widget colapsa para versão minimal (só o circle + número)
- Persiste em state local
- Re-expande automaticamente quando quota atinge 80%

---

### A.2 QUICK ACTIONS GRID (Extraído do VoiceLabs Dashboard)

**Referência:** VoiceLabs mostra "Good Morning, Jhon" com grid 2x3 de action cards com ícones ilustrados dentro de containers rounded-xl escuros (surface-variant), cada um sendo um atalho para uma feature core.

**Adaptação para Master Síndico — Home Dashboard (após login):**

Em vez de ir direto para o feed de vídeos (padrão YouTube), o Master Síndico pode ter uma seção de saudação + ações rápidas no topo da Home, seguida pelo feed. Isso reforça que a plataforma é uma ferramenta de governança, não um player de vídeo.

**Layout Desktop — Topo da Home:**

    ┌──────────────────────────────────────────────────────────────┐
    │                                                              │
    │  Minha Área                                                  │
    │  Bom dia, João                               Precisa de     │
    │  (Michroma 24px)                              ajuda? [?]    │
    │                                                              │
    │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
    │  │  [icon]  │ │  [icon]  │ │  [icon]  │ │  [icon]  │       │
    │  │  Nova    │ │ Meus     │ │Transparên│ │ Connect  │       │
    │  │Assembleia│ │ Vídeos   │ │  cia     │ │   Me     │       │
    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
    │                                                              │
    │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐       │
    │  │  [icon]  │ │  [icon]  │ │  [icon]  │ │  [icon]  │       │
    │  │ Buscar   │ │Avaliações│ │Benefícios│ │Configura │       │
    │  │Empresas  │ │          │ │  Locais  │ │  ções    │       │
    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘       │
    │                                                              │
    └──────────────────────────────────────────────────────────────┘

**Especificação do Action Card:**
- Container: 140x120px (desktop), aspect-ratio auto
- Background: var(--card) (light) ou var(--surface-variant) (dark)
- Border: 1px solid var(--border)
- Border-radius: rounded-xl (12px)
- Padding: 16px
- Ícone: 40px, centralizado, usando família lucide com cor var(--primary) (gold)
- Label: Manrope 500, 13px, text-center, mt-3, cor var(--foreground)
- Hover: border-color var(--primary), scale(1.03), shadow-sm, transition 200ms
- Active: scale(0.98), 100ms

**Ícones para cada Action Card:**

    Nova Assembleia:    i-lucide-clipboard-plus     (gold)
    Meus Vídeos:        i-lucide-video              (gold)
    Transparência:      i-lucide-bar-chart-3        (gold)
    Connect Me:         i-lucide-send               (gold)
    Buscar Empresas:    i-lucide-building-2         (gold)
    Avaliações:         i-lucide-star               (gold)
    Benefícios Locais:  i-lucide-map-pin            (gold)
    Configurações:      i-lucide-settings           (gold)

**Adaptação por Role (ABAC):**

O grid de ações muda conforme a persona logada:

Síndico N3 (todas as features):
[Nova Assembleia] [Meus Vídeos] [Transparência] [Connect Me]
[Buscar Empresas] [Avaliações]  [Benefícios]    [Exportar PDF]

Síndico N1 (capacitação apenas):
[Assistir Vídeos] [Buscar Empresas] [Fórum*] [Benefícios]

Morador Pagante:
[Meu Currículo] [Buscar Síndicos] [Assembleias] [Benefícios]
[Connect Me]    [Configurações]

Morador Base:
[Buscar Empresas] [Assembleias] [Benefícios] [Configurações]

Empresa Pro:
[Upload Vídeo] [Meus Conteúdos] [Buscar Fornecedores] [Fórum*]
[Cursos*]      [Benefícios]     [Connect Me]           [Config.]

**Mobile — Grid 2x2 com scroll:**
- Grid-cols-2, gap-3
- Container: scroll horizontal snap se mais de 4 items
- Dots indicator abaixo para indicar páginas

**Saudação Dinâmica:**
- Nome: Michroma 24px (desktop) / 20px (mobile), var(--foreground)
- Sub-label: Manrope 400, 14px, var(--muted-foreground)
- Exemplo: "Síndico Pro • Status Ouro • 3 condomínios"

---

### A.3 SIDEBAR REFINADA (Inspirada VoiceLabs + Ajustes)

**Padrões absorvidos do VoiceLabs:**

1. User Switcher no topo — Avatar + nome + chevron (para trocar de condomínio ativo)
2. Seções com labels uppercase — adaptamos para GESTÃO, CONTEÚDO, CONFIGURAÇÕES
3. Badge "New" — Pill gold ao lado de items novos
4. Toggle de collapse — Ícone discreto no header da sidebar
5. Credit Uses widget — No bottom da sidebar (item A.1)

**Sidebar Atualizada — Desktop Expandida (240px):**

    ┌──────────────────────────┐
    │  [🛡️ MS]  [□ collapse]   │  Logo compact + toggle
    │                           │
    │  ┌─────────────────────┐ │
    │  │ [👤] João Silva   ▾ │ │  User switcher
    │  │      Síndico Pro    │ │  (dropdown mostra condos)
    │  └─────────────────────┘ │
    │                           │
    │  ▣ Início                │  gold bg pill se ativo
    │  🔍 Buscar               │
    │                           │
    │  ── CONTEÚDO ──          │  Michroma 10px, gold, uppercase
    │  📹 Vídeos               │
    │  📻 Podcast         NEW  │  badge gold pill
    │                           │
    │  ── GESTÃO ──            │
    │  📋 Assembleias      •3  │  badge numeral
    │  📊 Transparência        │
    │  ⭐ Avaliações           │
    │  📄 Exportar PDF         │
    │                           │
    │  ── CONEXÕES ──          │
    │  📨 Connect Me           │
    │  🏢 Catálogo             │
    │  🏪 Vizinhança           │
    │                           │
    │  ── SISTEMA ──           │
    │  ⚙️ Configurações        │
    │  ❓ Central de Ajuda     │
    │                           │
    │  ─────────────────────── │
    │                           │
    │  ┌─────────────────────┐ │
    │  │ ┌──┐ Uso do Plano   │ │  Quota Widget
    │  │ │75│ 3/4 Connect Me  │ │
    │  │ │◕ │ este ano        │ │
    │  │ └──┘                 │ │
    │  │ [Gerenciar →]        │ │
    │  └─────────────────────┘ │
    │                           │
    └──────────────────────────┘

**User Switcher — Dropdown (para Síndicos multi-condomínio):**
- Fundo: var(--popover), border 1px var(--border), shadow-lg, rounded-lg
- Item selecionado: dot gold, texto bold
- Item não selecionado: dot muted, texto muted
- Hover: bg var(--muted)
- Animação: scale-y origin-top, 200ms

**Badge "NEW":**
- Pill: px-1.5 py-0.5, rounded-full, bg var(--accent), text var(--secondary), 10px, bold, uppercase

**Badge numérico:**
- Circle: min-w-5 h-5, rounded-full, bg var(--destructive), text white, 11px, bold

---

### A.4 PROFILE — TAB "STATUS" COM BADGES (Extraído Genaigurus)

**Tab "Status" no Perfil do Síndico:**

    ┌──────────────────────────────────────────────────────────┐
    │  [Vídeos]  [Gestão]  [Status]  [Sobre]                  │
    │                                                          │
    │  STATUS ATUAL                                            │
    │  ┌────────────────────────────────────────────────────┐  │
    │  │        ┌──────┐                                    │  │
    │  │        │  🥇  │   OURO                             │  │
    │  │        │      │   Síndico comprometido com         │  │
    │  │        └──────┘   capacitação e gestão ativa       │  │
    │  │                                                    │  │
    │  │   Para Diamante: 💎                                │  │
    │  │   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━░░░░░░ 78%     │  │
    │  └────────────────────────────────────────────────────┘  │
    │                                                          │
    │  CRITÉRIOS DE EVOLUÇÃO                                   │
    │  ┌────────────────────────────────────────────────────┐  │
    │  │  📹 Vídeos Assistidos (>70%)       47/60   78%    │  │
    │  │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━░░░░  ✅    │  │
    │  │                                                    │  │
    │  │  🏢 Condomínios Vinculados          3/3   100%    │  │
    │  │  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━  ✅    │  │
    │  │                                                    │  │
    │  │  📚 Módulos Concluídos              5/8    63%    │  │
    │  │  ━━━━━━━━━━━━━━━━━━━━━━━━━━░░░░░░░░░░░     ⏳    │  │
    │  └────────────────────────────────────────────────────┘  │
    │                                                          │
    │  HISTÓRICO DE STATUS                                     │
    │  🥇 Ouro      — desde Mar/2026                          │
    │  🥈 Prata     — Jan/2026 a Mar/2026                     │
    │  🥉 Bronze    — Out/2025 a Jan/2026                     │
    └──────────────────────────────────────────────────────────┘

**Badge de Status Grande — Design:**

    BRONZE:   gradient(#CD7F32, #A0522D), glow rgba(205,127,50,0.3)
    PRATA:    gradient(#E8E8E8, #A0A0A0), glow rgba(192,192,192,0.3)
    OURO:     gradient(var(--accent), var(--primary)), glow rgba(198,156,63,0.4)
    DIAMANTE: gradient(#B9F2FF, #E0C3FC, #B9F2FF), shimmer animation 3s, glow rgba(185,242,255,0.4)

---

### A.5 EXPLORE/BROWSE REFINADA (Extraído VoiceLabs)

Atualização da Search Page com curated collections:

    ┌──────────────────────────────────────────────────────────────┐
    │  Início > Buscar                                             │
    │  ┌──────────────────────────────────────────────────────┐   │
    │  │  🔍 Buscar síndicos, empresas, serviços...           │   │
    │  └──────────────────────────────────────────────────────┘   │
    │                                                              │
    │  TABS: [Tudo] [Empresas] [Síndicos] [Comércio]             │
    │                                                              │
    │  🔥 EM DESTAQUE (scroll horizontal)                         │
    │  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐        │
    │  │ Alpha Fach.  │ │ Beta Elét.   │ │ Gamma Seg.   │        │
    │  │ Empresa Pro  │ │ Empresa Plus │ │ Empresa Pro  │        │
    │  │ #fachada     │ │ #elétrica    │ │ #segurança   │        │
    │  └──────────────┘ └──────────────┘ └──────────────┘        │
    │                                                              │
    │  📂 CATEGORIAS TÉCNICAS (grid 4 cols)                       │
    │  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
    │  │Administ. │ │Jurídico  │ │Contabil. │ │Elétrica  │      │
    │  │  31 emp. │ │  18 emp. │ │  24 emp. │ │  45 emp. │      │
    │  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
    │  [Ver todas as 31 categorias →]                              │
    │                                                              │
    │  📍 PERTO DE VOCÊ (baseado no CEP)                          │
    │  [Card Comércio] [Card Comércio] [Card Comércio]            │
    └──────────────────────────────────────────────────────────────┘

**Category Collection Card:**
- 200x140px desktop, rounded-xl, bg var(--card) com subtle gradient
- Ícone 32px gold top-left, nome 14px bottom-left, "31 empresas" 12px muted
- Hover: border gold, scale(1.02)

---

### A.6 STEPPER DE EVOLUÇÃO (Adaptado Genaigurus Leaderboard)

O Master Síndico NÃO é competitivo. Mas o visual do "nível com progress" é aproveitável:

**Stepper Horizontal de Níveis:**
- 4 pontos conectados por linha horizontal
- Ponto concluído: 24px circle, bg var(--success), ícone ✓
- Ponto atual: 28px circle, bg var(--primary), glow, pulse suave 2s
- Ponto futuro: 24px circle, bg var(--muted), border 2px dashed
- Linha concluída: var(--success), Linha futura: var(--muted)

---

### A.7 MOBILE VIDEO PLAYER REFINADO (Genaigurus)

**Portrait Mode:**
- Player full width 16:9, barra gold
- Abaixo: título + canal + botões [Curtir] [Compartilhar] [Denunciar]
- "Mais Vídeos" em lista horizontal

**Landscape Mode:**
- Fullscreen com controles sobrepostos
- Barra: h-1 repouso → h-3 on touch, fill gold, thumb 12→16px
- Buffered: rgba(white, 0.3)

---

### A.8 BANNER DE COMUNICADO (Genaigurus Newsletter)

    ┌──────────────────────────────────────────────────────────┐
    │  📢 COMUNICADO OFICIAL                            [✕]   │
    │  Nova funcionalidade: Assembleia Digital agora           │
    │  disponível para Síndicos N2 e N3.                       │
    │  [Saiba Mais →]                                          │
    └──────────────────────────────────────────────────────────┘

- Fundo: var(--primary)/5, border 1px var(--primary)/20
- Dismissible, persiste em state

---

### A.9 MODAL DE DENÚNCIA (Genaigurus Feedback)

Motivos alinhados com Cap. 7 do Manual:
- Conteúdo comercial/propaganda
- Dados de contato no vídeo (tel, email, site)
- Conteúdo ofensivo ou inapropriado
- Informação técnica incorreta
- Spam ou conteúdo repetitivo
- Outro (com textarea 0/300)

---

## B. MICRO-INTERAÇÕES EXTRAÍDAS

### B.1 Like Animation (Heart Bounce)

    @keyframes heartBounce {
      0%   { transform: scale(1); }
      25%  { transform: scale(1.3); }
      50%  { transform: scale(0.95); }
      100% { transform: scale(1); }
    }

### B.2 Badge Shimmer (Diamante)

    @keyframes shimmer {
      0%   { background-position: -200% center; }
      100% { background-position: 200% center; }
    }

### B.3 Card Stagger Animation

    .stagger-item:nth-child(1) { animation-delay: 0ms; }
    .stagger-item:nth-child(2) { animation-delay: 50ms; }
    .stagger-item:nth-child(3) { animation-delay: 100ms; }
    ...até nth-child(8) { animation-delay: 350ms; }

### B.4 Sidebar Collapse Transition

    .sidebar { width: 240px; transition: width 300ms cubic-bezier(0.4, 0, 0.2, 1); }
    .sidebar.collapsed { width: 64px; }
    .sidebar .label-text { transition: opacity 200ms ease; }
    .sidebar.collapsed .label-text { opacity: 0; }

### B.5 Tooltip on Collapsed Sidebar Items

- Posição: right offset 8px, fundo popover, shadow-md, rounded-md
- Animação: fade + slide-right 150ms

---

## C. AJUSTES NO DOCUMENTO PRINCIPAL

### C.1 Home Page — Nova ordem de conteúdo

1. Saudação + Quick Actions Grid (NOVO)
2. Banner de Comunicado (NOVO, se existir)
3. Category Icons Row (existente)
4. Feed de Vídeos em seções (existente)
5. Seção "Perto de Você" (existente)

### C.2 Profile — Nova tab

Síndico: [Vídeos] [Gestão] [Status] [Sobre]  (Status é NOVO)

### C.3 Breakpoints Atualizados

    <768px   : MOBILE  → Bottom nav, drawer
    768-1023 : TABLET  → Sidebar colapsada 64px
    1024-1279: DESKTOP → Sidebar colapsada por padrão
    ≥1280    : WIDE    → Sidebar expandida 240px

---

## D. MAPEAMENTO VISUAL — REFERÊNCIAS POR TELA

| Tela | Referência | Padrão |
|------|-----------|--------|
| Home (saudação) | VoiceLabs Dashboard | Greeting + Action Grid |
| Home (feed) | YouTube + Avalanche | Category row + video grid |
| Sidebar | VoiceLabs | User switcher + seções + quota |
| Busca/Explore | VoiceLabs Voices | Tabs + trending + collections |
| Perfil (Status) | Genaigurus Badges | Badge + stepper + progress |
| Player Mobile | Genaigurus Video | Portrait + landscape + gold bar |
| Denúncia | Genaigurus Feedback | Modal com select + textarea |

---

*Documento complementar. Leia junto com MASTER_SINDICO_FRONTEND_DESIGN_SYSTEM.md*
