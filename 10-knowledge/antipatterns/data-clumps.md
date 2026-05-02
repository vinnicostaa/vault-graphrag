---
title: Data Clumps (AP-006)
type: antipattern
tags: [antipattern, code-smell, bloater, cohesion]
source: "https://refactoring.guru/smells/data-clumps"
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [data-clump, repeated-fields]
---

# AP-006 — Data Clumps

Conjunto de variáveis/campos que **aparece sempre junto** em múltiplos lugares (argumentos de função, campos de classe, estruturas de DB) mas nunca foi nomeado como conceito único.

## Sintomas

- Quatro+ funções com parâmetros `(street, city, state, zip)` sempre lado a lado
- Campos `startDate` e `endDate` em N classes sem um `DateRange`
- Remover um dos membros "quebra o sentido" — eles só fazem sentido juntos
- Copy-paste de validação para o mesmo grupo
- Query DB retorna o grupo completo; update modifica o grupo completo

## Por que é problema

**Conceito implícito sem nome**: o domínio tem a entidade `Address` ou `DateRange`, mas código finge que são campos soltos. Linguagem ubíqua não se materializa.

**Manutenção replicada**: formatar `Address` em 8 lugares; adicionar `country` obriga mexer em todos.

**Baixa coesão**: funções que recebem clumps tendem a virar procedurais ("calcular imposto com street+zip").

**Testes redundantes**: cada função testa validação dos mesmos 4 campos separadamente.

## Exemplo

```python
# ❌ data clump
def send_invoice(name, street, city, state, zip, amount): ...
def validate_shipping(street, city, state, zip): ...
def format_label(name, street, city, state, zip): ...

# ✅ extraído
@dataclass(frozen=True)
class Address:
    street: str
    city: str
    state: str
    zip: ZipCode

def send_invoice(name: str, addr: Address, amount: Money): ...
def validate_shipping(addr: Address): ...
def format_label(name: str, addr: Address): ...
```

## Como refatorar

- **Extract Class**: criar classe/struct com os campos agrupados
- **Introduce Parameter Object**: quando o clump aparece em assinaturas
- **Preserve Whole Object**: em vez de desmembrar a entidade em chamadas, passar ela inteira
- **Move Method**: métodos que operam sobre os campos migram para a nova classe
- Idealmente o agrupamento vira Value Object imutável (ver [[primitive-obsession]])

## Quando tolerar

- **Campos que passam juntos por coincidência**, sem invariante compartilhado (duas strings que só estão próximas no formulário)
- **Estruturas geradas** a partir de schema externo (OpenAPI, protobuf) — a geração dita a forma
- Quando a "extração" criaria uma classe com 2 campos usada em 1 lugar só — aguardar segunda ocorrência (regra de 3)

## Patterns relacionados (que resolvem)

- [[../patterns/builder]] — construção do agrupamento novo
- [[primitive-obsession]] — clump de primitivos vira VO
- [[long-parameter-list]] — clump em parâmetros explode assinatura
- [[anemic-domain-model]] — clump elevado a entidade rica
- [[../principles/solid]] — coesão / SRP

## Fontes

- Refactoring Guru — [Data Clumps](https://refactoring.guru/smells/data-clumps)
- Fowler, M. (2018). *Refactoring: Improving the Design of Existing Code*. 2nd ed. Addison-Wesley.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[primitive-obsession]]
- [[long-parameter-list]]
