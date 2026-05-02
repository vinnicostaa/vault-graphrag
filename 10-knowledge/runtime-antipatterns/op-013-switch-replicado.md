---
title: OP-013 — Switch/case gigante replicado em N lugares
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - duplication
  - ocp
id: OP-013
severity: High
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Replicated switch
  - Shotgun surgery by type
---

# OP-013 — Switch/case gigante replicado em N lugares

**Severidade**: High
**Categoria**: Duplicação

## Sintoma

O mesmo `switch (type)` aparece em `CreateXUseCase`, `UpdateXUseCase`, `GetPublicXUseCase`, `DeleteXUseCase`. Adicionar um novo tipo exige editar **N** arquivos — violando Open/Closed Principle. Típico *Shotgun Surgery* ([[../antipatterns/shotgun-surgery|AP-008]]).

## Por que é problema

- **Change preventer**: novo tipo quebra N pontos se um for esquecido.
- **Difícil verificar exhaustividade** quando a lista está espalhada.
- **Trial-and-error**: dev adiciona tipo novo, build passa, produção quebra no caminho não-coberto.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — switch duplicado em múltiplos use cases
// CreateVehicleUseCase.ts
switch (input.type) {
  case 'car':   await carRepo.create(input); break
  case 'truck': await truckRepo.create(input); break
  case 'bike':  await bikeRepo.create(input); break
}

// UpdateVehicleUseCase.ts — mesmo switch
switch (input.type) {
  case 'car':   await carRepo.update(input); break
  case 'truck': await truckRepo.update(input); break
  case 'bike':  await bikeRepo.update(input); break
}
```

## Fix preferido — Strategy Pattern com registry

```ts
interface VehicleStrategy {
  create(input: VehicleInput): Promise<Vehicle>
  update(id: string, input: VehicleInput): Promise<Vehicle>
  delete(id: string): Promise<void>
}

// Map imutável = registry central
const strategies: Record<VehicleType, VehicleStrategy> = {
  car:   new CarStrategy(carRepo),
  truck: new TruckStrategy(truckRepo),
  bike:  new BikeStrategy(bikeRepo),
}

// Use case genérico — adicionar novo tipo = nova Strategy, zero edit em use cases
class CreateVehicleUseCase {
  async execute(input: VehicleInput) {
    const strategy = strategies[input.type]
    if (!strategy) throw new ValidationError('tipo inválido')
    return strategy.create(input)
  }
}
```

## Alternativa

- **Polymorphism**: `Vehicle` como interface, subtipos implementam `create/update/delete` via dispatch natural. Funciona bem em linguagens OO clássicas; em Go/Rust/TS, Strategy + Map é idiomático.
- **Discriminated Unions** (sum types) com pattern matching exaustivo (TypeScript `never`, Rust `match`, Kotlin `sealed`) — compilador garante exhaustividade.

## Quando tolerar

Switch único, curto (≤ 3 casos), num só lugar, sobre tipo que raramente ganha novos valores. Comentário `// TODO: extrair Strategy se chegar ao 4º caso` ajuda manutenção futura.

## Relações

- **Patterns**: [[../patterns/strategy]] (fix canônico), [[../patterns/factory-method]] (criar Strategy certa), [[../antipatterns/shotgun-surgery|AP-008]] (code smell clássico relacionado)
- **OPs relacionados**: [[op-014-construtor-gigante]] (mesmo family — Strategy resolve ambos), [[op-015-validacao-repetida]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-013
- GoF — *Design Patterns*, Strategy Pattern
- Refactoring Guru — [Replace Conditional with Polymorphism](https://refactoring.guru/replace-conditional-with-polymorphism) (consultada 2026-04-24)

## Links

- [[_moc]]
- [[../patterns/strategy]]
- [[../antipatterns/shotgun-surgery]]
- [[../principles/solid]] — OCP, SRP
