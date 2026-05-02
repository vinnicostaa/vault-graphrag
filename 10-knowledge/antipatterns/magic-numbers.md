---
title: Magic Numbers (AP-009)
type: antipattern
tags: [antipattern, code-smell, readability, clean-code]
source: "Martin, R. C. (2008). Clean Code — Prentice Hall. Cap. 17 (Heuristics)"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [magic-literals, unnamed-constants]
---

# AP-009 — Magic Numbers

Literais numéricos (ou strings) com significado não-óbvio espalhados pelo código sem nome explicativo. O "3600" pode ser "segundos em uma hora" ou "limite de retry" — só quem escreveu sabe.

## Sintomas

- `if retries > 3`, `sleep(86400)`, `price * 1.23` sem constante nomeada
- Mesmo literal repetido em N lugares (3, 3, 3)
- Comentário `// 3 = max retries` do lado — o nome certo da constante eliminava o comentário
- Strings literais `status == "pending"` soltas; typo silencioso
- Tabelas de conversão (`rate * 9/5 + 32`) inline

## Por que é problema

**Código opaco**: leitor precisa decifrar intenção.

**Mudanças ocultas**: alterar regra obriga achar todas as cópias do literal — facilmente perde-se uma.

**Type safety fraca**: literais não tem tipo; trocar dois `int` que significam coisas diferentes compila.

**Testes repetem o literal**: replicação se propaga em fixtures.

## Exemplo

```go
// ❌
if user.FailedAttempts >= 5 {
    user.LockUntil = time.Now().Add(30 * time.Minute)
}
if elapsed > 86400 { ... }

// ✅
const (
    MaxFailedAttempts = 5
    AccountLockDuration = 30 * time.Minute
    OneDay = 24 * time.Hour
)
if user.FailedAttempts >= MaxFailedAttempts {
    user.LockUntil = time.Now().Add(AccountLockDuration)
}
if elapsed > OneDay { ... }
```

## Como refatorar

- **Replace Magic Number with Symbolic Constant**: refactoring clássico de Fowler
- **Replace Type Code with Enum**: strings/ints de categoria viram enum
- **Named parameters** em chamadas booleanas (`retry=true` melhor que `true`)
- **Constantes agrupadas por domínio** (pacote `limits`, `policy`) em vez de constantes espalhadas
- Se o valor vem de config externa: ler via env + validar no boot, não hardcode

## Quando tolerar

- `0`, `1`, `-1` em contextos óbvios (index, flag de sucesso, return value sentinela)
- Fórmulas matemáticas onde o literal **É** a constante universal (`2 * math.Pi`, `0.5` em média)
- Testes quando o literal é o próprio caso de teste (`assertEqual(sum(2, 3), 5)`)
- Conversões aritméticas triviais sem significado de domínio (`bytes / 1024`)

## Patterns relacionados (que resolvem)

- [[../principles/clean-code]] — naming como ferramenta primária
- [[primitive-obsession]] — muitas magic numbers sugerem falta de VO/enum
- [[duplicate-code]] — literal repetido é duplicação
- [[../principles/solid]] — OCP: config via constante permite extensão

## Fontes

- Martin, R. C. (2008). *Clean Code: A Handbook of Agile Software Craftsmanship*. Cap. 2 (Meaningful Names), Cap. 17 (G25).
- Fowler, M. (2018). *Refactoring*. "Replace Magic Number with Symbolic Constant".
- Refactoring Guru — [Replace Magic Number with Symbolic Constant](https://refactoring.guru/replace-magic-number-with-symbolic-constant).

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../principles/clean-code]]
- [[primitive-obsession]]
