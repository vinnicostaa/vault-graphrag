::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.page .text}
:::: {.illustration-container data-width="960" data-height="720"}
![Patrones de diseño en
Go](../../images/patterns/languages/go49a2.png?id=70faef6e92828d6bc0c486bc59081e22){.dp-bg
width="960" loading="lazy"
srcset="/images/patterns/languages/go-2x.png?id=aa34ed3bdda3e9e29bf73f24250a3082 2x,/images/patterns/languages/go-3x.png 3x"}

::: mob-image
![Design Patterns in
Go](../../images/patterns/languages/mini/go561f.png?id=4ae96539199ae1c0b15d49fccc7d84cc){.dp-bg-mini
width="256" loading="lazy"
srcset="/images/patterns/languages/mini/go-2x.png?id=3246b120fdee2b5b942c937509404f16 2x,/images/patterns/languages/mini/go-3x.png 3x"}
:::

# [PATRONES]{.dp1-h-1} [de]{.dp-h-article} [DISEÑO]{.dp1-h-2} [en]{.dp-lang-in} [Go]{.dp-lang} {#patrones-de-diseño-en-go .dp1-h .dp-abs .dp-c .dp-h1}
::::

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-text
## El catálogo de ejemplos en **Go** {#el-catálogo-de-ejemplos-en-go .dp-h2}

:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
#### Patrones creacionales {#patrones-creacionales .dp-h4 .dp-creational-title}

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Abstract
Factory](../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x,/images/patterns/cards/abstract-factory-mini-3x.png 3x"}
:::

#### Abstract Factory []{.dp-popularity} {#abstract-factory .dp-pattern-title}

::: dp-pattern-intent
Permite producir familias de objetos relacionados sin especificar sus
clases concretas.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](abstract-factory.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](abstract-factory/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Builder](../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x,/images/patterns/cards/builder-mini-3x.png 3x"}
:::

#### Builder []{.dp-popularity} {#builder .dp-pattern-title}

::: dp-pattern-intent
Permite construir objetos complejos paso a paso. Este patrón nos permite
producir distintos tipos y representaciones de un objeto empleando el
mismo código de construcción.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](builder.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](builder/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Factory
Method](../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x,/images/patterns/cards/factory-method-mini-3x.png 3x"}
:::

#### Factory Method []{.dp-popularity} {#factory-method .dp-pattern-title}

::: dp-pattern-intent
Proporciona una interfaz para la creación de objetos en una superclase,
mientras permite a las subclases alterar el tipo de objetos que se
crearán.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](factory-method.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](factory-method/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Prototype](../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x,/images/patterns/cards/prototype-mini-3x.png 3x"}
:::

#### Prototype []{.dp-popularity} {#prototype .dp-pattern-title}

::: dp-pattern-intent
Permite copiar objetos existentes sin que el código dependa de sus
clases.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](prototype.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](prototype/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

::::::::: {.dp-pattern-item .dp-creational}
::: dp-pattern-card
![Singleton](../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x,/images/patterns/cards/singleton-mini-3x.png 3x"}
:::

#### Singleton []{.dp-popularity} {#singleton .dp-pattern-title}

::: dp-pattern-intent
Permite asegurarnos de que una clase tenga una única instancia, a la vez
que proporciona un punto de acceso global a dicha instancia.
:::

:::::: dp-pattern-links
<div>

[ Artículo principal](singleton.html){.dp-pattern-link}

</div>

<div>

[ Naïve
Singleton](singleton/go/example.html#example-0){.dp-pattern-link}

</div>

<div>

[ Thread-safe
Singleton](singleton/go/example.html#example-1){.dp-pattern-link}

</div>
::::::
:::::::::

#### Patrones estructurales {#patrones-estructurales .dp-h4 .dp-structural-title}

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Adapter](../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x,/images/patterns/cards/adapter-mini-3x.png 3x"}
:::

#### Adapter []{.dp-popularity} {#adapter .dp-pattern-title}

::: dp-pattern-intent
Permite la colaboración entre objetos con interfaces incompatibles.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](adapter.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](adapter/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Bridge](../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x,/images/patterns/cards/bridge-mini-3x.png 3x"}
:::

#### Bridge []{.dp-popularity} {#bridge .dp-pattern-title}

::: dp-pattern-intent
Permite dividir una clase grande o un grupo de clases estrechamente
relacionadas, en dos jerarquías separadas (abstracción e implementación)
que pueden desarrollarse independientemente la una de la otra.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](bridge.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de código](bridge/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Composite](../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x,/images/patterns/cards/composite-mini-3x.png 3x"}
:::

#### Composite []{.dp-popularity} {#composite .dp-pattern-title}

::: dp-pattern-intent
Permite componer objetos en estructuras de árbol y trabajar con esas
estructuras como si fueran objetos individuales.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](composite.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](composite/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Decorator](../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x,/images/patterns/cards/decorator-mini-3x.png 3x"}
:::

#### Decorator []{.dp-popularity} {#decorator .dp-pattern-title}

::: dp-pattern-intent
Permite añadir funcionalidades a objetos colocando estos objetos dentro
de objetos encapsuladores especiales que contienen estas
funcionalidades.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](decorator.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](decorator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Facade](../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x,/images/patterns/cards/facade-mini-3x.png 3x"}
:::

#### Facade []{.dp-popularity} {#facade .dp-pattern-title}

::: dp-pattern-intent
Proporciona una interfaz simplificada a una biblioteca, un framework o
cualquier otro grupo complejo de clases.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](facade.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de código](facade/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Flyweight](../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x,/images/patterns/cards/flyweight-mini-3x.png 3x"}
:::

#### Flyweight []{.dp-popularity} {#flyweight .dp-pattern-title}

::: dp-pattern-intent
Permite mantener más objetos dentro de la cantidad disponible de memoria
RAM compartiendo las partes comunes del estado entre varios objetos en
lugar de mantener toda la información en cada objeto.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](flyweight.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](flyweight/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-structural}
::: dp-pattern-card
![Proxy](../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x,/images/patterns/cards/proxy-mini-3x.png 3x"}
:::

#### Proxy []{.dp-popularity} {#proxy .dp-pattern-title}

::: dp-pattern-intent
Permite proporcionar un sustituto o marcador de posición para otro
objeto. Un proxy controla el acceso al objeto original, permitiéndote
hacer algo antes o después de que la solicitud llegue al objeto
original.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](proxy.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de código](proxy/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

#### Patrones de comportamiento {#patrones-de-comportamiento .dp-h4 .dp-behavioral-title}

::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: dp-double-cols
:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Chain of
Responsibility](../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x,/images/patterns/cards/chain-of-responsibility-mini-3x.png 3x"}
:::

#### Chain of Responsibility []{.dp-popularity} {#chain-of-responsibility .dp-pattern-title}

::: dp-pattern-intent
Permite pasar solicitudes a lo largo de una cadena de manejadores. Al
recibir una solicitud, cada manejador decide si la procesa o si la pasa
al siguiente manejador de la cadena.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](chain-of-responsibility.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](chain-of-responsibility/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Command](../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x,/images/patterns/cards/command-mini-3x.png 3x"}
:::

#### Command []{.dp-popularity} {#command .dp-pattern-title}

::: dp-pattern-intent
Convierte una solicitud en un objeto independiente que contiene toda la
información sobre la solicitud. Esta transformación te permite
parametrizar los métodos con diferentes solicitudes, retrasar o poner en
cola la ejecución de una solicitud y soportar operaciones que no se
pueden realizar.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](command.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](command/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Iterator](../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x,/images/patterns/cards/iterator-mini-3x.png 3x"}
:::

#### Iterator []{.dp-popularity} {#iterator .dp-pattern-title}

::: dp-pattern-intent
Permite recorrer elementos de una colección sin exponer su
representación subyacente (lista, pila, árbol, etc.).
:::

::::: dp-pattern-links
<div>

[ Artículo principal](iterator.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](iterator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Mediator](../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x,/images/patterns/cards/mediator-mini-3x.png 3x"}
:::

#### Mediator []{.dp-popularity} {#mediator .dp-pattern-title}

::: dp-pattern-intent
Permite reducir las dependencias caóticas entre objetos. El patrón
restringe las comunicaciones directas entre los objetos, forzándolos a
colaborar únicamente a través de un objeto mediador.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](mediator.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](mediator/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Memento](../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x,/images/patterns/cards/memento-mini-3x.png 3x"}
:::

#### Memento []{.dp-popularity} {#memento .dp-pattern-title}

::: dp-pattern-intent
Permite guardar y restaurar el estado previo de un objeto sin revelar
los detalles de su implementación.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](memento.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](memento/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Observer](../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x,/images/patterns/cards/observer-mini-3x.png 3x"}
:::

#### Observer []{.dp-popularity} {#observer .dp-pattern-title}

::: dp-pattern-intent
Permite definir un mecanismo de suscripción para notificar a varios
objetos sobre cualquier evento que le suceda al objeto que están
observando.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](observer.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](observer/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![State](../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x,/images/patterns/cards/state-mini-3x.png 3x"}
:::

#### State []{.dp-popularity} {#state .dp-pattern-title}

::: dp-pattern-intent
Permite a un objeto alterar su comportamiento cuando su estado interno
cambia. Parece como si el objeto cambiara su clase.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](state.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de código](state/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Strategy](../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x,/images/patterns/cards/strategy-mini-3x.png 3x"}
:::

#### Strategy []{.dp-popularity} {#strategy .dp-pattern-title}

::: dp-pattern-intent
Permite definir una familia de algoritmos, colocar cada uno de ellos en
una clase separada y hacer sus objetos intercambiables.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](strategy.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](strategy/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Template
Method](../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x,/images/patterns/cards/template-method-mini-3x.png 3x"}
:::

#### Template Method []{.dp-popularity} {#template-method .dp-pattern-title}

::: dp-pattern-intent
Define el esqueleto de un algoritmo en la superclase pero permite que
las subclases sobrescriban pasos del algoritmo sin cambiar su
estructura.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](template-method.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](template-method/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::

:::::::: {.dp-pattern-item .dp-behavioral}
::: dp-pattern-card
![Visitor](../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x,/images/patterns/cards/visitor-mini-3x.png 3x"}
:::

#### Visitor []{.dp-popularity} {#visitor .dp-pattern-title}

::: dp-pattern-intent
Permite separar algoritmos de los objetos sobre los que operan.
:::

::::: dp-pattern-links
<div>

[ Artículo principal](visitor.html){.dp-pattern-link}

</div>

<div>

[ Ejemplo de
código](visitor/go/example.html#example-0){.dp-pattern-link}

</div>
:::::
::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
