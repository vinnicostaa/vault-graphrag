---
title: 'Livekit — Integration Guide (setup, token, client)'
type: note
tags:
  - provider
  - livekit
  - webrtc
  - realtime
  - integration
  - guide
source: 'https://docs.livekit.io'
source_date: '2026-04-24'
created: '2026-04-24'
updated: '2026-04-24'
---

# Livekit — Integration Guide

Passo-a-passo para levantar Livekit do zero (Cloud ou self-host) e conectar um client. Para conceitos, ver [[overview]]; para boas práticas operacionais, ver [[patterns]].

> Fonte: https://docs.livekit.io — consultada 2026-04-24.

---

## 1. Escolher deployment

### Opção A — Livekit Cloud (recomendado para começar)

1. Criar conta em `livekit.io` e criar um projeto
2. Dashboard → Settings → Keys → **Generate API Key**
3. Anotar `API Key`, `API Secret`, `URL` (formato `wss://<projeto>.livekit.cloud`)

### Opção B — Self-host via Docker

Stack mínima: SFU + Redis (para multi-node). Docker Compose essencial:

```yaml
services:
  livekit:
    image: livekit/livekit-server:latest
    command: --config /etc/livekit.yaml
    network_mode: host  # crítico — WebRTC precisa de host networking
    volumes:
      - ./livekit.yaml:/etc/livekit.yaml
  redis:
    image: redis:7-alpine
    ports: ["6379:6379"]
```

`livekit.yaml` mínimo:

```yaml
port: 7880
rtc:
  tcp_port: 7881
  port_range_start: 50000
  port_range_end: 60000
  use_external_ip: true
redis:
  address: localhost:6379
keys:
  devkey: <secret-de-32-bytes-em-base64>
turn:
  enabled: true
  domain: turn.example.com
  tls_port: 5349
  cert_file: /path/to/cert.pem
  key_file: /path/to/key.pem
```

**Gotchas de self-host**:
- UDP range 50000-60000 precisa estar aberto no firewall/security group
- `use_external_ip: true` se rodando em cloud (EC2/GCP) — descobre IP público via metadata
- TURN precisa de certificado TLS válido (self-signed falha no client)
- Redis é obrigatório para scale horizontal; opcional para single-node

### Gerar API keys via CLI (self-host)

```bash
livekit-server generate-keys
# output: API Key: APIxxx / API Secret: <base64>
```

---

## 2. Variáveis de ambiente

Backend sempre consome:

| Var | Significado | Exemplo |
|---|---|---|
| `LIVEKIT_URL` | WebSocket URL do SFU | `wss://proj.livekit.cloud` ou `wss://sfu.example.com` |
| `LIVEKIT_API_KEY` | Key público (idenfica projeto) | `APIxxx` |
| `LIVEKIT_API_SECRET` | Secret para assinar tokens | (guardar em secret manager; **nunca** mandar pro client) |

**Regra de segurança**: `LIVEKIT_API_SECRET` vive exclusivamente no backend. Frontend só recebe o **access token** já assinado (JWT).

---

## 3. Server SDK — gerar access token (Go)

Dependência:

```bash
go get github.com/livekit/server-sdk-go/v2
```

Função idiomática (domínio puro + injeção de clock e keys):

```go
package realtime

import (
    "time"

    lksdk "github.com/livekit/server-sdk-go/v2"
    "github.com/livekit/protocol/auth"
)

type TokenRequest struct {
    RoomName       string
    ParticipantID  string  // identity estável (ex: user_id do domínio)
    DisplayName    string
    CanPublish     bool
    CanSubscribe   bool
    CanUpdateMeta  bool
    TTL            time.Duration
}

func (s *Service) MintToken(req TokenRequest) (string, error) {
    at := auth.NewAccessToken(s.apiKey, s.apiSecret)
    grant := &auth.VideoGrant{
        RoomJoin:             true,
        Room:                 req.RoomName,
        CanPublish:           &req.CanPublish,
        CanSubscribe:         &req.CanSubscribe,
        CanUpdateOwnMetadata: &req.CanUpdateMeta,
    }
    at.AddGrant(grant).
        SetIdentity(req.ParticipantID).
        SetName(req.DisplayName).
        SetValidFor(req.TTL)

    return at.ToJWT()
}
```

**TTL recomendado**: 15min (ver [[patterns]]). O SFU reemite tokens para clients já conectados — TTL curto só limita janela de ataque em caso de vazamento.

---

## 4. Server SDK — orchestração de Room

Duas estratégias de criação:

### 4.a. Auto-create on join (simples)

Não faz nada no backend. Cliente conecta com token válido → Livekit cria room transparentemente. Bom para salas efêmeras/ad-hoc.

### 4.b. Pré-criar com config (controle)

```go
import lksdk "github.com/livekit/server-sdk-go/v2"

rs := lksdk.NewRoomServiceClient(url, apiKey, apiSecret)

_, err := rs.CreateRoom(ctx, &livekit.CreateRoomRequest{
    Name:            "assembly-2026-Q2",
    EmptyTimeout:    300,      // 5min sem ninguém → auto-delete
    MaxParticipants: 500,
    Metadata:        `{"kind":"assembly","topic":"..."}`,
})
```

Usar pré-create quando:
- `max_participants` precisa ser validado antes do primeiro join
- `metadata` da room precisa estar presente desde o join inicial
- Webhook `room_started` precisa disparar em tempo determinístico (ex: agendamento)

---

## 5. Client SDK — conectar, publicar, subscrever (Web)

```bash
npm i livekit-client
```

```typescript
import { Room, RoomEvent, Track } from 'livekit-client';

const room = new Room({
  adaptiveStream: true,
  dynacast: true,
});

room
  .on(RoomEvent.ParticipantConnected, (p) => console.log('joined:', p.identity))
  .on(RoomEvent.TrackSubscribed, (track, _pub, participant) => {
    if (track.kind === Track.Kind.Video) {
      const el = track.attach();
      document.querySelector(`#peer-${participant.identity}`)?.appendChild(el);
    }
  })
  .on(RoomEvent.Disconnected, () => console.log('bye'));

await room.connect(LIVEKIT_URL, accessToken);
await room.localParticipant.enableCameraAndMicrophone();
```

### Flutter

```bash
flutter pub add livekit_client
```

```dart
final room = Room();
await room.connect(livekitUrl, token);
await room.localParticipant!.setCameraEnabled(true);
await room.localParticipant!.setMicrophoneEnabled(true);
```

**Connect fallback**: Livekit tenta em ordem UDP → TURN-UDP → TCP → TURN-TLS. Isso é transparente ao client — aplicação só vê `RoomEvent.Connected` ou falha.

---

## 6. Egress — configurar target S3

Disparar recording (server-side, Go):

```go
eg := lksdk.NewEgressClient(url, apiKey, apiSecret)

_, err := eg.StartRoomCompositeEgress(ctx, &livekit.RoomCompositeEgressRequest{
    RoomName: "assembly-2026-Q2",
    Layout:   "grid",
    FileOutputs: []*livekit.EncodedFileOutput{{
        FileType: livekit.EncodedFileType_MP4,
        Filepath: "recordings/{room_name}-{time}.mp4",
        Output: &livekit.EncodedFileOutput_S3{
            S3: &livekit.S3Upload{
                AccessKey: s3Key,
                Secret:    s3Secret,
                Region:    "us-east-1",
                Bucket:    "recordings",
            },
        },
    }},
})
```

**Padrões importantes**:
- Egress é **assíncrono** — aguardar webhook `egress_ended` para confirmar persistência (ver [[patterns]])
- `{room_name}` e `{time}` são placeholders suportados no path
- Preferir bucket com object lock (imutabilidade) para registros que precisam de integridade

---

## 7. Webhooks — configurar endpoint

Dashboard (Cloud) → Settings → Webhooks → URL HTTPS + eventos.

Self-host: em `livekit.yaml`:

```yaml
webhook:
  urls:
    - https://api.example.com/hooks/livekit
  api_key: devkey  # key usado pra assinar JWT do webhook
```

Receiver em Go:

```go
import "github.com/livekit/protocol/webhook"

authProvider := auth.NewSimpleKeyProvider(apiKey, apiSecret)
event, err := webhook.ReceiveWebhookEvent(r.Body, r.Header, authProvider)
if err != nil { /* 401 */ }

switch event.Event {
case "room_started":        // ...
case "participant_joined":  // ...
case "egress_ended":        // ...
}
```

Eventos suportados (lista completa): `room_started`, `room_finished`, `participant_joined`, `participant_left`, `participant_connection_aborted`, `track_published`, `track_unpublished`, `egress_started`, `egress_updated`, `egress_ended`, `ingress_started`, `ingress_ended`.

---

## 8. Checklist de produção

- [ ] `LIVEKIT_API_SECRET` em secret manager — nunca em repo/env público
- [ ] Token TTL ≤ 15min; client faz refresh via backend antes de expirar
- [ ] `empty_timeout` configurado (evita rooms órfãs consumindo recursos)
- [ ] Webhook endpoint valida JWT signature antes de processar payload
- [ ] Egress com retry + DLQ para casos de falha no S3
- [ ] Monitoring de webhook `participant_connection_aborted` (sinal de problema de rede)
- [ ] TURN configurado se clients podem estar atrás de firewall corporativo
- [ ] Redis em HA (cluster ou Sentinel) se self-host multi-node

---

## Vizinhos no grafo

- [[_moc]]
- [[overview]] — arquitetura SFU e conceitos
- [[patterns]] — boas práticas
- [[antipatterns]]
- [[../_moc]]
- [[../mux/_moc]] — concorrente em broadcast
