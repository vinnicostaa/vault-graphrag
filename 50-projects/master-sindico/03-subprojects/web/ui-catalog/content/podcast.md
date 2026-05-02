---
title: Podcast (YouTube embed) — UI Spec
type: ui-spec
tags: [ui-catalog, master-sindico, content, podcast]
bc: content
source: _chaos/19-PODCAST.md (2026-02-23)
created: 2026-04-25
updated: 2026-04-25
absorbed_in: 2026-04-25 — Fase B do plano de consolidação _chaos/
status: absorbed
---

# 19 — Aba de Podcast (Integração YouTube)

> **Origem**: `_chaos/19-PODCAST.md` (gerado 2026-02-23 — pré-migração).
> **Absorvido em**: 2026-04-25 — Fase B do plano de consolidação.
> **Tradução aplicada**: vocabulário neutro; "Morador Pagante" removido (D-126); "My Síndico" → "Master Síndico".
> **Decisão João 2026-04-24**: Podcast é **feature interna de Conteúdo** (não sub-produto separado). Vai em `content/`.

> Sprint 4 · App `cms` / `lms` · Rota: `/podcast`
> Player YouTube embarcado. Listagem de episódios do canal oficial Master Síndico.

---

## Regras de negócio

### Conceito
- Aba Podcast integra com canal YouTube oficial do Master Síndico.
- Conteúdo sincronizado automaticamente via **YouTube Data API**.
- Cache de metadados (título, descrição, thumbnail) no backend.
- **NÃO é upload interno** — é embed de YouTube.

### Permissões

| Ação | Morador `base` | Morador `plus` (futuro) | Síndico | Empresa `plus`/`pro` | Marketing |
|---|---|---|---|---|---|
| Acessar podcast | ❌ | ✅ | ✅ | ✅ | ✅ |

> Disponível para usuários com plano pago / planos superiores. `base` não acessa.

---

## Layout — Listagem de episódios

Rota: `/podcast`. Estrutura:

1. **Último Episódio** (card destaque horizontal com play overlay)
2. **Todos os Episódios** (lista vertical com thumb 180×100)

### CSS

```css
.podcast-page { max-width: 900px; margin: 0 auto; }

/* Featured episode */
.podcast-featured {
  background: var(--card);
  border: 1px solid var(--primary);
  border-radius: var(--radius-2xl);
  overflow: hidden;
  display: flex; gap: 0; margin-bottom: 32px;
}
@media (max-width: 767px) { .podcast-featured { flex-direction: column; } }
.podcast-featured-thumb {
  width: 320px; min-height: 180px;
  object-fit: cover; background: var(--muted); position: relative; cursor: pointer;
}
.podcast-featured-play-icon {
  width: 56px; height: 56px;
  background: var(--primary); border-radius: 50%;
  display: flex; align-items: center; justify-content: center;
  color: var(--primary-foreground); font-size: 24px;
}
.podcast-featured-title {
  font-family: 'Manrope'; font-size: 20px; font-weight: 700;
  color: var(--foreground); margin-bottom: 8px; line-height: 1.3;
}

/* Episode list item */
.podcast-episode-item {
  display: flex; gap: 16px;
  padding: 16px;
  background: var(--card); border: 1px solid var(--border);
  border-radius: var(--radius-xl);
  transition: all var(--duration-moderate) var(--ease-out);
  cursor: pointer;
}
.podcast-episode-item:hover { border-color: var(--primary); }
@media (max-width: 767px) { .podcast-episode-item { flex-direction: column; } }
.podcast-episode-thumb {
  width: 180px; height: 100px; object-fit: cover;
  border-radius: var(--radius-md); background: var(--muted); flex-shrink: 0;
}
.podcast-episode-title {
  font-family: 'Manrope'; font-size: 16px; font-weight: 600;
  color: var(--foreground); margin-bottom: 4px;
}
.podcast-episode-meta {
  font-family: 'Manrope'; font-size: 13px; color: var(--muted-foreground);
  margin-bottom: 6px;
}
.podcast-episode-play {
  align-self: center;
  padding: 8px 16px;
  background: var(--primary); color: var(--primary-foreground);
  border: none; border-radius: var(--radius-md);
  font-family: 'Manrope'; font-size: 13px; font-weight: 600;
  cursor: pointer; white-space: nowrap; flex-shrink: 0;
}
```

---

## Layout — Player (Embed YouTube)

```css
.podcast-player { max-width: 900px; margin: 0 auto; }
.podcast-embed {
  width: 100%; aspect-ratio: 16/9;
  border-radius: var(--radius-xl);
  overflow: hidden; background: #000; margin-bottom: 24px;
}
.podcast-embed iframe { width: 100%; height: 100%; border: none; }
@media (max-width: 767px) { .podcast-embed { border-radius: 0; } }
```

## Componentes

`PodcastFeatured`, `PodcastEpisodeItem`, `PodcastPlayer` (iframe YouTube), `PodcastSearch`.

## Estados

Padrão (loading / empty / error / success / permission-denied) seguem [[../../patterns/ui-states|patterns/ui-states]].

## Links

- [[./video-player|video-player]] — para vídeos próprios (Mux), não YouTube
- [[../../../../00-product/sub-produtos/04-conteudo-videos]] — Podcast como feature interna
- [[../../patterns/ui-states|patterns/ui-states]]
- [[../../../../02-architecture/design-tokens|design-tokens]]
