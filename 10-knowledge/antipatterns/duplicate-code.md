---
title: Duplicate Code (AP-011)
type: antipattern
tags: [antipattern, code-smell, dispensable, dry]
source: "https://refactoring.guru/smells/duplicate-code"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [copy-paste-programming, dry-violation]
---

# AP-011 — Duplicate Code

Trechos de código idênticos (ou quase-idênticos) aparecem em múltiplos pontos. O smell *canônico*: tudo que é DRY atende isto como base.

## Sintomas

- Blocos iguais em dois métodos da mesma classe
- Métodos com corpo idêntico em classes irmãs (candidato a **Pull Up Method**)
- Expressões replicadas com mesma estrutura e 1-2 literais diferentes
- Bug fixado num lugar reaparece três semanas depois no clone que ficou
- Git grep do primeiro bloco retorna 10+ resultados

## Por que é problema

**Manutenção multiplicada**: bug fix, refactor, otimização precisam rodar N vezes.

**Divergência silenciosa**: com o tempo, cópias evoluem diferente; o que era "o mesmo" vira "quase o mesmo".

**Regra de negócio fragmentada**: a verdade do domínio está em vários lugares — qual é a fonte?

**Sinal de abstração faltante**: duplicação tipicamente expõe um conceito não nomeado.

## Exemplo

```javascript
// ❌
function getUserAge(user) {
    const now = new Date();
    const birth = new Date(user.birthDate);
    return Math.floor((now - birth) / (365.25 * 24 * 3600 * 1000));
}

function getAccountAgeInYears(account) {
    const now = new Date();
    const created = new Date(account.createdAt);
    return Math.floor((now - created) / (365.25 * 24 * 3600 * 1000));
}

// ✅
function yearsSince(date) {
    return Math.floor((Date.now() - new Date(date)) / (365.25 * 86_400_000));
}
```

## Como refatorar

- **Extract Method**: duplicação pequena dentro da mesma classe
- **Extract Class**: duplicação representa responsabilidade comum
- **Pull Up Method / Pull Up Field**: duplicação entre irmãs → superclasse
- **Form Template Method**: esqueleto compartilhado + partes variáveis
- **Substitute Algorithm**: quando as cópias divergiram, consolidar na melhor versão
- Regra prática: **rule of three** — duplicou 3 vezes, extrair; 2 vezes, aguardar

## Quando tolerar

- **Coincidência temporária**: dois trechos parecem iguais mas representam conceitos independentes que vão evoluir separadamente (acoplamento indevido se extrair)
- **Testes**: duplicação em arrange/assert é aceitável se aumenta legibilidade
- **Estruturas geradas** (migrations, fixtures) — não vale deduplizar
- **Bootstrapping / scaffolding** antes de estabilizar padrões

## Patterns relacionados (que resolvem)

- [[../patterns/template-method]] — parte compartilhada + variação controlada
- [[../patterns/strategy]] — variações viram estratégias
- [[../patterns/factory-method]] — construção duplicada vira factory
- [[shotgun-surgery]] — duplicação frequentemente causa shotgun surgery depois
- [[../principles/do-dont-checklist]] — DRY

## Fontes

- Refactoring Guru — [Duplicate Code](https://refactoring.guru/smells/duplicate-code)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Hunt, A. & Thomas, D. (1999). *The Pragmatic Programmer* — DRY como lema central.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[shotgun-surgery]]
- [[../principles/do-dont-checklist]]
