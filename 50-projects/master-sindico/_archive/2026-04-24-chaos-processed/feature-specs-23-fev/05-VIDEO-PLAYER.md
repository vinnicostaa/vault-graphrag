# 05 — VIDEO PLAYER & CONSUMO DE VÍDEO

> Sprint 1 · Rotas: /video/:id, /video/:id/fullscreen (mobile TikTok-style)
> Integração: Mux ou Cloudflare Stream

---

## REGRAS DE NEGÓCIO

- Player customizado com controles da marca
- Paywall 25%: Base vendo vídeo de Empresa → corta em 25%, overlay navy blur
- Contagem de visualização: apenas se >70% assistido (pontuação do síndico)
- Feedback Privado: likes/comentários visíveis APENAS para o dono do conteúdo
- Público geral vê APENAS o vídeo, sem contadores, sem comentários
- Sem editor de vídeo — upload de arquivo pronto

## LAYOUT — DESKTOP

```
┌────────────────────────────────────────────────────────────────┐
│  SIDEBAR │                                                      │
│          │  PLAYER (aspect 16:9, max-height 70vh)               │
│          │  ┌────────────────────────────────────────────────┐  │
│          │  │                                                │  │
│          │  │              VIDEO PLAYER                      │  │
│          │  │         (Mux/Cloudflare embed)                 │  │
│          │  │                                                │  │
│          │  │  [▶/⏸] [⏮] [Vol] [━━━━●━━━] [CC] [⚙] [⛶]  │  │
│          │  └────────────────────────────────────────────────┘  │
│          │                                                      │
│          │  METADATA                                            │
│          │  ┌────────────────────────────────────────────────┐  │
│          │  │ Título do Vídeo   h2 Michroma 20px             │  │
│          │  │                                                │  │
│          │  │ [Avatar] Canal  [Seguir] [Connect Me]          │  │
│          │  │          @handle • Status                      │  │
│          │  │                                                │  │
│          │  │ [👍 Curtir] [↗ Compartilhar] [⚑ Denunciar]   │  │
│          │  │                                                │  │
│          │  │ Descrição... #categorias                       │  │
│          │  └────────────────────────────────────────────────┘  │
│          │                                                      │
│          │  ┌─── 2 COLUNAS ───────────────────────────────┐    │
│          │  │ COMENTÁRIOS       │  VÍDEOS RELACIONADOS     │    │
│          │  │ (só se owner)     │  [VideoCard H]           │    │
│          │  │ OU                │  [VideoCard H]           │    │
│          │  │ SEÇÃO VAZIA       │  [VideoCard H]           │    │
│          │  │ (se não owner)    │                          │    │
│          │  └───────────────────┴──────────────────────────┘    │
└────────────────────────────────────────────────────────────────┘
```

## LAYOUT — MOBILE

```
┌─────────────────────────┐
│  PLAYER (full-width)    │
│  aspect-ratio 16:9      │
│  controles overlay      │
├─────────────────────────┤
│  Título do Vídeo        │
│  [Avatar] Canal • Status│
│                         │
│  [👍] [↗] [⚑]         │
│                         │
│  Descrição... [mais]    │
│                         │
│  Vídeos Relacionados    │
│  scroll horizontal      │
│  [VC][VC][VC] →         │
└─────────────────────────┘
```

## TIKTOK-STYLE FULLSCREEN (Mobile vídeos curtos)

Para vídeos curtos (≤60s), suportar modo vertical fullscreen estilo TikTok/Reels.
Swipe up = próximo vídeo, swipe down = anterior.

```
┌──────────────────────────┐
│  [← Back]  #categoria  [⋮]│  ← header overlay
│                            │
│                            │
│       VIDEO FULLSCREEN     │
│       (object-fit cover)   │
│                            │
│                            │
│                     [♡] 0  │  ← action buttons
│                     [💬] 0 │     (vertical stack)
│                     [↗]    │     right side
│                            │
│  [Avatar] @handle          │  ← bottom info overlay
│  #hashtag                  │
│  Descrição truncada...     │
│  ...mais                   │
└──────────────────────────┘
```

```css
.video-fullscreen { position: fixed; inset: 0; background: #000; z-index: 100; }
.video-fullscreen video { width: 100%; height: 100%; object-fit: cover; }
.video-fullscreen-header {
  position: absolute; top: 0; left: 0; right: 0; padding: 16px;
  background: linear-gradient(to bottom, rgba(0,0,0,0.6), transparent);
  display: flex; justify-content: space-between; z-index: 2;
}
.video-fullscreen-actions {
  position: absolute; right: 12px; bottom: 120px;
  display: flex; flex-direction: column; gap: 20px; align-items: center; z-index: 2;
}
.video-fullscreen-action-btn {
  width: 44px; height: 44px; border-radius: 50%;
  background: rgba(255,255,255,0.15); backdrop-filter: blur(8px);
  display: flex; align-items: center; justify-content: center;
  color: white;
}
.video-fullscreen-action-count { font-size: 11px; color: white; margin-top: 2px; }
.video-fullscreen-info {
  position: absolute; bottom: 0; left: 0; right: 80px; padding: 16px;
  background: linear-gradient(to top, rgba(0,0,0,0.7), transparent);
  z-index: 2;
}
.video-fullscreen-info .handle { color: white; font-weight: 600; font-size: 14px; }
.video-fullscreen-info .description { color: rgba(255,255,255,0.8); font-size: 13px; line-height: 1.4; }
```

PRIVACIDADE: Contadores de like/comentário em TikTok-style são visíveis APENAS se user == owner.
Para não-owner, mostrar apenas ícones sem números.

## PLAYER CONTROLS

```css
.player-controls {
  position: absolute; bottom: 0; left: 0; right: 0;
  padding: 8px 12px; background: linear-gradient(to top, rgba(0,0,0,0.8), transparent);
  display: flex; align-items: center; gap: 8px;
}
.player-progress {
  flex: 1; height: 4px; background: rgba(255,255,255,0.3);
  border-radius: 2px; cursor: pointer; position: relative;
}
.player-progress-fill { height: 100%; background: var(--primary); border-radius: 2px; position: relative; }
.player-progress-thumb {
  width: 12px; height: 12px; border-radius: 50%; background: var(--primary);
  position: absolute; right: -6px; top: -4px;
  opacity: 0; transition: opacity 200ms;
}
.player-controls:hover .player-progress-thumb { opacity: 1; }
.player-btn { color: white; padding: 4px; cursor: pointer; }
.player-time { font-family: 'Manrope'; font-size: 12px; color: rgba(255,255,255,0.8); }
```

## PAYWALL (25%)

Quando user Base vê vídeo de Empresa, player corta em 25%:

```css
.paywall-overlay {
  position: absolute; inset: 0; display: flex; flex-direction: column;
  align-items: center; justify-content: center; text-align: center;
  background: oklch(0.218 0.055 256.1 / 0.9); backdrop-filter: blur(8px);
  z-index: 10; padding: 24px;
}
.paywall-icon { width: 48px; height: 48px; color: var(--primary); margin-bottom: 16px; }
.paywall-title { font-family: 'Michroma'; font-size: 18px; color: white; margin-bottom: 8px; }
.paywall-text { font-family: 'Manrope'; font-size: 14px; color: rgba(255,255,255,0.7); margin-bottom: 24px; }
.paywall-cta {
  padding: 12px 32px; background: var(--primary); color: var(--primary-foreground);
  border-radius: var(--radius); font-weight: 600; cursor: pointer;
}
```

## VIDEO CARD (componente reutilizado em toda app)

```css
.video-card { cursor: pointer; transition: all 200ms; }
.video-card:hover { transform: scale(1.02); }
.video-card-thumbnail {
  aspect-ratio: 16/9; border-radius: 12px; overflow: hidden;
  background: var(--muted); position: relative;
}
.video-card-thumbnail img { width: 100%; height: 100%; object-fit: cover; }
.video-card-play-overlay {
  position: absolute; inset: 0; display: flex; align-items: center; justify-content: center;
  background: rgba(0,0,0,0.3); opacity: 0; transition: opacity 200ms;
}
.video-card:hover .video-card-play-overlay { opacity: 1; }
.video-card-play-icon { width: 48px; height: 48px; color: white; }
.video-card-duration {
  position: absolute; bottom: 8px; right: 8px;
  background: rgba(0,0,0,0.8); color: white; font-size: 12px;
  padding: 2px 6px; border-radius: 4px; font-family: 'Manrope';
}
.video-card-meta { display: flex; gap: 12px; padding: 8px 0; }
.video-card-avatar { width: 36px; height: 36px; border-radius: 50%; flex-shrink: 0; }
.video-card-title { font-family: 'Manrope'; font-size: 14px; font-weight: 600; line-height: 1.3; display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden; }
.video-card-channel { font-family: 'Manrope'; font-size: 13px; color: var(--muted-foreground); }
.video-card-stats { font-family: 'Manrope'; font-size: 12px; color: var(--muted-foreground); }
```

### Variantes do Card

| Variante | Badge | Posição |
|---|---|---|
| Institucional (Empresa) | "INSTITUCIONAL" bg gold, text navy | top-left thumbnail |
| Currículo (Morador) | "CURRÍCULO" bg accent, text navy | top-left thumbnail |
| Pauta (Assembleia) | "PAUTA #3" bg secondary, text white | top-left thumbnail |
| Preview 25% | overlay diagonal "Preview 25%" + 🔒 | sobre thumbnail |
| Travado 90 dias | grayscale + "Atualização em 45d" | sobre thumbnail |

## COMPONENTES

VideoPlayer, VideoPlayerControls, VideoMetadata, VideoActionBar, VideoComments, VideoRelated, VideoCard, VideoCardHorizontal, PaywallOverlay, TikTokPlayer, TikTokActionStack, TikTokInfoOverlay
