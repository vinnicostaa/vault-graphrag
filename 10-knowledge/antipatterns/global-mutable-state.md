---
title: Global Mutable State (AP-001)
type: antipattern
tags: [antipattern, design-smell, coupling, concurrency, state]
source: "Fowler, M. (2018). Refactoring 2nd ed. — Addison-Wesley"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [global-state, shared-mutable-state, singleton-abuse]
---

# AP-001 — Global Mutable State

Uso de variáveis, singletons ou contêineres estáticos acessíveis de qualquer parte do código que **mudam de valor em runtime**. Acopla tudo que toca, quebra isolamento de teste e introduz race conditions silenciosas.

## Sintomas

- Variáveis top-level mutáveis (`var config = ...`) lidas e escritas por N módulos
- Singletons com setters (`Logger.SetLevel(...)`, `DB.SetConnection(...)`)
- Testes falham em ordem aleatória (um contamina o próximo)
- Não consigo rodar dois testes em paralelo sem conflito
- Dependência invisível: função pura em assinatura, mas lê estado externo
- Mocks só funcionam via monkey-patching global

## Por que é problema

**Acoplamento escondido**: o estado global cria aresta entre todos os pontos que o lêem/escrevem sem aparecer no grafo de dependências. Mudar o contrato do estado obriga revisão global.

**Concorrência quebrada**: em runtimes com paralelismo real (Go, Rust, Java), leitura/escrita concorrente sem sincronização é UB. Mesmo em JS/Python single-thread, event loop interleaving produz bugs de ordem.

**Impossível testar unitariamente**: cada teste precisa resetar estado; ordem de execução passa a importar; paralelismo vira ilusão.

## Exemplo

```go
// ❌ estado mutável global
var CurrentUser *User

func Login(u *User) { CurrentUser = u }
func ChargeWallet(amount int) error {
    return CurrentUser.Wallet.Debit(amount) // quem é CurrentUser aqui?
}

// ✅ passar dependência explícita
func ChargeWallet(ctx context.Context, u *User, amount int) error {
    return u.Wallet.Debit(amount)
}
```

## Como refatorar

- **Injetar dependência**: parâmetro de função ou campo do struct/classe em vez de global
- **Request-scoped context**: propagar `ctx`/`Request` com valores específicos da chamada
- **Configuração imutável**: se realmente global, ler uma vez no boot, expor como valor imutável (sem setters)
- **Singleton só se estado é imutável + instanciação custosa**: caso legítimo é connection pool, não "estado atual do usuário"

## Quando tolerar

- **Logger** sem setters públicos em runtime de produção (inicializa no boot)
- **Connection pool** imutável após criação
- **Constantes derivadas de env** lidas no startup e congeladas
- **Métricas agregadas** (contadores atômicos, tipo Prometheus default registry) — mas aqui há sincronização explícita

## Patterns relacionados (que resolvem)

- [[../patterns/singleton]] — quando aplicado com disciplina (sem setters, lazy init controlado)
- [[../patterns/facade]] — encapsula subsistema com escopo local em vez de globals espalhados
- [[../patterns/strategy]] — injetar comportamento em vez de ler flag global
- [[../principles/solid]] — DIP: depender de interface passada, não de singleton concreto

## Fontes

- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.
- Martin, R. C. (2008). *Clean Code*. Cap. 6 (Objects and Data Structures) — discute acoplamento por estado compartilhado.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[../principles/solid]] — princípio DIP violado
- [[../principles/clean-code]]
