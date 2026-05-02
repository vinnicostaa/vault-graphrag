---
title: Property-Based Testing — invariantes sobre inputs aleatórios
type: concept
tags: [testing, pbt, property-based, fuzzing]
created: 2026-04-24
updated: 2026-04-24
aliases: [PBT, Property-based testing, Teste baseado em propriedades]
---

# Property-Based Testing — invariantes sobre inputs aleatórios

Em vez de escrever exemplos específicos (`f(2) == 4`), declarar uma **propriedade** que deve valer para qualquer input (`∀x: f(x) >= 0`). A ferramenta gera centenas a milhares de casos e tenta quebrar a propriedade.

## Example-based vs Property-based

| Dimensão | Example-based | Property-based |
|---|---|---|
| Input | Escolhido à mão | Gerado aleatoriamente pela lib |
| Intenção | "Este caso específico funciona" | "Esta invariante sempre vale" |
| Casos | 5-20 por função | 100-10.000 por propriedade |
| Cobre edge | Só o que lembrei | Zero-values, overflow, empty, NaN automáticos |
| Custo | Baixo inicial | Mais lento, mais pensar em invariante |

PBT **complementa** example-based; não substitui. Casos conhecidos continuam explícitos (regressão, bugs reportados).

## Ciclo da ferramenta

```
1. Generator        → produz input aleatório dentro do domínio
2. Property         → código afirma invariante sobre o input
3. Falha encontrada → lib tenta SHRINK o input (encontrar contraexemplo mínimo)
4. Repro salva      → seed + input registrado para re-rodar determinístico
```

### Shrinking (o que diferencia PBT de "só testar com rand")

Quando a propriedade falha com uma lista de 1324 elementos, a lib tenta reduzir pra `[0, -1]` mantendo a falha. Sem shrinking, o contraexemplo é ilegível; com shrinking, vira bug-report útil.

## Tools por stack

| Stack | Biblioteca |
|---|---|
| Haskell | QuickCheck (original, 1999) |
| Go | **pgregory.net/rapid**, gopter |
| TypeScript / JS | **fast-check** |
| Python | **Hypothesis** |
| Rust | **proptest**, quickcheck |
| Scala / JVM | ScalaCheck, jqwik |
| Elixir | StreamData |
| C# / .NET | FsCheck |
| Erlang | PropEr |

## Propriedades canônicas (catálogo pra pensar)

Quando olhar função nova e perguntar "que propriedade essa função deve respeitar?", usar este catálogo:

- **Identity**: `f(x) == x` para algum conjunto (e.g., `decode(encode(x)) == x`)
- **Idempotency**: `f(f(x)) == f(x)` (e.g., `normalize`, `canonicalize`, `dedupe`)
- **Commutativity**: `f(a, b) == f(b, a)` (e.g., soma, merge de sets)
- **Associativity**: `f(f(a, b), c) == f(a, f(b, c))` (e.g., concat, max)
- **Inverse / round-trip**: `decode(encode(x)) == x`, `parse(format(x)) == x`
- **Invariant preservation**: pós-condição mantém invariante (soma = 1, saldo ≥ 0)
- **Monotonicity**: `a <= b ⟹ f(a) <= f(b)`
- **Oracle (test against reference)**: nova implementação vs lenta-mas-correta
- **Never throws / never panics**: para qualquer input válido, função retorna erro tipado, nunca crasheia

## Exemplo (pseudocódigo)

```
property "reverse é sua própria inversa" {
  ∀ xs: List[int]:
    reverse(reverse(xs)) == xs
}

property "encode/decode round-trip JSON"  {
  ∀ obj: UserDTO:
    decode(encode(obj)) == obj
}

property "desconto nunca ultrapassa o total" {
  ∀ amount ∈ [0, 1_000_000], pct ∈ [0, 100]:
    let d = applyDiscount(amount, pct)
    0 <= d <= amount
}
```

## Onde PBT brilha

- **Parsers / serializers** — round-trip é invariante natural
- **State machines** — modelar transições legais e gerar sequências
- **Aritmética monetária** — nunca overflow, nunca negativo indevido
- **Quotas / rate limits** — nunca concede além do limite
- **ABAC / authz engines** — role × action, nenhum bypass
- **Frações / percentuais** — soma == 1 sem drift
- **Ordenação / deduplicação** — output ordenado, tamanho ≤ input, elementos preservados
- **Concorrência** (via PBT estendido com modelo) — stateful properties

## Onde PBT **não** compensa

- Lógica que depende de **texto natural em UI** (difícil gerar, fácil exemplificar)
- Fluxos E2E **longos** com muitos side-effects (usar BDD em vez)
- Regra com **poucos inputs enumeráveis** (table-driven resolve)

## Integração com fuzzing

PBT e fuzzing convergem na prática moderna:

- **Fuzzing clássico** (AFL, libFuzzer, `go test -fuzz`) — bytes aleatórios, detecta crash/UB
- **PBT com shrinking** — semântico, verifica propriedade
- **Differential fuzzing** — rodar duas implementações e comparar outputs; qualquer divergência é bug. Ex: novo parser vs libjson; nova crypto vs ref

Go 1.18+ traz fuzzing nativo compatível em espírito com PBT — corpus + shrinking automático.

## Boas práticas

- **Começar simples**: uma propriedade por função crítica. Expandir depois.
- **Seed gravada no repositório** quando PBT encontra contraexemplo — vira teste de regressão explícito.
- **Limitar domínio** para evitar falso-positivo (ex: `IntRange(0, 1_000_000)` quando `MaxInt` não é caso real).
- **Não gerar o resultado esperado com a mesma fórmula da implementação** — propriedade vira tautologia.
- **Tempo orçado**: 100 casos default; 1000 em regras críticas; 10_000 em nightly.

## Links

- [[_moc]]
- [[testing-strategy]]
- [[mutation-testing]]
- [[contract-testing]]
- [[../principles/do-dont-checklist]]
