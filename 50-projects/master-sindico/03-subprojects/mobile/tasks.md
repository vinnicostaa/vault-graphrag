---
title: Mobile — Tasks seed
type: note
tags: [subproject, mobile, master-sindico, v2, tasks, sprint, seed]
source:
  - Clientes/Joao/MasterSindico/Development/app/CLAUDE.md (§Próximas features, 2026-04-21)
  - ROADMAP M1 2026-05-08 / M2 2026-06-20 / M3 2026-07-20
  - ui-catalog.md (APP-### IDs)
  - STATE.md D-048 / D-049 / D-050
created: 2026-04-23
updated: 2026-04-23
---

# Mobile — Tasks seed

Tasks alvo do sub-projeto `app/` por sprint. Alinhado com [[../backend/tasks]] e [[../web/tasks]]. IDs `MB-###` (Mobile).

Formato: `MB-###` · complexidade (S/M/L/XL) · descrição · referência (`APP-###` UI, `BR-MOB-###` req, `D-###` decisão, `DT-MB-###` dívida).

> **M1 (2026-05-08)**: morador + auth + home shell + base infra. **M2 (2026-06-20)**: síndico light + Connect Me + vídeo + votação fornecedor. **M3 (2026-07-20)**: assembleia live + LMS thin + store submission.

---

## DT — Dívidas técnicas hoje (do diff `app/real` vs alvo)

Criar como DT-MB antes de arrancar M1:

- **DT-MB-001** — `minSdk = 19` no `build.gradle.kts`; BR-MOB-STORE-003 exige `26`. Elevar antes de M1 ship.
- **DT-MB-002** — Bundle ID Android atual `com.mastersindico.br.mastersindico`; alvo canônico `com.mastersindico.app`. Normalizar (atenção: muda update path se já publicado).
- **DT-MB-003** — Ausentes em `pubspec.yaml` mas citados em CLAUDE/ARCHITECTURE: `bloc_concurrency`, `hydrated_bloc`, `freezed`, `hive_flutter`, `local_auth`, `device_info_plus`, `firebase_messaging`, `app_links`, `easy_localization`, `freeRASP_flutter`. Adicionar em ondas conforme cronograma.
- **DT-MB-004** — `NoParams` não estende `Equatable`. Refactor + remover `registerFallbackValue(NoParams())` dos testes.
- **DT-MB-005** — Paleta `#6C63FF` é default Flutter create; reconciliar com Manual IV (ouro `#C69C3F` + navy `#071A33`) ou decidir paleta distinta com justificativa.
- **DT-MB-006** — `AuthRemoteDatasourceImpl` não tem `@LazySingleton(as: ...)` no imports list real (confirmar); já tem, OK. Dúvida: storage `CACHED_USER` vs chave canônica `current_user` de `AppConstants.userKey` — reconciliar.
- **DT-MB-007** — Keys reais `AUTH_TOKEN` em `auth_local_datasource.dart` vs `AppConstants.tokenKey = auth_access_token`. Normalizar.
- **DT-MB-008** — Refresh token não persiste hoje (re-login OIDC a cada expiração). M1 persistir em `flutter_secure_storage`.
- **DT-MB-009** — `Info.plist` sem `NSCameraUsageDescription`, `NSMicrophoneUsageDescription`, `NSPhotoLibraryUsageDescription`, `NSFaceIDUsageDescription`, `NSAppTransportSecurity`. Adicionar em bloco M1.
- **DT-MB-010** — `AndroidManifest.xml` sem `networkSecurityConfig`, sem permissions push/camera, sem intent-filter app links (apenas custom scheme). Adicionar em bloco M1.
- **DT-MB-011** — Flavors `dev/staging/prod` não configurados. Sprint 6.
- **DT-MB-012** — Rotas assembleia atuais em `/condominiums/:id/assemblies/*` — fora do `/home` shell M1; reorganizar rotas dentro do Shell + selector de condomínio (APP-HOME-002) em M1.
- **DT-MB-013** — `dio_web_adapter` presente nas transitive mas web não é target; validar que mobile não paga custo desnecessário.

---

## Sprint 10 — M1 (2026-05-08)

### Setup & infra

- **MB-001** · L · Elevar `minSdk 19 → 26` + reconciliar bundle ID `com.mastersindico.app` → DT-MB-001, DT-MB-002
- **MB-002** · M · Adicionar dependências M1 em `pubspec.yaml`: `bloc_concurrency`, `hydrated_bloc`, `freezed` + `freezed_annotation` + `json_serializable` + `json_annotation`, `hive_flutter` + `hive`, `local_auth`, `device_info_plus`, `firebase_messaging` + `firebase_core`, `app_links`, `crypto`, `flutter_windowmanager` · rodar `build_runner build` → DT-MB-003
- **MB-003** · S · Normalizar `AppConstants` keys + refactor `AuthLocalDatasourceImpl` → DT-MB-006, DT-MB-007
- **MB-004** · M · Refresh token persist + `RefreshInterceptor` Dio (concurrência com mutex) → DT-MB-008
- **MB-005** · M · Dio interceptors completos: Auth + Device (fingerprint + integrity) + Refresh + Error + Log (dev-only)
- **MB-006** · S · `NoParams extends Equatable` + remover `registerFallbackValue` → DT-MB-004
- **MB-007** · M · Reconciliar paleta (Manual IV ou decisão registrada) + atualizar `app_colors.dart` + `app_theme.dart` → DT-MB-005
- **MB-008** · M · Sentry Flutter + `beforeSend` scrubbing + obfuscation + symbol upload CI — **Sprint 7** mas placeholder scripts M1
- **MB-009** · S · `logger` wrapper + `pii_masker.dart` (maskCpf/maskEmail/maskToken)
- **MB-010** · M · `easy_localization` + `assets/translations/pt_BR.json` (dicionário base M1 morador)
- **MB-011** · S · Google Services config (Firebase project dev/staging/prod — stubs dev M1)
- **MB-012** · M · `Info.plist` permissions (camera/mic/photo/FaceID/ATS) → DT-MB-009
- **MB-013** · M · `AndroidManifest.xml` permissions (camera + notifications) + `network_security_config.xml` + app links intent-filter → DT-MB-010

### Auth (feature: auth)

- **MB-020** · M · Refactor `AuthRemoteDatasourceImpl` para usar `refresh_token` (hoje não persiste) · APP-AUTH-001
- **MB-021** · M · `AuthBloc` evoluir: `AuthCheckRequested` (bootstrap splash), `AuthRefreshRequested`, `AuthSessionExpired` state · APP-SYS-001
- **MB-022** · M · Wire-up OIDC custom scheme bidirecional (deep-link `com.mastersindico://auth` volta do browser externo) — confirmar `app_links` integração
- **MB-023** · S · `LoginForm` label pt-BR "[Entrar]" (remover "com Zitadel") + layout per design M1
- **MB-024** · S · APP-AUTH-002 Logout confirm modal
- **MB-025** · M · `DeviceFingerprint` helper (`device_info_plus` → SHA-256) + persist em secure storage
- **MB-026** · S · APP-SYS-004 Device mismatch page (`NewDeviceFailure` → redirect)
- **MB-027** · M · Health-check `AuthRepository.getCurrentUser()` + cache 5min

### Core / Design System widgets

- **MB-040** · M · Design system widgets base: `AppButton` (variants primary/secondary/text/destructive), `AppInput`, `AppCard`, `AppBadge`, `AppSpinner`, `AppEmptyState`, `AppErrorState`
- **MB-041** · M · Golden tests em design system widgets
- **MB-042** · M · `SemanticsLabel` helpers + a11y audit inicial (touch targets 44×44 / 48×48, contraste, textScaler)
- **MB-043** · S · Error boundary (via `ErrorWidget.builder` + Sentry capture)
- **MB-044** · S · `ToastHost` / `SnackbarHost` globais
- **MB-045** · S · Loading + empty state widgets reusáveis
- **MB-046** · S · `AppLifecycleState` overlay para telas sensíveis (`FLAG_SECURE` helper)

### Home shell (feature: institutional)

- **MB-060** · M · APP-SYS-001 Splash + bootstrap check token + `AuthCheckRequested`
- **MB-061** · M · APP-HOME-001 Shell com bottom nav 4 tabs (ShellRoute GoRouter)
- **MB-062** · M · APP-HOME-002 Selector condomínio (bottom sheet top) + cache memberships
- **MB-063** · L · APP-MOR-001 Painel morador (home feed) — substituir stub `home_page.dart`
- **MB-064** · M · APP-MOR-002 Timeline lista + pull-to-refresh + lazy load
- **MB-065** · M · APP-MOR-003 Timeline detalhe
- **MB-066** · M · APP-MOR-004 Plano Diretor leitura
- **MB-067** · M · APP-MOR-005 Eventos lista
- **MB-068** · M · APP-MOR-006 Evento detalhe + ciência/participação

### Onboarding morador

- **MB-080** · M · APP-MOR-011 Buscar condomínio por ID
- **MB-081** · M · APP-MOR-012 Identificação da unidade (wizard)
- **MB-082** · M · APP-MOR-013 Termo de ciência
- **MB-083** · M · APP-MOR-014 Status da unidade
- **MB-084** · M · APP-MOR-009 Avaliação obrigatória (gate via `GoRouter.redirect`)
- **MB-085** · M · APP-MOR-010 Gerenciar acessos

### Conta + assinatura

- **MB-100** · M · APP-ACC-001 Conta home
- **MB-101** · M · APP-ACC-002 Perfil + foto (camera + galeria + compressão)
- **MB-102** · M · APP-ACC-003 Assinatura + trial countdown + CTA Stripe Checkout em Custom Tab/ASWebAuthenticationSession
- **MB-103** · M · APP-ACC-004 Segurança + biometria toggle + sessões ativas (backend lista)
- **MB-104** · S · APP-ACC-006 Privacidade/LGPD (export + delete conta com biometria obrigatória)
- **MB-105** · S · APP-ACC-007 Sobre (versão + build + commit hash + termos)

### Offline M1 (Hive — D-041 alvo)

- **MB-120** · M · Hive setup + cipher com chave em `flutter_secure_storage` + `Hive.initFlutter()` em main
- **MB-121** · M · Boxes: `user`, `memberships`, `timeline:{condId}`, `events:{condId}`, `plano-diretor:{condId}` com TTL custom helper
- **MB-122** · M · `hydrated_bloc` storage config + `MembershipsBloc` primeiro consumer
- **MB-123** · M · Offline banner widget + cached data labels "atualizado há X min"
- **MB-124** · S · `NetworkInfoImpl` refactor — já existe, só validar D-048 integration

### Push M1 (FCM)

- **MB-140** · M · Firebase setup iOS + Android (dev + staging + prod flavors)
- **MB-141** · M · FCM token register backend `POST /api/v1/devices/register-push-token` + `onTokenRefresh`
- **MB-142** · M · Push handlers (foreground + background + terminated) + deep-link via `app_links`
- **MB-143** · M · Android notification channels (6 categorias — BR-MOB-PUSH-003)
- **MB-144** · M · iOS notification categories (6 categorias)
- **MB-145** · S · APP-NOT-001 Inbox de notificações
- **MB-146** · S · APP-NOT-002/003 Foreground toast + system notification flows

### Testing M1

- **MB-160** · M · Unit tests use cases + entities (coverage ≥ 95% domain)
- **MB-161** · M · Bloc tests (coverage ≥ 90% bloc)
- **MB-162** · M · Integration test — auth flow OIDC
- **MB-163** · M · Integration test — onboarding morador completo M1-M8
- **MB-164** · S · Widget tests páginas críticas (LoginPage, HomePage, TimelinePage, EventoDetailPage)
- **MB-165** · S · Golden tests design system widgets
- **MB-166** · S · CI coverage gate script (≥ 80% global, por camada)

### Security M1

- **MB-180** · M · Certificate pinning prod (`CertPinningAdapter` custom) — decidir vs `http_certificate_pinning` (P-APP-003)
- **MB-181** · S · HTTPS-only native config (ATS iOS + `network_security_config.xml` Android) — MB-012 + MB-013
- **MB-182** · M · freeRASP (Talsec) integração + header `X-Device-Integrity` report-only (D-050 M1) → `DeviceInterceptor`
- **MB-183** · M · `local_auth` integração + biometria toggle em APP-ACC-004 + usos críticos APP-ACC-006 delete conta
- **MB-184** · S · `FlutterSecureStorage` hardening (iOS: `kSecAttrAccessibleWhenPasscodeSetThisDeviceOnly`; Android: `StorageCipherAlgorithm.AES_GCM_NoPadding`)
- **MB-185** · S · Hive box encryption wire-up via cipher helper
- **MB-186** · S · Logout clean completo (Zitadel + secure + Hive + FCM unsubscribe + hydrated_bloc clear)

### Build & store

- **MB-200** · M · Android + iOS flavors (dev/staging/prod) → DT-MB-011
- **MB-201** · M · `Makefile` / scripts build/test (`make build-android-prod`, etc.)
- **MB-202** · M · CI GitHub Actions: `subosito/flutter-action` + analyze + test + build APK debug + build iOS no-codesign + build_runner
- **MB-203** · M · TestFlight + Google Play Internal upload manual (`fastlane` Sprint 8.9)

---

## Sprint 11 — M2 (2026-06-20) — Síndico light + Connect Me + vídeo

### Síndico mobile light (features: institutional + commercial)

- **MB-220** · M · APP-SIN-001 Governança entrada (bottom nav extra "Governança" 5ª aba)
- **MB-221** · M · APP-SIN-002 Central resumo
- **MB-222** · L · APP-SIN-003 Timeline novo registro voice-first (`speech_to_text` + fallback texto)
- **MB-223** · M · APP-SIN-004 Plano Diretor lista
- **MB-224** · M · APP-SIN-005 Plano Diretor — nova ação (stepper 4 steps)
- **MB-225** · M · APP-SIN-006 Eventos criar mobile
- **MB-226** · M · APP-SIN-007 Validações pendentes (swipe actions — `Dismissible`)
- **MB-227** · M · APP-SIN-008 Connect Me nova solicitação (síndico)
- **MB-228** · M · APP-SIN-009 Indicadores resumo swipe horizontal
- **MB-229** · S · APP-SIN-010 Modo apresentação rápida

### Connect Me morador

- **MB-240** · M · APP-CM-001 Hub morador + quota card (matriz D-094: Morador Base = 2/ano → CTA upgrade quando saldo = 0)
- **MB-241** · M · APP-CM-002 Nova solicitação morador (voice + LGPD obrigatório) · `bloc_concurrency.droppable()`
- **MB-242** · M · APP-CM-003 Minhas solicitações status
- **MB-243** · S · Cache local lista solicitações (Hive `connect_me:{userId}` TTL 15min)

### Vídeo morador

- **MB-260** · M · APP-VID-001 Mux Player wrapper (`mux_player_flutter` ou `better_player` + HLS signed URL) + lock badge
- **MB-261** · L · APP-VID-002 Upload vídeo-currículo (camera até 90s + Mux presigned + progress + cancel)
- **MB-262** · M · APP-VID-003 Unlock visibility durante votação fornecedor (signed URL TTL ≤ 1h)

### Votação fornecedor

- **MB-280** · M · APP-MOR-007 Votação fornecedor morador (cards + player) · `droppable()` obrigatório
- **MB-281** · M · APP-MOR-008 Ver proposta completa (bottom sheet full-height)

### Voice input (transversal)

- **MB-300** · M · `speech_to_text` Flutter plugin + permissão microfone + fallback textual
- **MB-301** · S · Tela permissão microfone + revoked handling (BR-MOB-CAM-004)

### Camera + fotos

- **MB-320** · M · Image picker + camera helpers + compressão cliente (BR-MOB-CAM-003)
- **MB-321** · M · Foto execução para síndico mobile (registrar serviço campo) — M2 ou M3 (P-APP-004)

### Offline M2

- **MB-340** · M · `SyncBloc` + `sync_queue` Hive box (ciência, confirmar participação, mark read) · BR-MOB-OFF-003
- **MB-341** · S · Retry policy exponencial + `sync_queue_failed` box pós-esgotamento

### Admin role (D-058) — entry mobile M2

- **MB-350** · M · APP-ADM-001 Admin home gated por permissão `admin.access` no token
- **MB-351** · M · APP-ADM-002 Bypass Lock 90d — solicitar (mobile entry)
- **MB-352** · M · APP-ADM-003 Bypass Lock 90d — aprovar (4-eyes — D-054, biometria obrigatória)

### Testing M2

- **MB-360** · M · E2E Connect Me morador + síndico light
- **MB-361** · M · E2E upload vídeo-currículo
- **MB-362** · M · E2E votação fornecedor
- **MB-363** · M · E2E sync queue offline → online

---

## Sprint 12 — M3 (2026-07-20) — Assembleia Live + LMS thin + Store submit

### Assembleia Live (feature: assembly)

- **MB-400** · M · APP-ASM-001 Lista abas Agendadas/Em andamento/Encerradas (refactor da página existente)
- **MB-401** · M · APP-ASM-002 Detalhe + edital PDF viewer (`flutter_pdfview` ou `pdfx`)
- **MB-402** · M · APP-ASM-003 Ciência da pauta — refactor: lista itens com checkbox individual (hoje é checkbox único geral)
- **MB-403** · L · APP-ASM-004 Outorgar procuração + assinatura biométrica (local_auth + backend signing payload)
- **MB-404** · M · APP-ASM-005 Check-in (`mobile_scanner` QR + código manual + check-in remoto)
- **MB-405** · XL · APP-ASM-006 Live session (`livekit_client` Flutter SDK + WS votação + fila fala + semanticsLabel live region + FLAG_SECURE)
- **MB-406** · M · APP-ASM-008 Ata — upgrade com PDF viewer + download

### LMS thin mobile

- **MB-430** · M · Lista cursos por área + filtros
- **MB-431** · M · Player lesson (Mux HLS)
- **MB-432** · M · Quiz UI (multiple choice + submit)
- **MB-433** · M · Certificado viewer + share (via `share_plus`)

### Modo assembleia apresentação

- **MB-450** · M · Tela full-screen apresentação (síndico) — esconde status bar + bottom nav

### Security M3 (D-050 gated)

- **MB-460** · M · freeRASP escalonar para gated — bloquear features críticas em `jailbroken`/`hooked` → APP-SYS-006
- **MB-461** · S · Revalidação periódica integridade durante sessão

### Polish

- **MB-470** · M · Performance audit (DevTools + Sentry Performance) — BR-MOB-PERF
- **MB-471** · M · Memory leak fix passes
- **MB-472** · M · Battery drain audit (WebSocket teardown + FCM handler eficiente)
- **MB-473** · M · Dark mode polish (se reconciliação de paleta MB-007 mantém dark)
- **MB-474** · S · App size audit (split AAB; dynamic delivery LMS se bundle > 40MB)

### Store submit

- **MB-490** · M · App Store submission + metadata + screenshots (5" + 6.5" + tablet) + Privacy Labels
- **MB-491** · M · Google Play submission + metadata + screenshots + Data Safety declaração
- **MB-492** · S · Review compliance checklist (IAP não aplicável, B2B SaaS declaração)

### Testing M3

- **MB-510** · L · E2E assembleia live full (ciência → check-in → vote → ata)
- **MB-511** · M · E2E LMS enrollment + certificate
- **MB-512** · M · Testes em device matrix (Android 8/14; iOS 15/17; tablet) — BR-MOB-QA-004
- **MB-513** · M · Accessibility audit manual + TalkBack/VoiceOver em critical paths

---

## Backlog pós-M3

- **MB-600** · XL · Empresa mobile app (se demanda validar M4+)
- **MB-601** · L · iOS widget + Android widget (timeline resumo na home)
- **MB-602** · L · Apple Watch / Wear OS notifications (stretch)
- **MB-603** · M · Offline pleno (todas rotas críticas cached read + estratégia write-queue completa)
- **MB-604** · M · Push rich notifications (image + action buttons interativos)
- **MB-605** · L · Accessibility audit externo WCAG AA
- **MB-606** · M · Feature flags mobile (backend `/api/v1/features/:feature/enabled` + cache 5min)
- **MB-607** · M · A/B testing framework (Firebase Remote Config ou Unleash)
- **MB-608** · L · Internacionalização EN (ativar chaves `en.json`)
- **MB-609** · M · Modo baixo-consumo (streaming qualidade adaptativa + sync pausado)

---

## Tasks técnicas transversais

- **MB-900** · M · CI/CD full (`fastlane` iOS + Android → TestFlight + Play Console automação)
- **MB-901** · S · Crashlytics opcional (complementar Sentry) — decidir redundância
- **MB-902** · M · Firebase Performance Monitoring (opcional M2+)
- **MB-903** · S · Pre-commit hooks (`analyze` + `format` + `build_runner` check)
- **MB-904** · M · `build_runner watch` ergonomia dev + CI caching
- **MB-905** · S · Release train cadence (1 release/mês para beta; 1/bimestre para prod)

---

## Gates de merge (DoD)

Toda PR mobile precisa:

- ✅ `fvm flutter analyze` zero warning.
- ✅ `fvm flutter test --coverage` thresholds respeitados ([[patterns#10-testing-patterns|patterns §10.1]]).
- ✅ `fvm dart run build_runner build --delete-conflicting-outputs` generated em dia (`injection.config.dart`, `*.g.dart`, `*.freezed.dart`).
- ✅ `fvm flutter build apk --debug` sem erro.
- ✅ `fvm flutter build ios --no-codesign` sem erro.
- ✅ PR review + commit pt-BR Conventional Commits.
- ✅ AUDIT sem 🔴 novo.
- ✅ Se toca security: security-review skill passa.
- ✅ Release Notes: seção no PR body.

---

## Links

- [[README]]
- [[architecture]]
- [[ui-catalog]]
- [[security]]
- [[requirements]]
- [[../backend/tasks]]
- [[../web/tasks]]
- [[../../05-tasks/_moc]]
- [[../../ROADMAP]]
- [[../../STATE#D-048|D-048]] · [[../../STATE#D-049|D-049]] · [[../../STATE#D-050|D-050]] · [[../../STATE#D-054|D-054]] · [[../../STATE#D-057|D-057]] · [[../../STATE#D-058|D-058]]
