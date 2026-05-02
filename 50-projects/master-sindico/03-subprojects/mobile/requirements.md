---
title: Mobile — Requirements específicos
type: requirement
tags: [subproject, mobile, master-sindico, v2, requirements, offline, push, deep-link, biometria]
source:
  - 04-requirements/global.md
  - Clientes/Joao/MasterSindico/Development/app/CLAUDE.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/ARCHITECTURE.md (2026-04-21)
  - STATE.md D-048..D-050 (2026-04-23)
created: 2026-04-23
updated: 2026-04-23
---

# Mobile — Requirements específicos

Requirements que **só o mobile implementa**. Complementa [[../backend/requirements]] e [[../web/requirements]]. Todos os requisitos globais (LGPD, ABAC, 1-device, audit, tenant isolation) **aplicam** — seção mostra as especializações mobile.

---

## 1. Offline behavior (BR-MOB-OFF) — D-041 aberto (Hive alvo)

### BR-MOB-OFF-001 — Offline-first seletivo

Cache local via **Hive** (decisão pendente D-041 — Hive vs sqflite vs drift; Hive é default canônico):

| Box | TTL | Conteúdo | Encryption |
|---|---|---|---|
| `user` | ∞ (até logout) | Snapshot `AuthUser` (id, name, email, avatar) | ✅ AES |
| `memberships` | 1h | Memberships ativas — refresh background | ✅ AES |
| `timeline:{condId}` | 30min | Últimos 30 itens por condomínio | ✅ AES |
| `events:{condId}` | 1h | Eventos agendados próximos 30 dias | ✅ AES |
| `plano-diretor:{condId}` | 1h | Plano diretor leitura | ✅ AES |
| `sync_queue` | até flush | Actions pendentes (ver BR-MOB-OFF-003) | ✅ AES |

Chave de encryption derivada de `flutter_secure_storage` (chave `hive_encryption_key`).

### BR-MOB-OFF-002 — Writes exigem online

Criar evento, votar, enviar Connect Me (primeira vez), registrar ciência crítica de edital, outorgar procuração, upload vídeo-currículo, pagamento — **requerem conexão**. Sem conexão → modal bloqueante com [Aguardar conexão] ou [Cancelar].

### BR-MOB-OFF-003 — Sync queue tolerante

Actions tolerantes (não críticas em tempo real) enfileiradas em `sync_queue` Hive box; worker `SyncBloc` dispara quando `NetworkInfo.isConnected` vira true:

- Registrar ciência simples de evento.
- Confirmar participação em evento.
- Marcar notificação como lida.
- Marcar item plano diretor como lido.

Retry policy: exponencial (1s → 2s → 4s → 8s → 30s cap) com max 5 tentativas; após falha permanente, notificar user e mover para box `sync_queue_failed`.

### BR-MOB-OFF-004 — UX offline

- **Banner topo persistente**: "📡 Sem conexão — mostrando dados em cache".
- **Badges** "atualizado há [X] min" em listas cached.
- **Botão [Tentar novamente]** prominente em tela de erro.
- **Loading skeleton** em placeholder enquanto rede tenta.

### BR-MOB-OFF-005 — Assembleia live = online obrigatório

Sessão live (APP-ASM-006) exige conexão estável. Se perder WS: banner "Reconectando..." com timeout 30s → se não recupera, modal erro com [Tentar novamente] / [Sair].

---

## 2. Push notifications (BR-MOB-PUSH) — FCM M1

### BR-MOB-PUSH-001 — FCM como provider único

iOS + Android via `firebase_messaging`. Token registrado no backend `POST /api/v1/devices/register-push-token` em startup + refresh (listener `onTokenRefresh`).

### BR-MOB-PUSH-002 — Tópicos

- `user:<user_id>` — notificações pessoais (Connect Me resposta, vídeo aprovado, sessão transferida).
- `tenant:<condId>` — broadcast do tenant (subscrever em cada membership ativa; unsubscribe ao sair).
- `tenant:<condId>:sindicos` — só quando role ativa é síndico (broadcast moderação, escalação validação).
- `system:all` — broadcast emergencial admin MS (raro).

### BR-MOB-PUSH-003 — Categorias e channels

| Categoria | Android channel | Importance | iOS category |
|---|---|---|---|
| `events` | `events-default` | DEFAULT | `event.default` |
| `connect_me` | `connect-me-high` | HIGH | `connect_me.interactive` |
| `votes` | `votes-high` | HIGH | `votes.interactive` |
| `assembly_live` | `assembly-urgent` | MAX | `assembly.urgent` |
| `compliance` | `compliance-default` | DEFAULT | `compliance.default` |
| `marketing` | `marketing-low` | LOW | `marketing.low` |

User override toggles em `/account/notificacoes` (APP-ACC-005). Opt-in explícito para `marketing`.

### BR-MOB-PUSH-004 — Payload canônico

```json
{
  "notification": { "title": "...", "body": "..." },
  "data": {
    "type": "vote_opened | event_created | connect_me_interest | assembly_live_starting | ...",
    "resource_id": "01HXXX...",
    "tenant_id": "01HXXX...",
    "category": "events | connect_me | votes | assembly_live | compliance | marketing",
    "deep_link": "com.mastersindico://condominium/.../votacao/..."
  }
}
```

### BR-MOB-PUSH-005 — PII restrito

Push **NUNCA** contém CPF/CNPJ/dados financeiros plain. Só título + preview curto (ex: "Nova proposta disponível", "Assembleia iniciando em 5min"); full content ao abrir app autenticado.

### BR-MOB-PUSH-006 — Taps (foreground / background / terminated)

- **Foreground**: `onMessage` → in-app toast/snackbar + badge; tap → deep-link.
- **Background**: system notification; tap → app retoma na rota `data.deep_link`.
- **Terminated**: FCM entrega para OS; tap → app abre e navega para rota.

---

## 3. Deep linking (BR-MOB-DL)

### BR-MOB-DL-001 — Schemes

- **Universal links iOS** (M1): `https://app.mastersindico.com.br/*` → `apple-app-site-association` configurado.
- **App links Android** (M1): intent-filter `android:autoVerify="true"` em `AndroidManifest.xml` + `https://app.mastersindico.com.br/.well-known/assetlinks.json`.
- **Custom scheme**: `com.mastersindico://` — usado **apenas** para OIDC callback (`/auth`) e casos não-web-compat. Já registrado em `Info.plist` e `AndroidManifest.xml` atuais.

### BR-MOB-DL-002 — Rotas deep-link suportadas M1

- `com.mastersindico://auth` (OIDC callback only)
- `https://app.mastersindico.com.br/condominium/:id`
- `https://app.mastersindico.com.br/condominium/:id/timeline`
- `https://app.mastersindico.com.br/condominium/:id/eventos/:id`
- `https://app.mastersindico.com.br/condominium/:id/votacoes/:id`
- `https://app.mastersindico.com.br/condominium/:id/assembleias/:id` (M3)
- `https://app.mastersindico.com.br/notifications`

### BR-MOB-DL-003 — Validação allowlist

Deep links validados contra allowlist de rotas conhecidas em `app_routes.dart`. Unknown scheme/path → navegar para tela 404 (`errorBuilder` do GoRouter) + log Sentry `deep_link_unknown`.

### BR-MOB-DL-004 — Auth guard

Se deep-link requer auth e user deslogado → redirect `/login?redirect=<original>` preservado; pós-login `GoRouter.redirect` retorna ao destino.

---

## 4. Biometria (BR-MOB-BIO) — `local_auth`

### BR-MOB-BIO-001 — Opt-in

Toggle em `/account/seguranca` (APP-ACC-004) → "Usar biometria na próxima abertura" e "Exigir biometria em ações críticas".

### BR-MOB-BIO-002 — Implementação

`local_auth` package:
- **iOS**: FaceID / Touch ID. String de uso em `Info.plist` `NSFaceIDUsageDescription` (pendente): "Use sua biometria para confirmar ações sensíveis."
- **Android**: BiometricPrompt (fingerprint, face — depende do device).

### BR-MOB-BIO-003 — Usos obrigatórios

- Re-abrir app após inatividade > 30min (opt-in).
- **Ação crítica — sempre**: outorgar procuração (APP-ASM-004), assinar ata, deletar conta, upgrade plano, bypass Lock 90d admin.

### BR-MOB-BIO-004 — NÃO substitui OIDC

Primeiro login **sempre** OIDC com senha (PKCE Zitadel). Biometria é **suplementar** — valida presença do titular sem revalidar credencial.

### BR-MOB-BIO-005 — Fallback

Device sem biometria OU biometric failure 3× → cai no fluxo padrão (OIDC + senha). Feedback: "Biometria indisponível, faça login novamente."

---

## 5. Camera & mídia (BR-MOB-CAM)

### BR-MOB-CAM-001 — Permissões on-demand

Pedir permissão **no momento** do uso (não na abertura do app). Explicação contextual modal antes do prompt nativo. Plataformas:

- iOS: `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSPhotoLibraryUsageDescription` em `Info.plist` (pendentes — ver [[README#2-nativo-plataformas-minimas]]).
- Android: `<uses-permission android:name="android.permission.CAMERA"/>` + runtime permission request (API 23+).

### BR-MOB-CAM-002 — Fluxos que usam camera

- Vídeo-currículo morador (90s max) — `/account/video-curriculo` (APP-VID-002).
- Foto de execução síndico mobile — `/sindico/servicos/:id/execucao/foto` (M2+).
- Foto de fachada condomínio (cadastro opcional) — `/sindico/condominium/new`.
- QR scan check-in assembleia (APP-ASM-005).

### BR-MOB-CAM-003 — Compressão cliente

- **Foto**: max 1920px longer side, JPEG quality 85, target < 1MB.
- **Vídeo**: backend Mux transcoda; cliente envia original via Mux direct upload presigned (máx 200MB).

### BR-MOB-CAM-004 — Revoked permission handling

Se permissão revogada em Settings do device → mostrar tela explicativa + botão [Abrir Configurações] (via `app_settings` ou `permission_handler` package).

---

## 6. Performance (BR-MOB-PERF)

### BR-MOB-PERF-001 — Cold start

- **iOS**: ≤ 2s até splash dismiss (dev build), ≤ 3s (release).
- **Android**: ≤ 2.5s (dev + release).
- Medida: desde `FlutterActivity onCreate` até `runApp` com Bloc inicializado.

### BR-MOB-PERF-002 — Frame rate

- 60fps em listas, transitions, scroll.
- Jank budget < 1%.
- `Performance.instance.addTimingsCallback` em dev; Sentry Performance em release.

### BR-MOB-PERF-003 — Memory footprint

- Target < 200MB em uso normal.
- Alerta automático > 250MB (Sentry).
- Leak detection via DevTools Memory em milestones.

### BR-MOB-PERF-004 — App size

- **iOS IPA**: target < 40MB (sem assets pesados bundled; fonts via system + google_fonts runtime).
- **Android AAB**: Dynamic delivery para features pesadas (LMS videos, se bundled).

### BR-MOB-PERF-005 — Battery

- Sem polling constante — push + lazy sync.
- WebSocket (assembleia live) mantido **apenas** durante sessão live; fechado ao sair da rota.
- Background tasks mínimos — apenas FCM + `hydrated_bloc` re-hydrate em start.

---

## 7. Internacionalização (BR-MOB-I18N)

### BR-MOB-I18N-001 — Pt-BR first

`easy_localization` (pendente M1) com dicionário `assets/translations/pt_BR.json`. EN preparado (chaves presentes) mas **não ativo** em M1-M3.

### BR-MOB-I18N-002 — Termos BR preservados

Idêntico ao [[../web/requirements#br-web-i18n-002|BR-WEB-I18N-002]]: "síndico" (não "trustee"), "condomínio" (não "HOA"), "assembleia" (não "meeting"), "edital", "ata", "procuração", "fração ideal".

### BR-MOB-I18N-003 — Formatação local

- `intl` package (já em pubspec `^0.20.0`) com locale `pt_BR`.
- Data (dd/MM/yyyy HH:mm), moeda (`R$ 1.234,56`), CPF/CNPJ/CEP formatados consistentemente com web.
- Timezone **apresentação** = `America/Sao_Paulo` (D-052); **storage** UTC.

---

## 8. Acessibilidade (BR-MOB-A11Y) — alvo WCAG 2.1 AA

### BR-MOB-A11Y-001 — `Semantics`

Todo botão, ícone interativo, tap target tem `Semantics(label: '...', button: true)` ou widget com `semanticsLabel`. Ex: `IconButton(tooltip: 'Ver ata', ...)` → tooltip vira label para screen reader.

### BR-MOB-A11Y-002 — Tamanho texto escalável

`MediaQuery.textScaler` respeitado. Font sizes responsivos — layout **não pode quebrar** em `textScaleFactor = 2.0`. Testar com `flutter run --enable-experiment=...` em device com "texto grande" nativo.

### BR-MOB-A11Y-003 — Touch targets

- Min **44×44 logical px** (iOS HIG).
- Min **48×48 dp** (Material Design 3 Android).
- Botões pequenos: `Padding` ou `InkWell` externo para touch area maior que a visual.

### BR-MOB-A11Y-004 — Contraste

Mesmo requisito web: **4.5:1** normal text; **3:1** large text (≥ 18pt ou ≥ 14pt bold).

### BR-MOB-A11Y-005 — Reduce motion

`MediaQuery.disableAnimations` → minimizar ou desligar transições decorativas (ex: cartões animados na Home).

### BR-MOB-A11Y-006 — Screen reader

TalkBack (Android) + VoiceOver (iOS) testados em critical paths (login, voto, Connect Me submit). Announce atualizações dinâmicas via `SemanticsService.announce` (ex: "Voto registrado com sucesso").

---

## 9. Observability (BR-MOB-OBS)

### BR-MOB-OBS-001 — Sentry Flutter (Sprint 7)

DSN via `--dart-define=SENTRY_DSN`. `tracesSampleRate: 0.2` prod; 1.0 dev. `beforeSend` scrubbing de `Authorization`, `X-Device-Fingerprint`, `Cookie`, `email`, `cpf`, `cnpj`, `token`.

### BR-MOB-OBS-002 — Symbol upload CI

`flutter build --obfuscate --split-debug-info=build/symbols/` + `sentry-cli upload-dif build/symbols` (iOS + Android) para stack traces legíveis.

### BR-MOB-OBS-003 — `logger` package

**Nunca** `print()`. `Logger().i/w/e(...)` com PII masking helper `maskCpf(...)`, `maskEmail(...)`, `maskToken(...)`.

### BR-MOB-OBS-004 — Crash-free rate target

- ≥ **99.5%** crash-free sessions.
- ≥ **99%** crash-free users.
- Dashboard Sentry: weekly review antes de cada release store.

### BR-MOB-OBS-005 — Performance

FCP (First Contentful Paint), TTI (Time To Interactive), scroll jank — Sentry Performance + Firebase Performance (opcional M2+).

---

## 10. Testing (BR-MOB-QA)

### BR-MOB-QA-001 — Coverage thresholds

Ver [[patterns#101-coverage-thresholds-claude-md-legado|patterns §10.1]].

### BR-MOB-QA-002 — Gates de merge

```bash
fvm flutter analyze                        # zero warning
fvm flutter test --coverage                # thresholds respeitados
fvm flutter build apk --debug              # Android sanity
fvm flutter build ios --no-codesign        # iOS sanity
fvm dart run build_runner build --delete-conflicting-outputs  # generated em dia
```

### BR-MOB-QA-003 — Integration tests

`integration_test/` para critical paths:
- Auth flow completo (OIDC PKCE + refresh + logout).
- Onboarding morador M1-M8 equivalente.
- Vote flow end-to-end (assembleia → pauta → voto → success).
- Upload vídeo-currículo.
- Sync queue offline → online.

### BR-MOB-QA-004 — Device matrix

Mínimo testado em CI:
- **Android**: API 26 (Android 8) + API 34 (Android 14).
- **iOS**: iOS 15 + iOS 17.
- Screen sizes: 5" (compact), 6.5" (default), tablet 10".

### BR-MOB-QA-005 — Visual regression

Golden tests em `lib/shared/widgets/` (design system) + páginas críticas. Re-gerar goldens só com aprovação explícita em PR (flag `--update-goldens`).

---

## 11. Paywall + quotas (BR-MOB-BILL)

### BR-MOB-BILL-001 — Frontend exibe, backend decide

Mesma regra web. Ver [[../backend/requirements]] BR-FE-001..004. Planos **globais** (D-057); app consulta `tenant.planId` + matriz de permissão retornada pelo backend e renderiza:
- Seções bloqueadas: cartão "Recurso não incluído no seu plano" + CTA [Fazer upgrade].
- Quotas: exibe uso atual / limite (ex: "3 de 5 Connect Me usados este ano").

### BR-MOB-BILL-002 — Upgrade flow

Link Stripe Checkout abre em **Custom Tab (Android) / ASWebAuthenticationSession (iOS)** — **não embed** WebView (compliance Apple App Review Guidelines 3.1.1 + Google Play Billing policy distinção IAP vs web billing).

### BR-MOB-BILL-003 — Apple IAP compliance

Se algum dia monetizar digital content na store (não é o caso MVP — assinatura é B2B SaaS), usar StoreKit / Billing Client. **MVP é B2B web billing** via Stripe Checkout — compliance: OK não precisar IAP.

### BR-MOB-BILL-004 — Trial countdown

Banner persistente quando `trial_ends_at - now < 3d`. CTA `/account/assinatura` → upgrade flow.

---

## 12. LGPD mobile (BR-MOB-LGPD)

### BR-MOB-LGPD-001 — Consent banner primeiro uso

Equivalente web, respeitando design mobile (modal full-screen com opções Granular). Armazena `consent_given_at` em backend via `POST /me/consent`.

### BR-MOB-LGPD-002 — Consent Connect Me

Antes de APP-CM-002 submit (morador envia solicitação): checkbox LGPD explícito com texto da política + versão.

### BR-MOB-LGPD-003 — Data export

`/account/privacidade` (APP-ACC-006) → [Baixar meus dados] → backend `POST /me/export` → email com link temporário (TTL 24h) para download PDF/JSON.

### BR-MOB-LGPD-004 — Delete account

Double-confirm (modal 1: "Confirma?" + texto detalhado) + re-auth OIDC senha + **biometria obrigatória** (BR-MOB-BIO-003). Backend faz soft delete (LGPD Art. 16 + 18); hard delete após 90d cumpre right to erasure.

### BR-MOB-LGPD-005 — No PII em logs

Enforcement via `logger` wrapper com PII masking helper. Lints custom (futuro) podem detectar `logger.i('...${user.cpf}')` e falhar build.

### BR-MOB-LGPD-006 — Clipboard data sensível

Não copiar automaticamente CPF/senha/token pra clipboard. User action explícita only.

---

## 13. Build variants (BR-MOB-BUILD)

### BR-MOB-BUILD-001 — Flavors (Sprint 6)

- `dev` → `API_BASE_URL=http://10.0.2.2:8000` (Android emu) / `http://localhost:8000` (iOS sim); banner "DEV" topo; `ZITADEL_ISSUER=http://localhost:8080`.
- `staging` → `https://staging.api.mastersindico.com.br`; banner "STG".
- `prod` → `https://api.mastersindico.com.br`; sem banner.

Config em `android/app/build.gradle` `productFlavors` + iOS `Runner.xcodeproj` configurations + Xcode schemes.

### BR-MOB-BUILD-002 — Env via dart-define

Nunca hardcode URLs ou secrets em código. `--dart-define=KEY=value`. Defaults em `AppConstants` apontam pra dev local — confirmado no código real (`String.fromEnvironment(...)` com fallback).

### BR-MOB-BUILD-003 — Obfuscation prod

`--obfuscate --split-debug-info=build/symbols` **obrigatório** em release. Stack traces legíveis só via Sentry symbol upload.

---

## 14. Store compliance (BR-MOB-STORE)

### BR-MOB-STORE-001 — Privacy label iOS App Store

Declarar coleta na App Store Connect:
- **Name** — linked to user — Functionality.
- **Email address** — linked to user — Functionality.
- **Phone number** — linked to user — Functionality, Audit.
- **User ID** — linked to user — Functionality.
- **Device ID** — linked to user — Functionality (1-device IR-011).
- **Coarse Location** (não-coletada M1).
- **Purchases** (via Stripe web) — linked to user — Functionality.

### BR-MOB-STORE-002 — Google Play Data Safety

Mesma declaração + **Data encryption in transit** = Yes (HTTPS + TLS 1.2+) + **Data deletion** = Yes (APP-ACC-006 LGPD delete).

### BR-MOB-STORE-003 — Minimum OS

- **iOS 15+** (cobertura > 95% em 2026).
- **Android API 26+** (Android 8+, cobertura > 95%). **Atual `minSdk = 19` no `build.gradle.kts` precisa elevar para 26 antes de M1 ship.**

### BR-MOB-STORE-004 — Crash report review

A cada release: review Sentry top 20 crashes antes de submeter store. Bloqueante se crash-free < 99%.

---

## 15. Security específico (BR-MOB-SEC) — resumo

Especialização de [[security]]. Resumo:

- **BR-MOB-SEC-001** — Bearer token **apenas** em `flutter_secure_storage` (Keychain/Keystore). Nunca `SharedPreferences`.
- **BR-MOB-SEC-002** — Certificate pinning prod obrigatório para `api.mastersindico.com.br`; rotação semestral com coexistência (N + N+1 cert pinnados).
- **BR-MOB-SEC-003** — HTTPS-only native (ATS iOS + networkSecurityConfig Android `cleartextTrafficPermitted="false"` prod).
- **BR-MOB-SEC-004** — OIDC PKCE via `ASWebAuthenticationSession` / Custom Tab. **Nunca WebView embed** Zitadel.
- **BR-MOB-SEC-005** — Device fingerprint `X-Device-Fingerprint` em toda request (IR-011).
- **BR-MOB-SEC-006** — Device integrity `X-Device-Integrity` em toda request (D-050): `ok | dev-mode | jailbroken | hooked`. M1 report-only; M3 gated em features críticas.
- **BR-MOB-SEC-007** — Biometria obrigatória em ações críticas (procuração, ata signing, delete conta, bypass Lock 90d).
- **BR-MOB-SEC-008** — `FLAG_SECURE` em telas com dados sensíveis (ata, assembleia live, docs compliance).
- **BR-MOB-SEC-009** — Obfuscation `--obfuscate --split-debug-info` release.
- **BR-MOB-SEC-010** — PII scrubbing em logs e Sentry `beforeSend`.

---

## 16. Links

- [[README]]
- [[architecture]]
- [[security]]
- [[ui-catalog]]
- [[../backend/requirements]]
- [[../web/requirements]]
- [[../../04-requirements/global]]
- [[../../STEERING]]
- [[../../STATE#D-048|D-048]] · [[../../STATE#D-049|D-049]] · [[../../STATE#D-050|D-050]] · [[../../STATE#D-057|D-057]] · [[../../STATE#D-058|D-058]]
