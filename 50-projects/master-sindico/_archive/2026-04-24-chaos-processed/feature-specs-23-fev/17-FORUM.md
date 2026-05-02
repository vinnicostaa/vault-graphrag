# 17 — FÓRUM E COMUNIDADE

> Sprint 4 · Rotas: /forum, /forum/:categoriaSlug, /forum/post/:id, /forum/novo
> Categorias definidas conforme Cap. 6 do Manual Técnico (31 categorias de serviço).

---

## REGRAS DE NEGÓCIO

### Estrutura
- Categorias de tópicos: baseadas nas 31 categorias técnicas do Manual Cap. 6
- Posts: Título + Conteúdo (texto) + Categoria
- Respostas: Texto simples em thread linear (sem sub-threads/replies aninhados)
- Sistema de Likes em posts/respostas (com visualização privada — doc 16)
- Busca por palavra-chave

### Permissões (Matriz Funcional)
| Ação | Base | Morador Pg | Síndico N1/N2/N3 | Emp. Plus/Pro | Marketing |
|------|------|-----------|-------------------|---------------|-----------|
| Acessar fórum | ❌ | ❌ | ✅ | ✅ | ✅ |
| Criar post | ❌ | ❌ | ✅ | ✅ | ✅ |
| Responder | ❌ | ❌ | ✅ | ✅ | ✅ |
| Curtir | ❌ | ❌ | ✅ | ✅ | ✅ |

### Moderação
- Reativa: botão de denúncia em cada post/resposta
- Sem IA de moderação automática
- MasterSíndico pode aprovar/reprovar/remover posts denunciados
- Fluxo: Denúncia → Fila de moderação → MasterSíndico avalia → Aprova ou Remove

### Privacidade
- Likes são "cegos" (doc 16): autor do post vê total, outros não
- Comentários/respostas são públicos para quem tem acesso ao fórum
- Diferente dos vídeos: respostas no fórum SÃO visíveis (faz sentido para discussão)
- Porém contadores de likes continuam privados

---

## LAYOUT — LISTAGEM DO FÓRUM

Rota: `/forum`

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  FÓRUM DA COMUNIDADE             [+ Novo Post →]      │
│          │  h2 Michroma 20px                                      │
│          │                                                        │
│          │  🔍 ┌──────────────────────────────────────────────┐   │
│          │     │ Buscar no fórum...                           │   │
│          │     └──────────────────────────────────────────────┘   │
│          │                                                        │
│          │  [Recentes] [Populares] [Minhas Perguntas]            │
│          │   ← tabs                                               │
│          │                                                        │
│          │  ─── CATEGORIAS ───────────────────────────── →       │
│          │  [📋Gestão] [⚖️Jurídico] [🔧Manutenção] [💰Finanças] │
│          │  [🏗️Engenharia] [🔒Segurança] [⚡Elétrica] [Todos]    │
│          │                                                        │
│          │  ┌────────────────────────────────────────────────┐    │
│          │  │  Como contratar impermeabilização?              │    │
│          │  │  [Manutenção Predial] • João S. • 2h atrás     │    │
│          │  │                                                │    │
│          │  │  Preciso fazer impermeabilização da cobertura   │    │
│          │  │  do meu condomínio e não sei por onde...        │    │
│          │  │                                                │    │
│          │  │  💬 3 respostas     [❤️ Curtir]  [⚠️]          │    │
│          │  └────────────────────────────────────────────────┘    │
│          │                                                        │
│          │  ┌────────────────────────────────────────────────┐    │
│          │  │  Elevador parado: qual o procedimento legal?    │    │
│          │  │  [Elevadores] • Maria A. • 5h atrás             │    │
│          │  │                                                │    │
│          │  │  O elevador do meu prédio está parado há 3     │    │
│          │  │  dias e a empresa de manutenção...              │    │
│          │  │                                                │    │
│          │  │  💬 7 respostas     [❤️ Curtir]  [⚠️]          │    │
│          │  └────────────────────────────────────────────────┘    │
│          │                                                        │
│          │  ┌────────────────────────────────────────────────┐    │
│          │  │  Dicas para primeira assembleia como síndico    │    │
│          │  │  [Gestão e Governança] • Carlos R. • 1d atrás   │    │
│          │  │                                                │    │
│          │  │  Fui eleito síndico há 2 semanas e preciso      │    │
│          │  │  organizar a primeira assembleia...              │    │
│          │  │                                                │    │
│          │  │  💬 12 respostas    [❤️ Curtir]  [⚠️]          │    │
│          │  └────────────────────────────────────────────────┘    │
│          │                                                        │
│          │  [Carregar mais...]                                    │
└──────────────────────────────────────────────────────────────────┘
```

### Nota sobre "💬 3 respostas"
- Contagem de RESPOSTAS é pública (necessário para dinâmica de fórum)
- Contagem de LIKES permanece privada (doc 16)
- O botão de like NÃO mostra número para público

### CSS — Post Card

```css
.forum-post-card {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  margin-bottom: 12px;
  transition: all 200ms;
}
.forum-post-card:hover {
  border-color: var(--primary);
  cursor: pointer;
}

.forum-post-title {
  font-family: 'Manrope';
  font-size: 17px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 6px;
  line-height: 1.3;
}

.forum-post-meta {
  display: flex;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
  margin-bottom: 10px;
}

.forum-post-category {
  display: inline-flex;
  padding: 2px 8px;
  background: oklch(0.715 0.120 84.2 / 0.1);
  color: var(--primary);
  border-radius: 6px;
  font-family: 'Manrope';
  font-size: 12px;
  font-weight: 600;
}

.forum-post-author {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

.forum-post-time {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

.forum-post-excerpt {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin-bottom: 12px;
}

.forum-post-footer {
  display: flex;
  align-items: center;
  gap: 16px;
  padding-top: 10px;
  border-top: 1px solid var(--border);
}

.forum-reply-count {
  display: flex;
  align-items: center;
  gap: 4px;
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
}

/* Botão curtir sem contador (público) */
/* Reutiliza .interaction-btn do doc 16 */
```

### CSS — Tabs e Search

```css
.forum-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 20px;
}

.forum-tabs {
  display: flex;
  gap: 0;
  border-bottom: 1px solid var(--border);
  margin-bottom: 20px;
}

.forum-tab {
  padding: 10px 16px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 500;
  color: var(--muted-foreground);
  background: none;
  border: none;
  border-bottom: 2px solid transparent;
  cursor: pointer;
  transition: all 200ms;
}
.forum-tab:hover { color: var(--foreground); }
.forum-tab[data-selected] {
  color: var(--primary);
  border-bottom-color: var(--primary);
  font-weight: 600;
}

.forum-search {
  position: relative;
  margin-bottom: 20px;
}

.forum-search-input {
  width: 100%;
  padding: 10px 14px 10px 40px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 10px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  outline: none;
}
.forum-search-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.1);
}

.forum-search-icon {
  position: absolute;
  left: 12px;
  top: 50%;
  transform: translateY(-50%);
  color: var(--muted-foreground);
}

/* Category pills — reutiliza padrão horizontal scroll */
.forum-categories {
  display: flex;
  gap: 8px;
  overflow-x: auto;
  padding-bottom: 8px;
  margin-bottom: 20px;
}
.forum-categories::-webkit-scrollbar { display: none; }

.forum-category-pill {
  padding: 6px 14px;
  border-radius: 9999px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 500;
  white-space: nowrap;
  border: 1px solid var(--border);
  background: var(--card);
  color: var(--muted-foreground);
  cursor: pointer;
  transition: all 200ms;
  flex-shrink: 0;
}
.forum-category-pill:hover {
  border-color: var(--primary);
  color: var(--primary);
}
.forum-category-pill.active {
  background: var(--primary);
  color: var(--primary-foreground);
  border-color: var(--primary);
}
```

---

## LAYOUT — DETALHE DO POST

Rota: `/forum/post/:id`

```
┌──────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                    │
│          │  ← Voltar ao fórum                                │
│          │                                                    │
│          │  Como contratar impermeabilização?                 │
│          │  h1 Manrope 22px bold                              │
│          │                                                    │
│          │  [Manutenção Predial]                              │
│          │  [avatar] João Silva • Síndico Pro • 2h atrás     │
│          │                                                    │
│          │  Preciso fazer impermeabilização da cobertura do   │
│          │  meu condomínio e não sei por onde começar. O      │
│          │  orçamento é limitado e temos 3 propostas...       │
│          │  ...conteúdo completo do post...                   │
│          │                                                    │
│          │  [❤️ Curtir]  [⚠️ Denunciar]                      │
│          │                                                    │
│          │  ─── RESPOSTAS (3) ─────────────────────────────  │
│          │                                                    │
│          │  ┌────────────────────────────────────────────┐    │
│          │  │ [avatar] Carlos A. • Emp. Pro • 1h atrás   │    │
│          │  │                                            │    │
│          │  │ A impermeabilização de cobertura exige      │    │
│          │  │ análise técnica prévia. Recomendo começar   │    │
│          │  │ por um laudo de...                          │    │
│          │  │                                            │    │
│          │  │ [❤️ Curtir]  [⚠️]                          │    │
│          │  └────────────────────────────────────────────┘    │
│          │                                                    │
│          │  ┌────────────────────────────────────────────┐    │
│          │  │ [avatar] Ana L. • Marketing • 45min atrás  │    │
│          │  │                                            │    │
│          │  │ Na nossa experiência, o processo tem 3      │    │
│          │  │ etapas principais...                        │    │
│          │  │                                            │    │
│          │  │ [❤️ Curtir]  [⚠️]                          │    │
│          │  └────────────────────────────────────────────┘    │
│          │                                                    │
│          │  ─── SUA RESPOSTA ──────────────────────────────  │
│          │  ┌────────────────────────────────────────────┐    │
│          │  │ Escreva sua resposta...                     │    │
│          │  │                                            │    │
│          │  │                                            │    │
│          │  └────────────────────────────────────────────┘    │
│          │  0/2000                        [Responder →]      │
└──────────────────────────────────────────────────────────────┘
```

### CSS — Post Detail

```css
.forum-detail { max-width: 780px; margin: 0 auto; }

.forum-detail-back {
  display: flex;
  align-items: center;
  gap: 6px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--muted-foreground);
  margin-bottom: 20px;
  cursor: pointer;
  background: none;
  border: none;
}
.forum-detail-back:hover { color: var(--primary); }

.forum-detail-title {
  font-family: 'Manrope';
  font-size: 22px;
  font-weight: 700;
  color: var(--foreground);
  line-height: 1.3;
  margin-bottom: 12px;
}

.forum-detail-author {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.forum-detail-author-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: var(--muted);
  object-fit: cover;
}

.forum-detail-author-info {
  display: flex;
  flex-direction: column;
}

.forum-detail-author-name {
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  color: var(--foreground);
}

.forum-detail-author-role {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
}

.forum-detail-content {
  font-family: 'Manrope';
  font-size: 15px;
  color: var(--foreground);
  line-height: 1.8;
  margin-bottom: 20px;
}

/* Respostas */
.forum-replies-section { margin-top: 28px; }

.forum-replies-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.12em;
  color: var(--muted-foreground);
  padding-bottom: 12px;
  border-bottom: 1px solid var(--border);
  margin-bottom: 16px;
}

.forum-reply-item {
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 10px;
  padding: 16px;
  margin-bottom: 12px;
}

.forum-reply-header {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 10px;
}

.forum-reply-avatar {
  width: 32px;
  height: 32px;
  border-radius: 50%;
  background: var(--muted);
  object-fit: cover;
}

.forum-reply-author-name {
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  color: var(--foreground);
}

.forum-reply-author-role {
  font-family: 'Manrope';
  font-size: 11px;
  color: var(--primary);
  font-weight: 500;
}

.forum-reply-time {
  font-family: 'Manrope';
  font-size: 12px;
  color: var(--muted-foreground);
  margin-left: auto;
}

.forum-reply-text {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.7;
  margin-bottom: 10px;
}

/* Input de resposta */
.forum-reply-input-section {
  margin-top: 24px;
  padding-top: 20px;
  border-top: 1px solid var(--border);
}

.forum-reply-input-title {
  font-family: 'Michroma';
  font-size: 11px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--muted-foreground);
  margin-bottom: 12px;
}

.forum-reply-textarea {
  width: 100%;
  min-height: 100px;
  padding: 12px 14px;
  background: var(--background);
  border: 1px solid var(--input);
  border-radius: 10px;
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  resize: vertical;
  outline: none;
}
.forum-reply-textarea:focus {
  border-color: var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}

.forum-reply-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 8px;
}

.forum-reply-charcount {
  font-size: 12px;
  color: var(--muted-foreground);
}

.forum-reply-submit {
  padding: 10px 20px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 14px;
  font-weight: 600;
  cursor: pointer;
}
.forum-reply-submit:hover { background: oklch(0.65 0.120 84.2); }
```

---

## FORM — NOVO POST

Rota: `/forum/novo`

```
┌──────────────────────────────────────────┐
│  NOVO POST NO FÓRUM                     │
│                                          │
│  Categoria *                             │
│  [▼ Selecione uma categoria        ]    │
│                                          │
│  Título *                                │
│  ┌──────────────────────────────────┐    │
│  │ Título da sua pergunta...        │    │
│  └──────────────────────────────────┘    │
│                                          │
│  Conteúdo *                              │
│  ┌──────────────────────────────────┐    │
│  │ Descreva seu problema ou dúvida  │    │
│  │ com o máximo de detalhes...      │    │
│  │                                  │    │
│  │                                  │    │
│  │                                  │    │
│  └──────────────────────────────────┘    │
│  0/5000                                  │
│                                          │
│  [Cancelar]             [Publicar →]    │
└──────────────────────────────────────────┘
```

### Validação (Zod)

```ts
import { z } from "zod";

export const ForumPostSchema = z.object({
  categoriaId: z.string().min(1, "Selecione uma categoria"),
  titulo: z.string().min(5, "Mín. 5 caracteres").max(150, "Máx. 150 caracteres"),
  conteudo: z.string().min(20, "Descreva com mais detalhes").max(5000),
});

export const ForumReplySchema = z.object({
  conteudo: z.string().min(5, "Mín. 5 caracteres").max(2000),
});
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `ForumPostCard` | Card na listagem | — |
| `ForumPostDetail` | Página do post com respostas | — |
| `ForumReplyItem` | Item de resposta | — |
| `ForumReplyInput` | Textarea de resposta | TanStack Form + Zod |
| `ForumNewPostForm` | Form de novo post | TanStack Form + Zod |
| `ForumSearch` | Input de busca | — |
| `ForumTabs` | Recentes/Populares/Minhas | Kobalte Tabs |
| `ForumCategoryPills` | Scroll de categorias | — |
| `InteractionBar` | Curtir/Denunciar (doc 16) | — |

---

## ESTADOS

| Estado | Comportamento |
|---|---|
| Loading | Skeleton: 4 post cards com shimmer |
| Empty (fórum sem posts) | Ilustração + "Seja o primeiro a postar!" + [Criar Post →] |
| Empty (filtro sem resultado) | "Nenhum post encontrado para '[busca]'" |
| Post sem respostas | Seção respostas com: "Ainda sem respostas. Seja o primeiro!" |
| Sem permissão (Base/Morador Pg) | Rota retorna 403, tela nem carrega |
| Denúncia enviada | Toast de confirmação (solid-sonner) |

---

## CHECKLIST

- [ ] Listagem com search + tabs + category pills
- [ ] Post cards com título, excerpt, meta, reply count
- [ ] Reply count visível (diferente de likes, que são privados)
- [ ] Detalhe do post com respostas em thread linear
- [ ] Author info com avatar + nome + role badge
- [ ] Input de resposta com validação 5-2000 chars
- [ ] Form novo post: categoria + título + conteúdo
- [ ] Botão curtir (sem contador público) em posts e respostas
- [ ] Botão denúncia em posts e respostas (modal doc 16)
- [ ] Pills de categoria scrolláveis (31 categorias do Cap. 6)
- [ ] Busca por palavra-chave (debounced)
- [ ] Dark mode tokens oklch
