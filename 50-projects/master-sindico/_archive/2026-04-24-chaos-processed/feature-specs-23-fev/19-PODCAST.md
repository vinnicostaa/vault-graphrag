# 19 — ABA DE PODCAST (INTEGRAÇÃO YOUTUBE)

> Sprint 4 · Rota: /podcast
> Player YouTube embarcado. Listagem de episódios do canal oficial Master Síndico.

---

## REGRAS DE NEGÓCIO

### Conceito
- Aba de Podcast integra com canal YouTube oficial do MasterSíndico
- Conteúdo sincronizado automaticamente via YouTube Data API
- Cache de metadados (título, descrição, thumbnail)
- NÃO é upload interno — é embed de YouTube

### Permissões
| Ação | Base | Morador Pg | Síndico N1+ | Emp. Plus/Pro | Marketing |
|------|------|-----------|-------------|---------------|-----------|
| Acessar podcast | ❌ | ✅ | ✅ | ✅ | ✅ |

- Disponível para usuários pagantes e planos superiores
- Base NÃO tem acesso

---

## LAYOUT — LISTAGEM DE EPISÓDIOS

Rota: `/podcast`

```
┌──────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                        │
│          │  PODCAST MASTER SÍNDICO          🔍 [Buscar...]       │
│          │  h2 Michroma 20px                                      │
│          │  Conteúdo em áudio e vídeo para sua gestão             │
│          │                                                        │
│          │  ┌─── ÚLTIMO EPISÓDIO ────────────────────────────┐   │
│          │  │                                                 │   │
│          │  │  ┌───────────────────┐                          │   │
│          │  │  │ [thumb 16:9]      │  Ep. 47: Fachadas e a   │   │
│          │  │  │   ▶ play overlay  │  Responsabilidade do     │   │
│          │  │  │                   │  Síndico                 │   │
│          │  │  └───────────────────┘                          │   │
│          │  │  Publicado em 15/02/2026 • 1h23min              │   │
│          │  │  Neste episódio, discutimos os aspectos...      │   │
│          │  │                                                 │   │
│          │  │  [▶ Assistir/Ouvir →]                           │   │
│          │  └─────────────────────────────────────────────────┘   │
│          │                                                        │
│          │  ─── TODOS OS EPISÓDIOS ───────────────────────────   │
│          │                                                        │
│          │  ┌──────────────────────────────────────────────────┐  │
│          │  │ [thumb] │ Ep. 46: Elevadores Modernos           │  │
│          │  │  16:9   │ 08/02/2026 • 55min                    │  │
│          │  │  120px  │ Os desafios da modernização...         │  │
│          │  │         │                            [▶ Ouvir]  │  │
│          │  └──────────────────────────────────────────────────┘  │
│          │                                                        │
│          │  ┌──────────────────────────────────────────────────┐  │
│          │  │ [thumb] │ Ep. 45: Seguros Condominiais          │  │
│          │  │  16:9   │ 01/02/2026 • 1h05min                  │  │
│          │  │  120px  │ Entenda as coberturas essenciais...    │  │
│          │  │         │                            [▶ Ouvir]  │  │
│          │  └──────────────────────────────────────────────────┘  │
│          │                                                        │
│          │  [Carregar mais episódios...]                           │
└──────────────────────────────────────────────────────────────────┘
```

### CSS

```css
.podcast-page { max-width: 900px; margin: 0 auto; }

/* Último episódio destaque */
.podcast-featured {
  background: var(--card);
  border: 1px solid var(--primary);
  border-radius: 16px;
  overflow: hidden;
  display: flex;
  gap: 0;
  margin-bottom: 32px;
}

@media (max-width: 767px) {
  .podcast-featured { flex-direction: column; }
}

.podcast-featured-thumb {
  width: 320px;
  min-height: 180px;
  object-fit: cover;
  background: var(--muted);
  position: relative;
  cursor: pointer;
}

@media (max-width: 767px) {
  .podcast-featured-thumb {
    width: 100%;
    aspect-ratio: 16/9;
  }
}

.podcast-featured-play {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: oklch(0 0 0 / 0.3);
  transition: background 200ms;
}
.podcast-featured-thumb:hover .podcast-featured-play {
  background: oklch(0 0 0 / 0.5);
}

.podcast-featured-play-icon {
  width: 56px;
  height: 56px;
  background: var(--primary);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--primary-foreground);
  font-size: 24px;
}

.podcast-featured-body {
  flex: 1;
  padding: 24px;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.podcast-featured-title {
  font-family: 'Manrope';
  font-size: 20px;
  font-weight: 700;
  color: var(--foreground);
  margin-bottom: 8px;
  line-height: 1.3;
}

.podcast-featured-meta {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 8px;
}

.podcast-featured-desc {
  font-family: 'Manrope';
  font-size: 14px;
  color: var(--foreground);
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin-bottom: 16px;
}

/* Lista de episódios */
.podcast-list { display: flex; flex-direction: column; gap: 12px; }

.podcast-episode-item {
  display: flex;
  gap: 16px;
  padding: 16px;
  background: var(--card);
  border: 1px solid var(--border);
  border-radius: 12px;
  transition: all 200ms;
  cursor: pointer;
}
.podcast-episode-item:hover {
  border-color: var(--primary);
}

@media (max-width: 767px) {
  .podcast-episode-item { flex-direction: column; }
}

.podcast-episode-thumb {
  width: 180px;
  height: 100px;
  object-fit: cover;
  border-radius: 8px;
  background: var(--muted);
  flex-shrink: 0;
}

@media (max-width: 767px) {
  .podcast-episode-thumb {
    width: 100%;
    height: auto;
    aspect-ratio: 16/9;
  }
}

.podcast-episode-body { flex: 1; }

.podcast-episode-title {
  font-family: 'Manrope';
  font-size: 16px;
  font-weight: 600;
  color: var(--foreground);
  margin-bottom: 4px;
}

.podcast-episode-meta {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  margin-bottom: 6px;
}

.podcast-episode-desc {
  font-family: 'Manrope';
  font-size: 13px;
  color: var(--muted-foreground);
  line-height: 1.5;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.podcast-episode-play {
  align-self: center;
  padding: 8px 16px;
  background: var(--primary);
  color: var(--primary-foreground);
  border: none;
  border-radius: 8px;
  font-family: 'Manrope';
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  white-space: nowrap;
  flex-shrink: 0;
}
```

---

## LAYOUT — PLAYER (Embed YouTube)

Ao clicar em um episódio, abre o player:

```
┌──────────────────────────────────────────────────────┐
│  ← Voltar                                            │
│                                                      │
│  ┌──────────────────────────────────────────────┐    │
│  │                                              │    │
│  │            [YOUTUBE IFRAME]                   │    │
│  │             16:9 / max-h 70vh                 │    │
│  │                                              │    │
│  └──────────────────────────────────────────────┘    │
│                                                      │
│  Ep. 47: Fachadas e a Responsabilidade do Síndico    │
│  h2 Manrope 20px bold                                │
│                                                      │
│  Publicado em 15/02/2026 • 1h23min                   │
│                                                      │
│  Descrição completa do episódio...                    │
│  ...texto do YouTube importado...                    │
│                                                      │
│  [← Anterior]              [Próximo Episódio →]      │
└──────────────────────────────────────────────────────┘
```

```css
.podcast-player { max-width: 900px; margin: 0 auto; }

.podcast-embed {
  width: 100%;
  aspect-ratio: 16/9;
  border-radius: 12px;
  overflow: hidden;
  background: #000;
  margin-bottom: 24px;
}

.podcast-embed iframe {
  width: 100%;
  height: 100%;
  border: none;
}

@media (max-width: 767px) {
  .podcast-embed { border-radius: 0; }
}
```

---

## COMPONENTES

| Componente | Tipo | Dependência |
|---|---|---|
| `PodcastFeatured` | Card destaque último episódio | — |
| `PodcastEpisodeItem` | Item na lista | — |
| `PodcastPlayer` | Página com embed YouTube | — |
| `PodcastSearch` | Busca por episódio | — |

---

## CHECKLIST

- [ ] Card destaque último episódio (horizontal com play overlay)
- [ ] Lista vertical de episódios com thumb 180x100
- [ ] Player: iframe YouTube responsivo 16:9
- [ ] Metadados via YouTube Data API (cache no backend)
- [ ] Busca por título de episódio
- [ ] Navegação anterior/próximo
- [ ] Sem acesso para Base (403)
- [ ] Mobile: cards verticais full-width
- [ ] Dark mode tokens oklch
