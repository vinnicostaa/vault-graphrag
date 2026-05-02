::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Padrões de
Projeto](../design-patterns.html){.type} /
[Catálogo](catalog.html){.type}
:::

# Padrões de projeto comportamentais {#padrões-de-projeto-comportamentais .title}

Padrões comportamentais são voltados aos algoritmos e a designação de
responsabilidades entre objetos.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Chain of
Responsibility](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Chain of Responsibility]{.pattern-name .pattern-name-long}
]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

Permite que você passe pedidos por uma corrente de handlers. Ao receber
um pedido, cada handler decide se processa o pedido ou passa para o
próximo handler da corrente.

[[
[![Command](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Command]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

Transforma o pedido em um objeto independente que contém toda a
informação sobre o pedido. Essa Transformação permite que você
parameterize métodos com diferentes pedidos, atrase ou coloque a
execução do pedido em uma fila, e suporte operações que não podem ser
feitas.

[[
[![Iterator](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Iterator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

Permite que você percorra elementos de uma coleção sem expor as
representações estruturais deles (lista, pilha, árvore, etc.)

[[
[![Mediator](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Mediator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

Permite que você reduza as dependências caóticas entre objetos. O padrão
restringe comunicações diretas entre objetos e os força a colaborar
apenas através do objeto mediador.

[[
[![Memento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Memento]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

Permite que você salve e restaure o estado anterior de um objeto sem
revelar os detalhes de sua implementação.

[[
[![Observer](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Observer]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

Permite que você defina um mecanismo de assinatura para notificar
múltiplos objetos sobre quaisquer eventos que aconteçam com o objeto que
eles estão observando.

[[
[![State](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[State]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

Permite que um objeto altere seu comportamento quando seu estado interno
muda. Parece como se o objeto mudasse de classe.

[[
[![Strategy](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Strategy]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

Permite que você defina uma família de algoritmos, coloque-os em classes
separadas, e faça os objetos deles intercambiáveis.

[[ [![Template
Method](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Template Method]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

Define o esqueleto de um algoritmo na superclasse mas deixa as
subclasses sobrescreverem etapas específicas do algoritmo sem modificar
sua estrutura.

[[
[![Visitor](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Visitor]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

Permite que você separe algoritmos dos objetos nos quais eles operam.
:::

::: next
#### Leia a seguir

[Chain of Responsibility []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Proxy](proxy.html){.btn .btn-default rel="prev"}
:::
:::::::
::::::::
:::::::::
