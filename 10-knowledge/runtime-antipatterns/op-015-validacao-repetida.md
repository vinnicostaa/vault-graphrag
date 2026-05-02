---
title: OP-015 — Validação repetida em N métodos
type: antipattern
tags:
  - antipattern
  - runtime
  - operational
  - duplication
  - dry
id: OP-015
severity: Medium
created: 2026-04-24
updated: 2026-04-24
doc-consulted: 2026-04-24
aliases:
  - Duplicate validation
  - Check copy-paste
---

# OP-015 — Validação repetida em N métodos

**Severidade**: Medium
**Categoria**: Duplicação

## Sintoma

`createA`, `createB`, `createC` cada um tem 10 linhas validando unicidade de documento (CPF/CNPJ/email) com pequenas diferenças. DRY violado — fix/mudança em uma regra obriga editar todas as N versões. Variação comum de [[../antipatterns/duplicate-code|AP-011]].

## Por que é problema

- **Bug drift**: correção aplicada em 2 de 3 lugares — o 3º continua quebrado em produção.
- **Teste custoso**: cada use case precisa cobrir casos da mesma validação.
- **Evolução lenta**: adicionar regra nova (ex.: blocklist) exige N patches.

## Exemplo (anti-pattern)

```ts
// ❌ ERRADO — validação duplicada com variações sutis
class CreateUserUseCase {
  async execute(input) {
    if (!isCPFValid(input.doc)) throw new ValidationError('CPF inválido')
    const existing = await userRepo.findByDoc(input.doc)
    if (existing) throw new ConflictError('CPF já cadastrado')
    // ...
  }
}

class CreateVendorUseCase {
  async execute(input) {
    if (!isCNPJValid(input.doc)) throw new ValidationError('CNPJ inválido')
    const existing = await vendorRepo.findByDoc(input.doc)
    if (existing) throw new ConflictError('CNPJ já cadastrado')
    // ...
  }
}
// CreatePartner, CreateSupplier... mesma estrutura replicada
```

## Fix preferido — helper/service de validação

```ts
// Helper centralizado
async function validateDocumentUniqueness(
  doc: string,
  type: 'cpf' | 'cnpj',
  repo: DocRepository,
): Promise<void> {
  const validator = { cpf: isCPFValid, cnpj: isCNPJValid }[type]
  if (!validator(doc)) {
    throw new ValidationError(`${type.toUpperCase()} inválido`)
  }
  const existing = await repo.findByDoc(doc)
  if (existing) {
    throw new ConflictError(`${type.toUpperCase()} já cadastrado`)
  }
}

// Use cases finos
class CreateUserUseCase {
  async execute(input) {
    await validateDocumentUniqueness(input.doc, 'cpf', this.userRepo)
    // ...
  }
}
```

## Alternativa

- **Value Object** (`CPF`, `CNPJ`) que valida na factory — elimina `isCPFValid(...)` espalhado. Continua precisando do check de unicidade no handler/UC, mas normalizado.
- **Specification Pattern**: regras complexas compostas por specs (`CPFValid().and(NotInBlocklist()).and(Unique(repo))`).

## Quando tolerar

Duplicação de 2 linhas em 2 use cases pode ficar. Regra de 3: a partir do 3º caso idêntico, extrair.

## Relações

- **Patterns**: Value Object, Specification Pattern, [[../patterns/strategy]] (pra por tipo)
- **OPs relacionados**: [[op-013-switch-replicado]] (mesma família), [[op-011-logica-negocio-infra]] (quando helper vai pra infra)
- **Code smells clássicos**: [[../antipatterns/duplicate-code|AP-011]]

## Fontes

- [[../../40-templates/AGENTS_SPEC]] §4 OP-015
- Fowler — *Refactoring 2nd ed.*, Extract Function
- [[../principles/do-dont-checklist]] — DRY, YAGNI, KISS

## Links

- [[_moc]]
- [[../antipatterns/duplicate-code]]
- [[../principles/do-dont-checklist]]
- [[op-013-switch-replicado]]
