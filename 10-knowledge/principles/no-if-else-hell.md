---
title: No if/else hell — extrair estratégias antes de aninhar
type: concept
tags: [principle, control-flow, strategy, polymorphism]
created: 2026-04-24
updated: 2026-04-24
aliases: [If-else hell, Nested conditionals]
---

# No if/else hell — extrair estratégias antes de aninhar

Aninhamento de `if/else` ≥ 3 níveis é sinal dado pela linguagem: **está faltando extrair**. Pode ser early return, strategy, tabela de dispatch ou polimorfismo. Aceitar o aninhamento é aceitar complexidade cognitiva multiplicativa.

## Sinais de problema

- `if` dentro de `if` dentro de `if`
- `else if` com 5+ ramos para tipos conhecidos
- Indentação que empurra código relevante para a coluna 40
- Função que cresce linearmente com cada nova variante de entrada

Code calisthenics (Jeff Bay) propõe regra extrema: **um nível de indentação por método**. Não é lei literal, mas calibra a intuição — sentir desconforto a partir do 2º nível.

## Estratégias de refactor

### 1. Early return / guard clauses

Eliminar o `else` invertendo a condição e retornando cedo.

```go
// ❌ 3 níveis de indent
func Charge(order Order) (Receipt, error) {
    if order.Paid {
        if order.Customer.Active {
            if order.Total > 0 {
                // feliz caso no nível 3
                return processPayment(order)
            } else {
                return Receipt{}, ErrZeroTotal
            }
        } else {
            return Receipt{}, ErrInactiveCustomer
        }
    } else {
        return Receipt{}, ErrAlreadyPaid
    }
}

// ✅ Early return, caminho feliz no nível 0
func Charge(order Order) (Receipt, error) {
    if !order.Paid           { return Receipt{}, ErrAlreadyPaid }
    if !order.Customer.Active { return Receipt{}, ErrInactiveCustomer }
    if order.Total <= 0      { return Receipt{}, ErrZeroTotal }

    return processPayment(order)
}
```

Ganho: leitura linear, sem rastrear indentação para entender qual condição casou.

### 2. Strategy Pattern

Múltiplos comportamentos selecionados por tipo/configuração. Ver [[../patterns/strategy]].

```ts
// ❌ branching que cresce por tipo novo
function calculateShipping(order: Order): number {
  if (order.type === "standard")      return order.weight * 2;
  else if (order.type === "express")  return order.weight * 5 + 10;
  else if (order.type === "overnight") return order.weight * 10 + 25;
  else if (order.type === "pickup")    return 0;
  else throw new Error("unknown");
}

// ✅ Strategies + registry
type ShippingStrategy = (order: Order) => number;
const shippingStrategies: Record<ShippingType, ShippingStrategy> = {
  standard:  o => o.weight * 2,
  express:   o => o.weight * 5 + 10,
  overnight: o => o.weight * 10 + 25,
  pickup:    _ => 0,
};

function calculateShipping(order: Order): number {
  const strategy = shippingStrategies[order.type];
  if (!strategy) throw new UnknownShippingType(order.type);
  return strategy(order);
}
```

Ganho: adicionar tipo novo toca 1 linha; função principal fechada para modificação (OCP).

### 3. Tabela de dispatch

Variante da estratégia para lookups simples:

```python
# ❌
if code == "A": return "Active"
elif code == "I": return "Inactive"
elif code == "B": return "Banned"
elif code == "P": return "Pending"
else: raise ValueError(code)

# ✅
STATUS_LABELS = {"A": "Active", "I": "Inactive", "B": "Banned", "P": "Pending"}
def status_label(code: str) -> str:
    try: return STATUS_LABELS[code]
    except KeyError: raise ValueError(f"unknown status: {code}")
```

### 4. Polimorfismo / subtipos

Se o ramo muda **comportamento**, não só valor, promover para método:

```python
# ✅
class Notification:
    def send(self): raise NotImplementedError

class EmailNotification(Notification):
    def send(self): smtp.send(...)

class SMSNotification(Notification):
    def send(self): twilio.send(...)

class PushNotification(Notification):
    def send(self): fcm.send(...)

def dispatch(notif: Notification): notif.send()
```

Chamador não faz `if type == ...`; dispatch é inerente.

### 5. Chain of Responsibility

Quando múltiplos handlers competem pela mesma entrada em ordem: ver [[../patterns/chain-of-responsibility]]. Útil em middleware, validação em camadas, fallback de providers.

## Linters que ajudam

| Ferramenta | Métrica |
|---|---|
| **ESLint** (`complexity`, `max-depth`) | ciclomática, profundidade |
| **SonarQube** | cognitive complexity (mais realista) |
| **gocyclo** / **gocognit** | Go |
| **radon** | Python — ciclomática e manutenabilidade |
| **clippy** (`cognitive_complexity`) | Rust |
| **PMD** (`CyclomaticComplexity`) | JVM |

Configurar **limite duro** (e.g., cognitive ≤ 15, depth ≤ 3) e bloquear PR que viola sem justificativa.

## Quando tolerar aninhamento

- **Switch exaustivo sobre tipo fechado** (sealed trait, enum, sum type) com match compilado — o compilador garante que todos os casos são cobertos; strategy artificial aqui seria piora.
- **Validação determinística com 4-6 regras ordenadas** onde cada uma depende da anterior — extração fragmenta o sentido.
- **Código crítico de performance** em hot loop — abstração pode custar alocações; medir antes de refatorar.

Regra geral: tolerar é **exceção documentada**, não default.

## Code calisthenics relacionados

Ver [[code-calisthenics]]:
- Regra 1: um nível de indentação por método
- Regra 2: não usar `else`
- Regra 3: encapsular primitivos (Value Objects removem `if`s de validação)
- Regra 7: pequenas entidades (< 50 linhas)

## Heurística de review

- [ ] Alguma função tem indentação ≥ 3 níveis? → candidato a refactor
- [ ] `if/else if/else if` com 4+ ramos sobre tipo/categoria? → strategy ou dispatch
- [ ] Ramo que muda **comportamento** por tipo? → polimorfismo
- [ ] `else` puramente negação do `if`? → early return
- [ ] Tabela de valor? → map/dict

## Links

- [[_moc]]
- [[clean-code]]
- [[code-calisthenics]]
- [[solid]]
- [[do-dont-checklist]]
- [[../patterns/strategy]]
- [[../patterns/chain-of-responsibility]]
- [[../antipatterns/_moc]]
