---
title: Notifications — Sistema de Notificações (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, cross, notifications]
bc: cross
source: _chaos/23-NOTIFICATIONS.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 23 — Sistema de Notificações

> **Origem**: `_chaos/23-NOTIFICATIONS.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".

> Sprint 1-4 · App `cms` · Todos os perfis autenticados.
> Dependências: [[../shell/app-shell|app-shell]] · [[./settings|settings]].

---

## 1. Regra fundamental

**O sistema NÃO envia notificações de interações sociais** (R-PRIV-MX, GR-028). Não existe:
- ❌ "Fulano curtiu seu vídeo"
- ❌ "Novo comentário no seu post"
- ❌ "Empresa postou novo vídeo"
- ❌ "Fulano começou a seguir você" (seguir não existe)

Notificações restritas a:
- ✅ Validação de cadastro / segurança
- ✅ Connect Me recebido
- ✅ Comunicados oficiais (broadcast Master Síndico)
- ✅ Eventos operacionais (assembleia, votação, avaliação)
- ✅ Progresso de cursos (certificado)
- ✅ Lives ao vivo (se habilitado em settings)
- ✅ Oportunidades (empresa demonstrou interesse no Banco Talentos)
- ✅ Cupons / promoções exclusivas do condomínio

### Canais

| Canal | Uso |
|---|---|
| **In-app** (badge + dropdown) | Todas |
| **Email** | Validação, Connect Me, Broadcasts, Certificados |
| **Push** (mobile / browser) | Lives ao vivo, Assembleias urgentes, Broadcasts |

Frontend controla apenas in-app. Email / Push são disparados pelo backend via `IEmailProvider` (Resend) e `IPushProvider` (FCM).

---

## 2. Notification Bell (topbar)

### Visual

- Sem notificações: 🔔 (`lucide:bell` 20 px, `--foreground`).
- Com não lidas: badge vermelho 16 px com número (Manrope 10 px bold branco), `--destructive` background, position absolute top-right.
- Se > 9: "9+". Se > 99: "99+".
- Animação de entrada do badge: scale 0 → 1 com bounce.
- Quando zero: badge desaparece (não mostra "0").

### Interação

- Click: abre dropdown (desktop) ou navega para `/notificacoes` (mobile).

---

## 3. Notification Dropdown (desktop)

Componente: **Kobalte Popover**. Largura 380 px, max-height 480 px, scroll interno.

```
┌──────────────────────────────────────┐
│  Notificações       Marcar todas     │
├──────────────────────────────────────┤
│  ● 📢 Comunicado Oficial            │
│  Atualização dos Termos de Uso       │
│  Há 15 minutos                       │
│  ────────────                        │
│  ● 📋 Assembleia Publicada          │
│  Cond. Sol Nascente publicou nova... │
│  Há 2 horas                         │
│  ────────────                        │
│  ○ 🔗 Connect Me Recebido           │
│  João Silva enviou formulário        │
│  Há 1 dia                           │
├──────────────────────────────────────┤
│  Ver todas as notificações →         │
└──────────────────────────────────────┘
```

```css
.notification-dropdown {
  width: 380px; max-height: 480px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md);
  box-shadow: var(--shadow-xl);
  overflow-y: auto;
}
.notification-item {
  padding: 12px 16px; border-bottom: 1px solid var(--border);
  cursor: pointer; transition: background var(--duration-moderate);
}
.notification-item:hover { background: oklch(var(--muted) / 0.3); }
.notification-item.unread {
  /* Dot indicator inline */
}
.notification-item.read { opacity: 0.7; }
.notification-time {
  font-family: 'Manrope'; font-size: 11px; color: var(--muted-foreground);
}
```

---

## 4. Página full — `/notificacoes`

### Estrutura

- Tabs: **Todas** / **Não lidas** / **Arquivadas** (Kobalte Tabs).
- Botão "Marcar todas como lidas".
- Agrupamento temporal: **HOJE** / **ESTA SEMANA** / **ANTERIORES** (label Manrope 12 px uppercase muted, letter-spacing 0.08em).
- Itens com ação contextual à direita (Arquivar, Baixar Certificado, Ver Resultado, Assistir Gravação).
- Paginação: infinite scroll com "Carregar mais" (20 / página, TanStack Query `useInfiniteQuery`).

### Mobile

Tabs scrolláveis, items full-width, swipe-left para "Arquivar".

---

## 5. Tipos de notificação

| Tipo | Ícone | Cor | Destino | Recebem |
|---|---|---|---|---|
| Broadcast | 📢 | `--primary` | `/comunicados/:id` | Todos (segmentado) |
| Assembleia Publicada | 📋 | navy | `/assembleias/:id` | Moradores condomínio |
| Votação Aberta | 🗳️ | navy | `/assembleias/:id/votacao` | Moradores |
| Votação Encerrada | 🗳️ | muted | `/assembleias/:id/resultado` | Moradores |
| Avaliação Disponível | ⭐ | `--warning` | `/avaliacoes/:id` | Moradores |
| Connect Me Recebido | 🔗 | `--success` | `/connect-me/recebidos` | Síndicos, Empresas |
| Certificado Gerado | 🎓 | `--success` | `/cursos/:id/certificado` | Síndico `plus`+ |
| Live ao Vivo | 🔴 | `--destructive` | `/lives/:id` | `plus`+ + Síndicos + Empresas |
| Live Gravada | 🔴 | muted | `/lives/:id/gravacao` | Todos com acesso |
| Interesse no Currículo | 💼 | `--primary` | `/oportunidade` | Morador c/ Banco Talentos |
| Promo Exclusiva | 🎁 | `--accent` | `/beneficios/:id` | Moradores |
| Conta Confirmada | ✅ | `--success` | `/` | Todos |
| Senha Redefinida | 🔒 | muted | `/configuracoes/seguranca` | Todos |
| Novo Login | 🛡️ | `--warning` | `/configuracoes/seguranca` | Todos |

### Estrutura de dados

```ts
interface Notification {
  id: string;
  type: NotificationType;
  title: string;
  description: string;
  createdAt: string; // ISO datetime
  readAt: string | null;
  archivedAt: string | null;
  actionUrl: string;
  actionLabel?: string;
  metadata?: Record<string, unknown>;
}

type NotificationType =
  | "broadcast"
  | "assembly_published"
  | "voting_open"
  | "voting_closed"
  | "evaluation_available"
  | "connect_me_received"
  | "certificate_generated"
  | "live_now"
  | "live_recorded"
  | "curriculum_interest"
  | "promo_exclusive"
  | "account_confirmed"
  | "password_reset"
  | "new_login";
```

---

## 6. Comportamento

### Recebimento
- **Polling** TanStack Query a cada 60 s (`refetchInterval: 60000`).
- Endpoint leve: `GET /notifications/unread-count`.
- Lista completa carregada sob demanda.
- Futuro: WebSocket / SSE (não obrigatório no MVP).
- **Sem som / vibração** ao receber — apenas badge atualiza silenciosamente.

### Marcar como lida
- **Automaticamente**: ao clicar (navegar para destino).
- **Manual**: botão "Marcar todas como lidas".
- Mutation: `PATCH /notifications/:id/read` ou `/read-all`.
- Optimistic update no front, reverte se falhar.

### Arquivar
- Mutation: `PATCH /notifications/:id/archive`.
- Não deleta — vai para tab "Arquivadas".

### Não-perturbe (implícito)
- Se usuário desativou categoria em [[./settings|settings]], backend não gera notificação.
- Comunicados oficiais e segurança são sempre enviados (não desativável).

---

## 7. Componentes

`NotificationBell`, `NotificationDropdown`, `NotificationItem`, `NotificationPage`, `NotificationTabs`, `NotificationGroup`, `NotificationIcon`, `EmptyNotifications`.

## 8. Estados

### Empty states

- **Sem notificações**: 🔔 "Nenhuma notificação. Quando houver atualizações relevantes, elas aparecerão aqui."
- **Sem não lidas**: ✅ "Tudo em dia! Você já leu todas as suas notificações."
- **Sem arquivadas**: 📂 "Nenhuma notificação arquivada."

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## 9. Notas de implementação

1. **Polling vs WebSocket**: MVP usa polling 60 s (`useNotificationCount` no `AppShell`).
2. **TanStack Query**: dropdown `staleTime: 30000`; página full `staleTime: 0`.
3. **Optimistic updates**: marcar / arquivar via `onMutate`.
4. **Limite backend**: últimas 200 notificações por usuário; mais antigas descartadas.
5. **NÃO é feed social** — sistema austero e funcional, alertas operacionais pontuais.

## 10. Acessibilidade

- Bell: `aria-label="Notificações, X não lidas"` (atualiza dinamicamente).
- Dropdown: `role="dialog"`, focus trap, Escape para fechar.
- Itens: `role="listitem"`, item não lido inclui "(não lida)" no `aria-label`.
- Keyboard: Enter / Space para abrir, Arrow keys para navegar, Escape para fechar.

## Links

- [[./settings|settings]] — toggles por categoria
- [[../../../web/ui-catalog#b8-conta-sistema-sys1-sys8]] — SYS7 Notificações (catálogo macro)
- [[../../../../00-product/sub-produtos/04-conteudo-videos]] — privacidade R-PRIV-MX
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
