---
title: DDD — Strategic and Tactical Design
type: concept
tags: [architecture, ddd, domain-driven-design, bounded-context, aggregate]
source: Eric Evans, "Domain-Driven Design" (2003); Vaughn Vernon, "Implementing DDD" (2013)
created: 2026-04-23
aliases: [Domain-Driven Design, DDD estratégico, DDD tático, Bounded Context]
---

# DDD — Strategic and Tactical Design

**Domain-Driven Design** (Evans, 2003) é abordagem de design de software para domínios **complexos**. Combina princípios estratégicos (como dividir o sistema em pedaços alinhados ao negócio) com táticos (como modelar cada pedaço). **Não é sobre orientação a objetos** — é sobre alinhamento código ↔ linguagem de negócio.

> **Filosofia central**: o código deve falar a linguagem do domínio. Se o negócio fala "assembleia homologada", o código tem `Assembly.Ratify()`, não `update_assembly_status_to_2`.

DDD tem duas dimensões que se potencializam mutuamente:

## Parte A — Strategic Design (macro)

Organiza o sistema como um todo. Responde: **onde ficam as fronteiras**.

### Ubiquitous Language

> A linguagem **usada no código** deve ser a mesma usada por **especialistas de domínio**.

Não existe "tradução" entre nome do negócio e nome do código. Se o síndico fala "edital", o banco tem tabela `editais`, a entidade se chama `Edital`, o método é `Edital.Publish()`.

- Glossário vivo no repositório (`01-domain/ubiquitous-language.md`)
- Novos termos passam pelos especialistas antes de virar código
- Divergência entre nome do negócio e do código = **dívida semântica**

### Bounded Context

> **Fronteira explícita** onde um modelo é **internamente consistente**.

Mesmo termo pode significar coisas diferentes em contextos diferentes — e isso **está ok**, desde que a fronteira seja explícita.

Exemplo: `User` em `identity` (dados de autenticação) ≠ `User` em `commercial` (perfil comercial com reputação). Ambos existem, são tipos **diferentes** em pacotes diferentes, conectados por `user_id`.

```
┌─────────────────────┐     ┌──────────────────────┐
│  Identity BC        │     │  Commercial BC       │
│                     │     │                      │
│  type User struct { │     │  type User struct {  │
│    ID     UserID    │     │    ID          UserID│
│    Email  Email     │     │    Reputation  Score │
│    Role   Role      │     │    Deals       []Deal│
│  }                  │     │  }                   │
└────────┬────────────┘     └──────────┬───────────┘
         │  integrados via ID (não     │
         │  compartilham struct)       │
         └──────────┬──────────────────┘
                   via events / shared/types interfaces
```

### Context Map

Diagrama que mostra **como bounded contexts se relacionam**:

- **Shared Kernel**: dois contextos compartilham um núcleo pequeno (raro)
- **Customer/Supplier**: downstream depende do contrato do upstream
- **Conformist**: downstream aceita o modelo do upstream sem traduzir (trade-off)
- **Anti-Corruption Layer (ACL)**: downstream traduz pra não contaminar seu modelo (comum em integração com legado/SDK externo)
- **Open Host Service**: upstream expõe protocolo público explícito (HTTP API)
- **Published Language**: formato universal (JSON schema, protobuf, AsyncAPI)
- **Separate Ways**: contextos simplesmente não conversam

### Subdomain

Tipos de complexidade dentro do negócio:

- **Core**: onde você **ganha dinheiro** / **se diferencia** — investe tempo e expertise, modela com cuidado máximo
- **Supporting**: necessário mas não diferenciador — investe moderado
- **Generic**: comprável pronto (pagamento, email, auth) — **não reinventar**, integrar SaaS

A decisão `core vs supporting vs generic` **guida investimento**. Errar aqui = gastar tempo de core modelando algo que SaaS resolveria.

## Parte B — Tactical Design (micro)

Modela **dentro** de um bounded context. Responde: **como escrever código que expressa regras**.

### Entity

Objeto com **identidade** que persiste através de mudanças de estado. Dois `User` com mesmos dados mas IDs diferentes são usuários **diferentes**.

Regras (ver [[../architecture/clean-architecture-layers]] para exemplos completos):
- Estado privado + factories `NewX`/`ReconstructX`
- Métodos expressam intenção: `subscription.Cancel(reason)`, não `setStatus(CANCELED)`
- Invariantes garantidas em todo método público
- Equality por **ID**, nunca por conteúdo

### Value Object

Sem identidade. Definido **totalmente pelo valor**. Dois `Email("a@b.com")` são **o mesmo email**, mesmo sendo objetos diferentes na memória.

Regras:
- **Imutável** após criação
- Validação **na factory**, nunca depois
- Equality **por valor**
- Zero setters
- Pode ter comportamento: `money.Add(other)`, `email.Domain()`, `cpf.Masked()`

### Aggregate

> **Cluster de entidades e value objects** tratado como uma unidade para fins de consistência.

Tem uma **raiz** (aggregate root) — a entidade pública. Tudo dentro é privado. Fora do aggregate, só conhece a raiz; não tem referência direta a filhos.

**Regras críticas** (violar estas é o erro mais comum em DDD):

1. **Uma transação = um aggregate**: operação de domínio modifica **um aggregate**, commit. Atualizar dois aggregates na mesma transação viola consistência final eventual.
2. **Referência entre aggregates por ID**, nunca por referência direta.
3. **Invariantes internas** do aggregate são garantidas **atomicamente**.
4. **Invariantes entre aggregates** são eventuais (via domain events).

```go
// ✅ Aggregate Assembly — raiz é Assembly, AgendaItem é parte
type Assembly struct {
    id       AssemblyID
    status   AssemblyStatus
    agenda   []AgendaItem   // privado, só Assembly manipula
    quorum   Quorum
}

func (a *Assembly) AddAgendaItem(title string, votingType VotingType) (AgendaItemID, error) {
    if a.status != AssemblyStatusDraft {
        return "", apperrors.ErrBusiness.WithMessage("assembleia já iniciada")
    }
    item := newAgendaItem(title, votingType)
    a.agenda = append(a.agenda, item)
    return item.ID(), nil
}

// ❌ Outro aggregate modificando AgendaItem diretamente
// externalCode.agendaItem.Vote(...) ← não faça, mutação escapa invariantes do Assembly
```

**Tamanho**: aggregate deve ser o **menor possível** que encapsula uma invariante. Aggregates gigantes viram hotspot de contenção de lock.

### Repository

Abstração de coleção de aggregates. **Um repository por aggregate root** (não por entidade interna).

```go
// ✅ Um repo por aggregate
type IAssemblyRepository interface {
    FindByID(ctx context.Context, id AssemblyID) (*Assembly, error)
    Save(ctx context.Context, a *Assembly) error
}
// (NÃO existe IAgendaItemRepository — AgendaItem vive dentro do Assembly)
```

Interface no domain, implementação na infrastructure. Ver [[../principles/solid]] (DIP) e [[clean-architecture-layers]].

### Domain Service

Lógica de domínio que **não pertence naturalmente a um aggregate**. Deve ser raro.

```go
// ✅ Transferência entre 2 aggregates Account — não cabe em um aggregate só
type TransferService struct { ... }
func (s *TransferService) Transfer(ctx context.Context, from, to *Account, amount Money) error { ... }
```

**Cuidado**: domain services viram lixeira de lógica que deveria estar nos aggregates. Regra de ouro: **prefira método em entidade; use service só se realmente orquestra múltiplos aggregates**.

### Domain Event

Fato ocorrido no domínio, relevante para outros bounded contexts.

- **Nome no passado**: `SubscriptionActivated`, `AssemblyRatified`, `ProxyRegistered`
- **Imutável** — evento é fato histórico
- Contém **somente** dados que handlers precisam
- Publicado **após** commit (dentro do UoW) — ou via Outbox se cross-service

Integração via eventos é a **forma canônica** de cruzar bounded contexts sem acoplar. Ver [[event-driven]] *(a criar)*.

### Factory

Função que cria aggregates/VOs com **validação e invariantes garantidas**.

```go
// ✅ Factory valida invariantes
func NewAssembly(condominiumID CondominiumID, scheduledFor time.Time, quorum Quorum) (*Assembly, error) {
    if scheduledFor.Before(time.Now().Add(48 * time.Hour)) {
        return nil, apperrors.ErrValidation.WithMessage("assembleia precisa ser agendada com 48h de antecedência")
    }
    if !quorum.IsValid() {
        return nil, apperrors.ErrValidation.WithMessage("quórum inválido")
    }
    return &Assembly{
        id:           NewAssemblyID(),
        status:       AssemblyStatusDraft,
        scheduledFor: scheduledFor,
        quorum:       quorum,
    }, nil
}

// ✅ Factory de reconstrução (da persistência) — não revalida invariantes (já foram válidas ao persistir)
func ReconstructAssembly(id AssemblyID, status AssemblyStatus, ...) *Assembly { ... }
```

### Specification

Pattern para **regra de negócio** reutilizável, combinable.

```go
type IsVotableByUserSpec struct { userID UserID }
func (s IsVotableByUserSpec) IsSatisfiedBy(assembly *Assembly) bool { ... }

// Combinação
canVote := isAssemblyActive.And(isUserMember).And(notAlreadyVoted)
```

Útil para regras **reaproveitadas em queries** e no domínio. Em prática, geralmente **over-engineering** — prefira método na entidade até aparecer 2ª cópia da regra.

## Por que DDD ganha em domínios complexos

- **Linguagem única** reduz bugs de interpretação
- **Bounded contexts** limitam blast radius — mudança em billing não quebra assembly
- **Aggregates** dão granularidade natural de transação e concorrência
- **Events** desacoplam temporalmente o que era acoplado via chamadas diretas

## Quando NÃO aplicar DDD

- **CRUD simples** (ex: admin panel com forms) — é overkill
- **Domínio genérico** (e-commerce padrão, blog) — use frameworks, não reinvente
- **Equipe sem especialista de domínio** disponível — DDD cai em invenção de termos
- **POCs e MVPs** onde o domínio ainda está sendo descoberto

## DDD vs Clean Architecture

Complementares, não excludentes:

- **DDD** define **o quê modelar** (aggregates, VOs, events, context boundaries)
- **Clean Architecture** define **como organizar layers** (domain/app/infra) e onde cada coisa vive

Clean Arch sem DDD → camadas vazias de significado (entidades anêmicas).
DDD sem Clean Arch → domínio rico mas acoplado a framework.

Ver [[clean-architecture-layers]].

## Anti-patterns DDD

- **Anemic Domain Model**: entidades com só getters/setters, lógica fora (em "services" gigantes) — fere o espírito DDD
- **God Aggregate**: aggregate com 50 entidades dentro — contenção, invariantes impossíveis
- **DDD-Lite**: rotular camada `domain/` mas não aplicar nada — "cargo cult DDD"
- **Cross-aggregate transactions**: atualizar 2 aggregates em 1 commit — viola consistência eventual
- **Repository que retorna tipos do ORM**: vaza infra pro domínio
- **Domain event com referência a entidade**: deve conter IDs/DTOs imutáveis, não ponteiros vivos

## Checklist de review

- [ ] Linguagem do código bate com linguagem do negócio?
- [ ] Bounded contexts têm fronteiras explícitas (pacotes, módulos)?
- [ ] Cada aggregate tem raiz clara e invariantes protegidas?
- [ ] Transações tocam **um** aggregate?
- [ ] Referência entre aggregates por ID?
- [ ] Entidades têm comportamento, não só dados?
- [ ] Value Objects são imutáveis?
- [ ] Domain events são nomeados no passado?
- [ ] Domínio está livre de `import`s de infra/SDK?

## Referências

- Eric Evans. *Domain-Driven Design: Tackling Complexity in the Heart of Software* (2003)
- Vaughn Vernon. *Implementing Domain-Driven Design* (2013) — mais prático que o original
- Vaughn Vernon. *Domain-Driven Design Distilled* (2016) — versão condensada
- Scott Millett. *Patterns, Principles, and Practices of DDD* (2015)
- Nick Tune. *Architecture Modernization* (2024) — bounded contexts em monolitos legados

## Links

- [[_moc]]
- [[clean-architecture-layers]]
- [[cqrs]]
- [[../patterns/transaction-patterns]]
- [[../principles/solid]]
- [[../principles/clean-code]]
- [[../methodology/legacy-as-reference]]
- [[../../20-stacks/go/go-patterns]]
