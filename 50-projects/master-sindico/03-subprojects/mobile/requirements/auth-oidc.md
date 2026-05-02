---
title: Auth OIDC — Mobile Flutter
type: requirement
tags: [master-sindico, mobile, flutter, identity, oidc, feature-req]
project: master-sindico
stack: mobile
priority: m1
status: pending
bounded_context: identity
created: 2026-04-24
---

# Auth OIDC — Mobile

Login Zitadel via **OIDC + PKCE** usando browser externo (ASWebAuthenticationSession iOS / Custom Tabs Android) — **nunca WebView embed** (padrão AppAuth + RFC 8252). 1-device enforcement via `device_fingerprint_id` (IR-011).

## Escopo M1/M2/M3

- **M1**: login PKCE + refresh + logout + 1-device enforcement + troca de dispositivo (403 NEW_DEVICE → modal "substituir").
- **M2**: biometria para re-auth em sessão existente (local_auth).
- **M3**: bypass Lock 90d com biometria 4-eyes (D-058 admin).

## Telas envolvidas

- `AUTH-M1` — Login CTA (logo + botão "Entrar com Master Síndico")
- `AUTH-M2` — External Browser Sheet (iOS/Android nativo — não customizável)
- `AUTH-M3` — Callback redirect (processa code → token)
- `AUTH-M4` — Device Mismatch 403 (outro device ativo → modal substituir)
- `AUTH-M5` — Session Expired (logout automático + retorno para AUTH-M1)

Relaciona web: `[[../../web/requirements/auth-oidc]]` (equivalente web via redirect normal; aqui o diferencial é o browser externo + deep link).

## Funcionais (FR-MOB-AUTH-N)

- **FR-MOB-AUTH-1** Gerar PKCE `code_verifier` (cryptographically secure 43-128 chars) + `code_challenge = base64url(sha256(verifier))`.
- **FR-MOB-AUTH-2** Abrir `/authorize?...&code_challenge=...&code_challenge_method=S256` via `flutter_appauth` / `oidc` package (nunca `WebView`).
- **FR-MOB-AUTH-3** Callback `com.mastersindico://oidc/callback?code=...&state=...` capturado por `app_links`; validar `state` + `nonce`.
- **FR-MOB-AUTH-4** Trocar `code` por tokens em `POST /token` incluindo `code_verifier`; persistir `access_token` + `refresh_token` em `flutter_secure_storage`.
- **FR-MOB-AUTH-5** Enviar `X-Device-Fingerprint` em todas as requisições; no primeiro login, backend registra fingerprint.
- **FR-MOB-AUTH-6** 403 com `code=NEW_DEVICE` → modal "Detectamos outro dispositivo. Substituir?" (Sim → `POST /api/v1/auth/device/replace` com biometria; Não → logout).
- **FR-MOB-AUTH-7** Refresh automático em interceptor Dio quando `access_token` expira (< 60s TTL restante); se refresh falha → logout + AUTH-M5.
- **FR-MOB-AUTH-8** Logout: `POST /api/v1/auth/logout` + limpar secure storage + limpar hydrated_bloc persistido + navegar AUTH-M1.
- **FR-MOB-AUTH-9** Scopes: `openid profile email offline_access`.

## Não-funcionais (NFR-MOB)

- **NFR-MOB-SEC-1** `access_token` e `refresh_token` **exclusivamente** em `flutter_secure_storage` (Keychain iOS / EncryptedSharedPreferences Android).
- **NFR-MOB-SEC-2** Nunca logar token em Sentry/logger (scrubbing `beforeSend`).
- **NFR-MOB-PERF-1** Login end-to-end (CTA → home) < 6s em rede 4G.
- **NFR-MOB-SEC-3** Certificate pinning ativo em prod (Dio interceptor).

## Dados locais

- **Secure Storage**: `oidc_access_token`, `oidc_refresh_token`, `oidc_id_token`, `oidc_expires_at`, `device_fingerprint_id`.
- **Hive `user_session` (encrypted, cipher key em secure_storage)**: `user_id`, `email`, `roles`, `last_login_at`.
- **Invalidação**: no logout, zerar tudo; em rotação de refresh, substituir atomically.

## Integração backend

- `GET /.well-known/openid-configuration` (Zitadel discovery, cacheado 24h).
- `POST https://auth.mastersindico.com.br/oauth/v2/token` (exchange + refresh).
- `POST /api/v1/auth/session/bind` — bind fingerprint no primeiro login.
- `POST /api/v1/auth/device/replace` — troca device (exige biometria em M2+).
- `POST /api/v1/auth/logout` — revoga refresh + audit event.

## Padrões Flutter aplicáveis

- `AuthBloc` (singleton GetIt; `hydrated_bloc` para sobreviver cold start).
- States freezed: `AuthUnauthenticated / AuthLoggingIn / AuthAuthenticated(user) / AuthRefreshing / AuthError(failure)`.
- `bloc_concurrency.droppable()` no evento `LoginRequested` (evita double-tap em botão).
- `Either<Failure, AuthSession>` no repositório; failures tipadas: `ServerFailure / AuthFailure / NewDeviceFailure / CancelledByUserFailure`.
- Router `go_router` com `redirect` consultando `AuthBloc.state`.

## Gaps/decisões abertas

- **Q-MOB-AUTH-01** Biometria obrigatória para device replace em M1 ou só M2? Assumido M2 (first replace sem biometria pela UX).
- **Q-MOB-AUTH-02** Logout em múltiplas abas/apps (revocation global): depende do backend implementar revoke token → ver `[[../../backend/requirements]]`.
- **Q-MOB-AUTH-03** Timeout de inatividade (30min D-050 security §13) requer `WidgetsBindingObserver` + timer global — tarefa aberta.

## Links

- [[../../../_moc]]
- [[../../../00-product/sub-produtos/01-governanca-institucional]]
- [[../../web/requirements/_moc]]
- [[../security]] §1, §4
- [[../../../STATE#D-050|D-050 Integrity]]
