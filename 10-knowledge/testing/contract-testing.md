---
title: Contract Testing — consumer-driven contracts entre serviços
type: concept
tags: [testing, contract-testing, integration, api]
created: 2026-04-24
updated: 2026-04-24
aliases: [CDC, Consumer-driven contracts, Pact testing]
---

# Contract Testing — consumer-driven contracts entre serviços

Testar o **contrato** (request/response) entre dois serviços sem precisar de ambos vivos ao mesmo tempo. Substitui boa parte do teste integrado caro por testes rápidos e desacoplados, sem perder garantia de compatibilidade.

## Problema que resolve

Microsserviços clássicos:

- Teste unitário de cada serviço: rápido, verde, mas não garante que `serviceA → serviceB` funciona
- Teste integrado full-stack: lento, flaky, acopla deploys
- Quando `serviceB` muda o contrato, `serviceA` só descobre em produção

Contract testing intermedeia: cada lado testa contra o **contrato**, não contra o outro serviço em execução.

## Consumer-Driven Contracts (CDC)

Fluxo canônico (exemplificado pelo **Pact**):

```
CONSUMER (ex: mobile app)
  ├── escreve teste "quando eu chamar GET /users/42, espero X"
  ├── framework grava o request e mock da response esperada
  └── gera PACT FILE (JSON)  ──────────────────┐
                                               │
                            publish            ▼
                           ┌────────── PACT BROKER ──────────┐
                           │ versionamento, matrix compat    │
                           └─────────────────┬────────────────┘
                                             │
PROVIDER (ex: users-service)                 │
  ├── recebe pact file                       │
  ├── replay: roda o request contra build    │
  ├── verify: response real casa com mock?   │
  └── emite resultado: verified / failed
```

Consumer **dita** o que precisa; provider **verifica** que entrega. Inverte a direção do contrato tradicional (spec → implementação) e garante que o consumer não está assumindo algo que o provider nunca prometeu.

## Schema tests vs Contract tests

| Dimensão | Schema test | Contract test |
|---|---|---|
| O que valida | Formato do payload (JSON Schema, OpenAPI) | **Interação**: request real + response real |
| Visão | Estrutural | Comportamental (incluindo semântica de status, headers) |
| Sabe o que o consumer usa? | Não | Sim — só os campos consumidos |
| Quebra por campo novo? | Não (compatível) | Não (consumer ignora) |
| Quebra por remover campo usado? | Talvez | **Sim** — é o ponto |

**Ambos convivem**: schema protege formato no runtime; contract protege uso real entre partes.

## Ferramentas

| Ferramenta | Público |
|---|---|
| **Pact** (pact-foundation) | CDC canônico, multi-linguagem (JS, Go, Java, Rust, Python, .NET, Ruby) |
| **Spring Cloud Contract** | JVM, stubs gerados |
| **Dredd** | Valida server contra **OpenAPI** ou API Blueprint (provider-side) |
| **Prism** (Stoplight) | Mock server + contract validation a partir de OpenAPI |
| **Schemathesis** | Property-based + OpenAPI (gera requests e valida contra spec) |

## Provider-side com OpenAPI + Dredd

Quando a spec OpenAPI é fonte da verdade (ver [[../architecture/openapi-contract-first]]), Dredd roda:

1. Lê `openapi.yaml`
2. Sobe servidor local
3. Para cada operação: dispara request de exemplo, compara response com schema
4. Reporta divergência

Útil em microsserviços com **muitos consumers desconhecidos** (API pública) onde CDC não escala — OpenAPI vira contrato unilateral estável.

## Benefícios vs teste integrado pesado

- **Desacoplado**: provider e consumer testam em CIs separadas
- **Rápido**: pact file é JSON leve, replay é síncrono
- **Versionado**: broker mantém histórico, faz matrix `consumer-vX vs provider-vY`
- **Deploy independente**: "este provider é seguro pra deploy? → verificar pact dos consumers em produção"
- **Detecção cedo**: quebra no CI do provider, não em staging

## Quando usar

- **Múltiplos consumers** de um mesmo provider (ex: mobile + web + outro serviço)
- **Microsserviços com deploys independentes** — Consumer-Driven Contracts (Ian Robinson)
- **API pública / SDK** — Dredd + OpenAPI + golden tests
- **Legacy com integração flaky** — reduzir matrix de testes integrados

## Quando **não** compensa

- **Monolito** único — unit + integration cobrem melhor
- **Único consumer + único provider** no mesmo time — overhead > benefício
- **Integração via fila assíncrona** sem contrato de mensagem estável — Pact suporta, mas começar mais simples

## Antipatterns

- **Provider escreve pact pelo consumer**: destrói o "consumer-driven" do CDC. Cada consumidor escreve os próprios pacts.
- **Pact file desatualizado no broker**: consumer atualizou mas não publicou — provider passa em falsas premissas. Fix: pact publish no pipeline do consumer.
- **Pact cobrindo lógica de negócio**: pact valida forma do contrato, não regra (se o saldo é calculado certo). Lógica → unit no provider.
- **Consumer com mocks inventados sem pact**: mock diverge do provider real. Ou escreve pact, ou não mocka.
- **Pact cobrindo 100% dos endpoints**: custo > valor. Cobrir **o que o consumer realmente consome**.

## Integração com CI/CD

```
consumer CI:
  └── test → pact publish   (broker)

provider CI:
  └── pact verify (pega latest pact de cada consumer)
  └── resultado em broker
  └── can-i-deploy provider-vNEW contra consumers-atuais ⇒ sim/não
```

`pact-broker can-i-deploy` é o gate de deploy — só publica provider se os consumers declaradamente aceitam a versão.

## Links

- [[_moc]]
- [[testing-strategy]]
- [[property-based-testing]]
- [[../architecture/openapi-contract-first]]
- [[../architecture/_moc]]
- [[../antipatterns/_moc]]
