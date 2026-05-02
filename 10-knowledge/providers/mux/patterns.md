---
title: Mux — patterns
type: pattern
tags: [provider, mux, video, patterns, webhook, jwt, live]
source: https://docs.mux.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Mux — patterns

Padrões de produção consolidados para integrações Mux. Cada padrão resolve um problema recorrente e aponta a antipattern oposta em [[antipatterns]].

---

## 1. Signed playback URLs

**Problema**: qualquer um com o `playback_id` acessa o HLS se o policy for `public`. Conteúdo privado precisa de gate por-requisição.

**Solução**: Signing Key + JWT emitido pelo backend a cada pedido de viewer autenticado.

Payload mínimo:

| Claim | Valor | Observação |
|---|---|---|
| `sub` | `playback_id` do asset/live | Obrigatório |
| `aud` | `v` (vídeo) \| `t` (thumb) \| `s` (storyboard) \| `g` (gif) | Cada recurso exige um aud separado |
| `exp` | unix timestamp | TTL curto: 10-60 min para VOD, 5-15 min para live |
| `kid` | Signing Key ID | Vai no header JWT, não no payload |

**Go (HS256)**:

```go
claims := jwt.MapClaims{
    "sub": playbackID,
    "aud": "v",
    "exp": time.Now().Add(30 * time.Minute).Unix(),
}
tok := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
tok.Header["kid"] = signingKeyID
signed, _ := tok.SignedString([]byte(signingKeyPrivate))
```

URL final: `https://stream.mux.com/{playback_id}.m3u8?token={signed}`.

**Estratégia de TTL**:

- **VOD comum**: 30-60 min. Se o usuário fica ≥ TTL, o player re-requisita segments e recebe 403 — player Mux faz refresh automático se `playback-token` for refrescado via JS.
- **Live**: 10-15 min, com refresh em event listener `token-expired` do Mux Player.
- **Download (MP4)**: 5-10 min. URL curta derrota sharing casual.

**Nunca armazenar JWT no DB**: gerar on-demand por request. Armazenar derrota todo o propósito do TTL curto.

---

## 2. Webhook lifecycle — state machine

Cada Asset atravessa estados determinísticos. Modelar como state machine no domínio evita bug de "liberar vídeo antes de estar pronto".

```
(upload)
   │
   ▼
video.upload.asset_created  ─────►  created, asset_id linked
   │
   ▼
video.asset.created         ─────►  preparing
   │
   ▼
video.asset.ready           ─────►  ready ✓ (disparar notificação downstream)
   │                                 │
   │                                 ▼
   │                        disponível para viewers
   │
   └── video.asset.errored  ─────►  errored (expor erro ao admin)
```

**Regra**: liberar o vídeo no app **somente** após `video.asset.ready`. Não pollar `GET /assets/{id}` a cada X segundos — webhook é autoritativo e barato.

**Downstream fan-out**: publicar evento de domínio `VideoReady(assetId, playbackIds[])` no event bus interno. Consumers: notificação push, email, invalidação de cache.

---

## 3. Upload direto cliente → Mux

**Problema**: server-proxied upload dobra banda e prende worker durante minutos.

**Solução**: backend gera `upload.url` via `POST /video/v1/uploads`, cliente faz `PUT` direto nesse URL.

Benefícios:
- Zero tráfego no backend
- Progress bar nativo via `XMLHttpRequest.upload.onprogress`
- Resumable uploads (Mux suporta Tus se habilitado)

Amarração de identidade: passar `new_asset_settings.passthrough` com IDs do domínio (`tenant:X;video:Y`). O webhook devolve esse valor sem transformação.

```ts
// backend
const upload = await mux.video.uploads.create({
  cors_origin: 'https://app.example.com',
  new_asset_settings: {
    playback_policy: ['signed'],
    passthrough: JSON.stringify({ tenant, videoId }),
  },
});
```

---

## 4. Live Stream — reconnect window

**Problema**: rede do broadcaster caiu por 20 segundos. Encerrar a live imediatamente quebra UX.

**Solução**: campo `reconnect_window` (segundos) na criação da live. Mux mantém o stream vivo por esse período esperando RTMP re-ingest.

```ts
mux.video.liveStreams.create({
  reconnect_window: 60,   // até 1800 (30 min)
  latency_mode: 'low',
});
```

**Protocolo cliente**:
1. OBS/cliente detecta erro de rede
2. Retry RTMP com backoff (1s, 2s, 4s, 8s, ...)
3. Se reconectar dentro de `reconnect_window`, live segue como mesma session (mesmo `playback_id`, mesmo Asset final)
4. Se exceder, Mux emite `video.live_stream.disconnected` e depois `idle`; próxima ingest cria sessão nova

Viewer vê um pequeno gap mas a URL HLS segue a mesma.

---

## 5. Simulcast targets — replicação RTMP

**Problema**: broadcaster quer aparecer simultaneamente em YouTube Live + Twitch + destino próprio.

**Solução**: `POST /video/v1/live-streams/{id}/simulcast-targets` com `url` RTMP + `stream_key` do destino.

```ts
await mux.video.liveStreams.simulcastTargets.create(liveId, {
  url: 'rtmp://a.rtmp.youtube.com/live2',
  stream_key: process.env.YOUTUBE_STREAM_KEY,
  passthrough: 'youtube-main',
});
```

Mux mantém o fan-out enquanto a live estiver `active`. Cobrança: por simulcast target ativo/minuto. Cada target tem seu próprio estado (`idle` / `starting` / `broadcasting` / `errored`) emitido via webhook.

---

## 6. Idempotência — `passthrough` como dedup key

**Problema**: Mux **não aceita `Idempotency-Key` header**. Retries de create podem gerar Assets duplicados.

**Solução**: usar `passthrough` com um `externalId` do domínio e **checar existência antes de criar**.

```ts
// checagem
const existing = await mux.video.assets.list({
  limit: 1,
  // filtro por passthrough não é direto; manter mapa local (idempotency_key → asset_id)
});

// ou manter tabela local: (dedup_key, asset_id) UNIQUE
// antes de POST, SELECT; se existe, reusa
```

Padrão usual: tabela `mux_idempotency` com `(dedup_key PK, asset_id, created_at)`. `dedup_key` = hash determinístico do comando de domínio (ex.: `sha256(tenant|entityId|version)`).

---

## 7. Rate limit + retry com backoff

Mux publica limites no response header:

- `X-RateLimit-Limit`
- `X-RateLimit-Remaining`
- `X-RateLimit-Reset`

Em `429 Too Many Requests`, respeitar `Retry-After` ou fallback para **exponential backoff com jitter**: `delay = min(cap, base * 2^attempt) + random(0, jitter)`.

```go
for attempt := 0; attempt < 5; attempt++ {
    resp, err := client.VideoAPI.CreateAsset(req)
    if err == nil { return resp, nil }
    if !isRetryable(err) { return nil, err }
    sleep := time.Duration(1<<attempt)*time.Second + jitter()
    time.Sleep(sleep)
}
```

**Retryable**: 429, 500, 502, 503, 504, erros de rede. **Não retryable**: 4xx excluindo 429 (input inválido não melhora com retry).

---

## 8. Thumbnail automático

Mux gera thumbnail on-the-fly via `https://image.mux.com/{playback_id}/thumbnail.{ext}?time={sec}`.

Parâmetros úteis:
- `time=30` — timestamp (padrão: frame médio do vídeo)
- `width=640&height=360` — redimensiona server-side
- `fit_mode=preserve | crop | smartcrop`
- `token=<JWT com aud=t>` — obrigatório se playback for signed

Uso recomendado: gerar URL no backend com JWT `aud=t` e TTL curto; passar para `<img>` ou pré-carregar no card de listagem.

Para animated preview: `animated.gif` ou `animated.webp` com `start` + `end`.

---

## 9. Storyboards (sprite sheet para hover preview)

`https://image.mux.com/{playback_id}/storyboard.vtt` devolve WebVTT + URLs de sprite sheet. Mux Player consome automaticamente quando `storyboard-token` é passado (signed playback) — exibe preview ao passar mouse sobre a timeline.

---

## 10. Environment separation

Duas chaves: uma por environment (`Development`, `Production`). **Nunca reutilizar**. Fluxo recomendado:

1. Dev roda contra `Development` (quota grátis, assets com watermark)
2. Staging roda contra `Production` em um environment Mux separado (`Staging`) — criar um environment extra no dashboard
3. Prod roda contra `Production` principal

Assets e webhooks são isolados por environment — não há risco de evento de dev vazar para prod handler.

---

## Links

- [[_moc]]
- [[../_moc]]
- [[overview]]
- [[integration-guide]]
- [[antipatterns]]
- [[../../security/_moc]] — JWT HMAC, verificação de webhook
- [[../livekit/_moc]] — vizinho (realtime)
