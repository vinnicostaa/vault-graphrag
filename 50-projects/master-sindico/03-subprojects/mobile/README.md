---
title: Mobile Master Síndico — README
type: spec
tags: [subproject, mobile, master-sindico, v2, flutter, dart, clean-arch, bloc]
source:
  - Clientes/Joao/MasterSindico/Development/app/CLAUDE.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/ARCHITECTURE.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/CODING_MANUAL.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/pubspec.yaml (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/pubspec.lock (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/lib/** (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/android/app/src/main/AndroidManifest.xml (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/ios/Runner/Info.plist (2026-04-21)
created: 2026-04-23
updated: 2026-04-23
aliases: [mobile-readme, app-readme]
---

# Mobile — Master Síndico (sub-projeto `app/`)

App **iOS + Android** construído uma única vez em **Flutter/Dart** e consumido por morador e síndico em mobilidade. Empresa, parceiro, agência de marketing e admin usam [[../web/README|web]] — melhor UX desktop. Admin = **`apps/admin` dedicada no monorepo web** (D-134 Fase H revoga D-058/D-072) com Cloudflare Access SSO+MFA — mobile **não tem telas admin** (acessadas via desktop em `admin.mastersindico.com.br`).

Planos são **globais estilo Stripe** (D-057 / D-066): sem slugs N1/N2/N3 — app consulta `tenant.planId` + matriz de permissão carregada pelo backend e renderiza UI conforme features liberadas. Tier universal para todas as personas.

> **Nota de pasta**: repo local chama-se `app/` (renomeado de `mobile/` em 2026-04-21 para casar com o repo GitHub). Qualquer doc antigo que mencione `mobile/` como folder local está stale — **sempre** `app/`.

Repo alvo: `github.com/mastersindico/app`.

---

## 1. Stack confirmada

Ancorada no `pubspec.yaml` e `pubspec.lock` lidos em 2026-04-21, cruzado com [[../../STATE#D-048|D-048]] (`flutter_bloc + bloc_concurrency + hydrated_bloc`) e [[../../STATE#D-049|D-049]] (`freezed` desde M1).

| Camada | Tech | Versão confirmada | Motivo |
|---|---|---|---|
| Gerenciamento SDK | **FVM** (`.fvmrc` → `stable`) | `.fvmrc` existe | SDK Flutter travado por máquina; parity com CI |
| Framework | **Flutter** canal stable | SDK `^3.11.4` (pubspec) | 1 codebase iOS+Android, hot reload, performance nativa |
| Linguagem | **Dart 3.x** null-safety | SDK `^3.11.4` | Type system maduro; sealed classes nativas (Dart 3) |
| Arquitetura | **Clean Architecture + Feature First** | `lib/app`, `lib/core`, `lib/features/<feat>/{domain,data,presentation}` | Paridade com backend Go; regra de dependência inviolável |
| Estado (D-048) | **flutter_bloc** | `^9.1.0` (direct main) | Event-driven audit trail natural; battle-tested enterprise |
| Estado — droppable | **bloc_concurrency** (D-048 mandatório) | ⚠️ ausente do `pubspec.yaml` — **dívida** | Evitar double-submit voto/proposta/pagamento |
| Estado — persistido | **hydrated_bloc** (D-048 mandatório) | ⚠️ ausente do `pubspec.yaml` — **dívida** | Hydrate state Bloc em offline |
| Imutabilidade/Unions (D-049) | **freezed + freezed_annotation** | ⚠️ ausente — **dívida M1** | Sealed states Bloc + VOs + copyWith sem boilerplate |
| DI | **get_it + injectable** (+ `build_runner`) | `get_it ^8.0.3`, `injectable ^2.5.0`, `injectable_generator ^2.7.0`, `build_runner ^2.4.14` | `@lazySingleton` / `@injectable` / `@module`; sem widget-tree coupling |
| Rotas | **go_router** | `^14.8.1` | Declarativo, type-safe, deep linking |
| HTTP | **Dio** | `^5.8.0` | Interceptors, retry, cancelToken; CVE-2024-41881 corrigido em 5.7+ |
| Connectivity | **connectivity_plus** | `^6.1.4` | Checar online/offline em `NetworkInfoImpl` |
| Secure storage (tokens) | **flutter_secure_storage** | `^10.0.0` | iOS Keychain + Android Keystore |
| Shared prefs (leve, não-sensível) | **shared_preferences** | `^2.5.3` | Flags UI, `hasSeenOnboarding`, cache user snapshot não-sensível |
| Functional | **dartz** | `^0.10.1` | `Either<Failure, T>` no cross-boundary |
| Value equality | **equatable** | `^2.0.7` | Entity/Event/State equality — obrigatório |
| OIDC (Zitadel) | **oidc + oidc_default_store** | `oidc ^0.14.0+2`, `oidc_default_store ^0.6.0+2` | PKCE Authorization Code Flow |
| i18n | **intl** | `^0.20.0` | Locale pt_BR, date/number |
| Logging | **logger** | `^2.5.0` | Nunca `print()`; wrapper com PII masking |
| Icons | **cupertino_icons** | `^1.0.8` | Material + Cupertino |
| Cache offline | **hive_flutter + hive** | ⚠️ ausente — **dívida M1** (D-041 aberto) | Cache seletivo (memberships, timeline, eventos, plano diretor) |
| Biometria | **local_auth** | ⚠️ ausente — **dívida M1** | FaceID / Touch ID / BiometricPrompt |
| Device info | **device_info_plus** | ⚠️ ausente — **dívida M1** | Fingerprint 1-device (IR-011) |
| Push | **firebase_messaging + firebase_core** | ⚠️ ausente — **dívida M1** | FCM iOS + Android |
| App links / Universal links | **app_links** | ⚠️ ausente — **dívida M1** | Deep links HTTPS + custom scheme |
| Jailbreak/Root (D-050) | **freeRASP (Talsec) flutter** | ⚠️ ausente — **dívida M1** (M1 report-only, M3 gated) | OWASP MASWE-0097 |
| i18n runtime | **easy_localization** | ⚠️ ausente — **dívida M2** | Dictionary pt_BR.json |
| Crash reporting | **sentry_flutter** | ⚠️ ausente — **Sprint 7** | Crash + performance; PII scrubbing |
| Video player | **mux_player_flutter** (ou `better_player` wrapper Mux) | ⚠️ ausente — **Sprint 5 / M2** | HLS signed Mux |
| Camera/mídia | **image_picker + camera** | ⚠️ ausente — **Sprint 5 / M2** | Vídeo-currículo (90s max), foto execução |
| Speech-to-text | **speech_to_text** | ⚠️ ausente — **Sprint 5 / M2** | Timeline voice-first síndico mobile |
| Livekit (live) | **livekit_client** | ⚠️ ausente — **Sprint 7 / M3** | Assembleia live WebRTC |
| Screen secure | **flutter_windowmanager** (Android `FLAG_SECURE`) | ⚠️ ausente — **Sprint 7 / M3** | Evitar screenshot em ata/assembleia live |
| Testing | **flutter_test + bloc_test + mocktail** (+ `integration_test`) | `bloc_test ^10.0.0`, `mocktail ^1.0.4` | Sem codegen em mocktail |
| Lints | **flutter_lints** | `^6.0.0` | `analysis_options.yaml` herda `package:flutter_lints/flutter.yaml` |

> **Dívida M1 consolidada** (registrar como DT-MB-###): `bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive`, `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`, `freeRASP`, `easy_localization`. Todas presentes como intenção em CLAUDE/ARCHITECTURE do legado, mas **ausentes em pubspec real**. M1 precisa destravar pelo menos: `bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive_flutter`, `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`.

---

## 2. Nativo — plataformas mínimas

| Item | iOS | Android |
|---|---|---|
| OS mínimo | iOS 15+ (cobertura > 95% 2026) | API 26 (Android 8+) — `minSdk = 19` atual no `build.gradle.kts` ⚠️ **elevar para 26** |
| Bundle ID / package | `com.mastersindico.app` alvo; atual Android `com.mastersindico.br.mastersindico` ⚠️ **normalizar** | idem |
| SDK/Compile | flutter `flutter.compileSdkVersion` | idem; `sourceCompatibility = VERSION_17`, `jvmTarget = 17` |
| Custom URL scheme | `Info.plist CFBundleURLSchemes` = `com.mastersindico` ✅ | `AndroidManifest.xml` intent-filter `<data android:scheme="com.mastersindico" />` ✅ |
| Universal / App links (M1) | `apple-app-site-association` via domínio `app.mastersindico.com.br` (pendente) | `intent-filter android:autoVerify="true"` + `assetlinks.json` (pendente) |
| Keychain / Keystore | `flutter_secure_storage` default (`accessibility: first_unlock`) — sem entitlement extra necessário | AES-256 GCM default |
| Biometria | FaceID + Touch ID (strings `NSFaceIDUsageDescription` pendente em `Info.plist`) | BiometricPrompt |
| ATS (HTTPS-only) | `NSAppTransportSecurity` pendente — configurar para bloquear cleartext exceto dev | `network_security_config.xml` pendente — `cleartextTrafficPermitted="false"` em prod |
| Camera/Mic (permission strings) | `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSPhotoLibraryUsageDescription` pendentes | `<uses-permission android:name="android.permission.CAMERA"/>` etc. pendentes |
| Notification (push) | APNs cert + `UIBackgroundModes remote-notification` pendente | Firebase SDK + channel registration (6 categorias) |
| `FLAG_SECURE` | `flutter_windowmanager` cross-platform | idem |
| Splash/launch | `LaunchScreen.storyboard` + `AppIcon.appiconset` | `@style/LaunchTheme` + `@style/NormalTheme` (já configurado) |
| Scene delegate iOS | `SceneDelegate.swift` (`UISceneConfigurationName: flutter`) ✅ | n/a |

> O `Info.plist` atual **declara apenas** `CFBundleURLSchemes = com.mastersindico`. Já está pronto para OIDC custom scheme, mas **ainda não declara** ATS, permissões de câmera/mic/biometria nem universal links. O `AndroidManifest.xml` atual **declara o custom scheme**, mas ainda não declara app links `autoVerify="true"` nem permissões de push/camera.

---

## 3. Módulos (features = BCs)

Paridade com [[../../01-domain/bounded-contexts]]. No código real hoje existem **apenas 3 features**: `auth` (scaffold OIDC), `assembly` (entities + usecases + bloc + 5 páginas) e `home` (stub). O restante é scaffold alvo M1-M3.

| Feature | BC backend | Personas-alvo mobile | Status no código real | Marco |
|---|---|---|---|---|
| `auth` | identity | Todos | 🟡 scaffold OIDC real via `oidc` package + user manager; token ainda salvo em `SharedPreferences` como `CACHED_USER` + token em `FlutterSecureStorage` | M1 |
| `institutional` | institutional | Morador + Síndico | ❌ ausente (apenas `home` stub) | M1 |
| `commercial` | commercial | Morador (Connect Me) + Síndico light | ❌ ausente | M2 |
| `content` | content | Morador (player + vídeo-currículo) + Síndico | ❌ ausente | M2 |
| `assembly` | assembly | Morador (ciência, voto, procuração, live) + Síndico (presidir) | 🟢 entities (Assembly, AgendaItem, Proxy, Minutes, ScienceRecord) + repository + 7 usecases + bloc + 5 páginas | M3 (live e procuração completas); leitura parcial já em M1 |
| `billing` | billing | Trial countdown + upgrade | ❌ ausente | M1 |

> Empresa, Parceiro e Agência de Marketing **não** têm app M1-M3 — usam web. M4+ pode avaliar app empresa. Admin é role em todos os perfis (D-058) — sem app separada.

---

## 4. Canal de distribuição

- **iOS**: App Store via TestFlight (beta interna) → App Store Connect (prod). Bundle ID `com.mastersindico.app` alvo.
- **Android**: Google Play Console Internal Testing → Closed → Open → Production. Package `com.mastersindico.app` alvo.
- **Beta interno**: Firebase App Distribution como canal alternativo em cross-team review.
- **Flavors** (pendente, Sprint 6): `dev`, `staging`, `prod` via `--dart-define` + `--flavor` em build scripts.
- **Obfuscation**: `--obfuscate --split-debug-info=build/symbols` obrigatório em release; símbolos enviados para Sentry via `sentry-cli upload-dif`.

---

## 5. Estrutura canônica

Refletindo o código real (lib/) estendido com os diretórios alvo M1-M3. `✅` = existe hoje no repo real; `🆕` = alvo.

```
app/
├── pubspec.yaml · pubspec.lock · analysis_options.yaml · .fvmrc
├── lib/
│   ├── main.dart                        ✅ bootstrap (WidgetsFlutterBinding + prefs + configureDependencies + runApp)
│   ├── app/
│   │   ├── app.dart                     ✅ MaterialApp.router + MultiBlocProvider (vazio hoje) + ThemeMode.system
│   │   ├── di/
│   │   │   ├── injection.dart           ✅ GetIt sl + @InjectableInit + RegisterModule (Connectivity, Dio, SharedPreferences-throw)
│   │   │   └── injection.config.dart    ✅ GERADO (build_runner) — NUNCA editar
│   │   └── router/
│   │       ├── app_router.dart          ✅ GoRouter top-level; rotas atuais: /, /login, /condominiums/:id/assemblies/*
│   │       └── app_routes.dart          ✅ AppRoutes abstract + helpers assemblyList/Detail/Science/Vote/Minutes
│   ├── core/
│   │   ├── error/
│   │   │   ├── exceptions.dart          ✅ ServerException · CacheException · NetworkException · UnexpectedException
│   │   │   └── failures.dart            ✅ Failure (abstract) → Server · Cache · Network · Auth · Unexpected
│   │   ├── network/
│   │   │   ├── network_info.dart        ✅ abstract
│   │   │   └── network_info_impl.dart   ✅ connectivity_plus — any(r != none)
│   │   ├── usecases/
│   │   │   └── usecase.dart             ✅ UseCase<T, Params> + NoParams
│   │   ├── utils/
│   │   │   ├── constants.dart           ✅ AppConstants abstract — baseUrl / timeouts / storage keys / Zitadel / verbose-logging
│   │   │   └── extensions/
│   │   │       ├── context_extensions.dart  ✅
│   │   │       └── string_extensions.dart   ✅
│   │   ├── theme/
│   │   │   ├── app_colors.dart          ✅ brand (primary #6C63FF, secondary #03DAC6), semantic, neutral light/dark, text
│   │   │   ├── app_text_styles.dart     ✅ MD3 scale: display/headline/title/body/label (large/medium/small)
│   │   │   └── app_theme.dart           ✅ AppTheme.light / AppTheme.dark — ColorScheme.fromSeed + MD3 component themes
│   │   └── security/                    🆕 fingerprint.dart · cert_pinning.dart · integrity.dart (freeRASP) · pii_masker.dart
│   ├── features/
│   │   ├── auth/                        ✅ domain (UserEntity + AuthRepository + LoginUseCase + LogoutUseCase)
│   │   │                                   data (UserModel + AuthRemoteDatasource[oidc] + AuthLocalDatasource[secure+prefs] + AuthRepositoryImpl)
│   │   │                                   presentation (AuthBloc + events Login/Logout/Check + states Initial/Loading/Auth/Unauth/Failure + LoginPage + LoginForm)
│   │   ├── home/                        ✅ presentation/pages/home_page.dart (stub "Olá, Template!")
│   │   ├── assembly/                    ✅ domain (5 entities + 7 usecases + AssemblyRepository)
│   │   │                                   data (5 models + AssemblyRemoteDatasource[Dio] + AssemblyRepositoryImpl)
│   │   │                                   presentation (AssemblyBloc + 7 events + 9 states + 5 pages + 3 widgets)
│   │   ├── institutional/               🆕 M1 — timeline, eventos, plano diretor, comunicados, avaliação obrigatória
│   │   ├── commercial/                  🆕 M2 — Connect Me morador + síndico, votação fornecedor
│   │   ├── content/                     🆕 M2 — vídeo-currículo, player Mux
│   │   └── billing/                     🆕 M1 — trial countdown + Stripe Checkout browser externo
│   └── shared/                          🆕 widgets (design system) + models (DTOs compartilhados)
├── assets/                              🆕 translations/pt_BR.json + images/ + fonts/
├── test/                                ✅ espelha lib/
│   ├── helpers/test_helper.dart         ✅ Mock* do repo/usecase
│   ├── core/error/failures_test.dart    ✅
│   ├── features/auth/
│   │   ├── domain/usecases/login_usecase_test.dart   ✅
│   │   └── presentation/bloc/auth_bloc_test.dart      ✅
│   ├── features/assembly/
│   │   ├── domain/usecases/cast_vote_usecase_test.dart        ✅
│   │   ├── domain/usecases/get_assemblies_usecase_test.dart   ✅
│   │   └── presentation/bloc/assembly_bloc_test.dart          ✅
│   └── widget_test.dart                 ✅
├── integration_test/                    🆕 critical paths (auth, onboarding, vote, upload)
├── android/                             ✅ app/build.gradle.kts (minSdk 19⚠️, compileSdk flutter, JVM 17) · AndroidManifest.xml (custom scheme registrado)
├── ios/                                 ✅ Runner/Info.plist (CFBundleURLSchemes com.mastersindico) · Runner/SceneDelegate.swift · AppDelegate.swift
└── .env.example                         🆕 template de --dart-define
```

---

## 6. Contratos com backend

### 6.1 REST

- **Base URL por ambiente** via `--dart-define=API_BASE_URL=...` — dev `http://localhost:8000`, prod `https://api.mastersindico.com.br` (confirmado em `lib/core/utils/constants.dart`).
- **Todas requests** recebem `Authorization: Bearer <access_token>` lido de `flutter_secure_storage` (Dio interceptor). **Nunca cookie httpOnly** em mobile — cookie é exclusivo do [[../web/README|web]].
- **`X-Device-Fingerprint: <sha256>`** em toda request, computado com `device_info_plus` (stub a implementar — IR-011).
- **`X-Device-Integrity: ok | dev-mode | jailbroken | hooked`** em toda request (D-050).
- **Response format** backend:

```json
// Success
{ "success": true, "data": { ... } }

// Error
{ "success": false, "error": { "code": "FORBIDDEN", "message": "..." } }
```

### 6.2 Auth OIDC + PKCE (Zitadel)

1. User abre `/login` → tap em **[Entrar]** → `AuthRemoteDatasourceImpl._getUserManager()` cria `OidcUserManager.lazy` (discovery well-known + client_id `366565963497287390` dev + redirect `com.mastersindico:/auth`).
2. `manager.loginAuthorizationCodeFlow()` dispara **browser externo**:
   - iOS: `ASWebAuthenticationSession` (o `oidc` package escolhe por padrão — **nunca WebView embed** Zitadel bloqueia).
   - Android: **Custom Tabs**.
3. Zitadel autentica → callback custom scheme `com.mastersindico:/auth` → app troca `code + verifier` por `access_token + refresh_token + id_token`.
4. `UserInfo.sub` → `UserModel.id`; `email` / `name` / `preferred_username` populados.
5. `AuthLocalDatasourceImpl.saveToken(token)` → `flutter_secure_storage` chave `AUTH_TOKEN`. **⚠️ Dívida**: hoje também faz `cacheUser` em `SharedPreferences` chave `CACHED_USER` (OK — snapshot leve não-sensível) e constants reserva `auth_access_token` / `auth_refresh_token` (alvo canônico; refactor pendente).
6. **Refresh automático** via interceptor Dio (pendente): `onError 401` → tenta refresh → retry; falha = logout global.
7. **Logout** → `manager.logout()` (`end_session` Zitadel) + `clearUser` + `clearToken` (secure_storage + prefs) + navega `/login`.

**Scopes OIDC mínimos** (em `AppConstants.zitadelScopes`): `openid`, `profile`, `email`, `offline_access` (obrigatório para refresh token).

### 6.3 Device fingerprint (IR-011)

`device_info_plus` → (`identifierForVendor` iOS / `id + fingerprint` Android) + `model` + `OS version` + `CPU` → SHA-256 → persistido em `flutter_secure_storage` chave `device_fingerprint` na primeira execução. Backend consolida e pode responder `403 NEW_DEVICE_DETECTED` → app navega para tela APP-SYS-004.

### 6.4 WebSocket assembleia live (M3)

`web_socket_channel` (`IOWebSocketChannel`) → `wss://api.mastersindico.com.br/ws/assembly/:id` com `Authorization: Bearer <token>` em header de conexão. Integração com `livekit_client` para mídia (vídeo/áudio) e WS separado para eventos de votação/fila de fala.

### 6.5 Push (FCM)

**Payload canônico**:

```json
{
  "notification": { "title": "...", "body": "..." },
  "data": {
    "type": "vote_opened | event_created | connect_me_interest | assembly_live_starting | ...",
    "resource_id": "01HXXX...",
    "tenant_id": "01HXXX...",
    "deep_link": "com.mastersindico://condominium/.../votacao/..."
  }
}
```

**Categorias/Channels** — ver [[requirements#2-push-notifications-br-mob-push|BR-MOB-PUSH]] (6 categorias: events, connect_me, votes, assembly_live, compliance, marketing).

---

## 7. Ligação com artefatos canônicos

- [[../../00-product/personas]]
- [[../../01-domain/bounded-contexts]]
- [[../../04-requirements/global]]
- [[../../04-requirements/functional/_moc]]
- [[../../02-architecture/api-design]]
- [[../../08-security/BEYOND_CORP]]
- [[../backend/README]]
- [[../web/design-system]] — tokens visuais espelhados
- [[../../STATE#D-048|D-048 Bloc + bloc_concurrency + hydrated_bloc]]
- [[../../STATE#D-049|D-049 freezed M1]]
- [[../../STATE#D-050|D-050 Jailbreak escalonado MASVS-R]]
- [[../../STATE#D-057|D-057 Planos globais Stripe-style]]
- [[../../STATE#D-058|D-058 Admin = role privilegiada]]

---

## 8. Documentos deste sub-projeto

- [[architecture]] — Clean Arch Flutter + DI + router + state mgmt Bloc + offline + networking + deep-link
- [[patterns]] — Bloc/UseCase/Factory/VO/Collections + Code Calisthenics Dart + testing
- [[ui-catalog]] — subset mobile das 141+ telas web, estruturado por seção
- [[requirements]] — offline, push, deep-link, biometria, camera, perf, i18n, a11y, obs, testing, paywall, LGPD, build, store
- [[security]] — Bearer em secure storage, cert pinning, OIDC PKCE, biometria, jailbreak escalonado, PII, OWASP Mobile Top 10
- [[tasks]] — seed MB-### por sprint M1-M3
- [[_moc]] — MOC do sub-projeto

## Links

- [[../../_moc]]
- [[../../CLAUDE]]
- [[../../STEERING]]
- [[../../ROADMAP]]
