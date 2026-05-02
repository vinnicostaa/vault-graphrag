---
title: Private Feedback — "Rede Social Cega" (UI Spec)
type: ui-spec
tags: [ui-catalog, master-sindico, cross, feedback, privacy, lgpd]
bc: cross
source: _chaos/16-PRIVATE-FEEDBACK.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 16 — Sistema de Feedback Privado ("Rede Social Cega")

> **Origem**: `_chaos/16-PRIVATE-FEEDBACK.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 4 · Transversal (aplicado em vídeos, posts do fórum, perfis)
> **A plataforma NÃO é rede social. Engajamento não é palco, é sinal.**

---

## Regras de negócio

### Princípio central
- Likes e Comentários existem no banco de dados.
- **Contadores e listas** são visíveis **EXCLUSIVAMENTE** ao dono do conteúdo (R-PRIV-MX, sub-produto Conteúdo).
- Para qualquer outro usuário: interface "cega" — sem contadores, sem lista de comentários.

### Regra de visualização (frontend)

```
Se Usuário_Logado == Dono_do_Conteúdo:
  → Exibe contador de likes
  → Exibe lista de comentários
  → Exibe métricas de visualização

Se Usuário_Logado != Dono_do_Conteúdo:
  → Oculta contadores
  → Oculta comentários
  → Usuário vê APENAS o conteúdo (vídeo, post)
  → Pode curtir e comentar, mas NÃO vê os de terceiros
```

### Interações permitidas
- **Curtir**: qualquer usuário com permissão (toggle).
- **Comentar**: qualquer usuário com permissão (texto simples — sem reply, sem thread).
- O autor NÃO vê sua interação listada (a menos que seja dono do conteúdo).

### Permissões (matriz)

| Ação | `base` | Síndico (todos) | Empresa `plus`/`pro` | Marketing |
|---|---|---|---|---|
| Curtir conteúdos | ❌ | ✅ | ✅ | ✅ |
| Comentar | ❌ | ✅ | ✅ | ✅ |

### Métricas visíveis ao dono
- Total de likes.
- Lista de comentários (autor, data, texto).
- Visualizações (contagem ≥ 70 % para vídeos).
- **Não gera ranking, não compara com outros conteúdos** (privacidade absoluta — D-058 GR-028).

### Admin Master Síndico
- Mesmas métricas do dono, para qualquer conteúdo.
- Para fins de moderação e análise.

---

## Layout — Visão pública (não-owner)

### Em vídeo

```
┌──────────────────────────────────────────┐
│  [VIDEO PLAYER]                          │
│  Título do vídeo                         │
│  🏢 Empresa Tal                          │
│                                          │
│  [❤️ Curtir]  [💬 Comentar]  [⚠️ Denúncia]│
│                                          │
│  ← SEM contadores visíveis              │
│  ← SEM lista de comentários             │
└──────────────────────────────────────────┘
```

### CSS — Botões de Interação (Público)

```css
.interaction-bar {
  display: flex; align-items: center; gap: 12px;
  padding: 12px 0; border-top: 1px solid var(--border); margin-top: 16px;
}
.interaction-btn {
  display: flex; align-items: center; gap: 6px;
  padding: 8px 14px;
  background: transparent; border: 1px solid var(--border);
  border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 13px; font-weight: 500;
  color: var(--muted-foreground);
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
}
.interaction-btn:hover {
  border-color: var(--foreground); color: var(--foreground);
}
.interaction-btn.liked {
  border-color: var(--primary); color: var(--primary);
  background: oklch(0.715 0.120 84.2 / 0.08);
}
.interaction-btn.liked .interaction-icon {
  animation: heartBounce var(--duration-slow) ease;
}
@keyframes heartBounce {
  0% { transform: scale(1); }
  30% { transform: scale(1.3); }
  60% { transform: scale(0.9); }
  100% { transform: scale(1); }
}
```

### Input de comentário (público — após clicar "Comentar")

```
┌──────────────────────────────────────────┐
│  ┌──────────────────────────────────┐    │
│  │ Escreva um comentário...         │    │
│  └──────────────────────────────────┘    │
│  0/500                    [Enviar →]     │
│                                          │
│  ℹ️ Seu comentário será visível apenas   │
│     para o autor do conteúdo.            │
└──────────────────────────────────────────┘
```

```css
.comment-textarea {
  width: 100%; min-height: 72px;
  padding: 10px 14px;
  background: var(--background); border: 1px solid var(--input);
  border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  resize: vertical; outline: none;
  transition: border-color var(--duration-moderate) var(--ease-out);
}
.comment-privacy-notice {
  display: flex; align-items: center; gap: 6px;
  font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground);
  margin-top: 8px; padding: 8px 12px;
  background: var(--muted); border-radius: var(--radius-sm);
}
```

---

## Layout — Visão do dono

### Painel privado de métricas

```
┌──────────────────────────────────────────────────────────┐
│  🔒 MÉTRICAS DO SEU CONTEÚDO (privado)                   │
│                                                          │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐                │
│  │   42     │ │   12     │ │  1.2k    │                │
│  │ Curtidas │ │Comentários│ │  Views   │                │
│  └──────────┘ └──────────┘ └──────────┘                │
│                                                          │
│  ─── COMENTÁRIOS RECEBIDOS ─────────────                 │
│  [avatar] João M. • 2h — Excelente explicação!          │
│  [avatar] Maria S. • 5h — Poderia fazer um vídeo...     │
│  [avatar] Carlos A. • 1d — Resultado ótimo.             │
│                                                          │
│  [Carregar mais comentários...]                          │
└──────────────────────────────────────────────────────────┘
```

```css
.owner-metrics {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); padding: 20px; margin-bottom: 24px;
}
.owner-metrics-title {
  font-family: 'Michroma'; font-size: 11px;
  text-transform: uppercase; letter-spacing: 0.12em; color: var(--primary);
  margin-bottom: 16px;
  display: flex; align-items: center; gap: 6px;
}
.owner-metrics-title::before { content: '🔒'; font-size: 14px; }

.owner-metrics-grid {
  display: grid; grid-template-columns: repeat(3, 1fr);
  gap: 16px; margin-bottom: 24px;
}
.owner-metric-value {
  font-family: 'Michroma'; font-size: 24px;
  color: var(--primary); letter-spacing: 0.05em;
}
```

---

## Modal de denúncia

```
┌─────────────────────────────────────────┐
│  DENUNCIAR CONTEÚDO                     │
│                                         │
│  Por que você está denunciando?         │
│  ○ Conteúdo ofensivo                    │
│  ○ Spam ou propaganda                   │
│  ○ Informação falsa                     │
│  ○ Dados de contato proibidos           │
│  ○ Outro                                │
│                                         │
│  Observação (opcional)                  │
│                                         │
│  [Cancelar]           [Enviar Denúncia] │
└─────────────────────────────────────────┘
```

---

## Componente padrão reutilizável

```tsx
<InteractionBar
  contentId={contentId}
  contentType="video" | "post" | "course"
  isOwner={isCurrentUserOwner}
/>
```

O componente `InteractionBar` decide internamente:
- `isOwner=false`: botões sem contadores + textarea on click.
- `isOwner=true`: botões COM contadores + lista de comentários.

## Componentes

`InteractionBar`, `LikeButton`, `CommentInput`, `CommentSentFeedback`, `OwnerMetrics`, `OwnerCommentList`, `CommentItem`, `ReportModal`.

## Integrações

- **Vídeos** ([[../content/video-player|content/video-player]]) — botões abaixo do player.
- **Posts do Fórum** ([[../forum/main|forum/main]]) — botões abaixo do post.
- **Perfil** ([[../cms/profile-management|cms/profile-management]]) — aba "Métricas" consolidada.

## Links

- [[../../../../00-product/sub-produtos/04-conteudo-videos]] — privacidade de métricas
- [[../../../../08-security/lgpd|08-security/lgpd]]
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
