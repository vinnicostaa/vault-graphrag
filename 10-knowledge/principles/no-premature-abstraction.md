---
title: No premature abstraction — YAGNI e a regra de três
type: concept
tags: [principle, yagni, abstraction, design]
created: 2026-04-24
updated: 2026-04-24
aliases: [YAGNI, Premature abstraction, Speculative generality]
---

# No premature abstraction — YAGNI e a regra de três

Abstração é custosa. Camada a mais, indireção, contrato a manter, lugar novo pra caçar bug. Só paga quando há **duas ou mais implementações reais requisitadas pela realidade**. Abstrair antes disso é apostar em futuro que estatisticamente não chega.

## YAGNI — You Aren't Gonna Need It

Princípio do XP (Kent Beck). Não codar para "caso precisemos depois":

- Não criar parâmetro que ninguém passa
- Não criar interface sem 2º implementador real
- Não criar hook de extensibilidade para cliente hipotético
- Não suportar formato alternativo "pra caso apareça"

Regra: **trabalho feito antes da hora** é trabalho a manter sem benefício até a hora chegar — **e** é trabalho jogado fora se a hora não chega ou chega com requisitos diferentes. A segunda situação é a mais comum.

## Rule of Three (Martin Fowler)

Refactor guiado por número de ocorrências:

```
1ª vez que escrevo este código     → codar direto, sem pensar em reuso
2ª vez (duplicação aparece)        → copiar, marcar mental ou TODO de duplicação
3ª vez (3 lugares iguais)          → AGORA refatorar para abstração
```

Antes de 3, o padrão real ainda não se revelou. Abstrair no 2º pode congelar interface errada — a 3ª ocorrência sempre traz variação que a abstração prematura não previu.

## Critérios para abstrair

Antes de criar interface/classe base/módulo reutilizável, responder SIM a todos:

1. **Existem 2+ implementações reais** requisitadas agora (não "pode ser que precise")?
2. **As implementações são suficientemente parecidas** para que uma mesma interface faça sentido, ou só se parecem em superfície?
3. **A abstração reduz LOC total** ou pelo menos complexidade cognitiva agregada?
4. **O contrato da abstração é estável** o bastante para não precisar mudar a cada nova implementação?

Se algum é NÃO: adiar. Deixar código duplicado até as respostas estarem claras.

## Speculative Generality (AP-012)

Antipattern clássico do catálogo de Fowler (*Refactoring*, cap. 3). Sintomas:

- Interface com **1 implementação**
- Classe abstrata com **0 subclasses reais** além de mocks de teste
- Parâmetro que **sempre recebe o mesmo valor**
- Hook/callback que **nunca foi sobrescrito**
- Generic type param (`<T>`) instanciado só com 1 tipo
- Método `do(x, context, config)` onde `config` não varia

Código parece "extensível" mas é só **ruído**. Remover:

```java
// ❌ interface com 1 implementação + factory que sempre retorna ela
interface UserValidator { boolean validate(User u); }
class StandardUserValidator implements UserValidator { ... }
class UserValidatorFactory { static UserValidator create() { return new StandardUserValidator(); } }

// ✅
class UserValidator { boolean validate(User u) { ... } }
```

Ver [[../antipatterns/_moc]] (AP-012).

## Custo da abstração prematura

Indireção não é grátis:

1. **Leitura mais cara** — leitor pula entre arquivos para seguir execução real
2. **Refactor futuro mais caro** — interface fixa congela assinatura incorreta
3. **Debugging mais difícil** — stack trace passa por camadas vazias
4. **Novo dev entra mais devagar** — gasta contexto entendendo estrutura que não ajuda
5. **Testes mais frágeis** — mocks pra interface fictícia acoplam teste à estrutura, não ao comportamento
6. **LLM/agente gasta contexto** entendendo níveis extras antes de editar

## Quando abstrair **cedo** faz sentido

Existem boundaries onde YAGNI não se aplica:

- **Boundary com sistema externo** (SDK Stripe, driver Postgres, file system) — wrappear **desde o dia 1** protege o resto do código de mudança externa. Ver [[clean-code]] §7 e [[solid]] DIP.
- **Cross-cutting concerns** (logging, metrics, authz, error handling) — abstrair cedo impede que virem fios espalhados pela base.
- **Limite entre camadas arquiteturais** (domain ↔ application ↔ infrastructure) — interface de repositório no domínio é obrigatória para que o domínio não dependa de ORM concreto.
- **Contratos públicos** (API, webhook, evento de domínio) — uma vez expostos, quebrar é caríssimo.

Nesses casos, a abstração **não é especulativa** — ela existe por um motivo hoje (isolamento), independente de quantas implementações haverá.

## Heurísticas práticas

### DRY com critério

**DRY** (Don't Repeat Yourself) não é "elimine toda string repetida". É **uma fonte da verdade por regra de conhecimento**. Duas linhas iguais que representam coisas diferentes **não** são duplicação; são coincidência de forma. Abstrair acopla dois domínios sem motivo.

Ver [[clean-code]] §10 (três tipos de duplicação).

### Pergunta antes de `<T>` ou `interface`

> "Quem é a segunda implementação real, e quando ela entra?"

Se a resposta é "sei lá" ou "talvez um dia": não abstrair. Voltar quando a 2ª aparecer.

### Copiar > abstrair errado

Quando em dúvida, **duplicar** por enquanto. Duplicação chama atenção (todo mundo lê); abstração errada **não** chama — fica escondida até causar dano.

### Refactor é barato; abstração errada é cara

Códigos modernos com testes sólidos permitem extrair abstração quando a 3ª ocorrência surge. Deixar emergir é quase sempre mais seguro que prever.

## Relação com outros princípios

- [[../methodology/gsd-get-shit-done]] — envia o mínimo que resolve; evolui quando vira necessidade
- [[../methodology/sdd-workflow]] — SDD pede spec antes de código, **não** pede abstração antes de uso
- [[solid]] — OCP e DIP **não** significam abstrair tudo; significam que **quando** há variação real, o ponto de extensão está preparado
- [[clean-code]] §7 — boundaries (fronteiras de terceiros) é o caso legítimo de abstração cedo

## Antipatterns relacionados

- **Speculative Generality** (AP-012) — gancho que ninguém usa
- **Inner-Platform Effect** — reimplementar metade da plataforma embaixo para "flexibilidade"
- **Big Design Up Front** — todo design antes da primeira linha
- **Framework-itis** — transformar aplicação em framework sem 2º app consumidor

## Links

- [[_moc]]
- [[clean-code]]
- [[solid]]
- [[code-calisthenics]]
- [[do-dont-checklist]]
- [[../methodology/gsd-get-shit-done]]
- [[../antipatterns/_moc]]
