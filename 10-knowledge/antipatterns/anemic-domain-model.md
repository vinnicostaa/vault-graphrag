---
title: Anemic Domain Model (AP-003)
type: antipattern
tags: [antipattern, design-smell, ddd, architecture]
source: "Fowler, M. (2003). AnemicDomainModel — martinfowler.com/bliki/AnemicDomainModel.html"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [anemic-model, data-class-domain]
---

# AP-003 — Anemic Domain Model

Entidades de domínio reduzidas a *bags* de getters/setters — toda regra de negócio vive em serviços externos (`OrderService.cancel(order)` em vez de `order.cancel()`). Parece OO, mas é procedural disfarçado. Termo cunhado por Martin Fowler.

## Sintomas

- `Order`, `User`, `Invoice` são classes só com campos públicos ou getter/setter
- A lógica mora em `*Service`, `*Manager`, `*Handler` que recebem a entidade e manipulam
- Invariantes validados no service, não na entidade
- Entidade entra em estado inválido "de fora" — qualquer setter pode corromper
- Linguagem ubíqua fragmentada: a regra "cancelar só se status = pending" fica escrita em docs, checagens espalhadas em serviços, testes — nunca na entidade

## Por que é problema

**Lógica espalhada**: a regra `order.cancel()` fica replicada em `OrderService.cancel`, `AdminService.cancelOrder`, `BatchCancelJob` — três implementações que divergem.

**Invariantes frágeis**: sem comportamento na entidade, qualquer código pode `order.status = "cancelled"` direto, pulando validação.

**DDD morto**: o modelo "rico" do DDD exige agregados que encapsulam regras. Anemic quebra a premissa central e transforma DDD em CRUD glorificado.

**Testes ruins**: testar `Order.cancel` exige montar service + mocks. Se a regra morasse na entidade, seria teste unitário puro.

## Exemplo

```go
// ❌ Anemic
type Order struct {
    ID     string
    Status string
    Total  int
}
func (o *Order) SetStatus(s string) { o.Status = s }

func (s *OrderService) Cancel(o *Order) error {
    if o.Status != "pending" { return ErrInvalidState }
    o.SetStatus("cancelled")
    return s.repo.Save(o)
}

// ✅ Rich — regra mora onde dados moram
type Order struct {
    id     OrderID
    status OrderStatus
    total  Money
}
func (o *Order) Cancel() error {
    if !o.status.IsPending() {
        return ErrOrderNotPending
    }
    o.status = OrderStatusCancelled
    return nil
}
```

## Como refatorar

- **Move Method**: migrar lógica de `XService.doY(x)` para `X.doY()`
- **Tell, Don't Ask**: em vez de ler campos e decidir fora, chamar método na entidade
- **Encapsular invariantes no construtor/factory**: se validação roda ao criar, estado inválido é impossível
- **Value Objects** para primitivos com regra (ver [[primitive-obsession]])
- Serviço de domínio só quando a lógica **não** pertence a uma entidade única (ex: transferência entre contas)

## Quando tolerar

- **DTOs de transporte** (request/response body, kafka events) — esses devem ser bags de dados por design
- **Read Models (CQRS)** — projeções só de leitura, sem invariantes
- **Read-only reports** onde não há comportamento a encapsular
- **Integração com ORMs** que exigem classes com setters — isolar a entidade de persistência do agregado de domínio

## Patterns relacionados (que resolvem)

- [[../architecture/ddd-strategic-tactical]] — agregados, entities, value objects ricos
- [[../patterns/strategy]] — variações de comportamento dentro da entidade
- [[../patterns/state]] — transições de estado como objetos
- [[../patterns/factory-method]] — criar entidade já em estado válido
- [[../principles/solid]] — SRP e Tell-Don't-Ask

## Fontes

- Fowler, M. *AnemicDomainModel* — https://martinfowler.com/bliki/AnemicDomainModel.html
- Evans, E. (2003). *Domain-Driven Design: Tackling Complexity in the Heart of Software*. Addison-Wesley.
- Vernon, V. (2013). *Implementing Domain-Driven Design*. Addison-Wesley.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[primitive-obsession]]
- [[../principles/solid]]
