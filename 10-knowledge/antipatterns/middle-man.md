---
title: Middle Man (AP-018)
type: antipattern
tags: [antipattern, code-smell, coupler, delegation]
source: https://refactoring.guru/smells/middle-man
source_date: 2026-04-24
created: 2026-04-24
updated: 2026-04-24
aliases: [pass-through-class, excessive-delegation]
---

# AP-018 — Middle Man

Classe cuja **maior parte dos métodos só delega pra outra**. Originalmente legítima (encapsulava algo), foi perdendo peso até virar pass-through puro. O cliente teria menos atrito falando direto com o alvo.

## Sintomas

- Métodos na forma `public X foo(Y y) { return inner.foo(y); }` repetidos em sequência
- Classe tem N métodos, N-1 delegam 1:1 pra um único campo
- Documentação `// see Inner.foo` em todos os métodos
- Cada método novo na `Inner` obriga adicionar método idêntico no Middle Man
- Refactor "otimização" quebra chamadores que só queriam o Inner

## Por que é problema

**Acoplamento vazio**: Middle Man amarra API do `Inner` a si mesmo sem valor agregado. Mudança em `Inner` obriga evoluir 2 classes.

**Surface inflada**: mesmos N métodos existem em 2 lugares; documentação, testes, mocks duplicados.

**Navegação penosa**: leitor vê `client.middle.x()` e precisa abrir `Middle` pra descobrir que é `Inner.x()`.

**Refactor blocker**: não dá pra evoluir `Inner` sem mexer no Middle Man, mesmo quando cliente não ligaria.

## Exemplo

```csharp
// ❌ Middle Man puro — só delega
public class PersonManager {
    private readonly PersonRepository _repo;
    public Person Find(int id)                => _repo.Find(id);
    public void Save(Person p)                => _repo.Save(p);
    public void Delete(int id)                => _repo.Delete(id);
    public IEnumerable<Person> All()          => _repo.All();
}
// uso: new PersonManager(repo).Find(1) — por que não repo.Find(1)?

// ✅ Remover middle man
// cliente fala direto: repo.Find(1);

// ✅ Ou dar trabalho real — Middle Man deixa de ser smell quando agrega
public class PersonService {
    private readonly PersonRepository _repo;
    private readonly IAuditLog _audit;
    private readonly IUnitOfWork _uow;

    public void Delete(int id, Actor actor) {
        // agrega: autorização, auditoria, transação
        _uow.Execute(() => {
            _repo.Delete(id);
            _audit.Record(actor, "person.delete", id);
        });
    }
}
```

## Como refatorar

- **Remove Middle Man**: substituir chamadas pelo alvo direto; deletar classe
- **Inline Class**: se Middle Man tem vestígio de lógica, absorver no `Inner` ou no chamador
- **Adicionar valor**: se a classe precisa existir por razão real (autorização, auditoria, cross-cutting concern), **fazer** algo — não só delegar
- **Replace Delegation with Inheritance** (com cuidado — ver [[refused-bequest]]): raro, só quando faz sentido semântico

## Quando tolerar

- **Facade** genuína — expõe **subconjunto simplificado** duma API complexa; não é delegação 1:1
- **Adapter** — traduz entre duas interfaces; métodos parecem delegar mas **convertem tipos** ou protocolos
- **Proxy** com trabalho real (cache, lazy-load, auth check, audit log)
- **Encapsulamento de lib externa** pra poder trocar depois (port em hexagonal) — **se** a lib for substituível de verdade, senão é [[speculative-generality]]
- **Hiding of implementation** — quando o `Inner` não deve ser parte da API pública

## Patterns relacionados (que resolvem)

- [[lazy-class]] — Middle Man é uma forma específica de Lazy Class
- [[../patterns/facade]] — Facade legítima agrega valor; Middle Man não
- [[../patterns/adapter]] — Adapter **traduz**, não só delega
- [[../patterns/proxy]] — Proxy **adiciona comportamento** (cache, auth, lazy)
- [[../patterns/decorator]] — Decorator adiciona capacidade; Middle Man não adiciona nada
- [[feature-envy]] — se o Middle Man só mexe em `Inner`, talvez o método pertença a `Inner`

## Fontes

- Refactoring Guru — [Middle Man](https://refactoring.guru/smells/middle-man)
- Fowler, M. (2018). *Refactoring*. 2nd ed. Addison-Wesley.
- Gamma et al. (1994). *Design Patterns* — Facade, Adapter, Proxy, Decorator.

## Links

- [[_moc]]
- [[../_moc]]
- [[../patterns/_moc]]
- [[lazy-class]]
- [[../patterns/facade]]
