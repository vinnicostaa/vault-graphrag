---
title: OP-016 — Force-unwrap / non-null assertion em mapeamento
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - null-safety
id: OP-016
severity: Critical
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Force unwrap
  - Non-null assertion abuse
---

# OP-016 — Force-unwrap / non-null assertion em mapeamento

**Severidade**: Critical (NPE em produção)
**Categoria**: Null-safety / Validação

## Sintoma

`raw.street!`, `user?.address!!.city!!`, `response.data as NotNullType`, `@SuppressWarnings("nullness")` espalhados pelo código. Nullable assertions em pontos onde o dado pode genuinamente faltar → NPE/crash em runtime em cenários reais de produção.

## Por que é problema

- **Crash silencioso**: campo opcional na fonte, obrigatório no destino, force-unwrap.
- **Cobertura ilusória**: linter passa porque suprimido; bug aparece em produção.
- **Inversão do valor de null-safety**: linguagens com nullability tipada (Dart, Kotlin, TypeScript strict, Rust Option) protegem quando usadas corretamente; `!`/`!!`/`as` anulam o benefício.

## Exemplo (anti-pattern)

```dart
// ❌ ERRADO — cada bang pode explodir
Address toDomain(Map<String, dynamic> raw) {
  return Address(
    street: raw['street']!,    // null? crash
    number: raw['number']!,
    city: raw['city']!,
    zipCode: raw['zipCode']!,
  );
}
```

## Fix preferido — validar explicitamente + tipo de retorno que reflete a possibilidade

```dart
Address? toDomain(Map<String, dynamic> raw) {
  final street = raw['street'] as String?;
  final number = raw['number'] as String?;
  final city = raw['city'] as String?;
  final zipCode = raw['zipCode'] as String?;

  if (street == null || number == null || city == null || zipCode == null) {
    return null;   // dado inconsistente; caller decide o que fazer
  }
  return Address(street: street, number: number, city: city, zipCode: zipCode);
}
```

Ou retornar `Either<ValidationError, Address>` quando o caller precisa saber a razão.

## Alternativa

- **Parser schema-based** (Zod, Valibot, io-ts, freezed JSONSerializable): decodifica e valida num passo; falha é tipada.
- **Failure-tolerant defaults**: campos com default seguro (`city = ''`) só quando fizer sentido de domínio.

## Nunca

- Desabilitar null-safety em arquivo inteiro (`// @ts-nocheck`, `biome-ignore` global, `@SuppressWarnings` em classe). Se a linguagem oferece null-safety, use.

## Quando tolerar

Logo após um check explícito que comprova non-null para o compilador/linter:

```ts
if (!user.address) return
// ok aqui — o compilador narrow já fez o trabalho
console.log(user.address.city)   // sem "!"
```

## Relações

- **Patterns**: [[../patterns/null-object]] (quando null precisa virar valor sentinel), Value Object
- **OPs relacionados**: [[op-017-mapeamento-parcial]] (sintoma adjacente), [[op-019-mutacao-sem-validacao]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-016
- Dart — [Sound null safety](https://dart.dev/null-safety) (consultada 2026-04-24)
- Kotlin — [Null safety](https://kotlinlang.org/docs/null-safety.html) (consultada 2026-04-24)
- TypeScript — [Strict null checks](https://www.typescriptlang.org/tsconfig#strictNullChecks) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../patterns/null-object]]
- [[op-017-mapeamento-parcial]]
- [[../principles/do-dont-checklist]]
