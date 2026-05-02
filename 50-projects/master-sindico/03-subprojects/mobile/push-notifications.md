---
title: Push Notifications — Mobile Flutter
type: spec
tags: [master-sindico, mobile, flutter, push, fcm, notifications]
project: master-sindico
stack: mobile
status: specification
created: 2026-04-24
---

# Push Notifications — Mobile

Arquitetura de notificações push unificada **FCM** (Android + iOS via APNs proxy). Topics + deep-links + agrupamento + opt-in granular por categoria.

## Registro de token

- FCM token obtido via `FirebaseMessaging.instance.getToken()` logo após login bem-sucedido.
- `POST /api/v1/devices/fcm-token { token, platform, app_version, device_fingerprint_id }`.
- Listener `onTokenRefresh` atualiza server-side automaticamente.
- Logout → `DELETE /api/v1/devices/fcm-token` + FCM `deleteToken()`.

## Topics (server-side subscription)

Atribuídos via backend no login; cliente não precisa `subscribeToTopic` manual:

- `user.{user_id}` — mensagens diretas (ex: device replace approved).
- `timeline.{condominium_id}.new` — nova entrada na timeline.
- `announcement.{condominium_id}.new` — comunicado normal.
- `announcement.{condominium_id}.urgent` — comunicado urgente (canal alta prioridade).
- `event.{event_id}.reminder` — 1h antes (agendamento server).
- `event.{event_id}.cancelled` — cancelamento.
- `connect-me.replied.{cm_id}` — morador recebeu resposta.
- `connect-me.new.{condominium_id}` — síndico recebeu novo Connect Me.
- `assembly.{assembly_id}.opened` — assembleia iniciada.
- `assembly.{assembly_id}.item.opened` — item entrou em votação.
- `vizinhanca.offer.new.{condominium_id}` — nova oferta relevante (opt-in).
- `vizinhanca.coupon.expiring.{user_id}` — cupom expira em 24h.

## Canais (Android notification channels)

| Channel ID | Importance | Som | Vibração | Uso |
|---|---|---|---|---|
| `ms_assembly_live` | HIGH | ✅ | padrão | `assembly.*` |
| `ms_urgent` | HIGH | ✅ | padrão | `announcement.*.urgent` |
| `ms_general` | DEFAULT | ✅ | padrão | timeline, announcement normal, event |
| `ms_connect_me` | DEFAULT | — | leve | `connect-me.*` |
| `ms_vizinhanca` | LOW | — | — | vizinhanca.* (opt-in por padrão off) |
| `ms_silent` | LOW | — | — | assessments background sync |

## iOS interruption levels

- `time-sensitive` para `ms_assembly_live` e `ms_urgent` (permite mesmo em Focus Mode).
- `active` (default) para general + connect-me.
- `passive` para vizinhanca.

## Payload canônico

```json
{
  "notification": {
    "title": "Novo comunicado",
    "body": "Aviso sobre elevador"
  },
  "data": {
    "type": "announcement.new",
    "deeplink": "mastersindico://announcements/abc123",
    "condominium_id": "cond_xxx",
    "entity_id": "abc123",
    "priority": "urgent"
  },
  "android": {
    "notification": {
      "channel_id": "ms_urgent",
      "tag": "announcement_cond_xxx",
      "notification_count": 3
    }
  },
  "apns": {
    "payload": {
      "aps": {
        "interruption-level": "time-sensitive",
        "thread-id": "announcement_cond_xxx",
        "badge": 3
      }
    }
  }
}
```

## Deep-links

Todos os pushes acionáveis levam ao app via **custom scheme + universal links**:

- iOS: universal links `https://app.mastersindico.com.br/*` + fallback scheme `mastersindico://*`.
- Android: app links `https://app.mastersindico.com.br/*` + intent filter scheme `mastersindico`.

Rotas deep-linkable (whitelist):
- `/announcements/:id` · `/timeline/:id` · `/events/:id` · `/connect-me/sent/:id` · `/connect-me/received/:id` · `/assembly/:id` · `/vizinhanca/offers/:id`.

Allowlist validação no handler — rotas desconhecidas → home.

## Agrupamento (stacking)

- **Android**: usar `tag` por `entity_id` para update-in-place; `notification_count` mostra badge.
- **iOS**: `thread-id` por tópico.
- Quando 3+ notifications do mesmo tópico em < 5min → agrupar em summary ("3 novos comunicados em Cond. X").

## Handlers Flutter

```dart
// Foreground
FirebaseMessaging.onMessage.listen((msg) {
  // exibir in-app toast + incrementar badge bloco, NÃO banner OS
});

// Background (app minimizado, tap no push)
FirebaseMessaging.onMessageOpenedApp.listen((msg) {
  _handleDeepLink(msg.data['deeplink']);
});

// Terminated (app killed, tap no push abre app)
final initial = await FirebaseMessaging.instance.getInitialMessage();
if (initial != null) _handleDeepLink(initial.data['deeplink']);

// Background isolate (Android-only, processamento headless)
FirebaseMessaging.onBackgroundMessage(_bgHandler);
```

## Opt-in granular (Perfil > Notificações)

6 toggles acionáveis pelo usuário (default ON exceto urgent):
1. Comunicados gerais
2. Comunicados urgentes (default ON, não desligável se plano morador base)
3. Eventos (lembretes 1h antes)
4. Connect Me (moradores sempre ON; síndicos toggle)
5. Vizinhança (default OFF; opt-in)
6. Assembleia Live (default ON; não desligável quando assembleia ativa)

Persistência via Hive + sync server `PATCH /api/v1/users/me/notification-prefs`.

## PII handling

- Body da notificação **nunca** contém CPF, endereço, valor monetário, texto integral de Connect Me.
- Título genérico ("Nova mensagem do síndico"); detalhe só ao abrir app (autenticação intacta).
- iOS Privacy Label: `Contact Info - not linked to you` falso (é linked); declarar em Privacy Manifest.

## Troubleshooting

- Push não entrega iOS → verificar APNs cert rotation + Firebase console diagnostics.
- Android low-delivery → verificar Doze mode + high-priority FCM flag.
- Dev/staging sempre reportam com prefix `[STAGING]` no title.

## Gaps/questões abertas

- **Q-PUSH-01** Rich push (imagem, ações inline) — M3+; tratar como backlog.
- **Q-PUSH-02** Push silent para refetch (data-only) — usado para `sync.trigger`; iOS limita frequência.
- **Q-PUSH-03** Notification history in-app (inbox) — feature M2.

## Links

- [[_moc]]
- [[requirements/_moc]]
- [[requirements/comunicados]] · [[requirements/eventos]] · [[requirements/connect-me-mobile]] · [[requirements/assembly-vote]]
- [[security]] §8 (PII handling)
