---
title: Mutation Testing — validando a qualidade dos testes
type: concept
tags: [testing, mutation-testing, coverage, quality]
created: 2026-04-24
updated: 2026-04-24
aliases: [Mutation testing, Teste de mutação]
---

# Mutation Testing — validando a qualidade dos testes

Cobertura de linhas diz **quanto** do código foi executado. Mutation testing diz **se** os testes realmente verificam comportamento. É o teste do teste.

## O que é

Processo em três passos, automatizado pela ferramenta:

```
1. Mutar o código-fonte em N variantes (cada mutante = 1 alteração pequena)
2. Rodar a suíte de testes contra cada mutante
3. Classificar:
     - killed   → pelo menos 1 teste falhou   (suite detectou)
     - survived → todos testes passaram       (suite NÃO detectou)
     - timeout  → mutação gerou loop infinito
     - no-coverage → linha nem é executada
```

Mutante **survived** é o alarme: alguém alterou a lógica e nenhum teste reclamou. A suite está cega naquele ponto.

## Por que cobertura tradicional é insuficiente

Linha coberta por teste sem `assert` é coberta de fachada. Em vários ecossistemas, é trivial chegar a 100% line coverage só exercitando o código — sem verificar nada.

```go
// código
func Discount(amount int, pct int) int {
    return amount - (amount * pct / 100)
}

// teste "coberto" que não testa nada
func TestDiscount(t *testing.T) {
    _ = Discount(100, 10) // 100% line coverage, 0% valor
}
```

Mutation testing gera variantes como `return amount + (amount * pct / 100)` ou `return amount - (amount * pct / 99)`. Se o teste acima continua verde, a suíte está mentindo sobre qualidade.

## Métrica: Mutation Score

```
MutationScore = killed / (total - no-coverage - timeout)
```

- **90-100%** — excelente, típico de módulos críticos maduros
- **70-90%** — bom em código recente; investigar survivors
- **< 60%** — cobertura cosmética; suíte precisa de asserts reais

Complemento (não substituto) de line coverage: combinar `coverage ≥ threshold` + `mutation score ≥ threshold`.

## Ferramentas por stack

| Stack | Ferramenta |
|---|---|
| JS / TS | **Stryker** (`@stryker-mutator/core`) |
| Go | **go-mutesting**, **ooze** |
| Python | **mutmut**, **cosmic-ray** |
| PHP | **Infection** |
| Java / Kotlin | **PITest** |
| Rust | **cargo-mutants** |
| .NET | **Stryker.NET** |

Rodar em pipeline **nightly** ou em PRs que tocam módulos críticos — é caro (roda a suíte N vezes). Não bloquear PR rápido; relatar divergência em canal de quality.

## Operadores de mutação clássicos

| Sigla | Operador | Exemplo |
|---|---|---|
| **AOR** | Arithmetic Operator Replacement | `a + b` → `a - b` |
| **ROR** | Relational Operator Replacement | `x >= y` → `x > y` |
| **COR** | Conditional Operator Replacement | `a && b` → `a \|\| b` |
| **UOI** | Unary Operator Insertion | `x` → `-x`, `!x` |
| **LOI** | Logical Operator Insertion | `x` → `~x` (bitwise) |
| **COI** | Conditional Operator Insertion | `if (x)` → `if (!x)` |
| **LCR** | Logical Connector Replacement | `true` → `false`, remover guard |
| **SDL** | Statement Deletion | remover linha inteira |

Ferramentas modernas aplicam vários em paralelo e filtram mutantes equivalentes (que não mudam semântica).

## Quando usar

- **Código crítico**: regras monetárias, controle de acesso, state machines
- **Libs/SDKs públicos**: quebrar contrato é caro
- **Core de domínio**: regras de negócio que não podem deslizar
- **Antes de remover teste "redundante"**: se o mutation score cai, o teste não era redundante

## Quando não compensa

- **UI-heavy** de componentes visuais (snapshot/E2E cobrem melhor)
- **Código performance-bound** com microbench crítico (mutantes distorcem timing)
- **Scripts descartáveis** e código exploratório
- **Configuração pura** sem lógica

## Workflow prático

1. Rodar mutation na baseline atual — registrar score
2. Focar em **survivors em código recém-modificado** (diff-driven)
3. Para cada survivor: escrever assert que mata aquele mutante
4. Proibir **queda** de mutation score entre PRs do módulo crítico
5. Revisar mutantes **equivalentes** (falso-positivos) manualmente a cada rodada

## Antipatterns

- Perseguir **100% mutation score** cegamente — retorno decrescente, mata produtividade
- Rodar mutation **a cada commit** de tudo — custo proibitivo; use diff-scope
- Acreditar em mutation score **sem** line coverage — código zero-coverage tem score vazio
- Ignorar survivors "porque o teste E2E pega" — E2E raramente mata mutante sutil de lógica

## Links

- [[_moc]]
- [[testing-strategy]]
- [[property-based-testing]]
- [[../principles/do-dont-checklist]]
- [[../antipatterns/_moc]]
