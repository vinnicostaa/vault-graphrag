---
title: Fórum & Comunidade — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, forum, commercial, vizinhanca]
bc: cross
source: _chaos/17-FORUM.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 17 — Fórum e Comunidade

> **Origem**: `_chaos/17-FORUM.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: `N1/N2/N3` → `trial/base/plus/pro` (ADR-0032 / D-081 / D-096); "Morador Pagante" removido (D-126).

> Sprint 4 · App `forum` (porta 3003) · Rotas: `/forum`, `/forum/:categoriaSlug`, `/forum/post/:id`, `/forum/novo`
> Categorias definidas conforme **Cap. 6 do Manual Técnico** (31 categorias de serviço).

---

## Regras de negócio

### Estrutura
- **Categorias**: 31 categorias técnicas (Manual Cap. 6).
- **Posts**: Título + Conteúdo (texto) + Categoria.
- **Respostas**: texto simples em thread linear (sem sub-threads / replies aninhados).
- **Likes**: visíveis apenas ao dono (ver [[../cross/private-feedback|cross/private-feedback]]). Contagem de **respostas** é pública (necessário para dinâmica de fórum).
- **Busca por palavra-chave** via `ISearchProvider` (D-120).

### Permissões (matriz)

| Ação | Morador `base` | Síndico (`base`+) | Empresa `plus`/`pro` | Marketing |
|---|---|---|---|---|
| Acessar fórum | ❌ | ✅ | ✅ | ✅ |
| Criar post | ❌ | ✅ | ✅ | ✅ |
| Responder | ❌ | ✅ | ✅ | ✅ |
| Curtir | ❌ | ✅ | ✅ | ✅ |

> Morador é gratuito (`base`) — não acessa fórum no MVP. Caso evolua para tier morador `plus` / `pro`, abrir acesso por matriz.

### Moderação
- **Reativa**: botão de denúncia em cada post / resposta.
- **Sem IA de moderação automática**.
- **Admin Master Síndico** aprova / reprova / remove posts denunciados (D-134 — admin em `apps/admin`).

---

## Layout — Listagem do Fórum

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  FÓRUM DA COMUNIDADE             [+ Novo Post →]      │
│          │  🔍 [Buscar no fórum...]                              │
│          │  [Recentes] [Populares] [Minhas Perguntas]            │
│          │                                                        │
│          │  ── CATEGORIAS ──                                      │
│          │  [📋Gestão] [⚖️Jurídico] [🔧Manutenção] [💰Finanças] │
│          │                                                        │
│          │  ┌──────────────────────────────────────────┐          │
│          │  │  Como contratar impermeabilização?        │          │
│          │  │  [Manutenção] • João S. • 2h             │          │
│          │  │  Preciso fazer impermeabilização da...    │          │
│          │  │  💬 3 respostas    [❤️] [⚠️]             │          │
│          │  └──────────────────────────────────────────┘          │
└──────────────────────────────────────────────────────────────────┘
```

### CSS — Post Card

```css
.forum-post-card {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl); padding: 20px; margin-bottom: 12px;
  transition: all var(--duration-moderate) var(--ease-out);
}
.forum-post-card:hover { border-color: var(--primary); cursor: pointer; }
.forum-post-title {
  font-family: 'Manrope'; font-size: 17px; font-weight: 700;
  color: var(--foreground); margin-bottom: 6px; line-height: 1.3;
}
.forum-post-category {
  display: inline-flex; padding: 2px 8px;
  background: oklch(0.715 0.120 84.2 / 0.1); color: var(--primary);
  border-radius: var(--radius-sm);
  font-family: 'Manrope'; font-size: 12px; font-weight: 600;
}
.forum-post-excerpt {
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  line-height: 1.6; margin-bottom: 12px;
  display: -webkit-box; -webkit-line-clamp: 2;
  -webkit-box-orient: vertical; overflow: hidden;
}
.forum-post-footer {
  display: flex; align-items: center; gap: 16px;
  padding-top: 10px; border-top: 1px solid var(--border);
}
.forum-reply-count {
  display: flex; align-items: center; gap: 4px;
  font-family: 'Manrope'; font-size: 13px; color: var(--muted-foreground);
}

/* Tabs e Search */
.forum-tabs {
  display: flex; border-bottom: 1px solid var(--border); margin-bottom: 20px;
}
.forum-tab {
  padding: 10px 16px; font-family: 'Manrope'; font-size: 14px; font-weight: 500;
  color: var(--muted-foreground); background: none; border: none;
  border-bottom: 2px solid transparent; cursor: pointer;
  transition: all var(--duration-moderate) var(--ease-out);
}
.forum-tab[data-selected] {
  color: var(--primary); border-bottom-color: var(--primary); font-weight: 600;
}

/* Category pills horizontal scroll */
.forum-categories {
  display: flex; gap: 8px; overflow-x: auto;
  padding-bottom: 8px; margin-bottom: 20px;
}
.forum-categories::-webkit-scrollbar { display: none; }
.forum-category-pill {
  padding: 6px 14px; border-radius: var(--radius-full);
  font-family: 'Manrope'; font-size: 13px; font-weight: 500;
  white-space: nowrap; border: 1px solid var(--border);
  background: var(--card); color: var(--muted-foreground);
  cursor: pointer; transition: all var(--duration-moderate) var(--ease-out);
  flex-shrink: 0;
}
.forum-category-pill.active {
  background: var(--primary); color: var(--primary-foreground);
  border-color: var(--primary);
}
```

---

## Layout — Detalhe do Post

Rota: `/forum/post/:id`.

Estrutura: header + título + author info + conteúdo + InteractionBar + lista de respostas em thread linear + input de nova resposta.

```css
.forum-detail { max-width: 780px; margin: 0 auto; }
.forum-detail-title {
  font-family: 'Manrope'; font-size: 22px; font-weight: 700;
  color: var(--foreground); line-height: 1.3; margin-bottom: 12px;
}
.forum-reply-item {
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-md); padding: 16px; margin-bottom: 12px;
}
.forum-reply-textarea {
  width: 100%; min-height: 100px; padding: 12px 14px;
  background: var(--background); border: 1px solid var(--input);
  border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 14px; color: var(--foreground);
  resize: vertical; outline: none;
}
.forum-reply-textarea:focus {
  border-color: var(--ring);
  box-shadow: 0 0 0 3px oklch(0.715 0.120 84.2 / 0.15);
}
```

---

## Form — Novo Post

Rota: `/forum/novo`. Validação Zod:

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

## Componentes

`ForumPostCard`, `ForumPostDetail`, `ForumReplyItem`, `ForumReplyInput`, `ForumNewPostForm`, `ForumSearch`, `ForumTabs`, `ForumCategoryPills`, `InteractionBar` (de [[../cross/private-feedback|private-feedback]]).

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[../cross/private-feedback|private-feedback]] — InteractionBar reutilizável
- [[../../../web/ui-catalog#d-app-forum]] — app `forum`
- [[../../../../00-product/business-model#41-matriz-consolidada]]
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
