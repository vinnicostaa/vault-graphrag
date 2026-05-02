# 16 — SISTEMA DE FEEDBACK PRIVADO ("REDE SOCIAL CEGA")

> Sprint 4 · Transversal (aplicado em vídeos, posts do fórum, perfis)
> A plataforma NÃO é rede social. Engajamento não é palco, é sinal.

---

## REGRAS DE NEGÓCIO

### Princípio Central
- Likes e Comentários existem no banco de dados
- Contadores e listas são visíveis **EXCLUSIVAMENTE** ao dono do conteúdo
- Para qualquer outro usuário: interface "cega" — sem contadores, sem lista de comentários

### Regra de Visualização (Frontend)
```
Se Usuario_Logado == Dono_do_Conteudo:
  → Exibe contador de likes
  → Exibe lista de comentários
  → Exibe métricas de visualização

Se Usuario_Logado != Dono_do_Conteudo:
  → Oculta contadores
  → Oculta comentários
  → Usuário vê APENAS o conteúdo (vídeo, post)
  → Pode curtir e comentar, mas NÃO vê os de terceiros
```

### Interações permitidas
- **Curtir**: Qualquer usuário com permissão pode curtir (conforme Matriz Funcional)
- **Comentar**: Qualquer usuário com permissão pode comentar
- O autor do comentário/curtida NÃO vê sua interação listada (a menos que seja dono do conteúdo)
- Curtida é toggle (curtir/descurtir)
- Comentário é texto simples (sem reply, sem thread)

### Permissões de Interação (Matriz Funcional)
| Ação | Base | Morador Pg | Síndico N1+ | Emp. Plus/Pro | Marketing |
|------|------|-----------|-------------|---------------|-----------|
| Curtir conteúdos | ❌ | ❌ | ✅ | ✅ | ✅ |
| Comentar | ❌ | ❌ | ✅ | ✅ | ✅ |

### Métricas visíveis ao dono
- Total de likes
- Lista de comentários (com autor, data, texto)
- Visualizações (contagem, >70% para vídeos)
- Não gera ranking, não compara com outros conteúdos

### Métricas visíveis ao MasterSíndico (admin)
- Mesmas métricas do dono, para qualquer conteúdo
- Para fins de moderação e análise

---

## LAYOUT — VISÃO DO PÚBLICO (Qualquer usuário vendo conteúdo de outro)

### Em vídeo:
```
┌──────────────────────────────────────────┐
│  [VIDEO PLAYER]                          │
│                                          │
│  Título do vídeo                         │
│  🏢 Empresa Tal                          │
│                                          │
│  [❤️ Curtir]  [💬 Comentar]  [⚠️ Denúncia]│
│                                          │
│  ← SEM contadores visíveis              │
│  ← SEM lista de comentários             │
│  ← SEM "X curtidas" ou "Y comentários"  │
└──────────────────────────────────────────┘
```

### Em post do fórum:
```
┌──────────────────────────────────────────┐
│  Título do Post                          │
│  Autor • 2h atrás • Categoria           │
│                                          │
│  Conteúdo do post...                     │
│                                          │
│  [❤️ Curtir]  [💬 Comentar]  [⚠️ Denúncia]│
│                                          │
│  ← NENHUM contador visível              │
└──────────────────────────────────────────┘
```

### CSS — Botões de Interação (Público)

```css
.interaction-bar {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
  border-top: 1px solid var(--border);
  margin-top: 16px;
}

.interaction-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 8px 14px;
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 500;
  color: var(--muted-foreground);
  cursor: pointer;
  transition: all 200ms;
}
.interaction-btn:hover {
  border-color: var(--foreground);
  color: var(--foreground);
}

/* Estado curtido (toggle) */
.interaction-btn.liked {
  border-color: var(--primary);
  color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.08);
}
.interaction-btn.liked .interaction-icon {
  animation: heartBounce 300ms ease;
}

@keyframes heartBounce {
  0% { transform: scale(1); }
  30% { transform: scale(1.3); }
  60% { transform: scale(0.9); }
  100% { transform: scale(1); }
}

.interaction-icon {
  width: 16px;
  height: 16px;
}

.interaction-btn-report {
  margin-left: auto;
  padding: 8px;
  background: transparent;
  border: none;
  color: var(--muted-foreground);
  cursor: pointer;
  border-radius: 6px;
  transition: all 200ms;
}
.interaction-btn-report:hover {
  color: var(--destructive);
  background: oklch(0.568 0.200 26.4 / 0.1);
}

/* IMPORTANTE: sem .interaction-count visível para público */
```

### Input de Comentário (Público — aparece ao clicar "Comentar")

```
┌──────────────────────────────────────────┐
│  ┌──────────────────────────────────┐    │
│  │ Escreva um comentário...         │    │
│  │                                  │    │
│  └──────────────────────────────────┘    │
│  0/500                    [Enviar →]     │
│                                          │
│  ℹ️ Seu comentário será visível apenas   │
│     para o autor do conteúdo.            │
└──────────────────────────────────────────┘
```

```css
.comment-input-area {
  margin-top: 12px;
  animation: fadeInUp 200ms ease;
}

.comment-textarea {
  width: 100%;
  min-height: 72px;
  padding: 10px 14px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  resize: vertical;
  outline: none;
  transition: border-color 200ms;
}
.comment-textarea:focus {
  border-color: var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}
.comment-textarea::placeholder {
  color: var(--muted-foreground);
}

.comment-input-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.comment-charcount {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}

.comment-submit-btn {
  padding: 8px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: all 200ms;
}
.comment-submit-btn:hover { background: oklch(0.65 0.120 84.2); }
.comment-submit-btn:disabled {
  background: var(--muted);
  color: var(--muted-foreground);
  cursor: not-allowed;
}

.comment-privacy-notice {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-top: 8px;
  padding: 8px 12px;
  background: var(--muted);
  border-radius: 6px;
}

/* Feedback pós-envio */
.comment-sent-feedback {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 12px 16px;
  background: oklch(0.627 0.170 149.2 / 0.1);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--success);
  animation: fadeIn 200ms ease;
}
```

---

## LAYOUT — VISÃO DO DONO DO CONTEÚDO

### Painel Privado de Métricas (no próprio conteúdo ou em /meu-perfil)

```
┌──────────────────────────────────────────────────────────┐
│  MÉTRICAS DO SEU CONTEÚDO (privado)                      │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │   42     │ │   12     │ │  1.2k    │                │
│  │ Curtidas │ │Comentários│ │  Views   │                │
│  └──────────┘ └──────────┘ └──────────┘                │
│                                                          │
│  ─── COMENTÁRIOS RECEBIDOS ──────────────────────────   │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  [avatar] João M. • 2h atrás                      │  │
│  │  Excelente explicação sobre impermeabilização!     │  │
│  │  Muito claro.                                      │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  [avatar] Maria S. • 5h atrás                     │  │
│  │  Poderia fazer um vídeo sobre manutenção           │  │
│  │  preventiva de telhados também?                    │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  ┌────────────────────────────────────────────────────┐  │
│  │  [avatar] Carlos A. • 1d atrás                    │  │
│  │  Já apliquei no meu condomínio. Resultado ótimo.   │  │
│  └────────────────────────────────────────────────────┘  │
│                                                          │
│  [Carregar mais comentários...]                          │
└──────────────────────────────────────────────────────────┘
```

### CSS — Métricas Privadas

```css
.owner-metrics {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 24px;
}

.owner-metrics-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--primary);
  margin-bottom: 16px;
  display: flex;
  align-items: center;
  gap: 6px;
}

/* Ícone de cadeado para indicar privacidade */
.owner-metrics-title::before {
  content: '🔒';
  font-size: 14px;
}

.owner-metrics-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 16px;
  margin-bottom: 24px;
}

.owner-metric-item {
  text-align: center;
  padding: 12px;
  background: var(--muted);
  border-radius: 8px;
}

.owner-metric-value {
  font-family: 'Michroma';
  font-size: 24px;
  color: var(--primary);
  letter-spacing: 0.05em;
}

.owner-metric-label {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-top: 4px;
}
```

### CSS — Lista de Comentários (visível ao dono)

```css
.owner-comments-section {
  margin-top: 20px;
}

.owner-comments-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted-foreground);
  margin-bottom: 12px;
  padding-bottom: 8px;
  border-bottom: 1px solid var(--border);
}

.comment-item {
  display: flex;
  gap: 12px;
  padding: 14px 0;
  border-bottom: 1px solid var(--border);
}
.comment-item:last-child { border-bottom: none; }

.comment-avatar {
  width: 36px;
  height: 36px;
  border-radius: 50%;
  background: var(--muted);
  flex-shrink: 0;
  object-fit: cover;
}

.comment-body { flex: 1; }

.comment-author {
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  color: var(--foreground);
}

.comment-time {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-left: 8px;
}

.comment-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.6;
  margin-top: 4px;
}

.comments-load-more {
  display: block;
  width: 100%;
  padding: 10px;
  background: transparent;
  border: 1px solid var(--border);
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  cursor: pointer;
  text-align: center;
  margin-top: 12px;
  transition: all 200ms;
}
.comments-load-more:hover {
  border-color: var(--primary);
  color: var(--primary);
}
```

---

## INTEGRAÇÃO COM OUTROS MÓDULOS

### Em Vídeos (doc 05)
- Botões ❤️ Curtir + 💬 Comentar abaixo do player
- Público: sem contadores
- Dono: painel de métricas acima da descrição

### Em Posts do Fórum (doc 17)
- Botões ❤️ Curtir + 💬 Comentar abaixo do post
- Público: sem contadores
- Dono: vê contadores inline discretos

### No Perfil (doc 04)
- Aba "Métricas" no perfil do dono com consolidado de todos os conteúdos
- Total de curtidas recebidas, total de comentários, total de views

### Padrão de Componentes Reutilizáveis

```
<InteractionBar
  contentId={contentId}
  contentType="video" | "post" | "course"
  isOwner={isCurrentUserOwner}
/>
```

O componente `InteractionBar` decide internamente o que mostrar:
- Se `isOwner=false`: botões sem contadores + textarea on click
- Se `isOwner=true`: botões COM contadores + lista de comentários

---

## MODAL DE DENÚNCIA

Ao clicar em ⚠️ Denúncia:

```
┌─────────────────────────────────────────┐
│  DENUNCIAR CONTEÚDO                     │
│                                         │
│  Por que você está denunciando?         │
│                                         │
│  ○ Conteúdo ofensivo                    │
│  ○ Spam ou propaganda                   │
│  ○ Informação falsa                     │
│  ○ Dados de contato proibidos           │
│  ○ Outro                                │
│                                         │
│  Observação (opcional):                 │
│  ┌─────────────────────────────────┐    │
│  │                                 │    │
│  └─────────────────────────────────┘    │
│                                         │
│  [Cancelar]           [Enviar Denúncia] │
└─────────────────────────────────────────┘
```

```css
.report-modal {
  max-width: 440px;
  width: 90%;
}

.report-title {
  font-family: 'Michroma';
  font-size: 16px;
  color: var(--foreground);
  letter-spacing: 0.05em;
  margin-bottom: 16px;
}

.report-reason-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
  margin-bottom: 16px;
}

.report-reason-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px 14px;
  border: 1px solid var(--border);
  border-radius: 8px;
  cursor: pointer;
  transition: all 200ms;
}
.report-reason-item:hover { border-color: var(--foreground); }
.report-reason-item.selected {
  border-color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.05);
}

.report-reason-radio {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 2px solid var(--border);
  flex-shrink: 0;
}
.report-reason-item.selected .report-reason-radio {
  border-color: var(--primary);
  background: var(--primary);
  box-shadow: inset 0 0 0 3px var(--card);
}

.report-reason-label {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
}
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `InteractionBar` | Barra com curtir/comentar/denúncia | Contexto de auth |
| `LikeButton` | Botão curtir com toggle | — |
| `CommentInput` | Textarea com enviar + aviso privacidade | — |
| `CommentSentFeedback` | Toast/banner de confirmação | — |
| `OwnerMetrics` | Grid 3 métricas (curtidas/comentários/views) | — |
| `OwnerCommentList` | Lista paginada de comentários | — |
| `CommentItem` | Item individual (avatar + nome + texto) | — |
| `ReportModal` | Modal de denúncia com motivos | Kobalte Dialog |

---

## CHECKLIST

- [ ] Botões curtir/comentar SEM contadores para público
- [ ] Toggle de curtida com animação heartBounce
- [ ] Textarea de comentário com aviso de privacidade
- [ ] Feedback visual "Comentário enviado" (fade green)
- [ ] Painel de métricas privadas para dono (3 counters Michroma gold)
- [ ] Lista de comentários visível APENAS ao dono
- [ ] Modal de denúncia com motivos pré-definidos
- [ ] Componente `InteractionBar` reutilizável (vídeos, posts, perfis)
- [ ] Zero contadores públicos em qualquer contexto
- [ ] Dark mode tokens oklch
