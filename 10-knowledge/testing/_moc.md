---
title: MOC — Testing
type: moc
tags: [moc, testing]
created: 2026-04-22
updated: 2026-04-24
---

# Testing

Filosofia (TDD, BDD), estratégia (pirâmide), tipos de testes (unit, integration, E2E, property-based, contract, mutation), thresholds.

## Notas

- [[testing-strategy]] — TDD filosofia, pirâmide, thresholds por camada, PBT, integration com containers, contract HTTP
- [[bdd-behavior-driven-development]] — Given-When-Then, Gherkin, Cucumber/godog, BDD vs TDD, outer/inner loop, living specification
- [[property-based-testing]] — invariantes sobre inputs gerados, shrinking, propriedades canônicas (identity/idempotency/round-trip), integração com fuzzing
- [[contract-testing]] — consumer-driven contracts (Pact), schema vs contract, Dredd + OpenAPI provider-side, `can-i-deploy`
- [[mutation-testing]] — operadores (AOR/ROR/UOI), Mutation Score, Stryker / go-mutesting / mutmut, quando usar e quando não

## A criar

- tdd-in-practice — red-green-refactor aplicado a task real
- testcontainers-patterns — reuso, truncate, parallelism
- coverage-thresholds-rationale — porque 95% domain vs 75% handler
- e2e-strategy — Playwright, smoke em produção, test data management

## Vizinhos

- [[../_moc]]
- [[../principles/do-dont-checklist]] — TDD obrigatório, regras duras
- [[../principles/clean-code]] — testes como código de produção
- [[../methodology/sdd-workflow]] — `<verify>` no plano casa com testes
- [[../methodology/gsd-get-shit-done]] — DoD explícito inclui coverage
- [[../observability/_moc]] — synthetic monitoring e smoke em prod são "testes vivos"
- [[../../20-stacks/_moc]] — implementações específicas (go-testing, typescript-testing, flutter-testing)
