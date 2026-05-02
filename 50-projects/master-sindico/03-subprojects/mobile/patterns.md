---
title: Mobile — Patterns Flutter/Dart
type: guide
tags: [subproject, mobile, master-sindico, v2, patterns, flutter, dart, bloc, freezed]
source:
  - Clientes/Joao/MasterSindico/Development/app/CODING_MANUAL.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/ARCHITECTURE.md (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/lib/** (2026-04-21)
  - Clientes/Joao/MasterSindico/Development/app/test/** (2026-04-21)
created: 2026-04-23
updated: 2026-04-23
aliases: [mobile-patterns]
---

# Mobile — Patterns Flutter/Dart

Patterns específicos do sub-projeto `app/`. Complementa [[architecture]] e [[ui-catalog]]. Estado management oficial: **`flutter_bloc`** ([[../../STATE#D-048|D-048]]); imutabilidade/unions: **`freezed`** ([[../../STATE#D-049|D-049]]) desde M1.

---

## 1. Bloc patterns

### 1.1 Event vs Method

Eventos disparados pela UI via `context.read<T>().add(...)`:

```dart
context.read<AssemblyBloc>().add(AssemblyVoteCast(
  condominiumId: condominiumId,
  assemblyId: assemblyId,
  agendaItemId: agendaItemId,
  voteValue: voteValue,
));
```

Bloc reage emitindo estados. UI reconstrói via `BlocBuilder` ou `BlocConsumer`; side effects (navegação, snackbar) em `BlocListener`.

### 1.2 Equatable obrigatório (CODING_MANUAL §4.2)

Todo `Entity`, `Event`, `State` extends `Equatable`. Sem `props`, comparação falha → Bloc não detecta mudança de estado e o widget não reconstrói.

**Exemplo real** — `AssemblyListLoaded`:

```dart
class AssemblyListLoaded extends AssemblyState {
  final List<AssemblyEntity> assemblies;
  const AssemblyListLoaded(this.assemblies);
  @override List<Object?> get props => [assemblies];
}
```

### 1.3 `part of`

Event e state files usam `part of '<feature>_bloc.dart'` — co-localizados. Padrão do código atual:

```dart
// assembly_bloc.dart
part 'assembly_event.dart';
part 'assembly_state.dart';

// assembly_event.dart
part of 'assembly_bloc.dart';
```

### 1.4 Navigation via BlocListener

Navegação é side-effect, **não estado**. Separar sempre:

```dart
BlocConsumer<AuthBloc, AuthState>(
  listener: (c, s) {
    if (s is AuthAuthenticated)  c.go(AppRoutes.home);
    if (s is AuthFailureState)   ScaffoldMessenger.of(c).showSnackBar(SnackBar(content: Text(s.message)));
  },
  builder: (c, s) => /* UI */,
)
```

### 1.5 BlocProvider local (não global)

Prefira `BlocProvider` no nível da página — Bloc disposed automaticamente em pop. Padrão real em `AssemblyListPage`:

```dart
BlocProvider(
  create: (_) => sl<AssemblyBloc>()
    ..add(AssemblyListRequested(condominiumId: condominiumId)),
  child: Scaffold(...),
)
```

Global só quando genuinamente shared (ex: `AuthBloc` observando sessão inteira) → em `App.build` via `MultiBlocProvider`.

### 1.6 `bloc_concurrency` droppable (D-048 obrigatório M1)

Em handlers de botões que podem ser apertados rápido (voto, Connect Me submit, pagamento):

```dart
import 'package:bloc_concurrency/bloc_concurrency.dart';

AssemblyBloc(...) : super(const AssemblyInitial()) {
  on<AssemblyVoteCast>(_onVoteCast, transformer: droppable()); // descarta novos até completar
}
```

### 1.7 `hydrated_bloc` (D-048 obrigatório M1)

Blocs cujo estado deve sobreviver restart (memberships, timeline cached, plano diretor lido):

```dart
class MembershipsBloc extends HydratedBloc<MembershipsEvent, MembershipsState> {
  // toJson / fromJson implementados para persistir estado
}
```

Storage encriptado atrás de `HydratedBloc.storage = HydratedStorage.build(storageDirectory: ...)` com chave derivada de `flutter_secure_storage`.

---

## 2. UseCase pattern

### 2.1 Interface real — `core/usecases/usecase.dart`

```dart
abstract class UseCase<T, Params> {
  Future<Either<Failure, T>> call(Params params);
}

class NoParams {}
```

> **Dívida**: `NoParams` hoje não estende `Equatable` — precisa para comparação em testes com `mocktail` sem ter que `registerFallbackValue(NoParams())` toda vez. M1 upgrade: `class NoParams extends Equatable { const NoParams(); @override List<Object?> get props => []; }`.

### 2.2 UseCase com params — `CastVoteUseCase`

```dart
@injectable
class CastVoteUseCase extends UseCase<Unit, CastVoteParams> {
  final AssemblyRepository _repository;
  CastVoteUseCase(this._repository);

  @override
  Future<Either<Failure, Unit>> call(CastVoteParams params) {
    return _repository.castVote(
      condominiumId: params.condominiumId,
      assemblyId: params.assemblyId,
      agendaItemId: params.agendaItemId,
      voteValue: params.voteValue,
    );
  }
}

class CastVoteParams extends Equatable {
  final String condominiumId;
  final String assemblyId;
  final String agendaItemId;
  final String voteValue;
  const CastVoteParams({required ...});
  @override List<Object?> get props => [condominiumId, assemblyId, agendaItemId, voteValue];
}
```

UseCase futuro pode adicionar validation, analytics, rate limiting local — **interface estável**, contratos downstream não mudam.

### 2.3 Um UseCase = uma ação (CODING_MANUAL §2.3)

```dart
// ✅ Bons UseCases
class LoginUseCase  extends UseCase<UserEntity, NoParams> { ... }
class LogoutUseCase extends UseCase<Unit, NoParams> { ... }

// ❌ UseCase guarda-chuva — NÃO fazer
class AuthUseCase {
  Future login()    { ... }
  Future logout()   { ... }
  Future register() { ... }
}
```

---

## 3. Factory/Reconstruct — DDD em Dart

Dart não tem private constructor puro (só library-private `_`). Convenção: `._` private + factories explícitas `create` (novo agregado) e `reconstruct` (rehidratar de JSON/DB):

```dart
@immutable
class Vote extends Equatable {
  final String id;
  final String tenantId;
  final String agendaItemId;
  final String voterId;
  final VoteOption option;
  final double weight;
  final DateTime castAt;

  const Vote._({required ..., required this.castAt});

  factory Vote.create({
    required String tenantId,
    required String agendaItemId,
    required String voterId,
    required VoteOption option,
    required double weight,
  }) {
    if (weight <= 0) throw ArgumentError('weight must be > 0');
    if (!option.isValid) throw ArgumentError('invalid option');
    return Vote._(
      id: uuidV7(),     // D-055
      tenantId: tenantId, agendaItemId: agendaItemId, voterId: voterId,
      option: option, weight: weight,
      castAt: DateTime.now().toUtc(),   // storage UTC — D-052
    );
  }

  factory Vote.reconstruct(Map<String, dynamic> json) => Vote._(
    id: json['id'],
    tenantId: json['tenant_id'],
    agendaItemId: json['agenda_item_id'],
    voterId: json['voter_id'],
    option: VoteOption.parse(json['option']),
    weight: (json['weight'] as num).toDouble(),
    castAt: DateTime.parse(json['cast_at']),
  );

  @override List<Object?> get props => [id];
}
```

Com `freezed` (M1, D-049) fica ainda mais enxuto — `@freezed` gera `copyWith`, `==`, `hashCode`, `toString`; factories `create`/`reconstruct` ficam explicitas.

---

## 4. Value Objects — branded types em Dart

Dart não tem branded types nativos; usar wrapper `@immutable` com factory validadora:

```dart
@immutable
class Email {
  final String value;
  const Email._(this.value);

  factory Email.parse(String raw) {
    final trimmed = raw.trim().toLowerCase();
    if (!RegExp(r'^[^@]+@[^@]+\.[^@]+$').hasMatch(trimmed)) {
      throw const FormatException('Email inválido');
    }
    return Email._(trimmed);
  }

  @override bool operator ==(Object o) => o is Email && o.value == value;
  @override int get hashCode => value.hashCode;
  @override String toString() => value;
}
```

Similar para:
- `CPF` / `CNPJ` — validador + dígitos verificadores + formatador.
- `CEP` — 8 dígitos + formatador.
- `FracaoIdeal` — `Decimal` com 6 casas; range [0, 1].
- `Money` — `int` cents (armazenamento), render `R$` via `intl`.
- `UnitIdentifier` — string livre ("101", "Apto 501", "Bloco B - 302") — ver [[../../01-domain/ubiquitous-language]].
- `Score` — int [0, 100] — reputação (D-051).

Com `freezed` M1:

```dart
@freezed
class Email with _$Email {
  const Email._();
  const factory Email(String value) = _Email;

  factory Email.parse(String raw) { /* validator */ return Email(trimmed); }
}
```

---

## 5. First-class collections

`VoteCollection` em vez de `List<Vote>` solto quando invariante coletivo importa:

```dart
@immutable
class VoteCollection extends Equatable {
  final List<Vote> _votes;
  const VoteCollection(this._votes);

  int    get count       => _votes.length;
  double get totalWeight => _votes.fold(0.0, (acc, v) => acc + v.weight);

  VoteCollection add(Vote v) {
    if (_votes.any((e) => e.voterId == v.voterId)) {
      throw StateError('voter already voted');
    }
    return VoteCollection([..._votes, v]);
  }

  @override List<Object?> get props => [_votes];
}
```

Uso canônico: `AgendaItem.votes` (VoteCollection), `Assembly.agendaItems` (AgendaItemCollection), `Timeline.entries` (TimelineEntryCollection com ordering invariant).

---

## 6. Code Calisthenics adaptado Dart/Flutter

9 regras traduzidas (CODING_MANUAL CLAUDE.md §9):

1. **1 nível de indent por método** — guard clauses, extract helpers, `if (!cond) return;`.
2. **Sem `else`** — ternário `cond ? a : b` ou guard; `else if` longo → Strategy com `Map<Enum, Function>` ou polimorfismo. Exceção: switch em `sealed class` (Dart 3) compilador exige exaustividade — sem `default:`.
3. **Envolver primitivos** — VOs §4.
4. **First-class collections** — §5.
5. **Um ponto por linha** (Demeter) — `user.profile.address.city` → método no agregado (`user.cityName`).
6. **Não abreviar** — `subscription` não `sub`; `condominium` não `condo`. Idiomatic Dart: `i` em loop curto, `e` em stack trace.
7. **Widgets pequenos** — máx ~100 linhas no `build`; acima: extrair sub-widgets privados (`_ActionBar`, `_AgendaSection` como no `assembly_detail_page.dart` real).
8. **Máx 3-5 params em widget** — muito: consolidar em config class. Exceção aceita: páginas receber IDs de rota (condominiumId + assemblyId + agendaItemId) — padrão de deep-link.
9. **Sem setters em entidades** — `@immutable` + `copyWith` (freezed gera). Mudança via factory ou use case no agregado.

Plugin `code-review` do Claude Code valida no PR.

---

## 7. `copyWith` pattern

**Manual** (CODING_MANUAL):

```dart
@immutable
class User extends Equatable {
  final String id;
  final String name;
  final Email email;
  const User({required this.id, required this.name, required this.email});

  User copyWith({String? id, String? name, Email? email}) =>
      User(id: id ?? this.id, name: name ?? this.name, email: email ?? this.email);

  @override List<Object?> get props => [id, name, email];
}
```

**`freezed` (M1, D-049) — preferido**:

```dart
@freezed
class User with _$User {
  const factory User({
    required String id,
    required String name,
    required Email email,
  }) = _User;
}
// copyWith, ==, hashCode, toString, JsonSerializable (se anotado) gerados.
```

---

## 8. Platform channels — evitar

Usar **apenas** se feature nativa crítica não coberta por plugin (ex: assinatura digital por certificado armazenado no Keychain, proxy custom). Exemplo hipotético:

```dart
class BiometricSigner {
  static const _channel = MethodChannel('com.mastersindico/biometric');
  Future<String> sign(String payload) async {
    return await _channel.invokeMethod('sign', {'payload': payload});
  }
}
```

Implementação nativa iOS (Swift) + Android (Kotlin) sincronizada com mesmo protocolo de payload. Biometric reauth padrão usa `local_auth` package — sem platform channel.

---

## 9. State management — signals UI local

`setState` permitido **apenas** para UI local trivial (toggle password obscured, tab selected, local form field focus). Qualquer state que afeta dados, navegação ou outros widgets → Bloc.

**Exemplo real** — `AssemblySciencePage`:

```dart
class _AssemblySciencePageState extends State<AssemblySciencePage> {
  bool _confirmed = false;    // ✅ toggle UI local

  // ... usa setState(() => _confirmed = v) no checkbox
  // mas o submit dispara event: context.read<AssemblyBloc>().add(AssemblyScienceRegistered(...))
}
```

---

## 10. Testing patterns

Stack: **`flutter_test`** (built-in) + **`mocktail`** (sem codegen, null-safe) + **`bloc_test`** (stream assertions).

### 10.1 Coverage thresholds (CLAUDE.md legado)

| Camada | Mínimo | Motivo |
|---|---|---|
| `lib/features/*/domain/` | **95%** | Regras de domínio — testar todos os paths |
| `lib/features/*/presentation/bloc/` | **90%** | Bloc é máquina de estados |
| `lib/features/*/data/repositories/` | **85%** | Único lugar com try/catch |
| `lib/features/*/data/datasources/` | **80%** | Mock Dio, testar error paths |
| `lib/features/*/presentation/pages/` | **70%** | Widget tests para fluxos críticos |
| `lib/features/*/presentation/widgets/` | **80%** | Golden tests para DS reusáveis |
| `lib/core/` | **90%** | Infra compartilhada |
| `lib/app/` | **60%** | Bootstrap/DI — teste manual OK |
| **Global** | **≥ 80%** | Sanidade |

### 10.2 AAA + `group()` — template canônico

```dart
group('CastVoteUseCase', () {
  late CastVoteUseCase useCase;
  late MockAssemblyRepository mockRepo;

  setUp(() {
    mockRepo = MockAssemblyRepository();
    useCase = CastVoteUseCase(mockRepo);
  });

  test('should return Unit on successful cast', () async {
    // Arrange
    when(() => mockRepo.castVote(condominiumId: any(named: 'condominiumId'), ...))
        .thenAnswer((_) async => const Right(unit));

    // Act
    final result = await useCase(tParams);

    // Assert
    expect(result, const Right(unit));
    verify(() => mockRepo.castVote(...)).called(1);
  });
});
```

### 10.3 `blocTest` — assertions de stream

**Exemplo real** — `auth_bloc_test.dart`:

```dart
blocTest<AuthBloc, AuthState>(
  'deve emitir [AuthLoading, AuthAuthenticated] em caso de sucesso',
  build: () {
    when(() => mockLoginUseCase(any())).thenAnswer((_) async => Right(tUser));
    return authBloc;
  },
  act: (bloc) => bloc.add(const AuthLoginRequested()),
  expect: () => [
    const AuthLoading(),
    AuthAuthenticated(tUser),
  ],
);
```

`registerFallbackValue(NoParams())` em `setUpAll` para `mocktail` aceitar matchers `any()` com tipos complexos.

### 10.4 Widget tests

```dart
testWidgets('LoginPage shows error on invalid credentials', (tester) async {
  await tester.pumpWidget(MaterialApp(home: LoginPage()));
  await tester.tap(find.text('Entrar'));
  await tester.pump();
  expect(find.text('Email inválido'), findsOneWidget);
});
```

### 10.5 Golden tests — design system

Para componentes reusáveis (`AppButton`, `AppBadge`, `AssemblyCard`):

```dart
testWidgets('AppButton matches golden', (tester) async {
  await tester.pumpWidget(MaterialApp(home: AppButton(label: 'Test')));
  await expectLater(find.byType(AppButton), matchesGoldenFile('app_button.png'));
});
```

### 10.6 Mocks via `mocktail` (sem codegen)

Arquivo atual: `test/helpers/test_helper.dart`:

```dart
class MockAuthRepository extends Mock implements AuthRepository {}
class MockLoginUseCase  extends Mock implements LoginUseCase {}
class MockLogoutUseCase extends Mock implements LogoutUseCase {}
```

Assembly tem Mocks in-file em `assembly_bloc_test.dart` (alvo: consolidar em `test_helper.dart`).

### 10.7 Integration tests (M1+)

`integration_test/` para critical paths:
- Auth flow completo (OIDC PKCE em browser emulado).
- Onboarding morador M1-M8.
- Vote flow end-to-end.
- Upload vídeo-currículo.

### 10.8 CI enforcement

```bash
fvm flutter test --coverage
lcov --remove coverage/lcov.info \
  'lib/app/di/injection.config.dart' \
  'lib/*/*.g.dart' \
  '*.freezed.dart' \
  -o coverage/lcov_filtered.info
genhtml coverage/lcov_filtered.info -o coverage/html
# Script asserta total ≥ 80%
```

---

## 11. Anti-patterns Flutter

| # | Anti-pattern | Por quê |
|---|---|---|
| 1 | Token em `SharedPreferences` | Plain storage; usar `flutter_secure_storage` (Keychain/Keystore) |
| 2 | `print()` em prod | Vaza PII sem structure; `logger` package |
| 3 | `http` package solto | Sem interceptors; Dio com stack padrão |
| 4 | `dio.options.validateStatus: (_) => true` em prod | Aceita qualquer status — mascara bugs |
| 5 | `onReceiveResponse` handler logando body inteiro | PII em log |
| 6 | `Color(0xff...)` hardcoded em widget | Quebra theming; `AppColors.*` ou `context.colorScheme` |
| 7 | `BuildContext` em Bloc | Violação Clean Arch (acopla ao widget tree) |
| 8 | `setState` em lógica business | Bloc emite state; widget reflete |
| 9 | `throw Exception(...)` do `domain/` ou `presentation/` | `Either<Failure, T>` |
| 10 | `@LazySingleton` em Bloc | Bloc deve ser fresh por página; `@injectable` |
| 11 | WebView embed para OIDC | Zitadel bloqueia + user não vê URL bar; `ASWebAuthenticationSession` / Custom Tabs |
| 12 | Deep-link scheme em runtime user input | Validar allowlist |
| 13 | `Dio` com `receiveDataWhenStatusError: false` + silenciar | Perde status code — error interceptor não funciona |
| 14 | `on<Event>(_handler)` sem `transformer` em botão crítico | Double-tap = double-submit (D-048 `droppable()`) |
| 15 | Entity com `fromJson` direto | Move pra Model em `data/` |
| 16 | `@freezed` em estado que precisa mutação interna (ex: controller) | Separar: freezed no state imutável + campos mutáveis em controller próprio |

---

## 12. Convenções de nomes

- **Files**: `snake_case.dart`.
- **Classes**: `PascalCase`.
- **Funções/variáveis**: `camelCase`.
- **Private**: `_camelCase`.
- **Constants**: `camelCase` (Dart convention — **não** SCREAMING_SNAKE).
- **Bloc**: `<Name>Bloc`, `<Name>Event`, `<Name>State`.
- **UseCase**: `<Verb><Noun>UseCase` (`GetTimelineUseCase`, `SubmitVoteUseCase`, `CastVoteUseCase`).
- **Repository**: `<Name>Repository` (abstract em `domain/`), `<Name>RepositoryImpl` (em `data/`).
- **Datasource**: `<Name>RemoteDatasource` / `<Name>LocalDatasource` (+ `Impl`).
- **Model**: `<Entity>Model` (`UserModel`, `AssemblyModel`).
- **VO**: nome do conceito (`Email`, `CPF`, `Money`, `FracaoIdeal`).

---

## 13. Imports — ordem obrigatória (CODING_MANUAL §1.3)

```dart
// 1. Dart SDK
import 'dart:convert';

// 2. Flutter SDK
import 'package:flutter/material.dart';

// 3. Third-party (alfabético)
import 'package:dartz/dartz.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

// 4. Project imports (relativo)
import '../../../../core/error/failures.dart';
import '../repositories/auth_repository.dart';
```

---

## 14. Comentários bilingues (CODING_MANUAL §1.1)

Todo `///` (dartdoc) **EN acima, PT abaixo**. `//` (comment) aceita single-language. **Não "limpar" a bilingualidade** — é padrão do projeto.

```dart
/// Authenticates the user with email and password.
/// Autentica o usuário com e-mail e senha.
Future<Either<Failure, UserEntity>> login(...);
```

User-facing strings, log messages, commits → **pt-BR**. Identifiers/filenames → **EN**.

---

## 15. Build runner workflow

Ao adicionar/alterar `@injectable`, `@lazySingleton`, `@module`, `@freezed`, `@JsonSerializable`:

```bash
fvm dart run build_runner build --delete-conflicting-outputs
```

Durante sessão intensa de DI/freezed:

```bash
fvm dart run build_runner watch --delete-conflicting-outputs
```

Regenerados: `injection.config.dart`, `*.g.dart` (json_serializable), `*.freezed.dart`.

### 15.1 Caveat `freezed`

`build_runner` pode demorar > 30s em projetos médios — usar `build.yaml` com `generate_for` targeted em paths que realmente usam freezed + CI caching (`~/.pub-cache` + `.dart_tool/build/`).

---

## 16. Commits (CODING_MANUAL §7)

Conventional Commits em pt-BR:

```
feat: adiciona Connect Me morador com fluxo LGPD
fix: corrige crash no logout quando refresh token expirado
docs: atualiza README com comandos fvm
refactor: extrai validação de voto para value object
test: adiciona bloc tests para AssemblyBloc
chore: atualiza dependências (bloc_concurrency + hydrated_bloc)
```

Branch naming:

```
feature/connect-me-morador
fix/logout-crash
refactor/vote-vo
```

PR body obrigatório: seção `Release Notes:`.

---

## 17. Links

- [[README]]
- [[architecture]]
- [[ui-catalog]]
- [[security]]
- [[../../02-architecture/patterns]]
- [[../../CLAUDE]] §3.9
- [[../../STATE#D-048|D-048 Bloc + bloc_concurrency + hydrated_bloc]]
- [[../../STATE#D-049|D-049 freezed M1]]
