---
title: OP-010 — Feature importa internals de outra feature
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - coupling
  - clean-architecture
id: OP-010
severity: High
created: 2026-04-24T00:00:00.000Z
updated: 2026-04-24T00:00:00.000Z
aliases:
  - Feature isolation broken
  - Cross-feature internals
doc-consulted: '2026-04-24'
---

# OP-010 — Feature importa internals de outra feature

**Severidade**: High
**Categoria**: Acoplamento

## Sintoma

`features/cart/presentation/` importa `features/auth/data/` (datasource, repo impl, DTO privado). Camada de apresentação de uma feature acessa internals de outra. Independent deployability e testabilidade quebrados.

## Por que é problema

- **Mudança em `data/` de auth quebra `cart`** — acoplamento invisível.
- **Testes explodem**: testar cart exige inicializar auth/data.
- **Clean Arch violada**: `presentation → domain ← data` para **ambas** as features; cross-feature só via `core/` ou `domain/entities/` pública.

## Exemplo (anti-pattern)

```dart
// ❌ ERRADO — presentation da Cart importa data da Auth
import 'package:app/features/auth/data/datasources/auth_remote_datasource.dart';

class CartPage extends StatelessWidget {
  final AuthRemoteDatasource auth;   // ← privado da Auth
  // ...
}
```

## Fix preferido — comunicação via `core/` ou entidade pública

```dart
// core/session/current_user.dart — contrato compartilhado estável
abstract class CurrentUser {
  UserId get id;
  bool get isLoggedIn;
}

// features/auth implementa e expõe via DI
// features/cart consome só o contrato
class CartPage extends StatelessWidget {
  final CurrentUser user;   // ← estável, atomic-sized, sem vazar internals
}
```

Outra via: **eventos de domínio** — Auth publica `UserLoggedIn`, Cart reage. Acoplamento por protocolo estável, não por import direto.

## Alternativa

- Expor `domain/entities/X.dart` como API pública da feature + tudo mais em `_internal/` (convenção Dart).
- Em monorepo: cada feature vira package separado; linting bloqueia `import '../other_feature/...'`.

## Quando tolerar

Nunca em feature madura. Em MVP exploratório, marcado com `// TODO feature boundary: extrair X pra core/` + ticket aberto.

## Relações

- **Patterns**: Feature isolation, [[../patterns/facade]] (API pública da feature), Event-driven communication
- **OPs relacionados**: [[op-009-vazamento-dominio-modulos]] (mesma raiz em escala BC), [[op-011-logica-negocio-infra]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-010, §3.17 (Mobile architecture)
- Uncle Bob — [[../architecture/the-clean-architecture]]
- Package-by-feature vs package-by-layer — amplamente documentado

## Links

- [[_moc]]
- [[../architecture/the-clean-architecture]]
- [[op-009-vazamento-dominio-modulos]]
- [[../architecture/clean-architecture-layers]]
