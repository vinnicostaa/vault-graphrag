# 🔍 Architecture Explained — Why Each File Exists
# 🔍 Arquitetura Explicada — Por Que Cada Arquivo Existe

This document walks through every directory and file in the project, explaining **why** it exists, **what** it does, and **how** it connects to the rest of the architecture.

Este documento percorre cada diretório e arquivo do projeto, explicando **por que** ele existe, **o que** faz e **como** se conecta ao restante da arquitetura.

---

## 🌳 High-Level Overview / Visão Geral de Alto Nível

```
lib/
├── main.dart        → Entry point (bootstrap)
├── app/             → Application shell (config)
├── core/            → Shared infrastructure
└── features/        → Business features (the actual app)
```

The project follows the **Dependency Rule**: outer layers know about inner layers, never the reverse.

O projeto segue a **Regra da Dependência**: camadas externas conhecem camadas internas, nunca o inverso.

```
┌──────────────────────────────────────┐
│          PRESENTATION                │  Widgets, Pages, Bloc
│  ┌──────────────────────────────┐    │
│  │          DOMAIN              │    │  Entities, UseCases, Repository contracts
│  │  ┌──────────────────────┐    │    │
│  │  │        DATA          │    │    │  Models, Datasources, Repository implementations
│  │  └──────────────────────┘    │    │
│  └──────────────────────────────┘    │
└──────────────────────────────────────┘
```

---

## 📄 `lib/main.dart`

**Why it exists / Por que existe:**
Every Flutter app needs a single entry point. This file does the **minimum** required to bootstrap the app — nothing more.

Todo app Flutter precisa de um ponto de entrada único. Este arquivo faz o **mínimo** necessário para iniciar o app — nada mais.

**What it does / O que faz:**
1. Calls `WidgetsFlutterBinding.ensureInitialized()` — required before any async operations.
2. Initializes `SharedPreferences` (async, must be done before DI).
3. Registers `SharedPreferences` in GetIt manually (because it can't be annotated).
4. Calls `configureDependencies()` to initialize all other DI registrations.
5. Runs `App()`.

**Why like this / Por que assim:**
`SharedPreferences.getInstance()` is async, but `@module` getters are synchronous. So we register it manually *before* the auto-generated DI runs. This avoids a common "`MissingPluginException`" crash.

---

## 📂 `lib/app/`

This folder holds **application-level configuration** — things that are not business logic but are required for the app to function.

Esta pasta contém **configuração de nível de aplicação** — coisas que não são lógica de negócio mas são necessárias para o app funcionar.

### `app/app.dart`

**Why:** Separates the `MaterialApp` widget from `main.dart` for cleanliness. Keeps widening the dependency tree predictable.

**Por que:** Separa o widget `MaterialApp` do `main.dart` por limpeza. Mantém a árvore de dependências previsível.

**What it does:**
- Wraps the app with `MultiBlocProvider` for global Blocs (currently empty — ready for when you need one).
- Configures `MaterialApp.router` with `GoRouter`, light/dark themes, and `ThemeMode.system`.

**Design decision / Decisão de design:** `ThemeMode.system` follows the device's dark mode setting automatically. Change to `ThemeMode.light` or `ThemeMode.dark` to force a specific theme.

---

### `app/di/injection.dart`

**Why:** Centralizes dependency injection (DI) configuration. DI is the **glue** that connects layers without creating hard dependencies.

**Por que:** Centraliza a configuração de injeção de dependência (DI). DI é a **cola** que conecta camadas sem criar dependências rígidas.

**What it does:**
- Exposes `sl` (service locator) — the single global `GetIt` instance.
- `configureDependencies()` triggers the auto-generated `init()` function.
- `RegisterModule` registers third-party classes (`Dio`, `Connectivity`, `SharedPreferences`) that can't use `@injectable` annotations.

**Why `@lazySingleton` vs `@injectable`:**
- `@lazySingleton` → created once, reused everywhere (Repositories, Datasources, Dio). Avoids creating 50 HTTP clients.
- `@injectable` → new instance every time (Blocs, UseCases). Each page gets its own fresh Bloc instance.

---

### `app/di/injection.config.dart`

**Why:** This file is **auto-generated** by `build_runner`. It wires all `@injectable` and `@lazySingleton` annotated classes into `GetIt`.

**Por que:** Este arquivo é **gerado automaticamente** pelo `build_runner`. Ele conecta todas as classes anotadas com `@injectable` e `@lazySingleton` ao `GetIt`.

**Never edit manually. Always regenerate:**
```bash
flutter pub run build_runner build --delete-conflicting-outputs
```

---

### `app/router/app_routes.dart`

**Why:** Route paths as string literals scattered across the code are a bug magnet. This file centralizes them as constants.

**Por que:** Caminhos de rotas como strings literais espalhadas pelo código são um ímã de bugs. Este arquivo as centraliza como constantes.

**Example of the problem / Exemplo do problema:**
```dart
context.go('/logn');  // Typo! No compile error, crashes at runtime.
context.go(AppRoutes.login);  // ✅ Compile-time safety.
```

---

### `app/router/app_router.dart`

**Why:** Separates routing configuration from the `App` widget. GoRouter is declared as a top-level variable so it doesn't need `BuildContext` to be constructed.

**Por que:** Separa a configuração de roteamento do widget `App`. GoRouter é declarado como variável top-level para não precisar de `BuildContext` para ser construído.

**What it does:**
- Defines all routes mapping paths → pages.
- Includes an `errorBuilder` for 404-style situations (e.g., deep linking to a deleted page).
- `debugLogDiagnostics: true` prints all navigation events to the console during development.

---

## 📂 `lib/core/`

**Why it exists:** Holds **shared infrastructure** that is used by multiple features but doesn't belong to any single one.

**Por que existe:** Contém **infraestrutura compartilhada** usada por múltiplas features mas que não pertence a nenhuma delas individualmente.

---

### `core/error/exceptions.dart`

**Why:** These are the **data layer's** error signals. When `Dio` returns a 500, or `SharedPreferences` returns null, a typed exception is thrown.

**Por que:** Estes são os sinais de erro da **camada de data**. Quando o `Dio` retorna 500 ou `SharedPreferences` retorna null, uma exceção tipada é lançada.

**Why typed exceptions instead of generic `Exception`:** So the `RepositoryImpl` can catch `ServerException` and `CacheException` separately and convert each to the appropriate `Failure`. Generic `catch (e)` loses this information.

---

### `core/error/failures.dart`

**Why:** Failures are the **domain layer's** way of expressing errors. They cross the domain boundary via `Either<Failure, T>` — never as thrown exceptions.

**Por que:** Failures são a forma da **camada de domain** de expressar erros. Eles cruzam a fronteira do domain via `Either<Failure, T>` — nunca como exceções lançadas.

**Why Equatable:** Bloc compares states with `==`. If `ServerFailure('error')` is not equatable, Bloc won't detect state changes correctly. Equatable makes `==` check the `message` and `code` fields instead of reference identity.

---

### `core/network/network_info.dart` + `network_info_impl.dart`

**Why split into two files:** The abstract class (`network_info.dart`) lives in `core/` as a contract. The implementation (`network_info_impl.dart`) uses `connectivity_plus` — a third-party package. If you ever swap `connectivity_plus` for another package, you only touch the `_impl` file. Zero changes in domain or presentation.

**Por que dividido em dois arquivos:** A classe abstrata (`network_info.dart`) vive no `core/` como contrato. A implementação (`network_info_impl.dart`) usa `connectivity_plus` — um pacote de terceiros. Se você trocar `connectivity_plus` por outro pacote, só muda o arquivo `_impl`. Zero mudanças no domain ou presentation.

---

### `core/usecases/usecase.dart`

**Why:** Defines the `UseCase<T, Params>` contract. Every use case in every feature extends this. It ensures:
1. Every use case returns `Future<Either<Failure, T>>` (no exceptions leak out).
2. Every use case is callable with a single `Params` object (consistent API).

`NoParams` exists for use cases that don't need input parameters (e.g., `LogoutUseCase`, `GetCurrentUserUseCase`).

---

### `core/utils/constants.dart`

**Why:** Prevents magic strings/numbers scattered across the codebase. Base URL, timeouts, storage keys — all in one place.

**Why `abstract class`:** Can't be instantiated. Used purely as a namespace for static constants.

---

### `core/utils/extensions/`

**Why:** Extension methods reduce boilerplate without creating utility classes. `context.colorScheme` is cleaner than `Theme.of(context).colorScheme` and still statically typed.

**Por que:** Extension methods reduzem boilerplate sem criar classes utilitárias. `context.colorScheme` é mais limpo que `Theme.of(context).colorScheme` e continua estaticamente tipado.

---

### `core/theme/`

**Why three files instead of one:**

| File / Arquivo | Purpose / Propósito |
|---|---|
| `app_colors.dart` | Raw color values. Change the brand color here, the entire app updates. |
| `app_text_styles.dart` | Typography definitions. Based on Material 3 type scale. |
| `app_theme.dart` | Combines colors + typography + component themes into `ThemeData`. |

Splitting allows a designer to update `app_colors.dart` without understanding `ThemeData` internals.

---

## 📂 `lib/features/auth/`

This is the **canonical example feature** — it demonstrates every layer of Clean Architecture with a real-world use case (authentication).

Esta é a **feature canônica de exemplo** — demonstra cada camada da Clean Architecture com um caso de uso real (autenticação).

---

### `domain/entities/user_entity.dart`

**Why:** The `UserEntity` is **pure Dart** — no `fromJson`, no `@injectable`, no Flutter imports. It represents `User` as the **business** sees it, not as the API returns it.

**Why `Equatable`:** Two `UserEntity` instances with the same `id`, `name`, `email` should be considered equal. This is critical for Bloc state comparison.

---

### `domain/repositories/auth_repository.dart`

**Why:** This is an **abstract class** (interface). The domain defines *what* operations exist but not *how* they work. The data layer provides the `Impl`.

**Why `Either<Failure, T>`:** Forces the caller (UseCase) to handle both success and failure explicitly. No hidden `try/catch` needed.

---

### `domain/usecases/login_usecase.dart`

**Why:** Encapsulates a single business operation. Even though `LoginUseCase` currently just forwards to the repository, it serves as a **stable entry point** where you can later add:
- Input validation
- Analytics tracking
- Rate limiting
- Cross-feature orchestration

`LoginParams` extends `Equatable` so it can be compared in tests.

---

### `data/models/user_model.dart`

**Why it extends `UserEntity`:** `UserModel` *is a* `UserEntity` with extra capabilities (`fromJson`, `toJson`). This means the Repository can return `UserModel` where `UserEntity` is expected — polymorphism in action.

**Por que estende `UserEntity`:** `UserModel` *é um* `UserEntity` com capacidades extras (`fromJson`, `toJson`). Isso significa que o Repository pode retornar `UserModel` onde `UserEntity` é esperado — polimorfismo em ação.

---

### `data/datasources/auth_remote_datasource.dart`

**Why abstract + impl:** The abstract class defines the contract. The impl uses `Dio`. If you switch from REST to GraphQL, you create a new impl — nothing else changes.

**Why Dart 3 records `({UserModel user, String token})`:** The login API returns both a user and a token. Instead of creating a wrapper class, we use a named record — lightweight, typed, and disposable.

---

### `data/datasources/auth_local_datasource.dart`

**Why:** Abstracts local storage. Currently uses `SharedPreferences`, but could be swapped for `Hive`, `sqflite`, or `drift` by creating a new `Impl`.

**Why not store the token in the entity:** The token is an infrastructure concern (HTTP headers). The domain entity shouldn't know about tokens.

---

### `data/repositories/auth_repository_impl.dart`

**Why:** This is where **everything connects**. The repository implementation:
1. Checks network connectivity.
2. Calls the remote datasource.
3. Caches the result locally.
4. Catches exceptions and converts them to `Failure`.

**Why it's the only place with `try/catch`:** Exceptions from `Dio` and `SharedPreferences` are caught here and converted to `Either.Left(Failure)`. From this point outward (UseCase → Bloc → UI), only `Either` is used. No more exception cascades.

---

### `presentation/bloc/auth_bloc.dart`

**Why Bloc and not Cubit:** Bloc separates events from methods, making it explicit:
- **What happened** (event): `AuthLoginRequested`
- **What's showing** (state): `AuthLoading`, `AuthAuthenticated`, `AuthFailureState`

This traceability is invaluable for debugging. You can replay events, log them, and test exact sequences.

**Why `part of` for event/state files:** Keeps the three files (`auth_bloc.dart`, `auth_event.dart`, `auth_state.dart`) tightly coupled while allowing separate file organization. Events and States are always co-located with their Bloc.

---

### `presentation/pages/login_page.dart`

**Why `BlocProvider` is local (not global):** The `AuthBloc` is created with `sl<AuthBloc>()` inside `BlocProvider` at the page level. When the user navigates away, the Bloc is automatically disposed. This prevents memory leaks.

**Why `BlocListener` for navigation:** Navigation is a side effect, not a UI state. `BlocListener` fires once per state change (doesn't rebuild the widget tree).

---

### `presentation/widgets/login_form.dart`

**Why separate from `LoginPage`:** Keeps the page (layout/structure) separate from the form (input fields/validation). The form could be reused in a different context (e.g., a dialog, a bottom sheet).

**Why `setState` is allowed here:** `_obscurePassword` is purely local UI state (show/hide password icon). It doesn't affect business logic or other widgets.

---

## 📂 `lib/features/home/`

**Why it exists with only `presentation/`:** Not every feature needs all three layers at the start. `home` is a simple static page — no API calls, no entities. If it grows, you add `domain/` and `data/` later.

**Por que existe só com `presentation/`:** Nem toda feature precisa de todas as três camadas desde o início. `home` é uma página estática simples — sem chamadas de API, sem entidades. Se crescer, adicione `domain/` e `data/` depois.

---

## 📂 `test/`

**Why mirror `lib/`:** Finding tests is instant. `lib/features/auth/domain/usecases/login_usecase.dart` → `test/features/auth/domain/usecases/login_usecase_test.dart`.

### `test/helpers/test_helper.dart`

**Why:** Centralizes mock definitions. Every test file imports the mocks from here instead of redeclaring them.

### Test files

| Test | What it validates |
|---|---|
| `failures_test.dart` | `Equatable` equality, `toString()` format |
| `login_usecase_test.dart` | UseCase calls repository and forwards result |
| `auth_bloc_test.dart` | Bloc emits correct state sequence for each event |

---

## 📦 Key Package Choices / Escolhas de Pacotes

| Package | Why chosen / Por que escolhido |
|---|---|
| **flutter_bloc** | Most structured state management for Clean Architecture. Separates events/states explicitly. |
| **get_it + injectable** | Simple DI without widget-tree coupling. Injectable generates boilerplate automatically. |
| **go_router** | Official Flutter team routing solution. Declarative, type-safe, supports deep linking. |
| **dartz** | Industry-standard `Either` for Dart. Makes errors visible in the type system. |
| **dio** | Interceptors, cancelation, retry — features that `http` lacks. |
| **equatable** | Value equality without manually overriding `==` and `hashCode`. |
| **mocktail** | No codegen needed, null-safe, minimal boilerplate. |
| **bloc_test** | `blocTest()` handles stream assertions elegantly. |
