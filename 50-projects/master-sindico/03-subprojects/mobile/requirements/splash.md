---
title: Splash — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, identity, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: identity
created: 2026-04-24
---

# Splash — Mobile

Tela de lançamento responsável por **pre-check de integridade** (freeRASP — D-050), decisão de rota inicial (login vs home) e boot do grafo DI. Não é apenas branding: é o primeiro gate de segurança.

## Escopo M1/M2/M3

- **M1**: splash + pre-check freeRASP report-only + decisão de rota (auth vs home) + branding Manual IV.
- **M2**: adicionar indicador de "verificando dispositivo" quando pre-check > 1s.
- **M3**: gate duro — se `integrity=hooked` bloqueia app inteira em M3 (ver D-050 roll-out).

## Telas envolvidas

- `SPL-M1` — Splash (logotipo + progress oculto)
- `SPL-M2` — Integrity Blocked (modal full-screen fallback quando `hooked` em M3)
- `SPL-M3` — App Outdated (se versão < mínima server-side)

Relacionadas web (sem equivalente direto; web não tem splash propriamente).

## Funcionais (FR-MOB-SPL-N)

- **FR-MOB-SPL-1** Ao iniciar, exibir splash nativo (iOS LaunchScreen.storyboard, Android splash_background.xml) + splash Flutter com logo Manual IV.
- **FR-MOB-SPL-2** Executar em paralelo: (a) boot GetIt/Injectable, (b) leitura `flutter_secure_storage` para sessão, (c) chamada freeRASP `Talsec.start()`.
- **FR-MOB-SPL-3** Se freeRASP reportar `integrity=hooked` → emitir audit event local + em M3 navegar para `SPL-M2` com CTA "Saiba mais" e botão fechar app.
- **FR-MOB-SPL-4** Se freeRASP reportar `integrity=jailbroken` ou `dev-mode` em M1 → apenas log + header `X-Device-Integrity` nas próximas requisições.
- **FR-MOB-SPL-5** Se sessão Zitadel válida → navegar para `/home` (shell bottom nav); senão → `/auth/login`.
- **FR-MOB-SPL-6** Se backend responder com `force_update_required=true` no healthcheck → navegar para `SPL-M3` com link para Store.
- **FR-MOB-SPL-7** Tempo máximo da splash = 3s iOS / 2.5s Android (NFR-MOB-PERF-1); após isso, navegar mesmo se freeRASP ainda não respondeu (timeout → report-only assumido).

## Não-funcionais (NFR-MOB)

- **NFR-MOB-PERF-1** Cold start < 3s iOS; < 2.5s Android (device de referência iPhone 11 / Pixel 5).
- **NFR-MOB-SEC-1** Sem PII em log; splash não acessa rede salvo healthcheck.
- **NFR-MOB-A11Y-1** Texto de erro em SPL-M2/M3 com `Semantics(label:...)`.

## Dados locais

- **Secure Storage**: `oidc_access_token`, `oidc_refresh_token`, `device_fingerprint_id` (lidos durante splash).
- **Hive box `app_settings`**: `last_integrity_status`, `last_seen_version`, `force_update_cached_until`.
- **SharedPreferences**: apenas flag `has_seen_onboarding_v1` (não-PII).
- **Invalidação**: token expirado → refresh automático no splash antes de navegar.

## Integração backend

- `GET /api/v1/healthz` (sem auth) — retorna `{ ok, min_app_version }` em < 300ms; se falhar, assume cache e navega.
- `GET /api/v1/auth/session` (com Bearer refresh) — valida sessão; 401 → login.

Nenhum WebSocket nesta feature.

## Padrões Flutter aplicáveis

- `BlocProvider<SplashBloc>` com states `freezed`: `SplashInitial / SplashBooting / SplashReady(route) / SplashBlocked(reason) / SplashOutdated`.
- `bloc_concurrency.droppable()` NÃO necessário (tela non-interactive).
- `go_router` redirect root-level consultando `SplashBloc.state`.
- `hydrated_bloc` para persistir `last_integrity_status` entre cold starts.
- `freezed` (D-049) obrigatório em events/states.

## Gaps/decisões abertas

- **Q-MOB-SPL-01** M3 gated: qual UX de bloqueio quando `hooked` fora da tela de votação? Assumido full-block; validar com compliance.
- **Q-MOB-SPL-02** Tempo de tolerância do timeout freeRASP ainda não benchmarked em devices low-end (pre-cal 3s).

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[../../../STATE#D-050|D-050 Integrity escalonada MASVS-R]]
- [[../security]] §7
- [[../../web/requirements/_moc]] (sem equivalente direto)
