---
title: God Object / Large Class (AP-002)
type: antipattern
tags: [antipattern, code-smell, bloater, srp]
source: "https://refactoring.guru/smells/large-class"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [god-class, large-class, blob]
---

# AP-002 — God Object / Large Class

Classe (ou módulo) que concentra responsabilidades de várias — vira "quem faz tudo". Conhecida como *God Class* ou *Blob*. Viola frontalmente SRP.

## Sintomas

- Arquivo com 500+ linhas, classe com 30+ métodos
- Nome genérico: `Manager`, `Controller`, `Helper`, `Service` sem escopo claro
- Métodos agrupam-se em cliques visíveis (um subset usa campos A-D, outro usa E-H) sugerindo ≥2 classes
- Toda feature nova adiciona mais um método aqui
- Difícil achar onde está a lógica — scroll vertical virou ferramenta de debug

## Por que é problema

**Testes lentos e frágeis**: montar a God Class exige N dependências; um bug em um método obriga revisar a classe toda.

**Merge conflicts constantes**: múltiplos devs tocam o mesmo arquivo porque toda feature passa por lá.

**Acoplamento acidental**: responsabilidades que deveriam evoluir independentes agora compartilham constantes, cache, locks — qualquer refactor quebra tudo.

**Bloqueia reuso**: você quer só 10% da classe mas precisa instanciar os 100%.

## Exemplo

```typescript
// ❌ God Object
class OrderManager {
  validateOrder(...) {}
  calculatePricing(...) {}
  applyDiscounts(...) {}
  chargePayment(...) {}
  sendConfirmationEmail(...) {}
  generateInvoicePDF(...) {}
  scheduleDelivery(...) {}
  updateInventory(...) {}
  logAuditTrail(...) {}
  refundOrder(...) {}
  // + 20 métodos privados
}

// ✅ responsabilidades separadas
class OrderValidator {}
class PricingCalculator {}
class PaymentCharger {}
class InvoiceGenerator {}
class DeliveryScheduler {}
class InventoryAdjuster {}
```

## Como refatorar

- **Extract Class**: identificar cliques de métodos+campos afins, mover para classe própria
- **Extract Subclass** quando há variações que justificam hierarquia
- **Move Method**: migrar métodos que usam mais campos de *outra* classe para lá
- **Decompose em use cases** (Clean Arch): cada use case/handler = uma operação
- Se for controller HTTP gigante: splitar por recurso ou por ação

## Quando tolerar

- **Data Transfer Objects** grandes com muitos campos e sem comportamento — aceitável se imutáveis
- **Configuration objects** agregadores lidos no boot
- **Adapters** para SDK externo com API grande — replicar a superfície é inevitável

## Patterns relacionados (que resolvem)

- [[../patterns/facade]] — expor subsistema coeso com interface enxuta
- [[../patterns/mediator]] — coordenar N pequenas classes em vez de God Object acoplando tudo
- [[../patterns/strategy]] — extrair algoritmos variáveis
- [[../patterns/command]] — transformar cada operação em objeto próprio
- [[../principles/solid]] — **SRP** diretamente violado
- [[../principles/code-calisthenics]] — regra "classe com no máximo 50 linhas"

## Fontes

- Refactoring Guru — [Large Class](https://refactoring.guru/smells/large-class)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Riel, A. (1996). *Object-Oriented Design Heuristics* — heurística clássica.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../principles/solid]]
- [[../principles/code-calisthenics]]
