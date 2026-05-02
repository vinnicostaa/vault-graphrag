---
title: OP-017 — Mapeamento parcial falha silenciosamente
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - null-safety
  - data-integrity
id: OP-017
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Silent partial mapping
  - Broken object construction
---

# OP-017 — Mapeamento parcial falha silenciosamente

**Severidade**: High
**Categoria**: Null-safety / Integridade de dados

## Sintoma

Código decide que objeto agregado "existe" baseado em subset de campos (`hasAddress = raw.street && raw.zipCode`), ignora o resto, e monta objeto meio-preenchido (`city = null`, `number = undefined`). Downstream consome o objeto inválido sem saber; bugs emergem distantes da origem.

## Por que é problema

- **Estado inconsistente**: `Address` sem `city` é semanticamente inválido; sistema opera com objeto inválido.
- **Bug-causality distance**: erro aparece 5 camadas depois do mapeamento — debug caro.
- **Corrupção de UI / persistência**: frontend renderiza `undefined`, banco aceita insert com null em coluna obrigatória (se schema permitir).

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — presence check parcial, monta Address quebrado
const hasAddress = raw.street && raw.zipCode   // ignora city, number
const address = hasAddress
  ? { street: raw.street, number: raw.number, city: raw.city, zipCode: raw.zipCode }
  : null
// address?.city pode ser undefined mesmo quando address !== null
```

## Fix preferido — validar **todos** os campos obrigatórios

```ts
function toAddress(raw: unknown): Address | null {
  const street  = getString(raw, 'street')
  const number  = getString(raw, 'number')
  const city    = getString(raw, 'city')
  const zipCode = getString(raw, 'zipCode')

  if (!street || !number || !city || !zipCode) {
    return null   // qualquer campo obrigatório faltando → sem address
  }
  return { street, number, city, zipCode }
}
```

Com schema (Zod/Valibot):

```ts
const AddressSchema = z.object({
  street: z.string().min(1),
  number: z.string().min(1),
  city:   z.string().min(1),
  zipCode: z.string().regex(/^\d{5}-?\d{3}$/),
})

function toAddress(raw: unknown): Address | null {
  const parsed = AddressSchema.safeParse(raw)
  return parsed.success ? parsed.data : null
}
```

## Alternativa

- **Either / Result** retornando razão específica da falha (`MissingField('city')`) quando caller precisa distinguir motivos.
- **Telemetria** (log warn + metric) quando mapeamento parcial é detectado — indica qualidade ruim na origem dos dados.

## Quando tolerar

Objetos onde **campos realmente opcionais** existem (ex.: `complemento` de endereço brasileiro). Aí o tipo reflete: `complemento?: string`.

## Relações

- **Patterns**: Parser, DTO com validação
- **OPs relacionados**: [[op-016-force-unwrap]] (mesma classe), [[op-019-mutacao-sem-validacao]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-017
- Zod — [Schema validation](https://zod.dev) (consultada 2026-04-24)
- Valibot — [docs](https://valibot.dev) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[op-016-force-unwrap]]
- [[../principles/clean-code]]
- [[../testing/property-based-testing]]
