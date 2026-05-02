---
title: Mux — integration guide
type: note
tags: [provider, mux, video, integration, sdk, webhook]
source: https://docs.mux.com
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
---

# Mux — integration guide

Passo-a-passo para subir uma integração Mux do zero: conta, credenciais, SDK, upload, webhook e player. Assume backend Go ou Node e frontend web (Flutter tem analogia direta via `@mux/mux-player-flutter`/community package).

---

## 1. Conta e credenciais

1. Criar conta em https://dashboard.mux.com/
2. Escolher um **Environment** (`Development` ou `Production`). Cada ambiente tem suas próprias chaves e assets — não há cross-environment read.
3. Em `Settings → Access Tokens`, gerar um **Access Token**. Retorna dois valores:
   - `Token ID` (público, equivale a username)
   - `Token Secret` (privado, exibido uma única vez — perdeu, revoga e gera outro)
4. Em `Settings → Signing Keys`, gerar uma **Signing Key** (par RSA/HMAC). Usada para assinar JWTs de playback. Ver [[patterns]] → Signed Playback.
5. Em `Settings → Webhooks`, criar um endpoint. O Mux gera um `Signing Secret` por webhook.

---

## 2. Variáveis de ambiente

Padrão que o SDK Mux espera:

```bash
MUX_TOKEN_ID=...              # Token ID (public-ish)
MUX_TOKEN_SECRET=...          # Token Secret (privado)
MUX_WEBHOOK_SECRET=...        # Signing secret do webhook
MUX_SIGNING_KEY_ID=...        # ID da Signing Key (entra no JWT header como 'kid')
MUX_SIGNING_KEY_PRIVATE=...   # Private key PEM (base64 ou raw) — NUNCA em logs
```

Guardar `MUX_TOKEN_SECRET` e `MUX_SIGNING_KEY_PRIVATE` em secret manager (Vault, AWS Secrets, Railway env encrypted). Nunca em `.env` commitado.

---

## 3. Instalação do SDK

**Go**:

```bash
go get github.com/muxinc/mux-go/v6
```

```go
import muxgo "github.com/muxinc/mux-go/v6"

client := muxgo.NewAPIClient(
    muxgo.NewConfiguration(
        muxgo.WithBasicAuth(os.Getenv("MUX_TOKEN_ID"), os.Getenv("MUX_TOKEN_SECRET")),
    ),
)
```

**Node/TypeScript**:

```bash
npm install @mux/mux-node
```

```ts
import Mux from '@mux/mux-node';
const mux = new Mux(); // lê MUX_TOKEN_ID e MUX_TOKEN_SECRET do env
```

Ambos SDKs expõem namespaces: `Video` (assets/uploads/live), `Data` (analytics), `System` (status).

---

## 4. Upload de VOD — dois caminhos

### 4.1. Direct Upload (recomendado)

Backend gera uma URL pré-assinada; o cliente (browser/mobile) faz o PUT direto para o Mux sem passar pelo servidor da aplicação. Preferível por dois motivos: reduz tráfego backend e não bloqueia worker durante transferência.

```ts
// backend (Node)
const upload = await mux.video.uploads.create({
  new_asset_settings: {
    playback_policy: ['signed'],
    passthrough: `tenant:${tenantId};video:${domainVideoId}`,
  },
  cors_origin: 'https://app.example.com',
});
// retorna upload.url (usar no cliente) e upload.id (usar no webhook)
return { uploadUrl: upload.url, uploadId: upload.id };
```

No frontend, `fetch(uploadUrl, { method: 'PUT', body: file })`.

### 4.2. Server-proxied (evitar)

Arquivo sobe para o backend, backend re-envia ao Mux. Dobra banda e segura worker. Só faz sentido se precisa inspecionar/transcodar antes (caso raríssimo — Mux já transcoda).

---

## 5. Webhook — lifecycle e verificação

Eventos principais:

| Evento | Quando | Ação recomendada |
|---|---|---|
| `video.upload.asset_created` | Upload terminou, Asset criado | Ligar `upload_id → asset_id` no DB |
| `video.asset.created` | Asset existe (status `preparing`) | Persistir como "em processamento" |
| `video.asset.ready` | Encoding terminou, playback disponível | Marcar `ready=true`, disparar downstream |
| `video.asset.errored` | Encoding falhou | Marcar `errored`, expor erro ao usuário |
| `video.asset.deleted` | Asset deletado (API ou TTL) | Remover do DB |
| `video.live_stream.active` | Live começou a receber RTMP | Notificar viewers, iniciar gravação downstream |
| `video.live_stream.idle` | Live encerrou | Marcar fim, esperar `asset.ready` do replay |

### Verificação de assinatura (obrigatória)

Todo webhook traz header `Mux-Signature: t=<ts>,v1=<hmac>`. O HMAC é SHA256 sobre `<ts>.<raw_body>` com a chave `MUX_WEBHOOK_SECRET`. Rejeitar se:

- `|now - ts| > 300s` (replay attack)
- HMAC recomputado ≠ `v1`

Em Node, o SDK tem `mux.webhooks.verifySignature(rawBody, headers, secret)`.

```ts
app.post('/webhooks/mux', express.raw({type:'application/json'}), (req, res) => {
  try {
    mux.webhooks.verifySignature(req.body, req.headers, process.env.MUX_WEBHOOK_SECRET!);
  } catch {
    return res.status(400).send('bad signature');
  }
  const event = JSON.parse(req.body.toString());
  enqueue(event); // processar async, responder 200 rápido
  res.sendStatus(200);
});
```

**Idempotência do handler**: Mux pode re-enviar o mesmo evento. Usar `event.id` como chave de deduplicação no DB (tabela `processed_events`).

---

## 6. Live Stream — ingest RTMP

```ts
const live = await mux.video.liveStreams.create({
  playback_policy: ['signed'],
  new_asset_settings: { playback_policy: ['signed'] }, // replay também signed
  reconnect_window: 60,                                // segundos pra re-ingestar sem encerrar
  low_latency: true,                                   // latência ~3-5s
});
// live.stream_key + rtmp://global-live.mux.com:5222/app → usar no OBS/cliente
// live.playback_ids[0].id → montar URL HLS de viewer
```

No cliente broadcaster (ex.: Flutter com `flutter_rtmp_publisher`): apontar para `rtmp://global-live.mux.com/app/{stream_key}`.

Durante a live, Mux emite `video.live_stream.active`. Ao encerrar, emite `idle` e, depois de alguns minutos, `video.asset.ready` do replay.

---

## 7. Mux Player — embed web

Web Component:

```html
<script src="https://cdn.jsdelivr.net/npm/@mux/mux-player"></script>

<mux-player
  playback-id="abc123..."
  playback-token="<JWT gerado no backend>"
  stream-type="on-demand"
  metadata-video-title="Assembleia 2026-04-24"
  metadata-viewer-user-id="user-42">
</mux-player>
```

Live: trocar `stream-type` para `"live"` ou `"ll-live"` (low-latency).

React:

```tsx
import MuxPlayer from '@mux/mux-player-react';

<MuxPlayer
  playbackId={playbackId}
  tokens={{ playback: playbackToken, thumbnail: thumbToken }}
  streamType="on-demand"
/>;
```

Flutter: usar `mux_video_player` (community) ou montar `video_player` + URL HLS + header `Authorization` se signed.

---

## 8. Signed playback — geração de JWT

Algoritmo: HS256 ou RS256 (depende da Signing Key gerada). Payload mínimo:

```json
{
  "sub": "<playback_id>",
  "aud": "v",                // "v" = video, "t" = thumbnail, "s" = storyboard, "g" = gif
  "exp": 1745520000,         // unix ts; TTL curto (ex: 10-60 min)
  "kid": "<signing_key_id>"  // header, não payload
}
```

Ver [[patterns]] → Signed playback URLs para exemplo em Go/Node e estratégias de TTL.

---

## 9. Checklist de onboarding

- [ ] Conta criada, environment `Production` separado do `Development`
- [ ] `MUX_TOKEN_ID` / `MUX_TOKEN_SECRET` no secret manager
- [ ] Signing Key gerada; `MUX_SIGNING_KEY_*` no secret manager
- [ ] Webhook configurado com URL pública + `MUX_WEBHOOK_SECRET` guardado
- [ ] SDK instalado na versão mais recente (verificar `go.mod` / `package.json`)
- [ ] Handler de webhook com verificação HMAC + dedup por `event.id`
- [ ] Tabela local com `asset_id` como chave, `playback_ids` como lista filha
- [ ] Fluxo de upload testado em `Development` antes de promover

---

## Links

- [[_moc]]
- [[../_moc]]
- [[overview]]
- [[patterns]]
- [[antipatterns]]
- [[../livekit/_moc]] — comparação com realtime WebRTC
