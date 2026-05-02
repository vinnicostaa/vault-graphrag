# 23 — SISTEMA DE NOTIFICAÇÕES

> **Sprint 1-4** · Web & Mobile · Todos os perfis autenticados
> Dependências: 00-DESIGN-TOKENS · 02-APP-SHELL · 22-SETTINGS

---

## 23.1 CONTEXTO E REGRAS DE NEGÓCIO

### 23.1.1 Definição
O sistema de notificações é o mecanismo de alerta in-app que informa o usuário sobre eventos relevantes à sua conta e atividade na plataforma.

### 23.1.2 Regra Fundamental (SOW 2.2 e 5.10)
**O sistema NÃO envia notificações de interações sociais.**

Não existe notificação para:
- ❌ "Fulano curtiu seu vídeo"
- ❌ "Novo comentário no seu post"
- ❌ "Empresa postou novo vídeo"
- ❌ "Fulano começou a seguir você" (seguir não existe)

Notificações são restritas a:
- ✅ Validação de cadastro/segurança (email confirmado, senha redefinida)
- ✅ Formulário Connect Me recebido
- ✅ Comunicados oficiais do MasterSíndico (broadcast)
- ✅ Eventos operacionais do condomínio (assembleia, votação, avaliação)
- ✅ Progresso de cursos (certificado gerado)
- ✅ Lives ao vivo (se habilitado nas configurações)
- ✅ Oportunidades (empresa demonstrou interesse no currículo)
- ✅ Cupons e promoções exclusivas do condomínio

### 23.1.3 Canais de Notificação
| Canal | Uso |
|---|---|
| **In-app** (badge + dropdown) | Todas as notificações listadas acima |
| **Email** | Validação de conta, Connect Me, Broadcasts, Certificados |
| **Push** (mobile/browser) | Lives ao vivo, Assembleias urgentes, Broadcasts |

O frontend controla apenas o canal **in-app**. Email e Push são disparados pelo backend.

---

## 23.2 COMPONENTE: NOTIFICATION BELL (TOPBAR)

### 23.2.1 Localização
O sino de notificações vive na topbar global (doc 02-APP-SHELL), à esquerda do avatar.

### 23.2.2 Especificação Visual

```
Estado sem notificações:
┌────┐
│ 🔔 │  (ícone sino 20px, --foreground)
└────┘

Estado com notificações não lidas:
┌────┐
│ 🔔 │  ← Badge vermelho 16px com número
│  ⓷ │     background: --destructive
└────┘     text: branco, Manrope 10px bold
           border-radius: 50%
           position: absolute, top: -4px, right: -4px
           Se > 9: exibe "9+"
           Se > 99: exibe "99+"
```

**Ícone do Sino:**
- Iconify: `lucide:bell` 20px
- Cor padrão: `--foreground`
- Hover: opacity 0.8
- Active (dropdown aberto): `--primary` (Gold)

**Badge de Contagem:**
- Círculo 16px (min), expande para pill se "9+" ou "99+"
- Background: `--destructive`
- Text: branco, Manrope 10px bold
- Posição: absolute, overlap no canto superior direito do sino
- Animação de entrada: scale de 0 → 1 com bounce (spring)
- Quando zero: badge desaparece completamente (não mostra "0")

### 23.2.3 Interação
- Click no sino: abre dropdown de notificações
- Se mobile (<768px): navega para página full `/notificacoes` em vez de dropdown

---

## 23.3 COMPONENTE: NOTIFICATION DROPDOWN (DESKTOP)

### 23.3.1 Layout

```
┌──────────────────────────────────────┐
│  Notificações              Marcar    │
│  Michroma 14px            todas como │
│                           lidas      │
├──────────────────────────────────────┤
│                                      │
│  ● 📢 Comunicado Oficial            │
│  Atualização dos Termos de Uso       │
│  Há 15 minutos                       │
│  ────────────────────────────────    │
│  ● 📋 Assembleia Publicada          │
│  Condomínio Sol Nascente publicou    │
│  nova assembleia: "Reforma do..."    │
│  Há 2 horas                         │
│  ────────────────────────────────    │
│  ○ 🔗 Connect Me Recebido           │
│  João Silva enviou formulário        │
│  de contato                          │
│  Há 1 dia                           │
│  ────────────────────────────────    │
│  ○ 🎓 Certificado Disponível        │
│  Curso "NR-10 Básico" concluído.    │
│  Baixe seu certificado.             │
│  Há 3 dias                          │
│  ────────────────────────────────    │
│  ○ 🔴 Live ao Vivo                  │
│  Master Síndico está ao vivo!       │
│  "Tendências do mercado condominial" │
│  Há 5 dias                          │
│                                      │
├──────────────────────────────────────┤
│  Ver todas as notificações →        │
│  (link para /notificacoes)           │
└──────────────────────────────────────┘
```

**Especificações do Dropdown:**
- Componente: Kobalte Popover
- Largura: 380px
- Max-height: 480px, com scroll interno
- Background: `--card`
- Border: `--border`
- Border-radius: `--radius` (0.5rem)
- Box-shadow: `0 4px 24px rgba(0, 0, 0, 0.12)`
- Posição: alinhado à direita do sino, abaixo da topbar

**Header do Dropdown:**
- "Notificações": Michroma 14px `--foreground`
- "Marcar todas como lidas": Manrope 12px `--primary` (Gold), text button, hover underline
- Border-bottom: 1px `--border`
- Padding: 12px 16px

**Item de Notificação:**
- Padding: 12px 16px
- Border-bottom: 1px `--border` (exceto último)
- Hover: background `--muted` opacity 0.3
- Click: navega para o destino da notificação + marca como lida
- Cursor: pointer

**Indicador de lido/não lido:**
- ● (não lido): dot 8px `--primary` (Gold) à esquerda
- ○ (lido): sem dot, texto com opacity 0.7

**Estrutura de cada item:**
```
┌─────────────────────────────────────┐
│ ● [ícone] Título da Notificação    │
│   Descrição curta (max 2 linhas,   │
│   truncada com ellipsis)            │
│   Há X tempo · Manrope 11px muted  │
└─────────────────────────────────────┘
```

- Ícone: emoji ou Iconify 16px, cor contextual (ver tabela de tipos abaixo)
- Título: Manrope 14px semibold `--foreground` (se não lido) ou Manrope 14px regular `--muted-foreground` (se lido)
- Descrição: Manrope 13px `--muted-foreground`, max 2 linhas
- Tempo: Manrope 11px `--muted-foreground`, relativo ("Há 5 min", "Há 2h", "Há 3 dias")

**Footer do Dropdown:**
- "Ver todas as notificações →": Manrope 13px `--primary` (Gold), link para `/notificacoes`
- Padding: 12px 16px, border-top: 1px `--border`
- Text-align: center

---

## 23.4 PÁGINA FULL: TODAS AS NOTIFICAÇÕES

### 23.4.1 Rota
```
/notificacoes
```

### 23.4.2 Layout (Desktop)

```
┌───────────────────────────────────────────────────────────┐
│  NOTIFICAÇÕES                                              │
│  Michroma 18px --foreground                                │
│                                                             │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐             │
│  │ Todas (12) │ │ Não lidas  │ │ Arquivadas │             │
│  │   ativo    │ │ (3)        │ │            │             │
│  └────────────┘ └────────────┘ └────────────┘             │
│                                                             │
│  [Marcar todas como lidas]  (text button, --primary Gold)  │
│                                                             │
│  HOJE                                                      │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ● 📢 Comunicado Oficial do Master Síndico           │  │
│  │   Atualização dos Termos de Uso. Revise as          │  │
│  │   alterações antes do próximo acesso.               │  │
│  │   Há 15 minutos                          [Arquivar] │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ ● 📋 Nova Assembleia — Condomínio Sol Nascente      │  │
│  │   O síndico publicou nova assembleia: "Reforma      │  │
│  │   do sistema hidráulico — Bloco B"                  │  │
│  │   Há 2 horas                             [Arquivar] │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ESTA SEMANA                                               │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ○ 🔗 Connect Me — Formulário Recebido               │  │
│  │   João Silva enviou um formulário de contato         │  │
│  │   para seu perfil empresarial.                      │  │
│  │   Há 1 dia                               [Arquivar] │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ ○ 🎓 Certificado Disponível                         │  │
│  │   Parabéns! Você concluiu o curso "NR-10 Básico".  │  │
│  │   Seu certificado está disponível para download.    │  │
│  │   Há 3 dias                    [Baixar Certificado] │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  ANTERIORES                                                │
│  ┌─────────────────────────────────────────────────────┐  │
│  │ ○ 🔴 Live Encerrada — Gravação Disponível           │  │
│  │   A live "Tendências do mercado condominial" foi    │  │
│  │   gravada. Assista a qualquer momento.              │  │
│  │   Há 5 dias                     [Assistir Gravação] │  │
│  ├─────────────────────────────────────────────────────┤  │
│  │ ○ 🗳️ Votação Encerrada                              │  │
│  │   A votação "Escolha do fornecedor de elevadores"   │  │
│  │   foi encerrada. Resultado disponível.              │  │
│  │   Há 1 semana                    [Ver Resultado]    │  │
│  └─────────────────────────────────────────────────────┘  │
│                                                             │
│  [Carregar mais]                                           │
│                                                             │
└───────────────────────────────────────────────────────────┘
```

**Tabs (Kobalte Tabs):**
- "Todas": todas as notificações
- "Não lidas": filtro por status unread, com contagem entre parênteses
- "Arquivadas": notificações que o usuário descartou

**Agrupamento Temporal:**
- "HOJE": notificações de hoje
- "ESTA SEMANA": últimos 7 dias (exceto hoje)
- "ANTERIORES": mais de 7 dias
- Label do grupo: Manrope 12px uppercase `--muted-foreground`, letter-spacing 0.08em, margin-top 24px

**Item de Notificação (Full Page):**
- Largura total: max-width 720px, centralizado
- Mais espaço que o dropdown: padding 16px 20px
- Ação contextual à direita: "Arquivar", "Baixar Certificado", "Ver Resultado", "Assistir Gravação"
- Ações: Manrope 13px `--primary` (Gold), text button

**Paginação:**
- Infinite scroll com "Carregar mais" (botão)
- 20 notificações por página
- TanStack Query `useInfiniteQuery`

### 23.4.3 Layout Mobile

Em mobile (<768px), a página full é a experiência primária (sem dropdown):
- Tabs horizontais scrolláveis no topo
- Itens ocupam full-width
- Ação contextual fica abaixo do texto (não à direita)
- Swipe left para revelar "Arquivar" (gesture nativa mobile, preparar estrutura)

---

## 23.5 TIPOS DE NOTIFICAÇÃO

| Tipo | Ícone | Cor Ícone | Destino ao Clicar | Perfis que Recebem |
|---|---|---|---|---|
| Broadcast/Comunicado | 📢 | `--primary` (Gold) | `/comunicados/:id` | Todos (conforme segmentação) |
| Assembleia Publicada | 📋 | oklch(0.5 0.08 256) (navy) | `/assembleias/:id` | Moradores do condomínio |
| Votação Aberta | 🗳️ | oklch(0.5 0.08 256) | `/assembleias/:id/votacao` | Moradores do condomínio |
| Votação Encerrada | 🗳️ | `--muted-foreground` | `/assembleias/:id/resultado` | Moradores do condomínio |
| Avaliação Disponível | ⭐ | `--warning` (amarelo) | `/avaliacoes/:id` | Moradores do condomínio |
| Connect Me Recebido | 🔗 | `--success` (verde) | `/connect-me/recebidos` | Síndicos, Empresas |
| Certificado Gerado | 🎓 | `--success` | `/cursos/:id/certificado` | Síndico N2/N3 |
| Live ao Vivo | 🔴 | `--destructive` (vermelho) | `/lives/:id` | Pagantes + Síndicos + Empresas |
| Live Gravada Disponível | 🔴 | `--muted-foreground` | `/lives/:id/gravacao` | Todos com acesso a lives |
| Interesse no Currículo | 💼 | `--primary` (Gold) | `/oportunidade` | Morador Pagante |
| Promo Exclusiva Cond. | 🎁 | `--accent` (Gold light) | `/beneficios/:id` | Moradores do condomínio |
| Conta Confirmada | ✅ | `--success` | `/` | Todos (no cadastro) |
| Senha Redefinida | 🔒 | `--muted-foreground` | `/configuracoes/seguranca` | Todos |
| Novo Login Detectado | 🛡️ | `--warning` | `/configuracoes/seguranca` | Todos |

### 23.5.1 Estrutura de Dados da Notificação

```typescript
interface Notification {
  id: string
  type: NotificationType // enum dos tipos acima
  title: string
  description: string
  createdAt: string // ISO datetime
  readAt: string | null // null = não lida
  archivedAt: string | null
  actionUrl: string // rota de destino
  actionLabel?: string // "Ver Resultado", "Baixar Certificado", etc.
  metadata?: Record<string, unknown> // dados extras (ex: nome do condomínio, nome do curso)
}

type NotificationType =
  | 'broadcast'
  | 'assembly_published'
  | 'voting_open'
  | 'voting_closed'
  | 'evaluation_available'
  | 'connect_me_received'
  | 'certificate_generated'
  | 'live_now'
  | 'live_recorded'
  | 'curriculum_interest'
  | 'promo_exclusive'
  | 'account_confirmed'
  | 'password_reset'
  | 'new_login'
```

---

## 23.6 COMPORTAMENTO E FLUXO

### 23.6.1 Recebimento de Notificações
- As notificações são carregadas via **polling** a cada 60 segundos (TanStack Query com `refetchInterval: 60000`)
- Alternativa futura: WebSocket/SSE para real-time (não obrigatório na Sprint 1-4)
- A cada poll: o frontend compara a contagem de não lidas. Se aumentou, atualiza o badge.
- Nenhum som, vibração ou animação intrusiva ao receber notificação — apenas o badge atualiza silenciosamente.

### 23.6.2 Marcar como Lida
- **Automaticamente**: ao clicar na notificação (navegar para o destino)
- **Manualmente**: botão "Marcar todas como lidas"
- Mutation via TanStack Query: `PATCH /notifications/:id/read` ou `PATCH /notifications/read-all`
- Optimistic update: marca como lida no frontend imediatamente, reverte em caso de erro

### 23.6.3 Arquivar
- O usuário pode arquivar notificações individuais (remove da lista principal, vai para "Arquivadas")
- Arquivar não deleta — o usuário pode consultar na aba "Arquivadas"
- Mutation: `PATCH /notifications/:id/archive`

### 23.6.4 Não Perturbe (implícito)
- Se o usuário desativou uma categoria de notificação em Settings (doc 22), o backend não gera a notificação para aquele canal.
- O frontend respeita: se a notificação não chega do backend, não mostra.
- Comunicados oficiais e notificações de segurança são sempre enviados (não desativáveis).

---

## 23.7 RESPONSIVIDADE

| Breakpoint | Sino | Dropdown | Página Full |
|---|---|---|---|
| ≥1024px | Topbar, abre dropdown | 380px popover | Max-width 720px, centralizado |
| 768-1023px | Topbar, abre dropdown | 380px popover | Full-width com padding |
| <768px | Topbar, navega para `/notificacoes` | Não existe | Full-width, tabs scrolláveis |

---

## 23.8 COMPONENTES

| Componente | Descrição | Props |
|---|---|---|
| `NotificationBell` | Sino com badge na topbar | `unreadCount` |
| `NotificationDropdown` | Popover com lista resumida | `notifications`, `onReadAll`, `onClickItem` |
| `NotificationItem` | Item individual (dropdown ou full) | `notification`, `variant: 'compact' \| 'full'`, `onArchive?` |
| `NotificationPage` | Página full com tabs e agrupamento | — |
| `NotificationTabs` | Tabs Todas/Não lidas/Arquivadas | `activeTab`, `counts` |
| `NotificationGroup` | Agrupamento temporal com label | `label`, `children` |
| `NotificationIcon` | Ícone contextual por tipo | `type: NotificationType` |
| `EmptyNotifications` | Estado vazio | `variant: 'all' \| 'unread' \| 'archived'` |

---

## 23.9 ESTADOS E FEEDBACK

### 23.9.1 Loading
- Dropdown: 3 skeleton items (ícone pulsante + 2 linhas de texto pulsantes)
- Página full: 5 skeleton items com agrupamento skeleton

### 23.9.2 Empty States

**Sem notificações (Todas):**
```
🔔
Nenhuma notificação
Quando houver atualizações relevantes,
elas aparecerão aqui.
```

**Sem não lidas:**
```
✅
Tudo em dia!
Você já leu todas as suas notificações.
```

**Sem arquivadas:**
```
📂
Nenhuma notificação arquivada
Notificações que você arquivar
aparecerão aqui.
```

- Ícone: 40px `--muted-foreground`
- Título: Manrope 16px semibold `--foreground`
- Descrição: Manrope 14px `--muted-foreground`
- Centralizado vertical e horizontalmente

### 23.9.3 Toasts
- "Marcar todas como lidas": toast info "Todas as notificações marcadas como lidas" (sutil, 2s)
- Nenhum toast para marcar individual (feedback visual inline é suficiente — dot some, item fica opaco)

---

## 23.10 ACESSIBILIDADE

- Bell button: `aria-label="Notificações, X não lidas"`, atualiza dinamicamente
- Dropdown: `role="dialog"`, `aria-label="Painel de notificações"`, focus trap
- Lista de notificações: `role="list"`, items `role="listitem"`
- Item não lido: `aria-label` inclui "(não lida)" no final
- "Marcar todas como lidas": `aria-live="polite"` no container para anunciar mudança
- Keyboard: Enter/Space para abrir dropdown, Arrow keys para navegar items, Escape para fechar
- Tab do sino: foco visível com ring `--ring` (Gold)

---

## 23.11 NOTAS DE IMPLEMENTAÇÃO

1. **Polling vs WebSocket**: Sprint 1-4 usa polling simples (60s). O backend retorna apenas a contagem de não lidas no poll leve (`GET /notifications/unread-count`). A lista completa é carregada sob demanda ao abrir o dropdown/página.

2. **Cache TanStack Query**: Notificações usam `staleTime: 30000` (30s) para o dropdown. A página full usa `staleTime: 0` (sempre fresco).

3. **Badge update**: O hook `useNotificationCount` faz polling e atualiza o badge na topbar. É global (monta uma vez no AppShell, não em cada página).

4. **Optimistic updates**: Marcar como lida e arquivar usam `onMutate` para atualizar o cache local imediatamente.

5. **Ordenação**: Notificações vêm do backend ordenadas por `createdAt DESC`. O agrupamento temporal é calculado no frontend.

6. **Limite de armazenamento**: O backend mantém as últimas 200 notificações por usuário. Mais antigas são descartadas automaticamente. O frontend não precisa se preocupar com isso.

7. **Não é feed social**: Reforçar que este sistema é austero e funcional. Não existe "feed de atividades" nem "timeline de interações". São alertas operacionais pontuais.
