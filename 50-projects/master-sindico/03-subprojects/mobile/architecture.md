---
title: Mobile — Arquitetura
type: spec
tags: [subproject, mobile, master-sindico, v2, architecture, flutter, clean-arch, feature-first, bloc]
source:
  - Clientes/Joao/MasterSindico/Development/app/ARCHITECTURE.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/CODING_MANUAL.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/lib/** (2026-04-21)
created: 2026-04-23
updated: 2026-04-23
aliases: [mobile-architecture, app-architecture]
---

# Mobile — Arquitetura

Arquitetura **Clean Architecture + Feature First** em Flutter/Dart. Complementa [[README]] com detalhes de estrutura, DI, router, state management, deep-linking, networking, offline, push e build.

---

## 1. Dependency Rule (inviolável — CODING_MANUAL §2.1)

```
presentation → domain ← data
```

- **`domain/`** — puro Dart. **Zero import** de Flutter, Dio, SharedPrefs, `flutter_secure_storage`, `oidc`. Só `dartz`, `equatable`, `injectable` annotations e pacote `intl` (formatação).
- **`data/`** — importa `domain/` para implementar repositórios; importa SDKs (`dio`, `flutter_secure_storage`, `oidc`, `connectivity_plus`).
- **`presentation/`** — importa `domain/` (via Bloc e UseCase) e UI Flutter. **Nunca** importa `data/`.

Features **nunca** importam `data/` ou `presentation/` de outras features. Compartilhar **apenas** via `core/` ou `domain/entities/` de outra feature (quando rigorosamente necessário).

### 1.1 Anti-patterns proibidos (CODING_MANUAL §3)

- ❌ `HttpClient` / `Dio` em `domain/`.
- ❌ `BuildContext` em Bloc.
- ❌ Exception atravessando camada — só `Either<Failure, T>`.
- ❌ `setState` em lógica de negócio — só UI local trivial (toggle password, selected tab).
- ❌ Token em `SharedPreferences`.
- ❌ `print()` em prod — `logger` package.
- ❌ `Colors.xxx` / `Color(0xff...)` hardcoded — `AppColors.*` ou `context.colorScheme`.
- ❌ Instanciação manual (`new AuthBloc(LoginUseCase(...))`) — sempre `sl<T>()`.

---

## 2. Camadas por feature

### 2.1 Domain

```
features/<feature>/domain/
├── entities/
│   └── <entity>_entity.dart          # Equatable, imutável, pure Dart (UserEntity, AssemblyEntity, AgendaItemEntity, ...)
├── repositories/
│   └── <feature>_repository.dart     # abstract class (contrato)
└── usecases/
    └── <verb>_<noun>_usecase.dart    # extends UseCase<T, Params>
```

**Regra**: `UseCase.call(Params) → Future<Either<Failure, T>>`. Nunca lança exceção. `Failure` cruza a fronteira, nunca `Exception`.

**Exemplo real** — `features/auth/domain/usecases/login_usecase.dart`:

```dart
@injectable
class LoginUseCase extends UseCase<UserEntity, NoParams> {
  final AuthRepository _repository;
  LoginUseCase(this._repository);

  @override
  Future<Either<Failure, UserEntity>> call(NoParams params) {
    return _repository.login();
  }
}
```

### 2.2 Data

```
features/<feature>/data/
├── models/
│   └── <entity>_model.dart              # extends <Entity> + fromJson/toJson/fromEntity
├── datasources/
│   ├── <feature>_remote_datasource.dart # abstract + impl (Dio / oidc)
│   └── <feature>_local_datasource.dart  # abstract + impl (SharedPrefs / SecureStorage / Hive)
└── repositories/
    └── <feature>_repository_impl.dart   # ÚNICO lugar com try/catch — converte Exception → Failure
```

**Regra**: `Model extends Entity` (polymorphism real — o Model retornado cabe onde Entity é esperado). Repository impl é o **único** local com `try/catch` — captura `ServerException` / `CacheException` / `NetworkException` e retorna `Left(Failure)`.

**Exemplo real** — `features/auth/data/repositories/auth_repository_impl.dart`:

```dart
@LazySingleton(as: AuthRepository)
class AuthRepositoryImpl implements AuthRepository {
  final AuthRemoteDatasource _remote;
  final AuthLocalDatasource _local;
  final NetworkInfo _networkInfo;

  AuthRepositoryImpl(this._remote, this._local, this._networkInfo);

  @override
  Future<Either<Failure, UserEntity>> login() async {
    if (!await _networkInfo.isConnected) return const Left(NetworkFailure());
    try {
      final result = await _remote.login();
      await _local.cacheUser(result.user);
      await _local.saveToken(result.token);
      return Right(result.user);
    } on ServerException catch (e) {
      return Left(ServerFailure(message: e.message ?? 'Erro na autenticação.', code: e.statusCode));
    }
  }
  // ...
}
```

O datasource remoto (`AuthRemoteDatasourceImpl`) usa **Dart 3 records** `({UserModel user, String token})` no retorno — lightweight wrapper sem criar classe dedicada.

### 2.3 Presentation

```
features/<feature>/presentation/
├── bloc/
│   ├── <feature>_bloc.dart           # Bloc principal
│   ├── <feature>_event.dart          # part of bloc
│   └── <feature>_state.dart          # part of bloc
├── pages/
│   └── <page>_page.dart              # Scaffold + BlocProvider local
└── widgets/
    └── <widget>.dart                 # UI reusável dentro da feature
```

**Regra**: Bloc emite estados. Página escuta (`BlocBuilder` / `BlocListener` / `BlocConsumer`) e renderiza. `BlocProvider` local por página (não global) para dispose automático em pop. Exceções: Blocs genuinamente globais (ex: futuro `AuthBloc` escutando sessão inteira) vão em `App` via `MultiBlocProvider`.

---

## 3. DI com GetIt + Injectable

### 3.1 Bootstrap real — `lib/main.dart`

```dart
Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();

  // SharedPreferences é async — registra manualmente ANTES de configureDependencies
  final prefs = await SharedPreferences.getInstance();
  sl.registerSingleton<SharedPreferences>(prefs);

  await configureDependencies();
  runApp(const App());
}
```

### 3.2 `injection.dart` — contrato com `build_runner`

```dart
final GetIt sl = GetIt.instance;

@InjectableInit(
  initializerName: 'init',
  preferRelativeImports: true,
  asExtension: true,
)
Future<void> configureDependencies() async => sl.init();

@module
abstract class RegisterModule {
  @lazySingleton
  Connectivity get connectivity => Connectivity();

  @lazySingleton
  Dio get dio => Dio(
        BaseOptions(
          baseUrl: AppConstants.baseUrl,
          connectTimeout: AppConstants.connectTimeout,
          receiveTimeout: AppConstants.receiveTimeout,
          headers: {'Content-Type': 'application/json'},
        ),
      );

  @lazySingleton
  SharedPreferences get sharedPreferences => throw UnimplementedError(
        'SharedPreferences deve ser registrado manualmente no main.dart...',
      );
}
```

> O `throw UnimplementedError` no getter de `SharedPreferences` é **intencional** (guarda contra DI ordem errada) — **não consertar**.

### 3.3 Anotações

| Anotação | Para | Vida |
|---|---|---|
| `@lazySingleton` | Repositories, Datasources, Dio, NetworkInfo, FlutterSecureStorage | Uma instância compartilhada, criada na 1ª resolução |
| `@injectable` | Blocs, UseCases | Instância fresca por resolução; Bloc scope à página |
| `@module` | Third-party não-anotável (Dio, Connectivity, SharedPreferences, FlutterSecureStorage) | Config no módulo |

### 3.4 Regra crítica

`lib/app/di/injection.config.dart` é **gerado**. **Nunca editar à mão**. Rerun ao adicionar/alterar anotação:

```bash
dart run build_runner build --delete-conflicting-outputs
# ou durante sessão de DI:
dart run build_runner watch --delete-conflicting-outputs
```

Troubleshooting comum:
- `MissingPluginException` no startup → `injection.config.dart` stale.
- `"X not registered in GetIt"` → anotação esqueceu ou `build_runner` não rodou.

---

## 4. Router — GoRouter top-level

Arquivo atual: `lib/app/router/app_router.dart`. Constantes em `app_routes.dart`. **Nunca** string literal em call site — sempre `AppRoutes.xxx(...)`.

### 4.1 Trecho real (assembleias já integradas)

```dart
final GoRouter appRouter = GoRouter(
  initialLocation: AppRoutes.home,
  debugLogDiagnostics: true,
  routes: [
    GoRoute(path: AppRoutes.home,  name: 'home',  builder: (c, s) => const HomePage()),
    GoRoute(path: AppRoutes.login, name: 'login', builder: (c, s) => const LoginPage()),
    GoRoute(
      path: '/condominiums/:condominiumId/assemblies',
      name: 'assembly-list',
      builder: (c, s) => AssemblyListPage(condominiumId: s.pathParameters['condominiumId']!),
    ),
    GoRoute(
      path: '/condominiums/:condominiumId/assemblies/:assemblyId/vote/:agendaItemId',
      name: 'assembly-vote',
      builder: (c, s) => AssemblyVotePage(
        condominiumId: s.pathParameters['condominiumId']!,
        assemblyId: s.pathParameters['assemblyId']!,
        agendaItemId: s.pathParameters['agendaItemId']!,
        agendaItemTitle: s.uri.queryParameters['title'] ?? '',
      ),
    ),
    // ... +3 rotas assembleia (detail, science, minutes)
  ],
  errorBuilder: (c, s) => /* 404 Page */,
);
```

### 4.2 Alvo M1 — estrutura completa

```dart
final appRouter = GoRouter(
  initialLocation: AppRoutes.splash,
  routes: [
    GoRoute(path: AppRoutes.splash, builder: ...),
    GoRoute(path: AppRoutes.login, builder: ...),
    GoRoute(path: AppRoutes.deviceMismatch, builder: ...),
    ShellRoute(  // shell com bottom nav 4 abas
      builder: (c, s, child) => MainShell(child: child),
      routes: [
        GoRoute(path: '/home', builder: ...),
        GoRoute(path: '/home/timeline', builder: ...),
        GoRoute(path: '/home/plano-diretor', builder: ...),
        GoRoute(path: '/home/eventos', builder: ...),
        GoRoute(path: '/account', builder: ...),
        // ...
      ],
    ),
    // M3
    GoRoute(path: '/home/assembleias/:id/live', builder: (c, s) => AssemblyLivePage(...)),
  ],
  redirect: (c, s) {
    // auth guard + avaliação obrigatória gate
  },
  errorBuilder: (c, s) => const NotFoundPage(),
);
```

### 4.3 Deep linking

- **iOS universal links** (M1): `https://app.mastersindico.com.br/*` → `apple-app-site-association` servido pelo [[../web/README|web]] em `.well-known/apple-app-site-association`.
- **Android app links** (M1): intent-filter `android:autoVerify="true"` em `AndroidManifest.xml` + `https://app.mastersindico.com.br/.well-known/assetlinks.json`.
- **Custom scheme `com.mastersindico://`**: usado **apenas** para OIDC callback (`/auth`) e casos não-web-compat. Já registrado em `Info.plist` `CFBundleURLSchemes` + `AndroidManifest.xml` intent-filter no código atual.
- **Allowlist**: `app_links` package entrega deep link; handler valida contra lista de rotas conhecidas; unknown → tela 404.
- **Auth guard**: se deep link requer auth e user deslogado → redirect `/login?redirect=<original>` preservado; pós-login retorna ao destino.

---

## 5. State management — flutter_bloc (D-048 fechado)

[[../../STATE#D-048|D-048]] decidiu: **manter `flutter_bloc`** (paridade legado + audit trail event-driven + battle-tested enterprise). **Adotar obrigatoriamente** desde M1:

- **`bloc_concurrency`** com transformer `droppable()` em botões que podem ser apertados rápido (votar, enviar Connect Me, pagar) — evita double-tap → double-submit.
- **`hydrated_bloc`** para estado persistido offline (memberships, timeline cached, plano diretor lido). Hidrata Bloc ao reabrir app.

### 5.1 Bloc canônico — `AssemblyBloc` (real)

```dart
@injectable
class AssemblyBloc extends Bloc<AssemblyEvent, AssemblyState> {
  final GetAssembliesUseCase _getAssemblies;
  final GetAssemblyUseCase _getAssembly;
  final GetAgendaItemsUseCase _getAgendaItems;
  final RegisterScienceUseCase _registerScience;
  final CastVoteUseCase _castVote;
  final CreateProxyUseCase _createProxy;
  final GetMinutesUseCase _getMinutes;

  AssemblyBloc({required ...}) : super(const AssemblyInitial()) {
    on<AssemblyListRequested>(_onListRequested);
    on<AssemblyDetailRequested>(_onDetailRequested);
    // ... 7 handlers
  }

  Future<void> _onVoteCast(AssemblyVoteCast e, Emitter emit) async {
    emit(const AssemblyLoading());
    final result = await _castVote(CastVoteParams(
      condominiumId: e.condominiumId,
      assemblyId: e.assemblyId,
      agendaItemId: e.agendaItemId,
      voteValue: e.voteValue,
    ));
    result.fold(
      (failure) => emit(AssemblyFailureState(failure.message)),
      (_) => emit(const AssemblyVoteSuccess()),
    );
  }
}
```

### 5.2 `part of` — co-localização

Event e state files usam `part of '<feature>_bloc.dart'`. Co-localizados:

```dart
// assembly_bloc.dart
part 'assembly_event.dart';
part 'assembly_state.dart';

// assembly_event.dart
part of 'assembly_bloc.dart';
```

### 5.3 Migração para sealed classes + freezed (D-049)

A partir de M1, novos Blocs adotam `sealed class` (Dart 3) + `freezed`:

```dart
@freezed
sealed class TimelineState with _$TimelineState {
  const factory TimelineState.initial() = TimelineInitial;
  const factory TimelineState.loading() = TimelineLoading;
  const factory TimelineState.loaded(List<TimelineEntry> entries) = TimelineLoaded;
  const factory TimelineState.error(String message) = TimelineError;
}
```

Switch exhaustivity checada pelo compilador, sem precisar `default:`.

---

## 6. Erros — `Either<Failure, T>` (CODING_MANUAL §4.1)

### 6.1 Hierarquia real — `core/error/failures.dart`

```dart
abstract class Failure extends Equatable {
  final String message;
  final int? code;
  const Failure({required this.message, this.code});

  @override List<Object?> get props => [message, code];
}

class ServerFailure     extends Failure { const ServerFailure({super.message = 'Erro no servidor.', super.code}); }
class CacheFailure      extends Failure { const CacheFailure({super.message = 'Erro ao acessar dados locais.', super.code}); }
class NetworkFailure    extends Failure { const NetworkFailure({super.message = 'Sem conexão com a internet.', super.code}); }
class AuthFailure       extends Failure { const AuthFailure({super.message = 'Não autorizado.', super.code}); }
class UnexpectedFailure extends Failure { const UnexpectedFailure({super.message = 'Erro inesperado.', super.code}); }
```

### 6.2 Alvo M1 — adicionar

- `ForbiddenFailure` — 403 ABAC.
- `NewDeviceFailure` — 403 `NEW_DEVICE_DETECTED` → redirect APP-SYS-004.
- `RateLimitFailure` — 429 → modal "Muitas solicitações, tente em Xs".
- `ValidationFailure(Map<String, String> fields)` — 400 field-level.

### 6.3 Exceções — `core/error/exceptions.dart`

```dart
class ServerException implements Exception {
  final String? message;
  final int? statusCode;
  const ServerException({this.message, this.statusCode});
}
class CacheException     implements Exception { final String? message; const CacheException({this.message}); }
class NetworkException   implements Exception { final String? message; const NetworkException({this.message = 'Sem conexão com a internet.'}); }
class UnexpectedException implements Exception { final String? message; final Object? error; const UnexpectedException({this.message, this.error}); }
```

Lançadas **apenas** na camada data (datasources) — capturadas **apenas** no repository impl.

---

## 7. Networking — Dio

### 7.1 Configuração base

`RegisterModule` cria um Dio `@lazySingleton` com base URL + timeouts + `Content-Type: application/json`.

### 7.2 Interceptors (ordem alvo M1)

```
RequestQueue (CancelToken holder)
    ↓
AuthInterceptor       → Authorization: Bearer <access_token>   (de flutter_secure_storage)
    ↓
DeviceInterceptor     → X-Device-Fingerprint, X-Device-Integrity
    ↓
RefreshInterceptor    → on 401, tenta refresh; falha = logout global
    ↓
ErrorInterceptor      → DioException → ServerException (com code + message do backend response.data.error)
    ↓
LogInterceptor (dev)  → logs sem PII
```

O `AssemblyRemoteDatasourceImpl` atual já tem `_mapDioException` que lê `response.data['error']['message']` e monta `ServerException(message, statusCode)` — padrão a replicar em todos os datasources.

### 7.3 Certificate pinning (prod)

Via `IOHttpClientAdapter` custom com `badCertificateCallback` + lista de SHA-256 dos certs (atual + backup, rotação semestral). Ver [[security#2-certificate-pinning|security §2]].

---

## 8. Offline-first seletivo (BR-MOB-OFF / D-041 aberto)

**Não é offline-pleno** — é seletivo. Decisão final Hive vs sqflite vs drift pendente ([[../../STATE|STATE]] D-041); intenção canônica: **Hive** (performático, sem SQL, box por tipo).

### 8.1 Reads cacheados

| Box | TTL | Conteúdo |
|---|---|---|
| `user` | ∞ (até logout) | Snapshot `AuthUser` |
| `memberships` | 1h | Memberships ativas — refresh background |
| `timeline:{condId}` | 30min | Últimos 30 itens |
| `events:{condId}` | 1h | Eventos agendados próximos 30 dias |
| `plano-diretor:{condId}` | 1h | Plano diretor leitura |

Boxes com dados sensíveis usam `HiveAesCipher` com chave em `flutter_secure_storage` (ver [[security#12-storage-encryption|security §12]]).

### 8.2 Writes online-only

Criar evento, votar, enviar Connect Me, registrar ciência crítica, outorgar procuração, upload vídeo — **exigem conexão**. Sem conexão → modal bloqueante.

### 8.3 Sync queue

Ações tolerantes (registrar ciência simples, confirmar participação, mark read) enfileiradas em `sync_queue` Hive box; worker `SyncBloc` dispara quando `NetworkInfo.isConnected` vira true.

### 8.4 Assembleia live = online obrigatório

Sessão live (APP-ASM-006) exige conexão estável. Se perder WS: banner "Reconectando..." com timeout 30s → se não recupera, modal erro com [Tentar novamente] / [Sair].

---

## 9. Push notifications (FCM — M1)

### 9.1 Registrar

Boot: `FirebaseMessaging.requestPermission` (iOS prompt) + `getToken()` → enviar backend `POST /api/v1/devices/register-push-token` + subscribe tópicos (`user:<id>`, `tenant:<condId>`, `tenant:<condId>:sindicos` se role ativa).

### 9.2 Tópicos

- Por user: `user:<user_id>` (notificações pessoais).
- Por tenant: `tenant:<condId>` (broadcast — subscribe em cada membership ativa; unsubscribe em remoção).
- Por role no tenant: `tenant:<condId>:sindicos` — broadcast moderação.

### 9.3 Categorias e channels (BR-MOB-PUSH-003)

6 categorias com channel Android + category iOS distinto. Ver [[requirements#2-push-notifications-br-mob-push]].

### 9.4 Handlers

- **Foreground**: `onMessage` → in-app toast/snackbar + update Bloc.
- **Background**: system notification → tap → deep-link via `app_links` + GoRouter.
- **Terminated**: FCM abre app na rota deep-linked.

### 9.5 PII restrito (BR-MOB-PUSH-005)

Push **nunca** contém CPF/CNPJ/dados financeiros plain. Título + preview curto; full content ao abrir app.

---

## 10. Video player (Mux HLS signed — M2)

Wrapper em `shared/widgets/mux_player.dart`:

```dart
class MuxVideoPlayer extends StatefulWidget {
  final String playbackId;
  final String signedToken;    // JWT gerado pelo backend com TTL ≤ 1h
  final bool showLockBadge;    // "🔒 Travado até DD/MM" (I-031)
  // ...
}
```

Backend emite signed URL/token via `GET /api/v1/videos/:id/playback` (expira 1h default). Frontend **nunca** hardcoda Mux secret (só o backend assina).

---

## 11. Upload flow (vídeo-currículo, foto execução — M2)

1. App captura (`image_picker` / `camera`).
2. App chama `POST /api/v1/videos/create-upload` → recebe `presigned_url` Mux direct.
3. App PUT direto pro Mux com progress callback + cancel.
4. Mux processa → webhook backend → status via polling `GET /api/v1/videos/:id` ou FCM push.
5. App atualiza status quando `approved` (desbloqueia player) ou `rejected` (mostra motivo).

---

## 12. Theming

Arquivos reais: `lib/core/theme/app_colors.dart` · `app_text_styles.dart` · `app_theme.dart`.

### 12.1 Tokens reais

```dart
abstract class AppColors {
  static const Color primary       = Color(0xFF6C63FF);  // roxo brand
  static const Color primaryLight  = Color(0xFF9D97FF);
  static const Color primaryDark   = Color(0xFF3D35CC);
  static const Color secondary     = Color(0xFF03DAC6);  // teal
  static const Color success       = Color(0xFF4CAF50);
  static const Color warning       = Color(0xFFFFC107);
  static const Color error         = Color(0xFFCF6679);
  static const Color info          = Color(0xFF2196F3);
  static const Color background    = Color(0xFFF8F9FA);
  static const Color surface       = Color(0xFFFFFFFF);
  static const Color onSurface     = Color(0xFF1C1B1F);
  static const Color outline       = Color(0xFFCAC4D0);
  static const Color backgroundDark = Color(0xFF121212);
  static const Color surfaceDark    = Color(0xFF1E1E1E);
  // ...
}
```

### 12.2 Alvo — reconciliar com Manual IV (ouro/navy)

As cores do [[../web/design-system|web design system]] (ouro `#C69C3F` primary + navy `#071A33` primary) **divergem** do mobile real. **Dívida**: reconciliar tokens mobile ao Manual IV do cliente antes de M1 ship — ou justificar paleta distinta em decisão registrada. Roxo `#6C63FF` é default do Flutter create, portanto provavelmente **placeholder**.

### 12.3 ThemeData

`useMaterial3: true`; `ColorScheme.fromSeed(seedColor: AppColors.primary)`; `ThemeMode.system` default em `App` widget; text theme derivado de MD3 type scale (`display/headline/title/body/label` × `large/medium/small`).

Component overrides no `app_theme.dart`: `appBarTheme`, `cardTheme`, `inputDecorationTheme`, `elevatedButtonTheme`, `textButtonTheme`, `dividerTheme`, `snackBarTheme`.

---

## 13. Observability

- **Sentry Flutter** (Sprint 7): DSN via `--dart-define=SENTRY_DSN`; `tracesSampleRate: 0.2` prod; `beforeSend` scrubbing de `Authorization`, `X-Device-Fingerprint`, `Cookie`, `email`, `cpf`, `cnpj`, `token`.
- **Symbol upload** em CI: `flutter build --obfuscate --split-debug-info=build/symbols` + `sentry-cli upload-dif build/symbols` para stack traces legíveis.
- **`logger` package** (já em pubspec `^2.5.0`) — nunca `print()`. Wrapper PII masking:
  ```dart
  Logger().i('User loaded: ${maskCpf(user.cpf)}');
  ```

---

## 14. Feature flags (M2+)

Simples: backend `/api/v1/features/:feature/enabled` retorna bool por user/tenant. Cache 5min. Backlog BE-###. Admin (D-058) consome a mesma API — role privilegiada só adiciona bits na matriz de permissão.

---

## 15. Build & deploy

### 15.1 Comandos canônicos

```bash
# Dev local (device conectado)
fvm flutter run
fvm flutter run -d <device_id>

# Android
fvm flutter build apk --release --obfuscate --split-debug-info=build/symbols \
  --dart-define=API_BASE_URL=https://api.mastersindico.com.br \
  --dart-define=SENTRY_DSN=... \
  --dart-define=ZITADEL_CLIENT_ID=... \
  --dart-define=ZITADEL_ISSUER=https://auth.mastersindico.com.br

fvm flutter build appbundle --release --obfuscate --split-debug-info=build/symbols \
  --dart-define=...

# iOS
fvm flutter build ipa --release --obfuscate --split-debug-info=build/symbols \
  --dart-define=...
```

### 15.2 Flavors (Sprint 6)

- `dev` → `API_BASE_URL=http://10.0.2.2:8000` (Android emu) / `http://localhost:8000` (iOS sim); banner "DEV" topo.
- `staging` → `https://staging.api.mastersindico.com.br`; banner "STG".
- `prod` → `https://api.mastersindico.com.br`; sem banner.

### 15.3 Gates (flutter analyze + test)

```bash
fvm flutter analyze                     # zero warning
fvm flutter test --coverage             # thresholds por camada
fvm dart run build_runner build --delete-conflicting-outputs
fvm flutter build apk --debug           # Android sanity
fvm flutter build ios --no-codesign     # iOS sanity
```

### 15.4 Distribuição

- **TestFlight (iOS)**: upload via Xcode ou `fastlane` (Sprint 8.9).
- **Google Play Internal (Android)**: upload via Play Console ou `fastlane`.
- **Firebase App Distribution** (opcional) para beta interna cross-team.

---

## 16. Admin = role privilegiada (D-058) em mobile

Admin **não** tem app separada. Em mobile, quando um usuário tem a role `admin` (flag ABAC retornada pelo backend no `/auth/me`), a mesma app mostra:

- Menu adicional em `/account` com itens gated: "Painel admin" (se M2+), "Relatórios globais", "Usuários" — rotas `/admin/*` só aparecem se permissão concede.
- Botões de ação privilegiada in-line (ex: "Reverter lock 90d" num vídeo) só renderizam se matriz concede.
- **Bypass Lock 90d** (D-054) exige **4-eyes policy** — mobile pode solicitar (admin A) mas aprovação do admin B pode vir via mobile ou web; UI mostra status "aguardando aprovação".

Em M1-M3 a expectativa é que admin opere predominantemente via [[../web/README|web]] (UX desktop superior para operações em lote, relatórios, auditoria). Mobile tem o **mesmo código** mas com telas gated por permissão.

---

## 17. Links

- [[README]]
- [[patterns]]
- [[ui-catalog]]
- [[security]]
- [[../backend/README]]
- [[../web/design-system]]
- [[../../02-architecture/c4-containers]]
- [[../../STATE#D-048|D-048 Bloc + extensões]]
- [[../../STATE#D-049|D-049 freezed]]
