---
title: BDD — Behavior-Driven Development
type: concept
tags: [testing, bdd, tdd, specification, gherkin, cucumber]
source: Dan North, "Introducing BDD" (2006)
created: 2026-04-23
aliases: [BDD, Behavior-Driven Development, Gherkin, Given-When-Then]
---

# BDD — Behavior-Driven Development

**BDD** (Dan North, 2006) é evolução de TDD que substitui vocabulário de **teste** por vocabulário de **comportamento de negócio**. Produz especificações executáveis — texto legível por stakeholders que **também** é suíte de testes automatizada.

> **Origem**: North notou que desenvolvedores fazendo TDD escreviam testes com nomes ruins ("`testCase42`"), perdendo a intenção. Mudou o vocabulário ("**Given**-**When**-**Then**") para forçar foco no comportamento que o negócio entende.

## BDD vs TDD — diferença real

Ambos são **test-first**. A diferença está na **linguagem** e no **público**.

| Dimensão | TDD (clássico) | BDD |
|---|---|---|
| Foco | Implementação correta | Comportamento observável |
| Linguagem | Técnica (método/classe) | Negócio (feature/cenário) |
| Leitura | Só dev | Dev + PO + QA + cliente |
| Granularidade | Unit test (um método) | Scenario (um fluxo) |
| Ferramenta típica | xUnit (Go `testing`, Jest, pytest) | Cucumber, Godog, Behat, SpecFlow |

Não são mutuamente exclusivos. **TDD para unidade + BDD para fluxo de negócio** é combinação comum.

## A sintaxe Gherkin

> Exemplo ilustrativo no domínio de governança condominial. O padrão Gherkin se aplica a qualquer domínio (e-commerce, banking, saúde, logística) — a estrutura **Given/When/Then** é universal.

```gherkin
Feature: Síndico pode cancelar assembleia antes da data marcada
  Como síndico
  Quero cancelar uma assembleia não iniciada
  Para que moradores parem de receber lembretes

  Scenario: Cancelamento com 24h de antecedência
    Given uma assembleia marcada para daqui a 48h
    And o síndico é o autor da assembleia
    When ele submete o cancelamento com motivo "quórum mínimo não atingido"
    Then a assembleia fica com status "cancelada"
    And todos os moradores convocados recebem notificação
    And o cancelamento é registrado na timeline imutável

  Scenario: Cancelamento proibido após início
    Given uma assembleia em andamento
    When o síndico tenta cancelar
    Then a operação é rejeitada com erro "assembleia_em_curso"
    And o status permanece "em_andamento"
```

**Given**: estado inicial (fixtures, setup)
**When**: ação (command, event, request)
**Then**: consequência observável (asserts)

Também:
- **And**: continuação de qualquer seção acima
- **Background**: `Given` comum a todos os scenarios da feature
- **Scenario Outline**: tabela de exemplos pra reusar o mesmo cenário com dados diferentes

## Mecânica — como vira código

Cada linha Gherkin se liga a um **step definition** em código:

```go
// godog exemplo (Go)
func aAssemblyMarcadaParaDaquiA(ctx context.Context, hours int) error {
    scheduledFor := time.Now().Add(time.Duration(hours) * time.Hour)
    assembly, err := assemblyRepo.Create(ctx, scheduledFor, ...)
    if err != nil { return err }
    testCtx.assembly = assembly
    return nil
}

func oSindicoSubmeteCancelamento(ctx context.Context, motivo string) error {
    return cancelUseCase.Execute(ctx, CancelAssemblyCommand{
        AssemblyID: testCtx.assembly.ID(),
        Reason:     motivo,
        ActorID:    testCtx.syndic.ID(),
    })
}

func aAssembleaFicaComStatus(ctx context.Context, status string) error {
    current, _ := assemblyRepo.FindByID(ctx, testCtx.assembly.ID())
    if string(current.Status()) != status {
        return fmt.Errorf("esperado %s, obtido %s", status, current.Status())
    }
    return nil
}

// Binding
func InitializeScenario(s *godog.ScenarioContext) {
    s.Step(`^uma assembleia marcada para daqui a (\d+)h$`, aAssemblyMarcadaParaDaquiA)
    s.Step(`^ele submete o cancelamento com motivo "([^"]+)"$`, oSindicoSubmeteCancelamento)
    s.Step(`^a assembleia fica com status "([^"]+)"$`, aAssembleaFicaComStatus)
}
```

**Regra dura**: step definitions são **finas** — só orquestram use case + fixtures. Lógica de negócio **não** entra no step.

## Ferramentas por stack

| Stack | Ferramenta | Observação |
|---|---|---|
| Go | [godog](https://github.com/cucumber/godog) | Cucumber oficial pra Go |
| TypeScript/Node | [cucumber-js](https://github.com/cucumber/cucumber-js) | Maturo |
| Python | [behave](https://behave.readthedocs.io/) | Gherkin completo |
| Ruby | [cucumber-ruby](https://cucumber.io/docs/installation/ruby/) | Original |
| Java/Kotlin | [cucumber-jvm](https://github.com/cucumber/cucumber-jvm) | Padrão enterprise |
| Rust | [cucumber-rust](https://github.com/cucumber-rs/cucumber) | Bem mantido |
| PHP | [Behat](https://docs.behat.org/) | — |
| C#/.NET | SpecFlow | — |

## Quando BDD agrega

- **Domínios complexos** com regras de negócio que stakeholders revisam frequentemente
- **Equipes cross-funcionais** (PO escreve feature; dev implementa step)
- **Compliance/regulação** onde rastrear regra-requisito-teste é obrigatório (LGPD, financeiro, saúde)
- **Específicações vivas** como documentação substitui docs estáticas desatualizadas

## Quando BDD **não** agrega (e atrapalha)

- **Unit testing puro** (um método, um cálculo) — overkill, use TDD xUnit
- **Equipe pequena, todo mundo dev** — ninguém lê Gherkin fora de dev; virar duplicação de esforço
- **Rotatividade de steps** muito alta — manter step catalog vira custo maior que o benefício
- **API interna sem stakeholders de negócio** — negócio é "dev cross-team" também, mas Gherkin não agrega valor

**Anti-pattern comum**: "BDD lite" onde dev escreve Gherkin sozinho e o PO nunca lê. **Sem o PO, é TDD com sintaxe chata**.

## BDD **sem** Gherkin (alternativa)

Pode-se ter espírito BDD sem a ferramenta Cucumber: **nomes de teste descritivos** como feature narrativa.

```go
// ✅ Espírito BDD em xUnit Go
func TestCancelAssembly_WhenUserIsSyndicAndAssemblyNotStarted_SetsStatusToCanceled(t *testing.T) {
    // Given
    assembly := fixtures.ScheduledAssembly(t, 48*time.Hour)
    syndic := fixtures.SyndicOf(t, assembly.CondominiumID())

    // When
    err := cancelUseCase.Execute(ctx, CancelAssemblyCommand{
        AssemblyID: assembly.ID(),
        ActorID:    syndic.ID(),
        Reason:     "quórum mínimo não atingido",
    })

    // Then
    require.NoError(t, err)
    updated, _ := repo.FindByID(ctx, assembly.ID())
    assert.Equal(t, AssemblyStatusCanceled, updated.Status())
}
```

Essa é a abordagem **mais comum** em equipes sem ator de negócio envolvido: **Given/When/Then visíveis nos comentários**, nomes de teste contam a história, sem custo de ferramenta.

## BDD + TDD — ciclo combinado

```
1. BDD outer loop (feature):
   Escreve .feature file + steps vazios → falha ("not implemented")

2. TDD inner loop (por unit):
   Dentro de cada step, implementa use case / aggregate / repo
   com TDD clássico (red → green → refactor)

3. BDD outer loop:
   Quando todos os steps verdes → feature completa → commit
```

**Outer/inner loop** é nome canônico. Outer garante **valor para negócio**, inner garante **implementação correta**.

## Regras de ouro

- **Feature file fala de domínio**, não de tela nem de tabela. "Given o síndico é o autor" (domínio), nunca "Given INSERT INTO assemblies VALUES (...)" (infra).
- **Step definitions em pt-BR** quando feature é em pt-BR (consistência de leitura).
- **Não compartilhar estado** entre scenarios via variável global — cada scenario isolado.
- **Fixtures reutilizáveis**: `fixtures.ScheduledAssembly(t, duration)` em vez de SQL solto.
- **Assertividade específica**: `assert.Equal("canceled", ...)` melhor que `assert.True(...)`.

## Living specification

BDD vira **documentação viva** quando:
- `.feature` files no repositório, revisados como código
- CI roda suíte BDD; vermelho bloqueia merge
- PR inclui novo `.feature` quando feature de produto muda
- Builds publicam relatório HTML legível (cucumber-html-reporter)

## Anti-patterns

- **Step genérico demais**: `Given um usuário "X"` com X variando ad-hoc — perde legibilidade
- **Cenário longo** (> 10 linhas): dividir
- **Asserção vaga**: `Then operação funciona` — o que "funciona" significa?
- **Mock de domínio**: BDD testa fluxo real; se mockar aggregate, é TDD com overhead
- **Feature sem Background comum** repetindo `Given` em todos os cenários
- **Gherkin em inglês com código em pt-BR** (ou vice-versa): inconsistência de idioma

## Checklist

- [ ] Feature file narra comportamento, não implementação?
- [ ] Given/When/Then têm um único verbo cada?
- [ ] Steps são reutilizáveis entre cenários?
- [ ] Cenários isolados (sem dependência de ordem)?
- [ ] CI bloqueia em feature vermelha?
- [ ] PO revisa `.feature` files?

## Referências

- Dan North. [Introducing BDD](https://dannorth.net/introducing-bdd/) (2006)
- Matt Wynne, Aslak Hellesøy. *The Cucumber Book* (2012)
- Gojko Adzic. *Specification by Example* (2011) — extensão conceitual de BDD
- *50 Quick Ideas to Improve Your User Stories* — Adzic (2014)

## Links

- [[_moc]]
- [[testing-strategy]]
- [[../principles/clean-code]]
- [[../architecture/ddd-strategic-tactical]]
- [[../methodology/sdd-workflow]]
- [[../methodology/spec-driven-development]]
