---
title: Offline Strategy — Mobile Flutter
type: spec
tags: [master-sindico, mobile, flutter, offline, cache, sync]
project: master-sindico
stack: mobile
status: specification
created: 2026-04-24
---

# Offline Strategy — Mobile

Estratégia de cacheamento, expiração e conflito para operações offline.

## Princípios

1. **M1 = read-only offline**. Escrita sempre online.
2. **M2 = sync queue seletivo** (Connect Me send, RSVP, ciência — podem aguardar).
3. **M3 = offline pleno não planejado**; votação assembleia/pagamento **nunca** offline (LGPD + auditoria exigem real-time).
4. **Fonte da verdade = servidor**. Cache local é otimização; conflito = servidor vence.

## Armazenamento — matriz

| Namespace | Mecanismo | TTL | Cap | Sensível? | Encrypted? |
|---|---|---|---|---|---|
| `oidc_*` tokens | flutter_secure_storage | session | — | ✅ | Keychain/EncryptedSP |
| `device_fingerprint` | flutter_secure_storage | permanent | — | ✅ | sim |
| `user_session` | Hive (encrypted, key em secure_storage) | session | — | ✅ | sim |
| `memberships` | Hive | 24h | — | parcial | opcional |
| `dashboard_resident` | Hive | 5min | — | não | não |
| `syndic_dashboard_{id}` | Hive | 10min | — | parcial | opcional |
| `timeline_{id}` | Hive | 1h | 100 | não | não |
| `timeline_detail_{id}` | Hive | 7d | — | não | não |
| `announcements_{id}` | Hive | 30min | 50 | não | não |
| `announcement_detail_{id}` | Hive | 7d | — | não | não |
| `events_{id}` | Hive | 15min | — | não | não |
| `connect_me_sent` | Hive | 30min | 100 | **PII** | ✅ |
| `connect_me_quota` | Hive | 5min | — | não | não |
| `vizinhanca_feed_{id}` | Hive | 10min | 50 | não | não |
| `vizinhanca_coupons_active` | Hive | até expirar | — | parcial | opcional |
| `onboarding_draft` | Hive | indef (limpa ao completar) | — | **PII** | ✅ |
| `app_settings` | Hive | permanent | — | não | não |
| `sync_queue` (M2+) | Hive | até flush | — | parcial | ✅ |

## Política de uso

- **Encrypted** obrigatório em: tokens, session, onboarding_draft, sync_queue, connect_me_sent.
- Hive boxes encriptados usam `HiveCipher` com `encryptionKey` persistida em `flutter_secure_storage` (chave gerada no primeiro launch).
- `hydrated_bloc` usa storage Hive dedicado (`hydrated_bloc/` path) e deve ser inicializado **antes** de qualquer Bloc.

## TTL expiration policy

- TTL expirado → cache entra em estado **stale-but-usable**: renderiza + banner "Você está offline" + fetch em background.
- Se fetch sucedir → atualiza Hive + emite state novo.
- Se fetch falhar → mantém stale + log Sentry.

## Sync Queue (M2+)

Operações enfileirabis:
- `connect_me.submit`
- `events.rsvp`
- `events.acknowledge`
- `announcements.mark_read`

**NÃO** enfileirabis (sempre online):
- `auth.*` (login, refresh, device replace)
- `assembly.*` (check-in, voto)
- `billing.*` (upgrade plano, pagamento)
- `onboarding.complete` (finalize binding)

### Estrutura do item

```dart
@freezed
class SyncQueueItem with _$SyncQueueItem {
  factory SyncQueueItem({
    required String id,          // uuid v4 local
    required String endpoint,    // '/api/v1/content/connect-me'
    required String method,      // 'POST'
    required Map<String, dynamic> payload,
    required DateTime enqueuedAt,
    required int retryCount,
    String? idempotencyKey,      // gerado local
    DateTime? lastAttemptAt,
    String? lastError,
  }) = _SyncQueueItem;
}
```

### Flush

- Dispara em: `ConnectivityResult.wifi | mobile` + app em foreground.
- Worker `SyncWorker` executa itens em ordem FIFO; droppable em paralelo.
- Retry exponential backoff (1min, 5min, 15min, 1h) → max 5 tentativas → marcar `failed` e notificar usuário.

## Conflict resolution

- **Always server-wins**: se local tiver payload enfileirado mas servidor mudou estado, resposta 409 → descartar item local + notificar usuário.
- Nunca merge automático em M2.

## Invalidação por push

| Push tópico | Ação local |
|---|---|
| `timeline.{id}.new` | marca `timeline_{id}` stale; refetch page 1 no foreground |
| `announcement.{id}.new` | stale + badge unread + se urgente notifica OS |
| `event.{id}.cancelled` | remove do cache events_{id} |
| `assembly.opened` | ignora cache; navega |
| `connect-me.replied.{id}` | stale connect_me_sent |

## Métricas / observability

- Sentry custom metric `offline_cache_hit_rate`.
- Sentry breadcrumb em cada cache read/write/invalidate.
- Log warn se Hive box > 10MB (potencial vazamento).

## Gaps/questões abertas

- **Q-OFF-01** Expiração automática de items em `sync_queue` que ficam > 7d pendentes (usuário sumiu do app)? Assumido discard + audit.
- **Q-OFF-02** Cota de disco total Hive (iOS pode limpar sob pressão) — monitorar via `path_provider`.
- **Q-OFF-03** Upgrade de app com mudança de schema Hive → plano de migração.

## Links

- [[_moc]]
- [[architecture]] (section offline-first seletivo)
- [[requirements/_moc]]
- [[../../STATE#D-048|D-048 hydrated_bloc]]
