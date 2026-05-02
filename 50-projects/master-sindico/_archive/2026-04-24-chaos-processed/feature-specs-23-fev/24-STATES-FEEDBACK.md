# 24 · ESTADOS DE UI, FEEDBACK & MICRO-INTERAÇÕES

> **Sprint**: Transversal (aplica-se a TODAS as sprints)
> **Prioridade**: Crítica — Todo componente da plataforma DEVE ter seus estados definidos
> **Dependências**: 00-DESIGN-TOKENS.md (cores, tipografia, motion), 02-APP-SHELL.md (layout)
> **Referências visuais**: YouTube (skeleton shimmer), Instagram (pull-to-refresh), Meta (error boundaries), Stripe (toast feedback)

---

## 1. FILOSOFIA DE ESTADOS

### 1.1 Princípio Fundamental

Nenhuma tela do Master Síndico pode existir em estado "indefinido". Todo componente visual deve ter resposta clara para 6 estados possíveis:

```
┌─────────────────────────────────────────────────────┐
│  ESTADO          │  QUANDO OCORRE                    │
├─────────────────────────────────────────────────────┤
│  Loading         │  Dados sendo buscados             │
│  Skeleton        │  Carregamento inicial da tela     │
│  Success         │  Dados carregados com sucesso     │
│  Empty           │  Sem dados para exibir            │
│  Error           │  Falha na operação                │
│  Restricted      │  Sem permissão (ABAC/Paywall)     │
└─────────────────────────────────────────────────────┘
```

### 1.2 Regra de Ouro

> **"O usuário nunca deve olhar para uma tela em branco."**
> Se algo está carregando, mostre skeleton. Se deu erro, mostre o que aconteceu e o que fazer. Se não tem dados, mostre o caminho. Se não tem permissão, mostre por quê.

### 1.3 Hierarquia de Feedback

```
Feedback Imediato (< 100ms)
├── Hover states, focus rings, active states
├── Botão pressed → feedback visual instantâneo
└── Toggle switches → mudança imediata

Feedback Rápido (100ms - 1s)
├── Toast notifications
├── Like/Unlike animation
├── Form field validation inline
└── Dropdown open/close

Feedback Demorado (> 1s)
├── Skeleton loading
├── Progress bars
├── Spinner com mensagem contextual
└── Upload progress
```

---

## 2. SKELETON LOADING

### 2.1 Conceito

Skeletons são placeholders visuais que replicam a estrutura exata do conteúdo que será carregado. Eles comunicam ao usuário o que esperar, reduzem percepção de espera e evitam layout shift (CLS).

### 2.2 Skeleton Base Component

```
┌──────────────────────────────────────────────────┐
│  SkeletonPrimitive                                │
│                                                   │
│  Props:                                           │
│  ├── variant: 'text' | 'circle' | 'rect' | 'card'│
│  ├── width: string (CSS)                          │
│  ├── height: string (CSS)                         │
│  ├── borderRadius: string (CSS)                   │
│  └── animated: boolean (default: true)            │
│                                                   │
│  Estilos:                                         │
│  ├── background: var(--muted)                     │
│  ├── Shimmer: gradiente linear animado            │
│  │   ├── from: var(--muted)                       │
│  │   ├── via: var(--muted) + 15% opacity branco   │
│  │   └── to: var(--muted)                         │
│  ├── animation: shimmer 1.5s ease infinite        │
│  └── border-radius: var(--radius)                 │
│                                                   │
│  @keyframes shimmer:                              │
│    0%   { background-position: -200% 0 }          │
│    100% { background-position: 200% 0 }           │
│                                                   │
│  background-size: 200% 100%                       │
└──────────────────────────────────────────────────┘
```

### 2.3 Skeleton Variants por Contexto

#### 2.3.1 SkeletonVideoCard (Home, Busca, Perfil)

Replica exatamente o VideoCard do doc 05-VIDEO-PLAYER.md:

```
┌─────────────────────────────────────┐
│  ┌─────────────────────────────┐    │
│  │                             │    │  ← Skeleton rect
│  │      THUMBNAIL 16:9        │    │     aspect-ratio: 16/9
│  │      shimmer animation     │    │     border-radius: 8px
│  │                             │    │
│  └─────────────────────────────┘    │
│                                      │
│  ○ ████████████████████████          │  ← Skeleton circle 40px (avatar)
│    ████████████████████              │    + Skeleton text width: 85%
│    ████████████                      │    + Skeleton text width: 55%
│                                      │
└─────────────────────────────────────┘

Mobile (< 768px):
- Grid 1 coluna, cards full-width
- Thumbnail aspect-ratio mantido 16:9
- Gap entre cards: 16px

Tablet (768px - 1023px):
- Grid 2 colunas
- Gap: 16px

Desktop (≥ 1024px):
- Grid 3-4 colunas (conforme tela)
- Gap: 20px
```

**Quantidade de skeleton cards por tela**:
- Home: 8 cards (2 linhas × 4 cols desktop, 4 linhas × 2 cols tablet, 8 linhas mobile)
- Busca: 6 cards
- Perfil (galeria): 6 cards

#### 2.3.2 SkeletonProfileHeader (Perfil)

Replica o header de perfil (doc 04-PROFILE-MANAGEMENT.md):

```
┌──────────────────────────────────────────────────────┐
│  ┌───────────────────────────────────────────────┐   │
│  │            BANNER SKELETON                     │   │  ← rect height: 200px
│  │            shimmer animation                   │   │     desktop: 280px
│  └───────────────────────────────────────────────┘   │
│                                                       │
│     ┌─────┐                                           │
│     │ ○○○ │  ← circle 96px (avatar skeleton)          │
│     └─────┘     overlap -48px sobre banner             │
│                                                       │
│     ████████████████████████  ← text h2 width: 200px  │
│     █████████████████        ← text small width: 140px │
│                                                       │
│     ████████████  ████████████  ████████████           │
│     (3 stat boxes skeleton)                            │
│                                                       │
└──────────────────────────────────────────────────────┘
```

#### 2.3.3 SkeletonForumPost (Fórum)

Replica card de post do fórum (doc 17-FORUM.md):

```
┌────────────────────────────────────────────────┐
│  ████████████████  ← pill categoria (60×24)     │
│                                                 │
│  ████████████████████████████████████           │  ← text h3 width: 80%
│  ████████████████████████████████████████████   │  ← text p width: 100%
│  █████████████████████████                      │  ← text p width: 60%
│                                                 │
│  ○ ████████████  ·  ████████                    │  ← circle 24px + text + text
│                                                 │
│  ██████  ██████  ██████                         │  ← 3 action pills
└────────────────────────────────────────────────┘

Quantidade: 5 skeleton posts na listagem inicial
```

#### 2.3.4 SkeletonAssemblyCard (Assembleias)

Replica card de assembleia (doc 09-ASSEMBLY-MANAGEMENT.md):

```
┌────────────────────────────────────────────────┐
│  ████████  ← badge status (80×24)               │
│                                                 │
│  ████████████████████████████████████           │  ← text h3 width: 75%
│                                                 │
│  ○  ████████████████                            │  ← icon placeholder + text
│  ○  ████████████████████████                    │  ← icon placeholder + text
│  ○  ████████████                                │  ← icon placeholder + text
│                                                 │
│  ████████████████████████████████████████       │  ← rect button full-width
└────────────────────────────────────────────────┘

Quantidade: 3 skeleton cards
```

#### 2.3.5 SkeletonCourseCard (LMS)

Replica card de curso (doc 15-LMS-COURSES.md):

```
┌─────────────────────────────────────┐
│  ┌─────────────────────────────┐    │
│  │      THUMBNAIL 16:9        │    │  ← rect shimmer
│  │                             │    │
│  └─────────────────────────────┘    │
│                                      │
│  ████████████████████████████████   │  ← text h3 width: 85%
│  ████████████████                   │  ← text small width: 50%
│                                      │
│  ┌─────────────────────────────┐    │
│  │ ████                        │    │  ← progress bar skeleton
│  └─────────────────────────────┘    │
│                                      │
│  ████████████  ████████████         │  ← 2 stat pills
└─────────────────────────────────────┘

Quantidade: 4 skeleton cards no catálogo
```

#### 2.3.6 SkeletonDashboard (Painéis Administrativos)

Para painéis como Transparência (doc 11), Admin (doc 20):

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  ┌────────┐  ┌────────┐  ┌────────┐  ┌────────┐     │  ← 4 stat cards
│  │ ██████ │  │ ██████ │  │ ██████ │  │ ██████ │     │     rect 160×80
│  │ █████  │  │ █████  │  │ █████  │  │ █████  │     │     shimmer
│  └────────┘  └────────┘  └────────┘  └────────┘     │
│                                                       │
│  ┌──────────────────────────────────────────────┐    │
│  │                                               │    │  ← chart area
│  │            CHART SKELETON                     │    │     rect height: 300px
│  │            shimmer animation                  │    │
│  │                                               │    │
│  └──────────────────────────────────────────────┘    │
│                                                       │
│  ┌──────────────────────────────────────────────┐    │
│  │  ████████████████████████  ████████           │    │  ← table rows
│  │  ████████████████████████  ████████           │    │     5 linhas skeleton
│  │  ████████████████████████  ████████           │    │
│  │  ████████████████████████  ████████           │    │
│  │  ████████████████████████  ████████           │    │
│  └──────────────────────────────────────────────┘    │
└──────────────────────────────────────────────────────┘

Mobile: stat cards 2×2 grid, chart e tabela full-width stacked
```

#### 2.3.7 SkeletonNotificationItem (Notificações)

```
┌────────────────────────────────────────────────┐
│  ○  ████████████████████████████████████████   │  ← circle 32px + text 90%
│     ████████████                               │  ← text timestamp 30%
├────────────────────────────────────────────────┤
│  ○  ████████████████████████████████████████   │
│     ████████████                               │
├────────────────────────────────────────────────┤
│  (repete 6x)                                   │
└────────────────────────────────────────────────┘
```

### 2.4 Regras de Implementação Skeleton

1. **Nunca usar spinner sozinho em tela cheia.** Spinners só aparecem em botões ou ações pontuais. Telas inteiras usam skeleton.

2. **Skeleton deve refletir layout real.** Se o card tem avatar + 2 linhas de texto + thumbnail, o skeleton tem exatamente esses shapes.

3. **Shimmer direction**: Sempre da esquerda para a direita (LTR). `background: linear-gradient(90deg, var(--muted) 25%, rgba(255,255,255,0.08) 50%, var(--muted) 75%)`

4. **Duração do shimmer**: `animation: shimmer 1.5s ease-in-out infinite`. Não usar animações mais rápidas que 1s (causa ansiedade) nem mais lentas que 2.5s (parece travado).

5. **Quantidade de skeletons**: Sempre mostrar a quantidade que caberia na viewport visível. Não mostrar 20 skeleton cards se só cabem 8.

6. **Border radius**: Todos os skeletons usam `border-radius: var(--radius)` (0.5rem) para consistência com cards reais.

7. **Transição skeleton → conteúdo real**: Fade suave `opacity 0 → 1` em 200ms quando dados chegam. Usar `<Show when={!isLoading()} fallback={<Skeleton />}>`.

---

## 3. EMPTY STATES

### 3.1 Conceito

Empty states são as telas exibidas quando não há dados para mostrar. São oportunidades de guiar o usuário para a ação correta, nunca apenas "tela vazia".

### 3.2 Anatomia do Empty State

```
┌──────────────────────────────────────────────────┐
│                                                   │
│              ┌──────────────┐                     │
│              │              │                     │
│              │  ILUSTRAÇÃO  │  ← 120×120 max      │
│              │  (ícone SVG) │     monocromática    │
│              │              │     var(--muted-fg)  │
│              └──────────────┘                     │
│                                                   │
│           Título do Estado Vazio                   │  ← Michroma 18px
│                                                   │     var(--foreground)
│        Descrição explicativa curta que             │  ← Manrope 14px
│        orienta o próximo passo do usuário          │     var(--muted-foreground)
│                                                   │     max-width: 320px
│                                                   │     text-align: center
│              ┌──────────────────┐                 │
│              │   Ação Primária  │  ← Botão gold   │
│              └──────────────────┘     se aplicável │
│                                                   │
│              Link secundário →                    │  ← text link opcional
│                                                   │
└──────────────────────────────────────────────────┘

Container: flex col, items-center, justify-center
Min-height: 400px (para ocupar viewport razoável)
Padding: 48px 24px
```

### 3.3 Catálogo de Empty States por Contexto

#### 3.3.1 Home — Sem Conteúdo

```
Ícone: Play circle com linha diagonal (vídeo indisponível)
Título: "Nenhum conteúdo disponível"
Descrição: "Novos vídeos e conteúdos serão exibidos aqui em breve."
Ação: Nenhuma (estado passivo)
```

#### 3.3.2 Busca — Sem Resultados

```
Ícone: Lupa com X
Título: "Nenhum resultado encontrado"
Descrição: "Tente ajustar os filtros ou buscar com outros termos."
Ação: [Limpar filtros] (botão outline)
```

#### 3.3.3 Perfil — Sem Vídeos Publicados

```
Para o DONO do perfil:
Ícone: Câmera com +
Título: "Você ainda não publicou vídeos"
Descrição: "Compartilhe seu conhecimento técnico com a comunidade."
Ação: [Publicar meu primeiro vídeo] (botão gold)

Para VISITANTE do perfil:
Ícone: Play circle vazio
Título: "Nenhum vídeo publicado"
Descrição: "Este perfil ainda não possui vídeos disponíveis."
Ação: Nenhuma
```

#### 3.3.4 Assembleias — Sem Eventos

```
Para SÍNDICO (dono):
Ícone: Calendário com +
Título: "Nenhuma assembleia criada"
Descrição: "Crie sua primeira assembleia e organize a gestão do condomínio."
Ação: [Criar assembleia] (botão gold)

Para MORADOR:
Ícone: Calendário vazio
Título: "Sem assembleias agendadas"
Descrição: "Quando seu síndico criar uma assembleia, ela aparecerá aqui."
Ação: Nenhuma
```

#### 3.3.5 Votação — Sem Pautas para Votar

```
Ícone: Cédula de votação vazia
Título: "Nenhuma votação aberta"
Descrição: "As votações serão exibidas aqui quando estiverem disponíveis."
Ação: Nenhuma
```

#### 3.3.6 Fórum — Sem Posts

```
Ícone: Balão de conversa vazio
Título: "Nenhuma discussão por aqui"
Descrição: "Seja o primeiro a iniciar uma conversa no fórum."
Ação: [Criar novo tópico] (botão gold) — se tiver permissão
```

#### 3.3.7 Fórum — Sem Posts na Categoria

```
Ícone: Filtro vazio
Título: "Nenhum post nesta categoria"
Descrição: "Tente selecionar outra categoria ou inicie a discussão."
Ação: [Ver todas as categorias] (botão outline)
```

#### 3.3.8 Cursos — Sem Cursos Inscritos

```
Ícone: Livro aberto vazio
Título: "Você ainda não iniciou nenhum curso"
Descrição: "Explore o catálogo e comece sua capacitação técnica."
Ação: [Explorar cursos] (botão gold)
```

#### 3.3.9 Cursos — Catálogo Vazio

```
Ícone: Estante vazia
Título: "Nenhum curso disponível no momento"
Descrição: "Novos cursos serão publicados em breve. Fique atento!"
Ação: Nenhuma
```

#### 3.3.10 Notificações — Sem Notificações

```
Ícone: Sino com check
Título: "Tudo em dia!"
Descrição: "Você não tem notificações pendentes."
Ação: Nenhuma
```

#### 3.3.11 Connect Me — Sem Histórico

```
Ícone: Envelope vazio
Título: "Nenhum formulário enviado"
Descrição: "Seus formulários de contato enviados aparecerão aqui."
Ação: [Buscar profissionais] (botão gold) → redireciona para Busca
```

#### 3.3.12 Vizinhança — Sem Promoções no CEP

```
Ícone: Pin de mapa com ?
Título: "Nenhuma promoção na sua região"
Descrição: "Ainda não há ofertas cadastradas para o CEP informado. Tente ampliar a área de busca."
Ação: [Alterar CEP] (botão outline)
```

#### 3.3.13 Vizinhança — CEP Não Cadastrado

```
Ícone: Pin de mapa com !
Título: "Informe seu CEP"
Descrição: "Para ver benefícios e promoções próximos a você, precisamos do seu CEP."
Ação: [Informar CEP] (botão gold) → abre modal com input CEP
```

#### 3.3.14 Currículos — Sem Currículos (para Empresa)

```
Ícone: Pessoa com +
Título: "Nenhum currículo disponível"
Descrição: "Quando moradores cadastrarem seus currículos, eles aparecerão aqui."
Ação: Nenhuma
```

#### 3.3.15 Métricas Privadas — Sem Interações

```
Ícone: Gráfico de barras vazio
Título: "Ainda sem interações"
Descrição: "Quando seu conteúdo receber curtidas e comentários, as métricas aparecerão aqui."
Ação: Nenhuma
```

#### 3.3.16 Transparência — Timeline Vazia (Síndico)

```
Ícone: Timeline vazia (linha pontilhada vertical com círculos vazios)
Título: "Sua linha do tempo está em branco"
Descrição: "Registre suas ações e decisões para construir seu histórico de gestão."
Ação: [Adicionar primeiro registro] (botão gold)
```

#### 3.3.17 Admin — Sem Denúncias Pendentes

```
Ícone: Escudo com check
Título: "Nenhuma denúncia pendente"
Descrição: "Não há conteúdos reportados para moderar no momento."
Ação: Nenhuma
```

#### 3.3.18 Lives — Sem Lives Disponíveis

```
Ícone: Câmera de vídeo com linha
Título: "Nenhuma live agendada"
Descrição: "Quando novas transmissões forem agendadas, você verá aqui."
Ação: Nenhuma
```

#### 3.3.19 Podcast — Sem Episódios

```
Ícone: Headphones vazio
Título: "Nenhum episódio disponível"
Descrição: "Novos episódios do podcast serão publicados em breve."
Ação: Nenhuma
```

### 3.4 Regras de Implementação Empty States

1. **Ilustrações**: Usar ícones SVG monocromáticos do Iconify (`@iconify/json`). Cor: `var(--muted-foreground)` com 60% opacity. Tamanho: 80px mobile, 120px desktop.

2. **Título**: Michroma 16px mobile / 18px desktop. `var(--foreground)`. `letter-spacing: 0.05em`. Máximo 40 caracteres.

3. **Descrição**: Manrope 13px mobile / 14px desktop. `var(--muted-foreground)`. `line-height: 1.5`. Máximo 2 linhas (80 caracteres). `text-align: center`. `max-width: 320px`.

4. **Botão de ação**: Só aparece quando existe ação prática possível. Gold primary para ação principal. Outline para ação secundária.

5. **Permissões**: O botão de ação só renderiza se o ABAC permitir. Ex: "Criar assembleia" só aparece para Síndico N3. Morador vê empty state sem botão.

6. **Nunca ser condescendente**: Não usar linguagem do tipo "Ops!", "Oopsie!", ou emojis tristes. Tom é profissional e orientador.

7. **Dark mode**: Ícones mantêm `var(--muted-foreground)`, que já adapta automaticamente via tokens oklch.

---

## 4. ERROR STATES

### 4.1 Tipos de Erro

```
┌─────────────────────────────────────────────────────┐
│  TIPO               │  HTTP    │  TRATAMENTO         │
├─────────────────────────────────────────────────────┤
│  Rede indisponível  │  0       │  Banner + Retry     │
│  Não autorizado     │  401     │  Redirect Login     │
│  Sem permissão      │  403     │  Paywall/Restricted │
│  Não encontrado     │  404     │  Empty state custom │
│  Validação          │  422     │  Inline nos campos  │
│  Rate limit         │  429     │  Toast + cooldown   │
│  Erro servidor      │  500     │  Error boundary     │
│  Timeout            │  408     │  Retry com backoff  │
│  Serviço externo    │  502/503 │  Toast informativo  │
└─────────────────────────────────────────────────────┘
```

### 4.2 Error Boundary (Falha Geral de Página)

Componente wrapper que captura erros não tratados e exibe fallback amigável:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│                    ⚠                                  │  ← Ícone warning
│                                                       │     var(--destructive)
│           Algo não saiu como esperado                 │  ← Michroma 20px
│                                                       │
│    Ocorreu um erro ao carregar esta página.           │  ← Manrope 14px
│    Nossa equipe foi notificada.                       │     var(--muted-foreground)
│                                                       │
│    ┌────────────────┐  ┌──────────────────┐          │
│    │  Tentar de novo │  │  Voltar ao início │          │  ← 2 botões
│    └────────────────┘  └──────────────────┘          │
│    (primary gold)       (outline secondary)           │
│                                                       │
│    Detalhes técnicos ▾                               │  ← Collapsible
│    ┌──────────────────────────────────────────┐      │     (aberto mostra)
│    │  Error: NetworkError                      │      │     código do erro
│    │  em /api/v1/assemblies                    │      │     monospace 12px
│    │  Timestamp: 2026-02-20T14:30:00Z          │      │     var(--muted)
│    └──────────────────────────────────────────┘      │     background
│                                                       │
└──────────────────────────────────────────────────────┘

Container: flex col, items-center, justify-center
Min-height: calc(100vh - topbar - bottomnav)
Padding: 48px 24px
```

**Implementação SolidJS:**
```
ErrorBoundary de solid-js
├── fallback recebe (err, reset)
├── "Tentar de novo" chama reset()
├── "Voltar ao início" usa navigate('/')
└── Log do erro → console.error + analytics futuro
```

### 4.3 Network Error Banner

Banner sticky no topo da tela quando não há conexão:

```
┌──────────────────────────────────────────────────────┐
│  ⚡ Sem conexão com a internet. Verificando...       │  ← Banner fixo
│                                                       │     top: 0 (abaixo topbar)
└──────────────────────────────────────────────────────┘     background: var(--destructive)
                                                             color: var(--destructive-foreground)
                                                             height: 40px
                                                             padding: 0 16px
                                                             font: Manrope 13px
                                                             display: flex
                                                             align-items: center
                                                             gap: 8px
                                                             z-index: 100

Quando conexão volta:
┌──────────────────────────────────────────────────────┐
│  ✓ Conexão restabelecida                              │  ← background: var(--success)
└──────────────────────────────────────────────────────┘     auto-dismiss após 3s
                                                             transition: height 300ms, opacity 300ms
```

**Detecção**: `navigator.onLine` + `window.addEventListener('online'/'offline')` + health check periódico (ping `/api/health` a cada 30s quando offline detectado).

### 4.4 Inline Form Errors

Erros de validação de formulário exibidos diretamente nos campos:

```
┌──────────────────────────────────────────────────┐
│  Label do campo                                   │
│  ┌──────────────────────────────────────────┐    │
│  │  valor inválido                           │    │  ← border: 2px solid var(--destructive)
│  └──────────────────────────────────────────┘    │     focus ring: var(--destructive)
│  ⚠ Mensagem de erro específica                   │  ← Manrope 12px
│                                                   │     var(--destructive)
└──────────────────────────────────────────────────┘     margin-top: 4px
                                                         display: flex
                                                         align-items: center
                                                         gap: 4px
                                                         icon: 14px
```

**Regras de validação Zod → mensagens amigáveis**:
- `z.string().min(1)` → "Este campo é obrigatório"
- `z.string().email()` → "Informe um e-mail válido"
- `z.string().min(6)` → "Mínimo de 6 caracteres"
- `z.string().max(N)` → "Máximo de N caracteres"
- `z.string().regex(CEP)` → "CEP inválido. Formato: 00000-000"
- `z.string().regex(CNPJ)` → "CNPJ inválido"
- Custom → Mensagem descritiva vinda do schema

**Timing**: Validação inline aparece `onBlur` (ao sair do campo) e atualiza em tempo real conforme o usuário corrige. Não validar `onChange` para cada tecla (causa ansiedade).

### 4.5 404 — Página Não Encontrada

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│                    404                                │  ← Michroma 72px
│                                                       │     var(--muted-foreground)
│                                                       │     opacity: 0.3
│           Página não encontrada                       │  ← Michroma 20px
│                                                       │
│    O conteúdo que você procura não existe             │  ← Manrope 14px
│    ou foi removido.                                   │     var(--muted-foreground)
│                                                       │
│           ┌──────────────────┐                       │
│           │  Voltar ao início │                       │  ← botão gold
│           └──────────────────┘                       │
│                                                       │
└──────────────────────────────────────────────────────┘

Rota: catch-all route no TanStack Router (routeTree)
```

### 4.6 Timeout & Retry

Para requests que demoram mais de 10s:

```
Estado 1 — Carregando (0-10s):
  Skeleton normal

Estado 2 — Demorado (10-20s):
  Skeleton + mensagem abaixo:
  "Isso está demorando mais que o normal..."
  Manrope 13px, var(--muted-foreground), text-align center

Estado 3 — Timeout (> 20s):
  Skeleton desaparece → Error state:
  "Não foi possível carregar os dados"
  [Tentar novamente] → retry com exponential backoff (1s, 2s, 4s, max 3 tentativas)
```

**TanStack Query config**:
```
retry: 3
retryDelay: (attempt) => Math.min(1000 * 2 ** attempt, 8000)
staleTime: 5 * 60 * 1000 (5 min para dados não-críticos)
```

---

## 5. ESTADOS RESTRITOS (PAYWALL & ABAC)

### 5.1 Conceito

Quando o usuário não tem permissão para acessar um recurso, o sistema NÃO mostra erro. Mostra o caminho para desbloqueio quando aplicável, ou simplesmente não renderiza o elemento.

### 5.2 Estratégias de Restrição

```
┌─────────────────────────────────────────────────────┐
│  ESTRATÉGIA       │  QUANDO USAR                     │
├─────────────────────────────────────────────────────┤
│  Não renderizar   │  Menu/botão que o perfil não tem │
│  Paywall overlay  │  Conteúdo parcialmente visível   │
│  Gate page        │  Rota inteira bloqueada           │
│  Upgrade prompt   │  Feature existe mas plano limita │
└─────────────────────────────────────────────────────┘
```

### 5.3 Não Renderizar (ABAC Silent)

Elementos que o perfil do usuário não tem acesso simplesmente **não existem** na interface. O componente não renderiza. Nenhuma mensagem, nenhum placeholder.

Exemplos:
- Morador Base → Não vê item "Fórum" no menu
- Morador Base → Não vê botão "Connect Me" no perfil do Síndico
- Empresa Plus → Não vê "Criar curso" no dashboard
- Síndico N1 → Não vê "Criar assembleia" no menu

**Implementação**: Guard component que lê `auth.role` e `auth.plan` do contexto. Se não tem permissão, retorna `null`. Sem fallback visual.

```
<PermissionGate requires={['assembly:create']} plan={['sindico_n3']}>
  <CreateAssemblyButton />
</PermissionGate>
```

### 5.4 Paywall Overlay (Vídeo 25%)

Para vídeos de empresas quando usuário Base assiste:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │                                                │   │
│  │              VÍDEO PLAYER                      │   │
│  │              (pausado em 25%)                   │   │
│  │                                                │   │
│  │  ┌────────────────────────────────────────┐   │   │
│  │  │                                         │   │   │  ← Overlay gradiente
│  │  │  ┌──────────────────────────────────┐  │   │   │     bottom → top
│  │  │  │                                   │  │   │   │     rgba(7,26,51, 0.95)
│  │  │  │  🔒 Conteúdo exclusivo            │  │   │   │
│  │  │  │                                   │  │   │   │  ← Michroma 18px
│  │  │  │  Para assistir o vídeo completo,  │  │   │   │     branco
│  │  │  │  faça upgrade do seu plano.       │  │   │   │
│  │  │  │                                   │  │   │   │  ← Manrope 14px
│  │  │  │  [Conhecer planos]                │  │   │   │     var(--primary)
│  │  │  │                                   │  │   │   │     botão gold
│  │  │  └──────────────────────────────────┘  │   │   │
│  │  └────────────────────────────────────────┘   │   │
│  │                                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
│  ▰▰▰▰▰▰▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱  25%              │  ← Progress bar trava
│                                                       │
└──────────────────────────────────────────────────────┘

Overlay: gradient linear, 
  from: transparent (top 30%)
  to: oklch(0.183 0.031 263.4 / 0.95) (bottom)
Padding interno do card: 32px
Backdrop-filter: blur(4px)
```

**Implementação**: Backend envia flag `isPreview: true` + `previewEndTime: seconds`. Frontend para o player no timestamp indicado e renderiza overlay. O player não permite seek além do ponto de corte.

### 5.5 Gate Page (Rota Bloqueada)

Quando o usuário tenta acessar uma rota inteira que não está no plano:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│              🔒                                       │  ← Ícone lock
│                                                       │     var(--muted-foreground)
│                                                       │     64px
│     Funcionalidade não disponível                     │  ← Michroma 20px
│         no seu plano atual                            │
│                                                       │
│     O acesso a {nome_funcionalidade}                  │  ← Manrope 14px
│     requer o plano {plano_minimo}.                    │     var(--muted-foreground)
│                                                       │
│     ┌──────────────────────────┐                     │
│     │  Conhecer planos         │                     │  ← botão gold
│     └──────────────────────────┘                     │
│                                                       │
│     ← Voltar                                         │  ← link text
│                                                       │
└──────────────────────────────────────────────────────┘

Rota: Middleware de rota (TanStack Router beforeLoad)
Se !hasPermission → renderiza GatePage component
```

**Variáveis dinâmicas**:
- `{nome_funcionalidade}`: "Fórum", "Cursos", "Lives ao vivo", "Criar assembleia", etc.
- `{plano_minimo}`: "Síndico N2", "Empresa Pro", "Morador Pagante", etc.

### 5.6 Trava Temporal de 90 Dias (Upload Trimestral)

Estado especial para vídeos institucionais (Empresa) e vídeo-currículo (Morador Pagante):

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  Vídeo Institucional                                  │  ← Michroma 18px
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │                                                │   │
│  │           VÍDEO ATUAL                          │   │  ← Player com vídeo
│  │           (publicado em 15/01/2026)             │   │     atual em reprodução
│  │                                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │  🔒 Atualização bloqueada                      │   │  ← Card informativo
│  │                                                │   │     background: var(--muted)
│  │  Você poderá atualizar seu vídeo               │   │     border: 1px var(--border)
│  │  institucional a partir de:                    │   │     border-radius: var(--radius)
│  │                                                │   │     padding: 20px
│  │  ┌─────┐  ┌─────┐  ┌─────┐                   │   │
│  │  │  42 │  │  07 │  │  15 │                    │   │  ← Countdown Michroma 28px
│  │  │ dias│  │horas│  │ min │                    │   │     var(--foreground)
│  │  └─────┘  └─────┘  └─────┘                   │   │     labels Manrope 11px
│  │                                                │   │     var(--muted-foreground)
│  │  Disponível em: 15/04/2026                     │   │
│  │                                                │   │  ← Manrope 13px
│  │  A regra de 90 dias garante estabilidade       │   │     var(--muted-foreground)
│  │  e profissionalismo ao seu perfil.             │   │
│  │                                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │  Substituir vídeo                              │   │  ← Botão DISABLED
│  └───────────────────────────────────────────────┘   │     opacity: 0.4
│                                                       │     cursor: not-allowed
│                                                       │     tooltip: "Disponível em DD/MM/YYYY"
└──────────────────────────────────────────────────────┘
```

**Countdown boxes**:
- Background: `var(--surface-variant)`
- Border-radius: `8px`
- Width: `72px`, height: `64px`
- Número: Michroma 28px, `var(--foreground)`
- Label: Manrope 11px, `var(--muted-foreground)`, uppercase
- Gap entre boxes: `12px`

**Quando liberado** (após 90 dias): O card informativo some. O botão "Substituir vídeo" fica ativo (gold primary). O upload segue fluxo normal do doc 06-VIDEO-UPLOAD.md.

### 5.7 Trava de Cooldown 4h (Cupons)

Para o sistema de cupons quando o morador já gerou cupom recentemente:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │  ⏳ Cupom ativo para este estabelecimento      │   │  ← Card warning
│  │                                                │   │     background: var(--warning) / 10%
│  │  Você já possui um cupom ativo.                │   │     border: 1px var(--warning) / 30%
│  │  Aguarde para gerar um novo:                   │   │     border-radius: var(--radius)
│  │                                                │   │     padding: 16px
│  │        02h 47min restantes                     │   │
│  │                                                │   │  ← Michroma 20px
│  │  ┌─────────────────────────────────────┐      │   │     var(--warning)
│  │  │ ▰▰▰▰▰▰▰▰▰▰▰▰▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱▱  │      │   │
│  │  └─────────────────────────────────────┘      │   │  ← progress bar
│  │                                                │   │     mostrando % do tempo
│  │  [Ver meu cupom ativo]                         │   │     decorrido
│  │                                                │   │
│  └───────────────────────────────────────────────┘   │
│                                                       │
└──────────────────────────────────────────────────────┘
```

### 5.8 Cota Connect Me Esgotada

Quando morador já usou todos os Connect Me do ano:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│  ┌───────────────────────────────────────────────┐   │
│  │  Cotas de contato esgotadas                    │   │  ← Michroma 16px
│  │                                                │   │
│  │  Você já utilizou suas {X} cotas anuais        │   │  ← Manrope 14px
│  │  de Connect Me para síndicos.                  │   │     var(--muted-foreground)
│  │                                                │   │
│  │  ┌────┐ ┌────┐ de ┌────┐ ┌────┐              │   │  ← Visual:
│  │  │ 🔒 │ │ 🔒 │    │ 🔒 │ │ 🔒 │              │   │     bolhas preenchidas = usadas
│  │  └────┘ └────┘    └────┘ └────┘              │   │     (Base: 2/2, Pagante: 4/4)
│  │  Usadas: {X}/{total}                          │   │
│  │                                                │   │
│  │  Suas cotas serão renovadas em                 │   │
│  │  janeiro de {próximo_ano}.                     │   │
│  │                                                │   │
│  │  [Fazer upgrade] — para mais cotas             │   │  ← botão outline gold
│  │                                                │   │     (se Base)
│  └───────────────────────────────────────────────┘   │
│                                                       │
└──────────────────────────────────────────────────────┘
```

---

## 6. TOAST NOTIFICATIONS (SOLID-SONNER)

### 6.1 Conceito

Toasts são notificações efêmeras que confirmam ações do usuário sem interromper o fluxo. Usamos `solid-sonner` como biblioteca base.

### 6.2 Posicionamento

```
Desktop (≥ 1024px): bottom-right, 24px de margem
Tablet (768-1023px): bottom-center, 16px de margem
Mobile (< 768px): top-center, 16px de margem (evitar conflito com bottom nav)

Stack: máximo 3 toasts visíveis simultaneamente
Novos empurram antigos para cima (desktop) ou para baixo (mobile)
```

### 6.3 Variantes de Toast

```
┌──────────────────────────────────────────────┐
│  ✓  Ação realizada com sucesso               │  ← SUCCESS
│                                               │     icon: check-circle
└──────────────────────────────────────────────┘     background: var(--card)
                                                     border-left: 4px var(--success)
                                                     color: var(--foreground)

┌──────────────────────────────────────────────┐
│  ✕  Não foi possível salvar                  │  ← ERROR
│     Tente novamente em instantes.            │     icon: x-circle
└──────────────────────────────────────────────┘     border-left: 4px var(--destructive)

┌──────────────────────────────────────────────┐
│  ⚠  Você atingiu o limite mensal de vídeos   │  ← WARNING
│                                               │     icon: alert-triangle
└──────────────────────────────────────────────┘     border-left: 4px var(--warning)

┌──────────────────────────────────────────────┐
│  ℹ  Novo episódio do podcast disponível      │  ← INFO
│                                               │     icon: info-circle
└──────────────────────────────────────────────┘     border-left: 4px var(--ring)
```

### 6.4 Anatomia do Toast

```
┌──────────────────────────────────────────────────┐
│  [icon]  Título da mensagem              [×]     │
│          Descrição opcional em segunda linha      │
│          [Ação]                                   │
└──────────────────────────────────────────────────┘

Estilos:
├── width: 360px (desktop), 100% - 32px (mobile)
├── padding: 14px 16px
├── background: var(--card)
├── border: 1px solid var(--border)
├── border-left: 4px solid {cor_variante}
├── border-radius: var(--radius)
├── box-shadow: 0 4px 12px rgba(0,0,0,0.15)
├── font-family: Manrope
├── title: 14px, font-weight: 600, var(--foreground)
├── description: 13px, var(--muted-foreground)
├── action: 13px, font-weight: 600, var(--primary), cursor pointer
└── close: 16px, var(--muted-foreground), hover: var(--foreground)
```

### 6.5 Duração

```
Success: 3 segundos (auto-dismiss)
Info: 4 segundos (auto-dismiss)
Warning: 5 segundos (auto-dismiss)
Error: 6 segundos (auto-dismiss) — ou persistente se ação necessária
Upload progress: persistente até conclusão

Todas: dismiss on swipe (mobile) ou click no ×
```

### 6.6 Catálogo de Mensagens de Toast

**Ações de Sucesso:**
- "Perfil atualizado com sucesso"
- "Vídeo publicado com sucesso"
- "Assembleia criada com sucesso"
- "Voto registrado com sucesso"
- "Avaliação enviada com sucesso"
- "Cupom gerado com sucesso"
- "Formulário Connect Me enviado"
- "Curso iniciado com sucesso"
- "Certificado disponível para download"
- "Post publicado no fórum"
- "Senha alterada com sucesso"
- "Foto de perfil atualizada"
- "Promoção cadastrada com sucesso"
- "Denúncia registrada. Obrigado pelo feedback"
- "Comunicado agendado com sucesso"

**Ações de Erro:**
- "Não foi possível salvar as alterações. Tente novamente."
- "Erro ao publicar vídeo. Verifique o formato do arquivo."
- "Falha ao enviar formulário. Verifique sua conexão."
- "Erro ao registrar voto. Tente novamente."
- "Não foi possível gerar o cupom. Tente mais tarde."
- "Erro ao carregar dados. Tente novamente."
- "Formato de arquivo não suportado."
- "Arquivo excede o tamanho máximo permitido."

**Ações de Warning:**
- "Você atingiu o limite mensal de vídeos ({N}/{max})"
- "Cupom ativo detectado. Aguarde {tempo} para gerar novo."
- "Cotas de Connect Me esgotadas para este ano."
- "Upload bloqueado. Próxima atualização em {data}."
- "Janela de votação encerrada."
- "Limite de caracteres atingido."

**Informativas:**
- "Live ao vivo agora! Assista em tempo real."
- "Novo episódio do podcast disponível."
- "Sua avaliação é importante. Avalie a gestão."
- "Novo comunicado do MasterSíndico."

---

## 7. LOADING STATES PONTUAIS

### 7.1 Button Loading

Quando uma ação está processando:

```
Estado Normal:          Estado Loading:
┌───────────────┐      ┌───────────────┐
│  Publicar     │  →   │  ⟳ Publicando │
└───────────────┘      └───────────────┘

Regras:
├── Botão fica disabled durante loading
├── Cursor: not-allowed
├── Opacity: 0.7
├── Spinner: 16px, animation rotate 1s linear infinite
├── Texto muda para gerúndio ("Salvando...", "Enviando...", "Criando...")
├── Width: min-width mantido (não encolher com texto menor)
└── Nunca mostrar spinner sem texto contextual
```

### 7.2 Upload Progress

Para uploads de vídeo (doc 06-VIDEO-UPLOAD.md):

```
┌──────────────────────────────────────────────────┐
│                                                   │
│  📹 video-institucional.mp4                       │  ← nome do arquivo
│                                                   │     Manrope 14px
│  ┌──────────────────────────────────────────┐    │
│  │ ▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▰▱▱▱▱▱▱▱▱▱▱▱▱      │    │  ← Progress bar
│  └──────────────────────────────────────────┘    │     height: 6px
│  62% · 1.2 MB de 1.9 MB · ~8s restantes         │     border-radius: 3px
│                                                   │     bg: var(--muted)
│  [Cancelar upload]                                │     fill: var(--primary) gold
│                                                   │     transition: width 200ms
└──────────────────────────────────────────────────┘

Progress bar cor:
├── 0-99%: var(--primary) gold
├── 100%: var(--success) verde
└── Erro: var(--destructive) vermelho
```

### 7.3 Pull to Refresh (Mobile)

Padrão Instagram-style para mobile:

```
↓ Puxe para atualizar

  ┌─────────────────┐
  │      ⟳           │  ← spinner aparece conforme puxa
  │  Atualizando...  │     threshold: 80px de pull
  └─────────────────┘     spinner: 24px gold
                          animation: rotate 1s linear infinite
                          release → fetch data → dismiss

Após atualizar:
  Spinner some → conteúdo atualizado → scroll volta ao topo

Implementação:
├── Touch events: touchstart, touchmove, touchend
├── Pull distance indicator: opacity proporcional à distância
├── Threshold: 80px para trigger
├── Max pull: 120px (rubber band effect após threshold)
└── Haptic feedback (se disponível): navigator.vibrate(10)
```

### 7.4 Infinite Scroll Loading

Para listagens longas (busca, fórum, notificações):

```
┌────────────────────────────────────────┐
│  Item 1                                 │
│  Item 2                                 │
│  ...                                    │
│  Item N                                 │
│                                         │
│           ⟳                             │  ← Spinner 24px
│     Carregando mais...                  │     var(--muted-foreground)
│                                         │     Manrope 13px
└────────────────────────────────────────┘     text-align: center
                                              padding: 24px
                                              margin-bottom: 80px (mobile, acima bottom nav)

Implementação:
├── IntersectionObserver no sentinel element
├── Threshold: 0.5 (dispara quando 50% visível)
├── Root margin: 200px (pre-fetch antes de chegar)
├── TanStack Query: useInfiniteQuery
├── hasNextPage → mostra spinner
├── !hasNextPage → mostra "Você chegou ao fim" (Manrope 13px, muted-foreground)
└── isError → mostra "Erro ao carregar. [Tentar novamente]"
```

---

## 8. MICRO-INTERAÇÕES

### 8.1 Like Animation

```
Estado: não curtido → curtido

1. Ícone coração outline → filled
2. Scale animation: 1 → 1.3 → 1 (200ms, ease-out)
3. Cor: var(--muted-foreground) → var(--primary) gold
4. Partículas opcionais: 3-5 dots gold saindo do ícone (150ms, fade out)

Estado: curtido → não curtido
1. Ícone coração filled → outline
2. Scale: 1 → 0.9 → 1 (150ms)
3. Cor: var(--primary) → var(--muted-foreground)
4. Sem partículas

Lembrete: O NÚMERO de likes NÃO é exibido para visitantes.
Apenas o toggle visual do ícone aparece para todos.
O dono do conteúdo vê o ícone + contador no painel privado.
```

### 8.2 Bookmark/Save Animation

```
Ícone bookmark outline → filled
Scale: 1 → 1.2 → 1 (200ms)
Cor: var(--muted-foreground) → var(--foreground)
```

### 8.3 Vote Confirmation

```
Ao clicar em "Aprovar" ou "Reprovar" na votação:

1. Botão selecionado: background → var(--primary) com scale 1 → 1.05 → 1
2. Checkmark aparece dentro do botão (fade in 200ms)
3. Botões não selecionados: opacity → 0.4, pointer-events: none
4. Toast success: "Voto registrado com sucesso"
5. Botões ficam locked (não pode mudar voto)
```

### 8.4 Form Field Focus

```
Input idle:
├── border: 1px solid var(--input)
├── background: transparent

Input focus:
├── border: 2px solid var(--ring) gold
├── outline: none
├── box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15)
├── transition: border-color 150ms, box-shadow 150ms

Input error:
├── border: 2px solid var(--destructive)
├── box-shadow: 0 0 0 3px oklch(0.568 0.200 26.4 / 0.15)

Input success (validação OK):
├── border: 2px solid var(--success)
├── Ícone check aparece no canto direito do input (16px, var(--success))
```

### 8.5 Card Hover (Desktop)

```
Card idle:
├── transform: none
├── box-shadow: none (ou 0 1px 3px rgba(0,0,0,0.08))

Card hover:
├── transform: translateY(-2px)
├── box-shadow: 0 8px 25px rgba(0,0,0,0.12)
├── transition: transform 200ms ease, box-shadow 200ms ease

Card active (pressed):
├── transform: translateY(0)
├── transition: transform 100ms

Thumbnail hover (video cards):
├── Overlay play button: opacity 0 → 1 (150ms)
├── Thumbnail: scale 1 → 1.03 (300ms, overflow hidden no container)
```

### 8.6 Toggle Switch

```
Off → On:
├── Track: var(--muted) → var(--primary) gold (200ms)
├── Thumb: translateX(0) → translateX(20px) (200ms, cubic-bezier)
├── Thumb shadow: 0 0 0 2px var(--primary)

On → Off:
├── Track: var(--primary) → var(--muted)
├── Thumb: translateX(20px) → translateX(0)
```

### 8.7 Tab Indicator

```
Tab navigation (perfil, settings, dashboard):
├── Indicador: barra inferior 2px var(--primary) gold
├── Transição: translateX + width para a tab ativa (300ms, ease-out)
├── Tab inativa: var(--muted-foreground)
├── Tab ativa: var(--foreground) + barra gold
├── Tab hover: var(--foreground) (sem barra)
```

### 8.8 Accordion/Collapsible

```
Fechar → Abrir:
├── Seta: rotate(0) → rotate(180deg) (200ms)
├── Conteúdo: height 0 → auto (300ms, ease-out)
├── Opacity: 0 → 1 (200ms, delay 100ms)

Abrir → Fechar:
├── Seta: rotate(180deg) → rotate(0)
├── Opacity: 1 → 0 (100ms)
├── Conteúdo: height auto → 0 (200ms, ease-in)
```

### 8.9 Page Transitions (TanStack Router)

```
Rota A → Rota B:
├── Saída: opacity 1 → 0 (150ms)
├── Entrada: opacity 0 → 1, translateY(8px) → translateY(0) (250ms, ease-out)
├── Classe CSS: .fade-in-up (já definida no main.css)

Componente Outlet:
├── Suspense fallback → skeleton da rota
├── Transition wrapper com as classes de animação
```

### 8.10 Notification Badge Pulse

```
Badge de contagem (sino, tabs):
├── Quando novo item chega: scale 1 → 1.4 → 1 (300ms, bounce)
├── Dot pulsante (se não lidas): animation pulse 2s infinite
│   @keyframes pulse:
│     0% { box-shadow: 0 0 0 0 oklch(0.568 0.200 26.4 / 0.6) }
│     70% { box-shadow: 0 0 0 6px oklch(0.568 0.200 26.4 / 0) }
│     100% { box-shadow: 0 0 0 0 oklch(0.568 0.200 26.4 / 0) }
```

---

## 9. ESTADOS ESPECIAIS

### 9.1 Maintenance Mode

Tela exibida quando o sistema está em manutenção programada:

```
┌──────────────────────────────────────────────────────┐
│                                                       │
│         [Logo Master Síndico]                         │
│                                                       │
│              🔧                                       │  ← Ícone wrench
│                                                       │     64px, var(--muted-fg)
│     Estamos em manutenção                            │  ← Michroma 24px
│                                                       │
│     A plataforma está passando por uma               │  ← Manrope 14px
│     atualização programada e retornará               │     var(--muted-foreground)
│     em breve.                                         │     max-width: 400px
│                                                       │     text-align: center
│     Previsão de retorno: 20/02/2026 às 18:00         │
│                                                       │  ← Manrope 14px bold
│                                                       │     var(--foreground)
└──────────────────────────────────────────────────────┘

Background: var(--background)
Centralizado vertical e horizontal
Sem nav, sem sidebar, sem footer
```

### 9.2 Session Expired

Modal sobreposto quando o JWT expira durante uso:

```
┌──────────────────────────────────────────────────────┐
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │  ← backdrop blur(8px)
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │     rgba(0,0,0,0.6)
│ ░░░░░┌──────────────────────────────────┐░░░░░░░░░░░ │
│ ░░░░░│                                   │░░░░░░░░░░░ │
│ ░░░░░│  Sessão expirada                  │░░░░░░░░░░░ │  ← Michroma 18px
│ ░░░░░│                                   │░░░░░░░░░░░ │
│ ░░░░░│  Sua sessão expirou por           │░░░░░░░░░░░ │  ← Manrope 14px
│ ░░░░░│  inatividade. Faça login          │░░░░░░░░░░░ │     var(--muted-fg)
│ ░░░░░│  novamente para continuar.        │░░░░░░░░░░░ │
│ ░░░░░│                                   │░░░░░░░░░░░ │
│ ░░░░░│  ┌──────────────────────────┐    │░░░░░░░░░░░ │
│ ░░░░░│  │  Fazer login novamente   │    │░░░░░░░░░░░ │  ← botão gold
│ ░░░░░│  └──────────────────────────┘    │░░░░░░░░░░░ │
│ ░░░░░│                                   │░░░░░░░░░░░ │
│ ░░░░░└──────────────────────────────────┘░░░░░░░░░░░ │
│ ░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░░ │
└──────────────────────────────────────────────────────┘

Modal:
├── width: 400px (desktop), 100% - 32px (mobile)
├── background: var(--card)
├── border-radius: 12px
├── padding: 32px
├── box-shadow: 0 20px 60px rgba(0,0,0,0.3)
└── Não pode ser fechado (sem botão ×, sem click fora)
```

### 9.3 Force Update (Mobile)

Quando uma nova versão obrigatória está disponível:

```
Título: "Atualização necessária"
Descrição: "Uma nova versão do Master Síndico está disponível com melhorias importantes."
Ação: [Atualizar agora] → link para loja
Não dismissível
```

---

## 10. ACESSIBILIDADE NOS ESTADOS

### 10.1 Screen Readers

- Skeletons: `aria-busy="true"` no container, `aria-label="Carregando conteúdo"` no skeleton
- Empty states: `role="status"`, `aria-label` descritivo do estado
- Error states: `role="alert"`, `aria-live="assertive"` para erros críticos
- Toasts: `role="status"`, `aria-live="polite"` (success/info), `aria-live="assertive"` (error)
- Loading buttons: `aria-disabled="true"`, `aria-busy="true"`, texto do gerúndio é o aria-label

### 10.2 Focus Management

- Error boundary: Foco automático no título do erro
- Modal session expired: Foco preso no modal (focus trap)
- Toast: Não rouba foco
- Empty state com ação: Foco no botão de ação

### 10.3 Motion Preferences

```css
@media (prefers-reduced-motion: reduce) {
  /* Desabilitar shimmer em skeletons */
  .skeleton { animation: none; }
  
  /* Reduzir transições */
  * { 
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
  
  /* Manter feedback visual sem animação */
  /* Cores e opacidade continuam funcionando */
}
```

---

## 11. IMPLEMENTAÇÃO TÉCNICA (SOLIDJS)

### 11.1 Componentes Base

```
src/components/ui/
├── skeleton.tsx          → SkeletonPrimitive + variants
├── empty-state.tsx       → EmptyState component genérico
├── error-boundary.tsx    → Wrapper com fallback customizado
├── error-page.tsx        → 404, 500 páginas
├── gate-page.tsx         → Paywall/restrição de plano
├── network-banner.tsx    → Banner de conexão
├── loading-button.tsx    → Botão com estado loading
├── upload-progress.tsx   → Barra de progresso de upload
├── countdown.tsx         → Countdown boxes (90 dias, 4h)
├── permission-gate.tsx   → ABAC conditional render
└── paywall-overlay.tsx   → Overlay de paywall em vídeo
```

### 11.2 Padrão SolidJS para Estados

```tsx
// Padrão recomendado com TanStack Query:
const query = createQuery(() => ({
  queryKey: ['resource', id],
  queryFn: () => fetchResource(id),
}));

return (
  <Switch>
    <Match when={query.isLoading}>
      <SkeletonVariant />
    </Match>
    <Match when={query.isError}>
      <ErrorState error={query.error} retry={() => query.refetch()} />
    </Match>
    <Match when={query.data?.length === 0}>
      <EmptyState variant="no-videos" />
    </Match>
    <Match when={query.data}>
      {(data) => <ContentList items={data()} />}
    </Match>
  </Switch>
);
```

### 11.3 CVA Variants para Skeleton

```tsx
const skeletonVariants = cva(
  'animate-shimmer bg-muted rounded',
  {
    variants: {
      variant: {
        text: 'h-4 rounded',
        circle: 'rounded-full',
        rect: 'rounded-lg',
        card: 'rounded-lg',
      },
      size: {
        sm: 'h-3',
        md: 'h-4',
        lg: 'h-6',
        xl: 'h-8',
      },
    },
    defaultVariants: {
      variant: 'text',
      size: 'md',
    },
  }
);
```

### 11.4 CVA Variants para Toast (via solid-sonner)

```tsx
// Configuração do Toaster no index.tsx:
<Toaster
  position="bottom-right"      // desktop
  toastOptions={{
    classes: {
      success: 'border-l-4 border-l-success',
      error: 'border-l-4 border-l-destructive',
      warning: 'border-l-4 border-l-warning',
      info: 'border-l-4 border-l-ring',
    },
    duration: {
      success: 3000,
      error: 6000,
      warning: 5000,
      info: 4000,
    },
  }}
/>
```

---

## 12. CHECKLIST DE IMPLEMENTAÇÃO

Para cada tela da plataforma, verificar se todos os estados estão cobertos:

```
□ Skeleton loading definido (shimmer, shapes corretas)
□ Empty state com texto e ação contextual
□ Error state com retry
□ Restricted state (ABAC/paywall) tratado
□ Toast de sucesso para ações mutativas
□ Toast de erro para falhas
□ Button loading para ações assíncronas
□ Inline validation nos formulários
□ Focus management para acessibilidade
□ aria-* attributes nos estados dinâmicos
□ Responsivo mobile: todos os estados testados em < 768px
□ Dark mode: todos os estados usam variáveis CSS (oklch)
□ prefers-reduced-motion respeitado
```

---

## 13. NOTAS PARA IMPLEMENTAÇÃO

1. **Skeleton antes de tudo**: Ao criar qualquer tela nova, o skeleton loading é o PRIMEIRO componente a ser implementado. Isso garante que nunca haverá tela em branco.

2. **Empty states como oportunidade**: Cada empty state é uma chance de guiar o usuário. Nunca deixar vazio sem direção.

3. **Erros honestos**: Mensagens de erro devem ser claras, sem jargão técnico. "Não foi possível carregar" é melhor que "Error 500: Internal Server Error".

4. **Consistência de timing**: Todos os componentes da plataforma devem seguir as mesmas durações de animação (150ms rápido, 200ms padrão, 300ms lento). Não inventar timings novos.

5. **Toast como confirmação, não como informação primária**: Toasts confirmam ações. Não usar toast para comunicar informações que o usuário precisa ler com atenção (para isso, usar modais ou banners inline).

6. **ABAC silencioso**: Elementos sem permissão NÃO existem na UI. O usuário não deve saber que algo existe mas está bloqueado, EXCETO nos casos de paywall explícito (vídeo 25%, gate page).

7. **Performance**: Skeleton shimmer animation usa `background-position` (GPU-accelerated). Nunca animar `width`, `height` ou `margin` em skeletons — causa jank.

8. **Testes visuais**: Cada estado deve ser testável isoladamente. Componentes de estado recebem props que simulam o estado desejado, facilitando revisão visual.
