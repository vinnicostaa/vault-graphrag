::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::: {.main-content-container .center-content-container}
::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} /
[Catálogo](catalog.html){.type}
:::

# Patrones de comportamiento {#patrones-de-comportamiento .title}

Los patrones de comportamiento tratan con algoritmos y la asignación de
responsabilidades entre objetos.

::: {.d-flex .flex-wrap .justify-content-between}
[[ [![Chain of
Responsibility](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}]{.pattern-image}
[Chain of Responsibility]{.pattern-name .pattern-name-long}
]{.pattern-card-top} [
]{.pattern-card-bottom}](chain-of-responsibility.html){.pattern-card-extended
.chain-of-responsibility}

Permite pasar solicitudes a lo largo de una cadena de manejadores. Al
recibir una solicitud, cada manejador decide si la procesa o si la pasa
al siguiente manejador de la cadena.

[[
[![Command](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}]{.pattern-image}
[Command]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](command.html){.pattern-card-extended .command}

Convierte una solicitud en un objeto independiente que contiene toda la
información sobre la solicitud. Esta transformación te permite
parametrizar los métodos con diferentes solicitudes, retrasar o poner en
cola la ejecución de una solicitud y soportar operaciones que no se
pueden realizar.

[[
[![Iterator](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}]{.pattern-image}
[Iterator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](iterator.html){.pattern-card-extended
.iterator}

Permite recorrer elementos de una colección sin exponer su
representación subyacente (lista, pila, árbol, etc.).

[[
[![Mediator](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}]{.pattern-image}
[Mediator]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](mediator.html){.pattern-card-extended
.mediator}

Permite reducir las dependencias caóticas entre objetos. El patrón
restringe las comunicaciones directas entre los objetos, forzándolos a
colaborar únicamente a través de un objeto mediador.

[[
[![Memento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}]{.pattern-image}
[Memento]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](memento.html){.pattern-card-extended .memento}

Permite guardar y restaurar el estado previo de un objeto sin revelar
los detalles de su implementación.

[[
[![Observer](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}]{.pattern-image}
[Observer]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](observer.html){.pattern-card-extended
.observer}

Permite definir un mecanismo de suscripción para notificar a varios
objetos sobre cualquier evento que le suceda al objeto que están
observando.

[[
[![State](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}]{.pattern-image}
[State]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](state.html){.pattern-card-extended .state}

Permite a un objeto alterar su comportamiento cuando su estado interno
cambia. Parece como si el objeto cambiara su clase.

[[
[![Strategy](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}]{.pattern-image}
[Strategy]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](strategy.html){.pattern-card-extended
.strategy}

Permite definir una familia de algoritmos, colocar cada uno de ellos en
una clase separada y hacer sus objetos intercambiables.

[[ [![Template
Method](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}]{.pattern-image}
[Template Method]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](template-method.html){.pattern-card-extended
.template-method}

Define el esqueleto de un algoritmo en la superclase pero permite que
las subclases sobrescriban pasos del algoritmo sin cambiar su
estructura.

[[
[![Visitor](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}]{.pattern-image}
[Visitor]{.pattern-name} ]{.pattern-card-top} [
]{.pattern-card-bottom}](visitor.html){.pattern-card-extended .visitor}

Permite separar algoritmos de los objetos sobre los que operan.
:::

::: next
#### Leer siguiente

[Chain of Responsibility []{.fa
.fa-arrow-right}](chain-of-responsibility.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Proxy](proxy.html){.btn .btn-default rel="prev"}
:::
:::::::
::::::::
:::::::::
