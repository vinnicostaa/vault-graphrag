---
title: Livekit — Patterns (boas práticas operacionais)
type: note
tags:
  - provider
  - livekit
  - webrtc
  - realtime
  - patterns
source: 'https://docs.livekit.io'
source_date: '2026-04-24'
created: '2026-04-24'
updated: '2026-04-24'
---

# Livekit — Patterns

Padrões consolidados para operar Livekit em produção. Cada padrão resolve uma classe de problema recorrente em realtime SFU. Para o que **não** fazer, ver [[antipatterns]]; para setup inicial, ver [[integration-guide]].

> Fonte: https://docs.livekit.io — consultada 2026-04-24.

---

## Token TTL curto + refresh server-driven

**Problema**: tokens JWT são bearer credentials. Se vazam (extensão maliciosa, log acidental, MITM em domínio sem HSTS), qualquer um pode entrar na Room até expirar.

**Padrão**:

- TTL default de **15 minutos** para tokens emitidos a clients
- Backend expõe endpoint `POST /realtime/token/refresh` que valida sessão HTTP (cookie/JWT do app) e devolve novo token Livekit
- Client SDK detecta `tokenExpired` ou renova proativamente a cada ~10min
- Livekit Cloud **reemite internamente** tokens para clients já conectados (sem reconnect), mas **novos** joins exigem token válido

Em self-host sem Cloud, o mecanismo de reemissão não existe — TTL curto é a única defesa. Ver [[antipatterns]].

---

## Permissions granulares no JWT

**Problema**: client-side permission check é inútil — atacante reescreve o JS. Precisa ser **server-enforced**.

**Padrão**: embutir exatamente os direitos necessários no VideoGrant:

| Papel típico | `CanPublish` | `CanSubscribe` | `CanUpdateOwnMetadata` | `RoomAdmin` |
|---|---|---|---|---|
| Speaker/host | true | true | true | false |
| Audience/viewer | false | true | false | false |
| Moderator | true | true | true | true |
| Bot recorder | false | true | false | false |

Princípio: **least privilege por default**. Um viewer jamais deveria receber `CanPublish=true` mesmo que "o frontend nunca use". Se o frontend nunca usa, o backend também não precisa emitir.

Para mudança de papel mid-session, usar `RoomServiceClient.UpdateParticipant` em vez de re-emitir token + reconnect.

---

## Room cleanup automático via `empty_timeout`

**Problema**: rooms órfãs (último participant caiu, ninguém voltou) consomem recursos do SFU e aparecem em `ListRooms` para sempre.

**Padrão**:

```go
rs.CreateRoom(ctx, &livekit.CreateRoomRequest{
    Name:         "assembly-2026-Q2",
    EmptyTimeout: 300,  // 5 minutos
})
```

- `empty_timeout` em segundos: após último participant sair, Livekit aguarda esse intervalo; se ninguém entrar de volta, Room é deletada
- **Não usar 0** (desliga cleanup → vazamento garantido)
- Calibrar por caso de uso: sala de reunião pontual = 60s; sala de evento com intervalo = 600s

Backend deve **ouvir** `room_finished` webhook para persistir "sala encerrada" no domínio — não depender de polling em `ListRooms`.

---

## Egress assíncrono com webhooks como fonte da verdade

**Problema**: `StartEgress` retorna imediatamente com um `EgressInfo` em estado `STARTING`. O arquivo **ainda não existe** no S3. Backend que trata a resposta como "gravação pronta" gera bugs sutis.

**Padrão** (state machine guiada por webhook):

```
client chama StartRoomCompositeEgress
        │
        ▼
backend salva Egress{id, status=STARTING}
        │
        ├── webhook "egress_started"  → status=ACTIVE
        ├── webhook "egress_updated"  → (opcional: progresso)
        └── webhook "egress_ended"    → status=COMPLETE + fileUrl
                                        ou status=FAILED + error
```

Somente em `egress_ended` com `status=COMPLETE` o backend:
1. Persiste URL final do arquivo no agregado de domínio
2. Dispara evento de domínio `RecordingAvailable`
3. (Opcional) copia para bucket com object lock para imutabilidade

Se `egress_ended` não chegar em X minutos, alarme e reconciliação via `ListEgress`.

---

## Graceful disconnect

**Problema**: fechar a aba do browser dispara WebSocket close, mas às vezes antes do Livekit limpar o participant — causa flicker e métricas ruidosas.

**Padrão**:

- **Web**: chamar `room.disconnect()` em `beforeunload` (já é default em alguns SDKs, mas explicitar garante)
- **Mobile**: chamar no `onDestroy`/`dispose` do container
- **Flutter**: já chama automaticamente no dispose se o Room foi criado com `autoSubscribe=true`
- Backend nunca deveria precisar forçar desconexão de client "fantasma"; se precisa, usar `RoomServiceClient.RemoveParticipant`

---

## Participant identity estável e imutável

**Problema**: identity é a **chave** pela qual outros participants referenciam esse usuário. Se muda, o grafo de menção/moderation quebra.

**Padrão**:

- Identity = **user_id do domínio** (UUID ou sub do JWT de auth principal)
- Nunca usar email (muda), username (muda), session_id (muda cada login)
- Display name separado em `SetName()` — esse pode mudar livremente

```go
at.SetIdentity(user.ID.String()).
   SetName(user.DisplayName)  // este muda; o acima nunca
```

Consequência: se um usuário abre 2 abas com a mesma identity, **a segunda kick a primeira** (default Livekit). Se quiser multi-device real, usar `identity = f"{user_id}:{device_id}"`.

---

## Reconnect com exponential backoff + resync

**Problema**: client SDK tenta reconectar automaticamente, mas em cenários extremos (laptop sleep 30min) o token expirou e o reconnect falha em loop.

**Padrão**:

1. Escutar `RoomEvent.Reconnecting` — mostrar UI de "reconectando"
2. Se 3+ falhas consecutivas, backoff exponencial (1s → 2s → 4s → 8s, cap 30s)
3. Antes de cada retry pós-backoff, **buscar token fresh** no backend
4. Após `RoomEvent.Reconnected`, resyncar estado da UI (participants, mute states, metadata) — o SDK já restaura internamente, mas listeners da app podem ter stale state

Cliente nunca deveria armazenar token em localStorage para reusar — sempre pegar novo do backend, que re-valida a sessão do usuário.

---

## Monitoring via webhook events

**Problema**: entender saúde do sistema sem logar dentro do SFU.

**Padrão**: espelhar eventos Livekit em pipeline de observability:

| Evento | Métrica derivada | Alerta se... |
|---|---|---|
| `room_started` | `livekit_rooms_started_total` | desvio >3σ do baseline |
| `room_finished` | `livekit_rooms_duration_seconds` (histogram) | p99 < 30s (rooms morrendo cedo) |
| `participant_joined` | `livekit_participants_joined_total` | drop súbito durante evento agendado |
| `participant_connection_aborted` | `livekit_connection_aborted_total` | taxa > 5% dos joins |
| `egress_ended{status=FAILED}` | `livekit_egress_failures_total` | qualquer incremento fora de janela de manutenção |

Endpoint webhook deve ser idempotente (mesmo event_id pode chegar 2x por retry). Usar `egress_id` + `event_type` + timestamp como chave de idempotência.

---

## Room naming determinístico

**Problema**: Room names vindos de input livre causam colisão, DoS e confusão de search.

**Padrão**: nome determinístico derivado do agregado de domínio:

```
room_name = f"{bounded_context}:{aggregate_id}:{occurrence_id}"
# exemplos:
# "meeting:f3a1-...:2026-Q2"
# "lesson:7b2c-...:session-4"
```

- Prefixo por bounded context facilita routing em webhook handlers
- Aggregate ID garante unicidade
- Occurrence/session ID permite múltiplas instâncias do mesmo agregado

Validação no backend antes de criar: regex `^[a-z]+:[a-z0-9-]+(:[a-z0-9-]+)?$`.

---

## Vizinhos no grafo

- [[_moc]]
- [[overview]]
- [[integration-guide]]
- [[antipatterns]]
- [[../_moc]]
- [[../mux/_moc]]
