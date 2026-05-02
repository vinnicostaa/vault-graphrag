---
title: Shotgun Surgery (AP-008)
type: antipattern
tags: [antipattern, code-smell, change-preventer, coupling]
source: "https://refactoring.guru/smells/shotgun-surgery"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [scattered-change, buckshot-refactor]
---

# AP-008 — Shotgun Surgery

Uma mudança conceitual simples obriga alterar **pedaços pequenos em muitas classes/arquivos**. Oposto do *Divergent Change*: lá uma classe muda por muitos motivos; aqui um motivo obriga mudar muitas classes.

## Sintomas

- Adicionar um campo novo a `User` exige editar 15 arquivos (mapper, DTO, migration, controller, service, test, ...)
- Trocar formato de moeda obriga passar por toda camada
- Regra de negócio replicada em N lugares, cada um com pequena diferença
- Commits típicos: "small change" tocando 20 arquivos
- Toda nova feature exige tour pelo código inteiro

## Por que é problema

**Alto custo de mudança**: features simples viram épicos. Roadmap vira refém do acoplamento.

**Risco de inconsistência**: esquece um dos 15 lugares → bug sutil em produção (regra aplicada só em 14/15).

**Testes frágeis**: cada ponto de mudança tem seu próprio teste; cobertura por caminho explode.

**Symptom de baixa coesão de conceito**: a "coisa que muda junto" não está junta no código.

## Exemplo

```typescript
// ❌ Shotgun — mudar taxa de IVA obriga editar tudo
class InvoiceCalculator { total() { return sub * 1.23 } }
class ReportService   { render() { ... sub * 1.23 ... } }
class EmailFormatter  { body() { ... sub * 1.23 ... } }
class TaxForm         { fill() { ... sub * 1.23 ... } }

// ✅ concentrado
class TaxPolicy { static VAT = 0.23; static apply(m: Money) { ... } }
// todos passam por TaxPolicy.apply()
```

## Como refatorar

- **Move Method / Move Field**: concentrar em uma só classe/módulo
- **Inline Class**: se a fragmentação for herança abusiva
- **Introduce Policy / Strategy**: regra que varia vira objeto único ([[../patterns/strategy]])
- **Mediator**: coordenar mudança via ponto central em vez de N-a-N ([[../patterns/mediator]])
- **Encapsulate Field**: se a mesma regra de acesso a um campo estava replicada

## Quando tolerar

- **Mudanças estruturais reais cross-cutting** (upgrade de framework, migração de lib) — aí a dispersão reflete o impacto real
- **Schemas contratuais** (OpenAPI + handler + test fixture) — tocar os três é o design esperado
- **Propagação de feature-flag** em rollout — temporário e rastreável

## Patterns relacionados (que resolvem)

- [[../patterns/mediator]] — ponto central de coordenação
- [[../patterns/strategy]] — extrair política variável
- [[../patterns/facade]] — esconder subsistema
- [[duplicate-code]] — shotgun surgery vem geralmente de duplicação
- [[../principles/solid]] — OCP diretamente violado

## Fontes

- Refactoring Guru — [Shotgun Surgery](https://refactoring.guru/smells/shotgun-surgery)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../patterns/mediator]]
- [[duplicate-code]]
