---
title: Alternative Classes with Different Interfaces (AP-021)
type: antipattern
tags: [antipattern, code-smell, oo-abuser, polymorphism]
source: https://refactoring.guru/smells/alternative-classes-with-different-interfaces
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [divergent-names, same-behavior-different-api]
---

# AP-021 — Alternative Classes with Different Interfaces

Duas (ou mais) classes fazem **basicamente a mesma coisa** mas com **nomes/assinaturas diferentes**. Cliente não consegue trocar uma pela outra sem adaptador — perde polimorfismo e tem [[duplicate-code]] embaixo do tapete.

## Sintomas

- Classes com método semanticamente igual mas nomes divergentes: `Queue.enqueue()` vs `Buffer.push()` vs `List.addLast()`
- Duas classes com "load/fetch/get/retrieve" pra mesma operação
- Impossível escrever função que aceita `A | B` sem adaptador por causa de API
- Dev novo pergunta "qual eu uso, `Foo` ou `Bar`?" — e a diferença é só cosmética
- Testes copiam estrutura com nomes diferentes
- Review recorrente: "isso não é igual àquele que o time X fez?"

## Por que é problema

**Perda de polimorfismo**: se as classes não compartilham interface, cliente precisa de `if (thing instanceof A)` ou `switch` — anti-OO.

**Duplicação escondida**: mesma regra reimplementada de formas ligeiramente diferentes — fix e bug viajam em N cópias ([[duplicate-code]]).

**Ubiquitous language rompida** (DDD): mesmo conceito com 3 nomes diferentes espalhados pelo código sinaliza que o domínio não está claro.

**Refactor caro quando aparece necessidade real**: quando o time finalmente percebe "é a mesma coisa", unificar é mexer em todos os chamadores de ambos.

## Exemplo

```typescript
// ❌ Mesma semântica, APIs divergentes
class EmailNotifier {
    sendEmail(to: string, subject: string, body: string) { /* ... */ }
}
class SmsNotifier {
    dispatchMessage(phone: string, text: string) { /* ... */ }
}
class PushNotifier {
    pushToDevice(deviceId: string, title: string, body: string) { /* ... */ }
}

function notifyUser(u: User, msg: Msg) {
    if (u.channel === 'email') new EmailNotifier().sendEmail(u.email, msg.title, msg.body);
    else if (u.channel === 'sms') new SmsNotifier().dispatchMessage(u.phone, msg.title + ' ' + msg.body);
    else new PushNotifier().pushToDevice(u.deviceId, msg.title, msg.body);
}

// ✅ Interface comum
interface Notifier { send(recipient: Recipient, msg: Msg): Promise<void>; }
class EmailNotifier implements Notifier { send(r, m) { /* ... */ } }
class SmsNotifier   implements Notifier { send(r, m) { /* ... */ } }
class PushNotifier  implements Notifier { send(r, m) { /* ... */ } }

function notifyUser(n: Notifier, r: Recipient, m: Msg) { return n.send(r, m); }
```

## Como refatorar

- **Rename Method**: unificar nomes que fazem a mesma coisa (`fetchUser` + `loadUser` → `findUser`)
- **Move Method / Add Parameter**: alinhar assinaturas antes de extrair interface
- **Extract Interface / Extract Superclass**: uma vez que os métodos combinam em nome e assinatura, extrair contrato comum
- **Form Template Method**: quando variam em alguns passos mas o esqueleto é igual
- **Strategy**: usar interface comum + múltiplas implementações intercambiáveis

## Quando tolerar

- **Semântica genuinamente diferente** apesar de nomes parecidos — não unificar à força. `HttpClient.send` vs `SmsClient.send` podem parecer irmãos mas têm contratos muito distintos.
- **Estágio inicial de design** — é cedo pra extrair interface comum sem ter evidência de 2+ casos reais ([[speculative-generality]])
- **Libs externas** com APIs legadas que você não controla — usar Adapter pra dar interface comum, não forçar mudança
- **Bounded contexts diferentes** (DDD) — mesmo conceito pode ter APIs diferentes em contextos distintos porque o **significado** é diferente

## Patterns relacionados (que resolvem)

- [[duplicate-code]] — smell quase sempre presente junto
- [[../patterns/strategy]] — interface comum, implementações intercambiáveis
- [[../patterns/template-method]] — esqueleto comum, passos variáveis
- [[../patterns/adapter]] — unificar APIs que você não pode mudar
- [[../architecture/ddd-strategic-tactical]] — ubiquitous language consolidada
- [[../principles/solid]] — DIP, OCP

## Fontes

- Refactoring Guru — [Alternative Classes with Different Interfaces](https://refactoring.guru/smells/alternative-classes-with-different-interfaces)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Gamma et al. (1994). *Design Patterns* — Strategy, Template Method, Adapter.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[duplicate-code]]
- [[../patterns/strategy]]
