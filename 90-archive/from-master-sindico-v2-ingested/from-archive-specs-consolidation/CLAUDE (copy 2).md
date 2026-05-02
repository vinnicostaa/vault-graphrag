# CLAUDE.md

Guia para Claude Code operando no sub-projeto **`app/`** do Master Síndico (Flutter).

- **GitHub**: https://github.com/mastersindico/app
- **Stack**: Flutter/Dart (SDK `^3.11.4`) · Clean Architecture + Feature First · `flutter_bloc` · `get_it` + `injectable` · `go_router` · `dio` · `dartz` · `equatable` · `mocktail` + `bloc_test` · `flutter_secure_storage` · `oidc` (Zitadel)
- **Papel no ecossistema**: cliente mobile que consome a API Go em `../backend/` via Zitadel PKCE + `Authorization: Bearer`.

O `../CLAUDE.md` cobre contexto cross-repo (produto, personas, money em centavos, UUIDv7, Zitadel). **`ARCHITECTURE.md` e `CODING_MANUAL.md` deste diretório são autoritativos** sobre invariantes do código mobile — leia-os antes de trabalho não-trivial.

## Nota sobre pasta

A pasta local foi **renomeada de `mobile/` para `app/`** em 2026-04-21 para casar com o nome do repo GitHub (`mastersindico/app`). Qualquer doc antigo que mencione `mobile/` como folder local está desatualizado — **sempre** `app/`.

## Comandos essenciais

```bash
flutter pub get                                                  # deps
flutter run                                                      # device/emulador conectado
flutter test                                                     # todos os testes
flutter test test/features/auth/presentation/bloc/auth_bloc_test.dart    # arquivo único
flutter test --name "should return UserEntity on successful login"       # por nome
flutter test --coverage && genhtml coverage/lcov.info -o coverage/html   # com coverage
flutter analyze                                                  # static analysis (analysis_options.yaml → flutter_lints)

# Code generation (OBRIGATÓRIO ao adicionar/alterar @injectable/@lazySingleton/@module)
dart run build_runner build --delete-conflicting-outputs
dart run build_runner watch --delete-conflicting-outputs         # deixa regenerando durante sessão DI

# Formatação
dart format lib/ test/
```

`lib/app/di/injection.config.dart` é **gerado** — nunca editar à mão. Se DI falhar com `MissingPluginException` ou "not registered" no startup, a causa mais comum é `injection.config.dart` stale — rode `build_runner` de novo.

Dart SDK constraint: `^3.11.4` em `pubspec.yaml`. Menções a `^3.10.4` em docs antigos estão stale.

## Fontes de verdade (cross-repo)

- **Product/requirements** (Reqs, decisões, personas, planos, matriz funcional): `../backend/.kiro/specs/master-sindico/requirements/` — ler **antes** de implementar feature
- **Design de contratos** (API endpoints, data model, fluxos auth): `../backend/.kiro/specs/master-sindico/design/` — especialmente `api-design.md` (contrato HTTP) e `authentication-flow.md` (PKCE)
- **Antipatterns a evitar**: `../backend/.kiro/specs/master-sindico/reference/antipatterns-to-avoid.md` — aplicar adaptações Flutter (ex: A3 `if/else hell` vira "switch de N widgets" → Strategy)
- **Deploy / infra**: `../backend/.kiro/specs/master-sindico/design/deploy-config.md`
- **Bloqueadores vivos**: `../backend/.kiro/STATE.md`
- **Convenções técnicas transversais**: `../backend/.kiro/steering/` (aplicabilidade abaixo)

### Steering do backend — aplicabilidade ao app

| Steering | Aplicável? | Como aplicar em Flutter |
|---|---|---|
| `sdd-workflow.md` | ✅ sim | Ciclo 5 fases (Discuss → Plan → Execute → Verify → Ship) — ver seção SDD abaixo |
| `mcp-integration.md` | ✅ sim | Context7 para docs oficiais (Flutter, flutter_bloc, go_router, dio, oidc, dartz) |
| `plugins.md` | ✅ sim | `figma` (design → código), `feature-dev`, `superpowers`, `security-guidance`, `commit-commands` |
| `security.md` | ✅ parcial | "Never trust the frontend" + regras de secrets + LGPD; seção específica mobile abaixo (certificate pinning, jailbreak detection) |
| `testing.md` | ⚠️ adaptado | TDD obrigatório; thresholds mobile abaixo (bloc/usecases 90%, domain 95%, widgets 75%) |
| `do-dont.md` | ⚠️ parcial | Várias regras transversais; divergências documentadas aqui |
| `code-calisthenics.md` | ⚠️ adaptado | 9 regras adaptadas a Dart — regras específicas abaixo |
| `transaction-patterns.md` | ❌ não | Transações são problema do backend |
| `go-patterns.md`, `database.md` | ❌ não | Específicos de Go |

## Architecture — o que importa antes de editar

O walkthrough detalhado está em `ARCHITECTURE.md` (por arquivo) e `CODING_MANUAL.md` (regras e proibições). Invariantes carregadoras:

- **Layout é Feature First + Clean Architecture**: `lib/app/` (bootstrap, DI, router), `lib/core/` (shared infra), `lib/features/<nome>/{domain,data,presentation}/`. Testes em `test/` espelham `lib/` exatamente.
- **Dependency Rule (inviolável)**: `presentation → domain ← data`. `domain/` não importa de `data/` ou `presentation/`. Features não importam `data/` ou `presentation/` de outras features — compartilhar **apenas** via `core/` ou `domain/entities/` de outra feature.
- **Erros nunca propagam como exceção além da camada data**: `data/datasources/` lançam `ServerException` / `CacheException` tipadas. **`RepositoryImpl` é o único lugar** com `try/catch` — converte exceção em `Failure` e retorna `Either<Failure, T>` (dartz). UseCases, Blocs e widgets só veem `Either`.
- **`SharedPreferences` registrado manualmente em `main.dart` antes de `configureDependencies()`** — é async, não cabe no getter do `@module` (que é sync). O `throw UnimplementedError(...)` no getter é **intencional** para forçar o registro manual. **Não "consertar" esse throw.**
- **DI annotation é semântica, não estilo**:
  - `@lazySingleton` → Repositories, Datasources, `Dio`, `NetworkInfo` (instância compartilhada)
  - `@injectable` → Blocs e UseCases (instância fresca por resolução; Blocs scopados à página via `BlocProvider(create: (_) => sl<AuthBloc>())` — disposam no pop)
  - `@module` → third-party que não aceita anotação (`Dio`, `Connectivity`, `SharedPreferences`)
- **Bloc plumbing**: `auth_event.dart` e `auth_state.dart` usam `part of 'auth_bloc.dart'`. Event/state são co-localizados com o Bloc. Sempre.
- **Routing**: `appRouter` é `GoRouter` top-level (sem `BuildContext` para construir). Paths em `lib/app/router/app_routes.dart` como constantes — **nunca** string literal em call site.
- **Theming**: nunca hardcode cor ou text style. Usar `AppColors.*`, `AppTextStyles.*`, ou extensões `context.colorScheme` / `context.textTheme` em `lib/core/utils/extensions/`.

## Auth Zitadel (estado atual e alvo)

**Estado atual** (scaffold): `AuthRemoteDatasource` / `AuthLocalDatasource` são shape-template; token em `SharedPreferences` sob `AppConstants.tokenKey`. `lib/core/utils/constants.dart` tem placeholders:
- `baseUrl = 'https://api.example.com'` → **deve virar** `https://api.mastersindico.com.br` (prod) ou `http://localhost:8000` (dev)
- `zitadelIssuer = 'https://YOUR_ZITADEL_DOMAIN'` → preencher com instância Zitadel dev/prod
- `zitadelClientId` e redirect URI (`com.mastersindico:/auth`) **são reais**

**Alvo** (Sprint 1.14 no backend/tasks.md):

```
1. Login → oidc (PKCE Authorization Code Flow) com Zitadel
2. AccessToken + RefreshToken recebidos
3. Ambos armazenados em flutter_secure_storage (NÃO SharedPreferences)
4. Dio interceptor injeta `Authorization: Bearer <access_token>` em toda request a api.mastersindico.com.br
5. Refresh automático antes de expirar
6. Logout → end_session Zitadel + secure_storage clear + navegação para login
```

Contrato com backend (`../backend/.kiro/specs/master-sindico/requirements/identity-mirror-pattern.md`):
- Backend valida JWT via introspection Zitadel
- 1 dispositivo ativo por conta — device fingerprint enviado como header `X-Device-Fingerprint`
- Backend pode responder `403 NEW_DEVICE_DETECTED` → app navega para tela "sessão transferida"

**Regra dura**: token nunca em `SharedPreferences`, nunca em log, nunca em logs da Crashlytics/Sentry. `flutter_secure_storage` em iOS Keychain + Android Keystore.

## Workflow SDD adaptado ao Flutter

Ciclo de 5 fases (do `../backend/.kiro/steering/sdd-workflow.md`) adaptado:

```
Discuss     →  Plan           →  Execute     →  Verify                  →  Ship
requisito      + <verify>         TDD              flutter analyze +         PR +
+ API           + arquivos         bloc_test        flutter test --coverage   Release Notes
+ Figma         + Context7         Arrange/Act/     + build iOS + Android     + TestFlight/Play
                                   Assert                                       internal
```

### Discuss (antes de qualquer código)

1. Ler requisito em `../backend/.kiro/specs/master-sindico/requirements/*.md`
2. Confirmar endpoint no backend em `../backend/internal/modules/{ctx}/infrastructure/http/routes/` — se não existe, **parar** e implementar backend primeiro
3. Ler frame Figma via plugin `figma` — `/implement-design <frame>` (URL: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX)
4. Ler feature existente próxima (ex: `auth/`) como template
5. Checar `../backend/.kiro/STATE.md`

### Plan

```xml
<plan feature="commercial/connect-me — morador envia para síndico">
  <files>
    lib/features/connect_me/domain/entities/connect_me_request_entity.dart   [novo]
    lib/features/connect_me/domain/repositories/connect_me_repository.dart   [novo]
    lib/features/connect_me/domain/usecases/submit_request_usecase.dart      [novo]
    lib/features/connect_me/data/models/connect_me_request_model.dart        [novo]
    lib/features/connect_me/data/datasources/connect_me_remote_datasource.dart [novo]
    lib/features/connect_me/data/repositories/connect_me_repository_impl.dart [novo]
    lib/features/connect_me/presentation/bloc/connect_me_bloc.dart            [novo]
    lib/features/connect_me/presentation/bloc/connect_me_event.dart           [novo]
    lib/features/connect_me/presentation/bloc/connect_me_state.dart           [novo]
    lib/features/connect_me/presentation/pages/connect_me_form_page.dart     [novo]
    lib/app/router/app_router.dart                                            [editar — add rota]
    lib/app/router/app_routes.dart                                            [editar — add constante]
  </files>

  <docs-oficiais-via-context7>
    - flutter/flutter — topic: "Form validation with StatefulWidget"
    - felangel/bloc — topic: "bloc_test blocTest with seed/act/expect"
    - rrousselGit/dartz — topic: "Either fold chain"
  </docs-oficiais-via-context7>

  <figma>
    Frame "Morador / Connect Me / Síndico" — validar tokens, estados (idle, loading, success, error), LGPD disclaimer
  </figma>

  <verify>
    - ConnectMeRequestEntity equality via Equatable
    - SubmitRequestUseCase chama repo com LoginParams corretos
    - ConnectMeBloc: initial → ConnectMeInitial; add(Submit) + success → [Loading, Success]; add(Submit) + failure → [Loading, Failure(message)]
    - Data → Model fromJson/toJson roundtrip
    - RepositoryImpl: sem conexão → Left(NetworkFailure); 200 → Right; 500 → Left(ServerFailure)
    - Page: campos obrigatórios bloqueiam submit; disclaimer LGPD checkbox required; loading desabilita submit
    - flutter analyze sem warning
    - Coverage em domain/ ≥ 95%, bloc/ ≥ 90%, repository_impl ≥ 85%
  </verify>

  <api-integration>
    POST /api/v1/connect-me
    Body: { targetId, description, urgency, consentVersion }
    Response: { id, status, autoCloseAt }
    Headers: Authorization: Bearer <token>, X-Device-Fingerprint: <fp>
  </api-integration>
</plan>
```

### Execute — TDD

1. **RED**: `test/features/connect_me/domain/usecases/submit_request_usecase_test.dart` — teste falha (classe não existe)
2. **GREEN**: implementa classes mínimas para teste passar
3. **REFACTOR**: extrai helpers, elimina duplicação, simplifica

Ordem por feature:
1. Entity (domain) + teste de equatable
2. Repository interface (domain)
3. UseCase + teste com mocktail
4. Model (data) + teste fromJson/toJson
5. Datasource interface + impl
6. RepositoryImpl + teste de success/failure paths
7. Bloc + event + state + teste com bloc_test
8. Page + widget test
9. Wire-up DI (roda `build_runner`)
10. Rota em `app_router.dart`

### Verify

```bash
flutter analyze                                       # sem warning
flutter test                                          # todos verdes
flutter test --coverage
# opcional: genhtml coverage/lcov.info -o coverage/html && open coverage/html/index.html
dart run build_runner build --delete-conflicting-outputs
# Assert coverage manualmente ou via script — ver thresholds abaixo
flutter build apk --debug                             # Android dev build limpo
flutter build ios --no-codesign                        # iOS dev build limpo
```

**Gates duros**:
- `flutter analyze` zero warning
- `flutter test` 100% verde (com `-race`? não se aplica a Dart)
- Coverage por camada (abaixo)
- Build Android + iOS sem erro

### Ship

```bash
# Commits pt-BR via plugin commit-commands
/commit
# ex: "connect-me: Add flow morador envia para síndico"

# PR via plugin github
/pr

# Release candidate:
flutter build apk --release          # Android → upload Play Console Internal
flutter build ipa --release          # iOS → upload TestFlight
```

PR body obrigatório: seção `Release Notes:`.

## TDD & Testing — thresholds + padrões

Stack: `flutter_test` (built-in) + `mocktail` (sem codegen) + `bloc_test` (stream assertions).

### Coverage thresholds por camada

| Camada | Alvo | Motivo |
|---|---|---|
| `lib/features/*/domain/` | **95%** | Regras de domínio — testar todos os paths |
| `lib/features/*/presentation/bloc/` | **90%** | Bloc é máquina de estados — cobrir transições |
| `lib/features/*/data/repositories/` | **85%** | Único lugar com try/catch — success + failure paths |
| `lib/features/*/data/datasources/` | **80%** | Mock Dio/SharedPrefs, testar path de erro |
| `lib/features/*/presentation/pages/` | **70%** | Widget tests para fluxos críticos |
| `lib/features/*/presentation/widgets/` | **80%** | Golden tests para componentes reusáveis |
| `lib/core/` | **90%** | Infra compartilhada |
| `lib/app/` | **60%** | Bootstrap/DI — teste manual é ok |
| **Global** | **≥ 80%** | Sanidade |

CI enforcement (Sprint 6):

```bash
flutter test --coverage
lcov --remove coverage/lcov.info \
  'lib/app/di/injection.config.dart' \
  'lib/*/*.g.dart' \
  '*.freezed.dart' \
  -o coverage/lcov_filtered.info
# Tooling: lcov_cobertura ou script custom
genhtml coverage/lcov_filtered.info -o coverage/html
# Assert total ≥ 80% via script no CI
```

### Padrões obrigatórios

- **Red-Green-Refactor sempre** — teste antes do código
- **Arrange / Act / Assert** dentro de `group()`:

```dart
group('SubmitRequestUseCase', () {
  late SubmitRequestUseCase useCase;
  late MockConnectMeRepository mockRepo;

  setUp(() {
    mockRepo = MockConnectMeRepository();
    useCase = SubmitRequestUseCase(mockRepo);
  });

  test('should return ConnectMeRequestEntity on success', () async {
    // Arrange
    when(() => mockRepo.submit(any()))
        .thenAnswer((_) async => Right(tRequestEntity));

    // Act
    final result = await useCase(tParams);

    // Assert
    expect(result, Right(tRequestEntity));
    verify(() => mockRepo.submit(tParams)).called(1);
  });
});
```

- **`blocTest`** para Bloc:

```dart
blocTest<ConnectMeBloc, ConnectMeState>(
  'emits [Loading, Success] when Submit succeeds',
  build: () {
    when(() => mockUseCase(any()))
        .thenAnswer((_) async => Right(tEntity));
    return ConnectMeBloc(submitRequestUseCase: mockUseCase);
  },
  act: (bloc) => bloc.add(SubmitRequested(...)),
  expect: () => [
    const ConnectMeLoading(),
    ConnectMeSuccess(tEntity),
  ],
);
```

- **Mocks** em `test/helpers/test_helper.dart` — estender, não redeclarar
- **Testes independentes** — sem ordem, sem estado compartilhado mutável
- **Golden tests** para widgets de design system:

```dart
testWidgets('AppButton matches golden', (tester) async {
  await tester.pumpWidget(MaterialApp(home: AppButton(label: 'Test')));
  await expectLater(find.byType(AppButton), matchesGoldenFile('app_button.png'));
});
```

## Segurança mobile — específica

Além das regras transversais de `../backend/.kiro/steering/security.md`:

### Nunca fazer

- ❌ Token em `SharedPreferences` — use `flutter_secure_storage` (iOS Keychain + Android Keystore)
- ❌ PII em `print()` / `logger.i(..., user.cpf)` — **nenhum** log com CPF, token, senha
- ❌ `dio.options.validateStatus: (_) => true` em produção — aceita qualquer status
- ❌ HTTP plain em produção — TLS obrigatório (`api.mastersindico.com.br` → HTTPS)
- ❌ Certificate pinning removido ou comentado (ver seção abaixo)
- ❌ `http` package solto — usar Dio com interceptors
- ❌ `dart:io WebView` com `javascriptMode: JavascriptMode.unrestricted` sem CSP
- ❌ Secrets em `analysis_options.yaml`, `pubspec.yaml`, constantes do app — **nunca** chegarem ao bundle

### Sempre fazer

- ✅ Tokens em `flutter_secure_storage` (nome genérico `auth_access_token`, `auth_refresh_token`)
- ✅ Certificate pinning para `api.mastersindico.com.br` em prod — via interceptor Dio ou `http_certificate_pinning`
- ✅ HTTPS-only — config nativa (`NSAppTransportSecurity` iOS, `networkSecurityConfig` Android)
- ✅ Device fingerprint header `X-Device-Fingerprint: <hash>` — calcular com `device_info_plus`
- ✅ Biometric unlock para re-auth em app (quando `local_auth` for integrado no Sprint 1.14)
- ✅ Logout → clear de `flutter_secure_storage` + clear de `Dio` cookie jar + navegação
- ✅ Detecção de jailbreak/root básica via `flutter_jailbreak_detection` — reportar ao backend, não bloquear sem justificativa
- ✅ CSP em WebViews quando houver (LGPD terms page, etc)

### LGPD no mobile

- Tela de consentimento explícito antes de enviar dados pessoais (Connect Me, cadastro)
- Opção "exportar meus dados" — chama endpoint backend, abre PDF
- Opção "deletar conta" — soft delete no backend, clear local
- Zero coleta de analytics sem opt-in (ver política com time jurídico antes de integrar Firebase Analytics)

## Code Calisthenics adaptado a Dart/Flutter

9 regras de `../backend/.kiro/steering/code-calisthenics.md` traduzidas:

1. **Um nível de indent por método** — guard clauses, extrair helpers, `if (!cond) return;`
2. **Sem `else`** — ternário (`cond ? a : b`) ou guard; `else if` longo → Strategy pattern com `Map<Enum, Function>` ou classes
3. **Envolver primitivos** — `UserId`, `Email`, `CPF` como wrapper classes com validação no construtor:

```dart
class Email {
  final String value;
  Email._(this.value);

  factory Email.parse(String raw) {
    if (!RegExp(r'^[^@]+@[^@]+\.[^@]+$').hasMatch(raw)) {
      throw const FormatException('Email inválido');
    }
    return Email._(raw.trim().toLowerCase());
  }
}
```

4. **First-class collections** — `VoteCollection` em vez de `List<Vote>` solto com invariante
5. **Um ponto por linha** — `user.profile.address.city` viola Demeter; método no agregado (`user.cityName`)
6. **Não abreviar** — `subscription`, não `sub`; `condominium`, não `condo`. Idiomatic Dart: `i` em loop curto, `e` em stack trace
7. **Widgets pequenos** — máx ~100 linhas de `build`; acima: extrair sub-widgets
8. **Máx 3-5 params em widget** — muito: consolidar em config class ou composition
9. **Sem setters em entidades** — entidades são `@immutable`, mudança via `copyWith`

Plugin `code-review` valida no PR.

## Plugins Claude Code relevantes ao app

Configurados em `../backend/.claude/settings.json` (workspace-level):

| Plugin | Uso no app |
|---|---|
| `typescript-lsp` | ❌ não se aplica |
| `gopls-lsp` | ❌ não se aplica |
| `figma` | ✅ sempre que implementar tela — `/implement-design <frame>` (projeto: https://www.figma.com/design/pxJkmn8rzezKnCYXcA7lyL/Master-sindico-APP---UX) |
| `security-guidance` | ✅ pre-tool hook detecta `eval` em `dart:mirrors`, `dart:io Process.run` inseguro, etc |
| `code-review` | ✅ PR review com scoring |
| `feature-dev` | ✅ agents de exploration/design/review na fase Discuss/Plan/Verify |
| `superpowers` | ✅ `/brainstorm`, `/write-plan`, TDD skills |
| `commit-commands` | ✅ `/commit` em pt-BR |
| `claude-md-management` | ✅ mantém este `CLAUDE.md`, `ARCHITECTURE.md`, `CODING_MANUAL.md` coerentes |
| `context7` | ✅ docs oficiais: flutter, flutter_bloc, go_router, dio, dartz, oidc, injectable, mocktail, bloc_test, equatable, freezed (se adotar) |
| `playwright` | ❌ não se aplica (E2E de app mobile via `integration_test` nativo, quando chegar) |
| `railway` | ❌ não se aplica (app não deploy em Railway) |
| `sentry` | ⏳ Sprint 7 — `sentry_flutter` para crash reporting |

## Conventions enforced across the codebase

- **Bilingual doc comments** — todo `///` começa em inglês, português abaixo. `//` comment single-language ok. **Não "limpar" a bilingualidade** — é padrão (CODING_MANUAL §1.1)
- **User-facing strings, log messages, commits**: pt-BR. Identifiers/filenames em inglês (`snake_case` files, `PascalCase` classes, `camelCase` everything else, `_camelCase` private)
- **Todas entities, Bloc events, Bloc states extends `Equatable`** — Bloc usa `==`; `props` ausente silenciosamente quebra state-change
- **Logging usa `logger` package**, nunca `print()`
- **Sem business logic em widgets**, sem `BuildContext` em Blocs — widget dispatcha evento, Bloc emite estado, `BlocListener` cuida de navegação
- **`setState` só para toggle UI local** (obscure password, selected tab); API-driven → Bloc

## Build e deploy

### Dev local

```bash
flutter run                 # device/emulator padrão
flutter run -d <device_id>  # device específico
flutter run --flavor dev    # quando flavors forem configurados (Sprint 6)
```

### Build produção

```bash
# Android
flutter build apk --release --obfuscate --split-debug-info=build/symbols
flutter build appbundle --release --obfuscate --split-debug-info=build/symbols

# iOS
flutter build ipa --release --obfuscate --split-debug-info=build/symbols
```

`--obfuscate` ofuscar código Dart. `--split-debug-info` separa símbolos para stack traces legíveis via `flutter symbolize`.

### Distribuição

- **TestFlight (iOS)**: upload via Xcode ou `fastlane` (Sprint 8.9)
- **Google Play Internal Testing (Android)**: upload via Play Console ou `fastlane` (Sprint 8.9)
- **Firebase App Distribution** (opcional) para beta interna

### CI/CD (Sprint 6)

GitHub Actions em `mastersindico/app/.github/workflows/`:

```yaml
# pseudo-config
on: [pull_request]
jobs:
  test:
    steps:
      - uses: subosito/flutter-action@v2
        with: { channel: 'stable' }
      - run: flutter pub get
      - run: dart run build_runner build --delete-conflicting-outputs
      - run: flutter analyze
      - run: flutter test --coverage
      - run: flutter build apk --debug
      - run: flutter build ios --no-codesign
```

## MCP e credenciais

Os MCPs (Context7, GitHub, Obsidian) configurados em `../backend/.claude/settings.json` e `../backend/.mcp.json`. Abrindo Claude Code em `app/`:
- Se workspace root for `backend/`, MCPs estão ativos — ótimo
- Se abrir só em `app/`, replicar `.mcp.json` + `.claude/settings.json` aqui OU abrir da raiz `Development/`

Env vars ficam em `../backend/.env` (gitignored). `FIGMA_ACCESS_TOKEN` para plugin Figma já lá.

## Integração com backend

Backend vive em `../backend/` (repo: `mastersindico/backend`). Endpoints em `http://localhost:8000/api/v1/*` (dev). Contrato:

```
Auth:         /auth/login, /auth/callback, /auth/logout    (OIDC redirects — só via browser externo em app, ou `oidc` package)
API:          /api/v1/auth/me                               (GET — AuthUser atual)
              /api/v1/...                                   (todos protegidos, Authorization: Bearer <token>)
Webhooks:     /webhooks/stripe, /webhooks/mux               (não aplicável ao app)
```

Response format do backend:
```json
// Success:
{ "success": true, "data": { ... } }

// Error:
{ "success": false, "error": { "code": "FORBIDDEN", "message": "..." } }
```

`Dio` interceptor em `lib/core/network/`:
1. **Auth interceptor**: injeta `Authorization: Bearer ${accessToken}` de `flutter_secure_storage`
2. **Device interceptor**: injeta `X-Device-Fingerprint: ${fingerprint}` computed via `device_info_plus`
3. **Refresh interceptor**: ao receber 401, tenta refresh token; se falhar, logout
4. **Error interceptor**: mapeia `DioException` → `ServerException` tipado com `code` e `message` do backend

## Troubleshooting

| Sintoma | Causa provável |
|---|---|
| `MissingPluginException` no startup | `injection.config.dart` stale — rodar `build_runner` |
| "X not registered in GetIt" | Anotação esqueceu (`@injectable`/`@lazySingleton`) ou build_runner não rodou |
| `SharedPreferences` crash no startup | Esqueceu registrar manualmente em `main.dart` antes de `configureDependencies()` |
| Test falha com "state never emitted" | `props` ausente em Bloc state — Bloc usa `==` |
| Deep link não navega | Scheme `com.mastersindico:/auth` tem que estar em `AndroidManifest.xml` + `Info.plist` |
| Login OIDC abre browser mas não volta | Redirect URI errado ou scheme não registrado |
| Build iOS falha em release | Provisioning profile ausente / codesigning — `flutter build ios --no-codesign` em dev, fastlane em CI |

## Próximas features (roteiro alinhado com backend)

Pelo `../backend/.kiro/specs/master-sindico/tasks/sprint-1.md`:

- **1.14**: auth OIDC real + onboarding por persona (morador, síndico, empresa, parceiro)

Sprints seguintes (mirror do tasks.md monolítico do backend):
- **Sprint 2**: feature `institutional` — home morador, timeline leitura, plano diretor leitura
- **Sprint 3**: feature `institutional` completa — painel morador, eventos, comunicados, avaliação de gestão, push notifications (FCM)
- **Sprint 4**: feature `commercial` — Connect Me mobile, dashboard empresa, registrar execução (câmera)
- **Sprint 5**: feature `content` — player nativo, upload foto/vídeo, busca
- **Sprint 6**: integrações push + SMS + email (via backend)
- **Sprint 7**: feature `assembly` + QA em device real
- **Sprint 8.9**: submit TestFlight + Google Play Internal

Cada Sprint, atualizar `app/` feature correspondente seguindo o protocolo SDD acima.
