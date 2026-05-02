---
title: Speculative Generality (AP-012)
type: antipattern
tags: [antipattern, code-smell, dispensable, yagni, over-engineering]
source: "https://refactoring.guru/smells/speculative-generality"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [over-engineering, premature-abstraction, yagni-violation]
---

# AP-012 — Speculative Generality (YAGNI violation)

Abstrações, parâmetros, hooks e hierarquias criadas *"para o caso de precisar"* que nunca foram necessários. Tipicamente resultam de design dirigido a futuros hipotéticos — viola YAGNI (*You Aren't Gonna Need It*).

## Sintomas

- Interfaces com uma única implementação real
- Hierarquias de herança com 1 classe filha há meses
- Parâmetros opcionais nunca usados fora de testes
- Hooks/callbacks instalados "caso alguém queira override"
- Comentários do tipo `// extensibility for future plugins`
- Factory elaborada para criar 1 tipo
- Configs em arquivo `.yaml` que só têm 1 valor possível

## Por que é problema

**Complexidade sem retorno**: o custo é pago hoje (mais classes, mais testes, mais navegação); o benefício nunca chega.

**Abstração errada**: quando o segundo caso real aparece, quase sempre ele **não encaixa** na abstração especulativa — refactor obrigatório.

**Bloqueio cognitivo**: leitor vê hierarquia e assume que há múltiplas implementações — tempo perdido entendendo algo que não existe.

**Acoplamento artificial**: dependências de interface prematuras obrigam tocar em N lugares para cada mudança simples.

## Exemplo

```csharp
// ❌ especulativo — 1 implementação, interface inútil
public interface IOrderProcessor { void Process(Order o); }
public class DefaultOrderProcessor : IOrderProcessor { ... }
// abstract base, strategy factory, config injection — tudo para 1 caso

// ✅ direto
public class OrderProcessor {
    public void Process(Order o) { ... }
}
// se aparecer 2ª variante, EXTRAIR então
```

## Como refatorar

- **Inline Class / Inline Method / Collapse Hierarchy**: remover camadas sem uso
- **Remove Parameter**: parâmetro não-usado sai
- **Rename Method**: métodos com sufixos `Base`, `Abstract`, `Generic` viram nomes concretos
- **Regra prática**: YAGNI + regra de três — abstrair só ao ver a 2ª/3ª ocorrência real
- Aceitar que extrair tarde é mais barato que prever errado

## Quando tolerar

- **Abstrações ditadas por framework** (interfaces que o DI container exige)
- **Fronteiras arquiteturais reais** (ports em Clean Arch) — DIP é princípio, não especulação; mas: só para ports que *realmente* trocam (DB, HTTP, provider externo)
- **Bibliotecas públicas** com contratos estáveis externos — extensibilidade é requisito, não especulação
- **Testes duplos (mocks/fakes)** que exigem interface

## Patterns relacionados (que resolvem)

- [[../methodology/gsd-get-shit-done]] — GSD + YAGNI
- [[../principles/solid]] — OCP/DIP aplicados **no momento certo**, não preventivamente
- [[../principles/do-dont-checklist]] — regra YAGNI
- [[long-method]] — oposto especular: complexidade explícita vs abstrata

## Fontes

- Refactoring Guru — [Speculative Generality](https://refactoring.guru/smells/speculative-generality)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2000). *Extreme Programming: A Gentle Introduction* — YAGNI.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../methodology/gsd-get-shit-done]]
- [[../principles/do-dont-checklist]]
