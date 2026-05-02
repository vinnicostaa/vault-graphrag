# 📖 Coding Manual — Feature First + Clean Architecture
# 📖 Manual de Codificação — Feature First + Clean Architecture

This document defines the coding standards, mandatory patterns, and explicit prohibitions for any project built on this template.

Este documento define os padrões de codificação, padrões obrigatórios e proibições explícitas para qualquer projeto construído sobre este template.

---

## 📐 1. General Rules / Regras Gerais

### 1.1 Language / Idioma
- All code comments must be bilingual: **English above, Portuguese below**.
- Todos os comentários de código devem ser bilíngues: **inglês acima, português abaixo**.

```dart
// ✅ CORRECT / CORRETO
/// Authenticates the user with email and password.
/// Autentica o usuário com e-mail e senha.
Future<Either<Failure, UserEntity>> login(...);

// ❌ WRONG — single language only / ERRADO — idioma único
/// Autentica o usuário com e-mail e senha.
Future<Either<Failure, UserEntity>> login(...);
```

### 1.2 Naming Conventions / Convenções de Nomenclatura

| Element / Elemento | Style / Estilo | Example / Exemplo |
|---|---|---|
| Classes | `PascalCase` | `AuthRepository`, `LoginUseCase` |
| Files / Arquivos | `snake_case` | `auth_repository.dart`, `login_usecase.dart` |
| Variables / Variáveis | `camelCase` | `userName`, `isLoading` |
| Constants / Constantes | `camelCase` | `baseUrl`, `tokenKey` |
| Private fields / Campos privados | `_camelCase` | `_repository`, `_dio` |
| Bloc Events / Eventos Bloc | `PascalCase` + verb | `AuthLoginRequested` |
| Bloc States / Estados Bloc | `PascalCase` + adjective | `AuthAuthenticated`, `AuthLoading` |

### 1.3 Imports Order / Ordem dos Imports

Always follow this order, separated by blank lines:

Sempre siga esta ordem, separados por linhas em branco:

```dart
// 1. Dart SDK
import 'dart:convert';

// 2. Flutter SDK
import 'package:flutter/material.dart';

// 3. Third-party packages (alphabetical) / Pacotes de terceiros (alfabético)
import 'package:dartz/dartz.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// 4. Project imports (relative) / Imports do projeto (relativo)
import '../../../../core/error/failures.dart';
import '../repositories/auth_repository.dart';
```

---

## 🏗 2. Architecture Rules / Regras de Arquitetura

### 2.1 The Dependency Rule / A Regra de Dependência

**THE MOST IMPORTANT RULE. NEVER BREAK THIS.**

**A REGRA MAIS IMPORTANTE. NUNCA QUEBRE ESTA.**

```
presentation → domain ← data
```

- `domain/` **MUST NOT** import from `data/` or `presentation/`.
- `presentation/` **MUST NOT** import from `data/`.
- `data/` imports from `domain/` (to implement interfaces).
- `presentation/` imports from `domain/` (to use entities and usecases).

- `domain/` **NÃO PODE** importar de `data/` ou `presentation/`.
- `presentation/` **NÃO PODE** importar de `data/`.
- `data/` importa de `domain/` (para implementar interfaces).
- `presentation/` importa de `domain/` (para usar entidades e usecases).

```dart
// ❌ FORBIDDEN — domain importing from data
// ❌ PROIBIDO — domain importando de data
import '../data/models/user_model.dart'; // INSIDE domain/ folder!

// ❌ FORBIDDEN — presentation importing from data
// ❌ PROIBIDO — presentation importando de data
import '../data/datasources/auth_remote_datasource.dart'; // INSIDE presentation/ folder!
```

### 2.2 Feature Isolation / Isolamento de Features

- Each feature is **self-contained**. Features **MUST NOT** import from each other's `data/` or `presentation/` layers.
- Cada feature é **auto-contida**. Features **NÃO PODEM** importar das camadas `data/` ou `presentation/` de outra feature.

```dart
// ❌ FORBIDDEN — cross-feature import
// ❌ PROIBIDO — import entre features
// Inside features/cart/presentation/
import '../../auth/data/datasources/auth_local_datasource.dart';

// ✅ ALLOWED — sharing via domain entities or core
// ✅ PERMITIDO — compartilhar via entidades do domínio ou core
import '../../../core/error/failures.dart';
import '../../auth/domain/entities/user_entity.dart';
```

- Shared logic belongs in `core/`, not inside a feature.
- Lógica compartilhada pertence ao `core/`, não dentro de uma feature.

### 2.3 One UseCase = One Action / Um UseCase = Uma Ação

- Each `UseCase` class does **exactly one thing**.
- Cada classe `UseCase` faz **exatamente uma coisa**.

```dart
// ✅ CORRECT — single responsibility / CORRETO — responsabilidade única
class LoginUseCase extends UseCase<UserEntity, LoginParams> { ... }
class LogoutUseCase extends UseCase<Unit, NoParams> { ... }

// ❌ WRONG — multiple responsibilities / ERRADO — múltiplas responsabilidades
class AuthUseCase {
  Future login() { ... }
  Future logout() { ... }
  Future register() { ... }
}
```

---

## 🚫 3. What NOT To Do / O Que NÃO Fazer

### 3.1 Never use `print()` for logging / Nunca use `print()` para logging

```dart
// ❌ WRONG
print('User logged in: $userId');

// ✅ CORRECT — use the logger package
import 'package:logger/logger.dart';
final logger = Logger();
logger.i('User logged in: $userId');
```

### 3.2 Never throw exceptions from domain or presentation / Nunca lance exceções do domain ou presentation

Exceptions are **only thrown in the `data/` layer** (datasources). Repositories catch them and convert to `Failure`.

Exceções são **lançadas apenas na camada `data/`** (datasources). Repositories as capturam e convertem em `Failure`.

```dart
// ❌ WRONG — throwing in UseCase
// ❌ ERRADO — lançando no UseCase
class LoginUseCase {
  Future<UserEntity> call(params) async {
    try { ... } catch (e) { throw Exception('Failed'); } // NO!
  }
}

// ✅ CORRECT — return Either
// ✅ CORRETO — retornar Either
class LoginUseCase extends UseCase<UserEntity, LoginParams> {
  Future<Either<Failure, UserEntity>> call(params) { ... }
}
```

### 3.3 Never use BuildContext in Bloc / Nunca use BuildContext no Bloc

```dart
// ❌ WRONG — Bloc should never depend on UI / ERRADO — Bloc nunca deve depender de UI
class AuthBloc {
  void showError(BuildContext context, String msg) {
    ScaffoldMessenger.of(context).showSnackBar(...);
  }
}

// ✅ CORRECT — Bloc emits state, widget reacts
// ✅ CORRETO — Bloc emite estado, widget reage
// In Bloc: emit(AuthFailureState(message))
// In Widget: BlocListener reacts to state
```

### 3.4 Never put business logic in widgets / Nunca coloque lógica de negócio em widgets

```dart
// ❌ WRONG — API call inside a widget / ERRADO — chamada de API dentro de widget
class LoginPage extends StatelessWidget {
  void _login() async {
    final response = await Dio().post('/auth/login', ...);  // NO!
  }
}

// ✅ CORRECT — widget only dispatches events / CORRETO — widget só despacha eventos
class LoginPage extends StatelessWidget {
  void _login() {
    context.read<AuthBloc>().add(AuthLoginRequested(email: ..., password: ...));
  }
}
```

### 3.5 Never instantiate dependencies manually / Nunca instancie dependências manualmente

```dart
// ❌ WRONG — manual instantiation / ERRADO — instanciação manual
final bloc = AuthBloc(
  loginUseCase: LoginUseCase(AuthRepositoryImpl(...)),
);

// ✅ CORRECT — use GetIt service locator / CORRETO — usar GetIt service locator
final bloc = sl<AuthBloc>();
```

### 3.6 Never use hardcoded colors or text styles / Nunca use cores ou estilos de texto hardcoded

```dart
// ❌ WRONG
Text('Hello', style: TextStyle(fontSize: 24, color: Color(0xFF123456)));
Container(color: Colors.blue);

// ✅ CORRECT — use theme system / CORRETO — usar sistema de tema
Text('Hello', style: context.textTheme.headlineMedium);
Container(color: context.colorScheme.primary);
// or / ou
Text('Hello', style: AppTextStyles.headlineMedium);
Container(color: AppColors.primary);
```

### 3.7 Never use `setState` for complex state / Nunca use `setState` para estado complexo

```dart
// ❌ WRONG — setState for API-driven state / ERRADO — setState para estado de API
class _MyPageState extends State<MyPage> {
  bool isLoading = false;
  String? error;
  UserEntity? user;

  void _login() async {
    setState(() => isLoading = true);
    try {
      user = await api.login(...);
    } catch (e) {
      error = e.toString();
    }
    setState(() => isLoading = false);
  }
}

// ✅ setState is ONLY acceptable for local UI state
// ✅ setState é APENAS aceitável para estado local de UI
setState(() => _obscurePassword = !_obscurePassword);  // OK — toggle visibility
setState(() => _selectedTab = 2);                       // OK — tab selection
```

### 3.8 Never put Models in the domain layer / Nunca coloque Models na camada domain

```dart
// ❌ WRONG — UserModel in domain/
lib/features/auth/domain/models/user_model.dart  // NO!

// ✅ CORRECT
lib/features/auth/domain/entities/user_entity.dart  // Pure entity, no JSON
lib/features/auth/data/models/user_model.dart       // Model with fromJson/toJson
```

---

## ✅ 4. Mandatory Patterns / Padrões Obrigatórios

### 4.1 Either for error handling / Either para tratamento de erros

All repository methods and use cases **must** return `Either<Failure, T>`.

Todos os métodos de repositório e use cases **devem** retornar `Either<Failure, T>`.

```dart
Future<Either<Failure, UserEntity>> login({...});   // ✅
Future<UserEntity> login({...});                     // ❌
```

### 4.2 Equatable for entities and states / Equatable para entidades e estados

All entities, Bloc events, and Bloc states **must** extend `Equatable`.

Todas as entidades, eventos Bloc e estados Bloc **devem** estender `Equatable`.

```dart
class UserEntity extends Equatable {
  @override
  List<Object?> get props => [id, name, email];
}
```

### 4.3 Injectable annotations for DI / Anotações Injectable para DI

| Annotation / Anotação | Use for / Usar para |
|---|---|
| `@injectable` | Bloc, UseCase (new instance each time / nova instância cada vez) |
| `@lazySingleton` | Repository, Datasource, NetworkInfo (single instance / instância única) |
| `@module` | Third-party classes that can't be annotated / Classes de terceiros |

### 4.4 Const constructors / Construtores const

Always use `const` constructors when possible.

Sempre use construtores `const` quando possível.

```dart
const AuthLoading();            // ✅
const AuthUnauthenticated();    // ✅
AuthLoading();                  // ❌ missing const
```

### 4.5 Prefer `final` / Prefira `final`

```dart
final String name;        // ✅ immutable
String name;               // ❌ mutable without reason
```

---

## 📁 5. File Organization / Organização de Arquivos

### 5.1 New Feature Checklist / Checklist de Nova Feature

When creating a new feature, **always** create the full structure:

Ao criar uma nova feature, **sempre** crie a estrutura completa:

```
features/
└── <feature_name>/
    ├── data/
    │   ├── datasources/
    │   │   ├── <feature>_local_datasource.dart
    │   │   └── <feature>_remote_datasource.dart
    │   ├── models/
    │   │   └── <feature>_model.dart
    │   └── repositories/
    │       └── <feature>_repository_impl.dart
    ├── domain/
    │   ├── entities/
    │   │   └── <feature>_entity.dart
    │   ├── repositories/
    │   │   └── <feature>_repository.dart
    │   └── usecases/
    │       └── <action>_usecase.dart
    └── presentation/
        ├── bloc/
        │   ├── <feature>_bloc.dart
        │   ├── <feature>_event.dart
        │   └── <feature>_state.dart
        ├── pages/
        │   └── <feature>_page.dart
        └── widgets/
            └── <widget_name>.dart
```

### 5.2 Where does each thing go? / Onde cada coisa vai?

| What / O quê | Where / Onde |
|---|---|
| Reusable UI utilities (extensions, helpers) | `core/utils/` |
| Colors, fonts, themes | `core/theme/` |
| Error handling (Failure, Exception) | `core/error/` |
| Base classes (UseCase, NoParams) | `core/usecases/` |
| Network checking | `core/network/` |
| Feature-specific logic | `features/<name>/` |
| App-level config (DI, Router, MaterialApp) | `app/` |
| Tests mirroring the lib structure | `test/` |

---

## 🧪 6. Testing Standards / Padrões de Testes

### 6.1 Test file naming / Nomeação de arquivo de teste

Mirror `lib/` structure exactly:

Espelhe a estrutura de `lib/` exatamente:

```
lib/features/auth/domain/usecases/login_usecase.dart
test/features/auth/domain/usecases/login_usecase_test.dart
```

### 6.2 Test structure / Estrutura de teste

Use **Arrange / Act / Assert** pattern with `group()`:

Use o padrão **Arrange / Act / Assert** com `group()`:

```dart
group('LoginUseCase', () {
  test('should return UserEntity on successful login', () async {
    // Arrange
    when(() => mockRepo.login(...)).thenAnswer((_) async => Right(tUser));

    // Act
    final result = await useCase(tParams);

    // Assert
    expect(result, Right(tUser));
    verify(() => mockRepo.login(...)).called(1);
  });
});
```

### 6.3 What to test / O que testar

| Layer / Camada | What to test / O que testar | Tool / Ferramenta |
|---|---|---|
| `domain/usecases/` | UseCase calls repository correctly | `mocktail` |
| `presentation/bloc/` | Bloc emits correct state sequence | `bloc_test` |
| `data/repositories/` | Repository handles success + failure | `mocktail` |
| `data/models/` | fromJson / toJson roundtrip | `flutter_test` |

---

## 🔄 7. Git Conventions / Convenções de Git

### 7.1 Commit messages / Mensagens de commit

Use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add cart feature with checkout flow
fix: resolve null token crash on logout
docs: update README with i18n guide
refactor: extract payment validation to usecase
test: add bloc tests for CartBloc
chore: update dependencies
```

### 7.2 Branch naming / Nomeação de branch

```
feature/cart-checkout
fix/logout-crash
refactor/auth-flow
docs/coding-manual
```

---

## ⚡ 8. Quick Reference Card / Cartão de Referência Rápida

| ✅ DO / FAÇA | ❌ DON'T / NÃO FAÇA |
|---|---|
| Use `Either<Failure, T>` for results | Throw exceptions from domain/presentation |
| Use `sl<T>()` for dependency access | Instantiate dependencies with `new` |
| Use `AppColors.primary` | Use `Colors.blue` or `Color(0xFF...)` |
| Use `context.textTheme.bodyMedium` | Use `TextStyle(fontSize: 14)` |
| Use `const` constructors | Omit `const` when possible |
| Put shared code in `core/` | Put shared code inside a feature |
| Create one UseCase per action | Create a UseCase with multiple methods |
| Keep Bloc free of BuildContext | Pass BuildContext to Bloc |
| Use `final` for immutable fields | Use `var` without reason |
| Mirror `lib/` in `test/` | Put all tests in one flat folder |
| Use `mocktail` for mocking | Create manual mock classes |
| Comment in EN + PT-BR | Comment in one language only |
