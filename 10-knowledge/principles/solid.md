---
title: SOLID — os 5 princípios
type: concept
tags: [principles, solid, design, oop, clean-code]
source: Robert C. Martin, "Design Principles and Design Patterns" (2000)
created: 2026-04-23
aliases: [SOLID principles, SRP OCP LSP ISP DIP]
---

# SOLID — os 5 princípios

5 princípios de **design orientado a objeto** formulados por Robert C. Martin (Uncle Bob) em 2000, condensados no acrônimo por Michael Feathers. Aplicam-se a qualquer linguagem com abstrações (Go, TS, Rust, Java, Python). **Não são regras** — são heurísticas que previnem 80% da dívida técnica de design.

> **Filosofia comum**: cada princípio ataca um modo diferente de rigidez/fragilidade que códigos OOP tendem a desenvolver à medida que crescem.

## 1. SRP — Single Responsibility Principle

> "Uma classe/módulo deve ter **apenas um motivo** para mudar."

Não é "faz uma coisa só" (confusão comum). É "**responde a uma audiência** — um stakeholder, um domínio, um eixo de mudança".

```go
// ❌ 3 motivos para mudar: impressão, cálculo fiscal, persistência
type Invoice struct { ... }
func (i *Invoice) CalculateTax() Money { ... }    // muda se regra fiscal muda
func (i *Invoice) SaveToDatabase() error { ... }  // muda se schema DB muda
func (i *Invoice) RenderPDF() []byte { ... }      // muda se layout muda

// ✅ Um motivo cada
type Invoice struct { ... }                         // dados + regras de invariante
type InvoiceTaxCalculator struct { ... }            // regras fiscais
type InvoiceRepository interface { Save(...) }      // persistência
type InvoiceRenderer interface { RenderPDF(...) }   // apresentação
```

**Sinais de violação**:
- Classe com métodos de categorias muito diferentes (cálculo + I/O + formatação)
- Mudança de requisito em uma área obriga tocar múltiplas áreas da mesma classe
- Nome da classe precisa de conjunção ("AccountManager**And**Logger")

**Exceção pragmática**: Value Objects podem ter múltiplos "aspectos" (`Email` tem `Validate()`, `Domain()`, `Masked()`) — tudo serve à **mesma audiência** (quem manipula email). SRP é sobre audiências, não sobre contagem de métodos.

## 2. OCP — Open/Closed Principle

> "Entidades devem estar **abertas** para extensão e **fechadas** para modificação."

Comportamento novo se adiciona via **nova implementação de abstração**, não editando código existente. Reduz risco de regressão.

```go
// ❌ Adicionar novo tipo de pagamento força editar a função
func ProcessPayment(p Payment) error {
    switch p.Type {
    case "stripe":
        return processStripe(p)
    case "pix":
        return processPix(p)
    // Novo tipo? Editar aqui — toda vez — risco de regressão.
    }
}

// ✅ Strategy via interface — fechado para modificação, aberto para extensão
type PaymentProcessor interface {
    Process(ctx context.Context, p Payment) error
}

type StripeProcessor struct { ... }
type PixProcessor struct { ... }
// Novo tipo? Nova struct implementando PaymentProcessor. Zero edição do código existente.

func (uc *ProcessPaymentUseCase) Execute(ctx context.Context, cmd Cmd) error {
    processor := uc.processors[cmd.Method]
    return processor.Process(ctx, cmd.Payment)
}
```

**Sinais de violação**:
- `switch`/`if-else` extenso por tipo (dispatch manual em vez de polimorfismo)
- Comentário "adicionar novo tipo aqui"
- Função que cresce linearmente com número de variantes

**Exceção pragmática**: YAGNI. Criar interface para **um** caso concreto sem vislumbre de segundo = abstração prematura. Aplicar OCP a partir da **2ª implementação**.

## 3. LSP — Liskov Substitution Principle

> "Objetos de uma classe derivada devem **substituir** os da classe base **sem alterar o programa**."

Subtipo não pode **reforçar pré-condições**, **enfraquecer pós-condições**, ou **violar invariantes** do tipo base.

```go
// ❌ Violação clássica: Square herda Rectangle
type Rectangle struct { width, height float64 }
func (r *Rectangle) SetWidth(w float64)  { r.width = w }
func (r *Rectangle) SetHeight(h float64) { r.height = h }

type Square struct { Rectangle }
func (s *Square) SetWidth(w float64)  { s.width = w; s.height = w }
func (s *Square) SetHeight(h float64) { s.width = h; s.height = h }

// Qualquer código que espera Rectangle quebra com Square:
func resize(r *Rectangle) {
    r.SetWidth(5)
    r.SetHeight(10)
    assert(r.width == 5 && r.height == 10) // FALHA se r for Square
}
```

Em Go/Rust/TS moderno, LSP se manifesta em **conformidade com contratos de interface**:

```go
// Se IRepository.Save promete "retorna erro SÓ em falha de infra":
type IRepository interface {
    Save(ctx context.Context, e Entity) error
}

// ❌ Esta impl viola LSP — retorna erro de negócio via Save
type NaiveRepo struct{}
func (r *NaiveRepo) Save(ctx context.Context, e Entity) error {
    if e.IsExpired() {
        return ErrBusiness // ← chamador espera só erro de infra
    }
    // ...
}

// ✅ Regra de negócio fica no domain/use case, não no repo
```

**Sinais de violação**:
- Subclasses que sobrescrevem métodos com `throw new UnsupportedOperationException()`
- Type check (`is`, `instanceof`) antes de usar — se precisa do tipo concreto, a abstração mente
- Hierarquia com "exceções" em subclasses ("exceto `AdminUser` que não tem `Deactivate`")

## 4. ISP — Interface Segregation Principle

> "Clientes **não** devem ser forçados a depender de métodos que não usam."

Interfaces **amplas demais** acoplam consumidores a métodos irrelevantes. Preferir **múltiplas interfaces específicas** a uma genérica.

```go
// ❌ Mega-interface
type Worker interface {
    Work(ctx context.Context) error
    Eat(ctx context.Context) error
    Sleep(ctx context.Context) error
}
// RobotWorker precisa implementar Eat() e Sleep() absurdos

// ✅ Segregadas
type Worker interface { Work(ctx context.Context) error }
type Eater  interface { Eat(ctx context.Context) error }
type Sleeper interface { Sleep(ctx context.Context) error }

type Human = interface { Worker; Eater; Sleeper }
type Robot = interface { Worker }
```

Em Go, isso é **idiomático**: interfaces pequenas, no ponto de uso ("**accept interfaces, return structs**"). `io.Reader`, `io.Writer`, `io.Closer` são o exemplo canônico — três métodos, um propósito cada.

```go
// Função só precisa ler — aceita interface mínima
func processStream(r io.Reader) error { ... }

// Em vez de exigir *os.File ou interface gigante
```

**Sinais de violação**:
- Interface com 15+ métodos
- Implementações com `panic("not implemented")`
- Consumidor usa 2 dos 10 métodos da dependência

## 5. DIP — Dependency Inversion Principle

> "Módulos de alto nível **não** devem depender de módulos de baixo nível. **Ambos** devem depender de **abstrações**."

É o princípio **mais importante** para Clean Architecture. Inverte o fluxo natural de dependência: domain define a interface, infrastructure a implementa.

```go
// ❌ Fluxo natural — domínio depende de infra
// domain/use_cases/checkout.go
import "github.com/stripe/stripe-go"  // ← domínio conhece Stripe, errado
func Checkout(...) {
    stripe.Subscriptions.New(...)
}

// ✅ Invertido — infra depende do domínio
// domain/providers/i_payment_gateway.go
type IPaymentGateway interface {
    CreateSubscription(ctx context.Context, in CreateInput) (*Output, error)
}

// domain/use_cases/checkout.go
type CheckoutUseCase struct {
    gateway IPaymentGateway  // ← domínio só conhece a abstração
}

// infrastructure/providers/stripe_gateway.go
type StripeGateway struct { client *stripe.Client }
var _ domain.IPaymentGateway = (*StripeGateway)(nil)  // infra IMPLEMENTA a interface do domain
```

**Regra de dependência de Clean Architecture é DIP aplicado**:

```
infrastructure → application → domain
     ↑              ↑            ↑
     └── depende ───┴── depende ─┘
     (todos apontam para dentro, domain não aponta para ninguém)
```

**Sinais de violação**:
- `import` de SDK externo (`stripe-go`, `mux-go`, pgx) em `domain/`
- Use case faz `database.Query(...)` direto em vez de usar `IRepository`
- Testes do domínio precisam de container Docker de Postgres

## Por que estes 5

Cada princípio ataca uma **classe específica de rigidez**:

| Princípio | Rigidez atacada | Sintoma resolvido |
|---|---|---|
| SRP | Acoplamento temporal | "Mudar X obriga mudar Y, Z, W" |
| OCP | Inflação do switch | "Toda feature nova quebra regressão em features antigas" |
| LSP | Herança enganosa | "Subclasse A joga exceção onde a base retorna valor" |
| ISP | Dependência inútil | "Preciso implementar 15 métodos pra usar 2" |
| DIP | Inversão frágil | "Não consigo testar use case sem banco real" |

Não aplicar um isolado: violação de SRP geralmente arrasta OCP atrás; violação de DIP geralmente força ISP a falhar.

## Quando NÃO aplicar

- **Scripts e utilitários de 100 linhas**: SOLID é overkill
- **Provas de conceito**: priorizar velocidade de validação
- **Código gerado** (sqlc, ORMs com codegen): não tocar estrutura
- **Testes**: setup direto geralmente > abstração indireta
- **Abstração prematura (YAGNI)**: OCP/DIP antes da 2ª implementação é dívida

## Checklist de review

- [ ] Cada classe/módulo responde a **uma** audiência? (SRP)
- [ ] Adição de variante nova requer edição de código antigo? (OCP ⚠️)
- [ ] Subtipo quebra contrato do supertipo? (LSP ⚠️)
- [ ] Interface força implementar métodos irrelevantes? (ISP ⚠️)
- [ ] Domínio importa de infra? (DIP ⚠️⚠️⚠️)

## Referências

- Robert C. Martin. *Design Principles and Design Patterns* (2000)
- Robert C. Martin. *Clean Architecture* (2017) — cap. 7-11
- Barbara Liskov. *Data Abstraction and Hierarchy* (1987) — origem do LSP
- Go Proverbs. "The bigger the interface, the weaker the abstraction" — ISP idiomático em Go

## Links

- [[_moc]]
- [[clean-code]]
- [[code-calisthenics]]
- [[do-dont-checklist]]
- [[../architecture/clean-architecture-layers]]
- [[../architecture/ddd-strategic-tactical]]
- [[../architecture/cqrs]]
- [[../patterns/_moc]]
