:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Builder {#builder .title}

::: aka
También llamado: [Constructor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Builder** es un patrón de diseño creacional que nos permite construir
objetos complejos paso a paso. El patrón nos permite producir distintos
tipos y representaciones de un objeto empleando el mismo código
de construcción.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-ese7de.png?id=6a685e06fc69071a93b7282298a55837"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-es.png?id=6a685e06fc69071a93b7282298a55837 640w,/images/patterns/content/builder/builder-es-1.5x.png?id=c09f576b2875fd570d4ab228bb6778b4 960w,/images/patterns/content/builder/builder-es-2x.png?id=23aa59f21675a8694523a6d03c00a8ff 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Builder" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina un objeto complejo que requiere una inicialización laboriosa,
paso a paso, de muchos campos y objetos anidados. Normalmente, este
código de inicialización está sepultado dentro de un monstruoso
constructor con una gran cantidad de parámetros. O, peor aún: disperso
por todo el código cliente.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Una gran cantidad de subclases genera otro problema" />
<figcaption><p>Crear una subclase por cada configuración posible de un
objeto puede complicar demasiado el programa.</p></figcaption>
</figure>

Por ejemplo, pensemos en cómo crear un objeto `Casa`. Para construir una
casa sencilla, debemos construir cuatro paredes y un piso, así como
instalar una puerta, colocar un par de ventanas y ponerle un tejado.
Pero ¿qué pasa si quieres una casa más grande y luminosa, con un jardín
y otros extras (como sistema de calefacción, instalación de fontanería y
cableado eléctrico)?

La solución más sencilla es extender la clase base `Casa` y crear un
grupo de subclases que cubran todas las combinaciones posibles de los
parámetros. Pero, en cualquier caso, acabarás con una cantidad
considerable de subclases. Cualquier parámetro nuevo, como el estilo del
porche, exigirá que incrementes esta jerarquía aún más.

Existe otra posibilidad que no implica generar subclases. Puedes crear
un enorme constructor dentro de la clase base `Casa` con todos los
parámetros posibles para controlar el objeto casa. Aunque es cierto que
esta solución elimina la necesidad de las subclases, genera otro
problema.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="El constructor telescópico" />
<figcaption><p>Un constructor con un montón de parámetros tiene su
inconveniente: no todos los parámetros son necesarios todo
el tiempo.</p></figcaption>
</figure>

En la mayoría de los casos, gran parte de los parámetros no se
utilizará, lo que provocará que [las llamadas al constructor sean
bastante feas](../smells/long-parameter-list.html). Por ejemplo, solo
una pequeña parte de las casas tiene piscina, por lo que los parámetros
relacionados con piscinas serán inútiles en nueve de cada diez casos.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Builder sugiere que saques el código de construcción del
objeto de su propia clase y lo coloques dentro de objetos independientes
llamados *constructores*.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Aplicación del patrón Builder" />
<figcaption><p>El patrón Builder te permite construir objetos complejos
paso a paso. El patrón Builder no permite a otros objetos acceder al
producto mientras se construye.</p></figcaption>
</figure>

El patrón organiza la construcción de objetos en una serie de pasos
(`construirParedes`, `construirPuerta`, etc.). Para crear un objeto, se
ejecuta una serie de estos pasos en un objeto constructor. Lo importante
es que no necesitas invocar todos los pasos. Puedes invocar sólo
aquellos que sean necesarios para producir una configuración particular
de un objeto.

Puede ser que algunos pasos de la construcción necesiten una
implementación diferente cuando tengamos que construir distintas
representaciones del producto. Por ejemplo, las paredes de una cabaña
pueden ser de madera, pero las paredes de un castillo tienen que ser de
piedra.

En este caso, podemos crear varias clases constructoras distintas que
implementen la misma serie de pasos de construcción, pero de forma
diferente. Entonces podemos utilizar estos constructores en el proceso
de construcción (por ejemplo, una serie ordenada de llamadas a los pasos
de construcción) para producir distintos tipos de objetos.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-1-es2f26.png?id=0ce8ce42120a359bab252d8d057e294d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-es.png?id=0ce8ce42120a359bab252d8d057e294d 600w,/images/patterns/content/builder/builder-comic-1-es-1.5x.png?id=a52a738873cc95d141258018f43b925e 900w,/images/patterns/content/builder/builder-comic-1-es-2x.png?id=a9f38324cd69f3b232b36d63c50e60ab 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Los distintos constructores ejecutan la misma tarea de
formas distintas.</p></figcaption>
</figure>

Por ejemplo, imagina un constructor que construye todo de madera y
vidrio, otro que construye todo con piedra y hierro y un tercero que
utiliza oro y diamantes. Al invocar la misma serie de pasos, obtenemos
una casa normal del primer constructor, un pequeño castillo del segundo
y un palacio del tercero. Sin embargo, esto sólo funcionaría si el
código cliente que invoca los pasos de construcción es capaz de
interactuar con los constructores mediante una interfaz común.

#### Clase directora

Puedes ir más lejos y extraer una serie de llamadas a los pasos del
constructor que utilizas para construir un producto y ponerlas en una
clase independiente llamada *directora*. La clase directora define el
orden en el que se deben ejecutar los pasos de construcción, mientras
que el constructor proporciona la implementación de dichos pasos.

<figure class="image">
<img
src="../../images/patterns/content/builder/builder-comic-2-es2589.png?id=b6ea6473add16bc853f87086217721c6"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-es.png?id=b6ea6473add16bc853f87086217721c6 343w,/images/patterns/content/builder/builder-comic-2-es-1.5x.png?id=4d2e118f6e71bb49a803ad770b82d7b5 515w,/images/patterns/content/builder/builder-comic-2-es-2x.png?id=55601a71e3c066fe003b3ebadfd0fa71 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>La clase directora sabe qué pasos de construcción
ejecutar para lograr un producto que funcione.</p></figcaption>
</figure>

No es estrictamente necesario tener una clase directora en el programa,
ya que se pueden invocar los pasos de construcción en un orden
específico directamente desde el código cliente. No obstante, la clase
directora puede ser un buen lugar donde colocar distintas rutinas de
construcción para poder reutilizarlas a lo largo del programa.

Además, la clase directora esconde por completo los detalles de la
construcción del producto al código cliente. El cliente sólo necesita
asociar un objeto constructor con una clase directora, utilizarla para
iniciar la construcción, y obtener el resultado del objeto constructor.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Estructura del patrón de diseño Builder" /><img
src="../../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Estructura del patrón de diseño Builder" />
</figure>
:::

1.  La interfaz **Constructora** declara pasos de construcción de
    producto que todos los tipos de objetos constructores tienen en
    común.

2.  Los **Constructores Concretos** ofrecen distintas implementaciones
    de los pasos de construcción. Los constructores concretos pueden
    crear productos que no siguen la interfaz común.

3.  Los **Productos** son los objetos resultantes. Los productos
    construidos por distintos objetos constructores no tienen que
    pertenecer a la misma jerarquía de clases o interfaz.

4.  La clase **Directora** define el orden en el que se invocarán los
    pasos de construcción, por lo que puedes crear y reutilizar
    configuraciones específicas de los productos.

5.  El **Cliente** debe asociar uno de los objetos constructores con la
    clase directora. Normalmente, se hace una sola vez mediante los
    parámetros del constructor de la clase directora, que utiliza el
    objeto constructor para el resto de la construcción. No obstante,
    existe una solución alternativa para cuando el cliente pasa el
    objeto constructor al método de producción de la clase directora. En
    este caso, puedes utilizar un constructor diferente cada vez que
    produzcas algo con la clase directora.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo del patrón **Builder** ilustra cómo se puede reutilizar el
mismo código de construcción de objetos a la hora de construir distintos
tipos de productos, como automóviles, y crear los correspondientes
manuales para esos automóviles.

<figure class="image">
<img
src="../../images/patterns/diagrams/builder/example-es90a4.png?id=c326e99e007a469951240da70b4294d7"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-es.png?id=c326e99e007a469951240da70b4294d7 500w,/images/patterns/diagrams/builder/example-es-1.5x.png?id=9d63e68861ad2bb294e23deca6c8d8d9 750w,/images/patterns/diagrams/builder/example-es-2x.png?id=dcace51331439c48a6cd9fc6f4f930f7 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Ejemplo de la estructura del patrón Builder" />
<figcaption><p>Ejemplo de una construcción paso a paso de automóviles y
de los manuales de usuario para esos modelos
de automóvil.</p></figcaption>
</figure>

Un automóvil es un objeto complejo que puede construirse de mil maneras
diferentes. En lugar de saturar la clase `Automóvil` con un constructor
enorme, extrajimos el código de ensamblaje del automóvil y lo pusimos en
una clase constructora de automóviles independiente. Esta clase tiene un
grupo de métodos para configurar las distintas partes de un automóvil.

Si el código cliente necesita ensamblar un modelo de automóvil con
ajustes especiales, puede trabajar directamente con el objeto
constructor. Por otro lado, el cliente puede delegar el ensamblaje a la
clase directora, que sabe cómo utilizar un objeto constructor para
construir varios de los modelos más populares de automóviles.

Puede que te sorprenda, pero todo automóvil necesita un manual (en
serio, ¿quién se los lee?). El manual explica cada característica del
automóvil, de modo que los detalles del manual varían de un modelo a
otro. Por eso tiene lógica reutilizar un proceso de construcción
existente para automóviles reales y sus respectivos manuales. Por
supuesto, elaborar un manual no es lo mismo que fabricar un automóvil,
por lo que debemos incluir otra clase constructora especializada en
elaborar manuales. Esta clase implementa los mismos métodos de
construcción que su hermana constructora de automóviles , pero, en lugar
de fabricar piezas del automóvil, las describe. Al pasar estos
constructores al mismo objeto director, podemos construir tanto un
automóvil como un manual.

La última parte consiste en buscar el objeto resultante. Un automóvil de
metal y un manual de papel, aunque estén relacionados, son objetos muy
diferentes. No podemos colocar un método para buscar resultados en la
clase directora sin acoplarla a clases de productos concretos. Por lo
tanto, obtenemos el resultado de la construcción del constructor que
realizó el trabajo.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// El uso del patrón Builder sólo tiene sentido cuando tus
// productos son bastante complejos y requieren una
// configuración extensiva. Los dos siguientes productos están
// relacionados, aunque no tienen una interfaz común.
class Car is
    // Un coche puede tener un GPS, una computadora de
    // navegación y cierto número de asientos. Los distintos
    // modelos de coches (deportivo, SUV, descapotable) pueden
    // tener distintas características instaladas o habilitadas.

class Manual is
    // Cada coche debe contar con un manual de usuario que se
    // corresponda con la configuración del coche y explique
    // todas sus características.


// La interfaz constructora especifica métodos para crear las
// distintas partes de los objetos del producto.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// Las clases constructoras concretas siguen la interfaz
// constructora y proporcionan implementaciones específicas de
// los pasos de construcción. Tu programa puede tener multitud
// de variaciones de objetos constructores, cada una de ellas
// implementada de forma diferente.
class CarBuilder implements Builder is
    private field car:Car

    // Una nueva instancia de la clase constructora debe
    // contener un objeto de producto en blanco que utiliza en
    // el montaje posterior.
    constructor CarBuilder() is
        this.reset()

    // El método reset despeja el objeto en construcción.
    method reset() is
        this.car = new Car()

    // Todos los pasos de producción funcionan con la misma
    // instancia de producto.
    method setSeats(...) is
        // Establece la cantidad de asientos del coche.

    method setEngine(...) is
        // Instala un motor específico.

    method setTripComputer(...) is
        // Instala una computadora de navegación.

    method setGPS(...) is
        // Instala un GPS.

    // Los constructores concretos deben proporcionar sus
    // propios métodos para obtener resultados. Esto se debe a
    // que varios tipos de objetos constructores pueden crear
    // productos completamente diferentes de los cuales no todos
    // siguen la misma interfaz. Por lo tanto, dichos métodos no
    // pueden declararse en la interfaz constructora (al menos
    // no en un lenguaje de programación de tipado estático).
    //
    // Normalmente, tras devolver el resultado final al cliente,
    // una instancia constructora debe estar lista para empezar
    // a generar otro producto. Ese es el motivo por el que es
    // práctica común invocar el método reset al final del
    // cuerpo del método `getProduct`. Sin embargo, este
    // comportamiento no es obligatorio y puedes hacer que tu
    // objeto constructor espere una llamada reset explícita del
    // código cliente antes de desechar el resultado anterior.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// Al contrario que otros patrones creacionales, Builder te
// permite construir productos que no siguen una interfaz común.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // Documenta las características del asiento del coche.

    method setEngine(...) is
        // Añade instrucciones del motor.

    method setTripComputer(...) is
        // Añade instrucciones de la computadora de navegación.

    method setGPS(...) is
        // Añade instrucciones del GPS.

    method getProduct():Manual is
        // Devuelve el manual y rearma el constructor.


// El director sólo es responsable de ejecutar los pasos de
// construcción en una secuencia particular. Resulta útil cuando
// se crean productos de acuerdo con un orden o configuración
// específicos. En sentido estricto, la clase directora es
// opcional, ya que el cliente puede controlar directamente los
// objetos constructores.
class Director is
    // El director funciona con cualquier instancia de
    // constructor que le pase el código cliente. De esta forma,
    // el código cliente puede alterar el tipo final del
    // producto recién montado. El director puede construir
    // multitud de variaciones de producto utilizando los mismos
    // pasos de construcción.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ...


// El código cliente crea un objeto constructor, lo pasa al
// director y después inicia el proceso de construcción. El
// resultado final se extrae del objeto constructor.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // El producto final a menudo se extrae de un objeto
        // constructor, ya que el director no conoce y no
        // depende de constructores y productos concretos.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Builder para evitar un "constructor telescópico".
:::

::: applicability-solution
Digamos que tenemos un constructor con diez parámetros opcionales.
Invocar a semejante bestia es poco práctico, por lo que sobrecargamos el
constructor y creamos varias versiones más cortas con menos parámetros.
Estos constructores siguen recurriendo al principal, pasando algunos
valores por defecto a cualquier parámetro omitido.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Crear un monstruo semejante sólo es posible en lenguajes
que soportan la sobrecarga de métodos, como C# o Java.</p></figcaption>
</figure>

El patrón Builder permite construir objetos paso a paso, utilizando tan
solo aquellos pasos que realmente necesitamos. Una vez implementado el
patrón, ya no hará falta apiñar decenas de parámetros dentro de los
constructores.
:::

::: applicability-problem
Utiliza el patrón Builder cuando quieras que el código sea capaz de
crear distintas representaciones de ciertos productos (por ejemplo,
casas de piedra y madera).
:::

::: applicability-solution
El patrón Builder se puede aplicar cuando la construcción de varias
representaciones de un producto requiera de pasos similares que sólo
varían en los detalles.

La interfaz constructora base define todos los pasos de construcción
posibles, mientras que los constructores concretos implementan estos
pasos para construir representaciones particulares del producto. Entre
tanto, la clase directora guía el orden de la construcción.
:::

::: applicability-problem
Utiliza el patrón Builder para construir árboles con el patrón
[Composite](composite.html) u otros objetos complejos.
:::

::: applicability-solution
El patrón Builder te permite construir productos paso a paso. Podrás
aplazar la ejecución de ciertos pasos sin descomponer el producto final.
Puedes incluso invocar pasos de forma recursiva, lo cual resulta útil
cuando necesitas construir un árbol de objetos.

Un constructor no expone el producto incompleto mientras ejecuta los
pasos de construcción. Esto evita que el código cliente extraiga un
resultado incompleto.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Asegúrate de poder definir claramente los pasos comunes de
    construcción para todas las representaciones disponibles del
    producto. De lo contrario, no podrás proceder a implementar el
    patrón.

2.  Declara estos pasos en la interfaz constructora base.

3.  Crea una clase constructora concreta para cada una de las
    representaciones de producto e implementa sus pasos de construcción.

    No olvides implementar un método para extraer el resultado de la
    construcción. La razón por la que este método no se puede declarar
    dentro de la interfaz constructora es que varios constructores
    pueden construir productos sin una interfaz común. Por lo tanto, no
    sabemos cuál será el tipo de retorno para un método como éste. No
    obstante, si trabajas con productos de una única jerarquía, el
    método de extracción puede añadirse sin problemas a la interfaz
    base.

4.  Piensa en crear una clase directora. Puede encapsular varias formas
    de construir un producto utilizando el mismo objeto constructor.

5.  El código cliente crea tanto el objeto constructor como el director.
    Antes de que empiece la construcción, el cliente debe pasar un
    objeto constructor al director. Normalmente, el cliente hace esto
    sólo una vez, mediante los parámetros del constructor del director.
    El director utiliza el objeto constructor para el resto de la
    construcción. Existe una manera alternativa, en la que el objeto
    constructor se pasa directamente al método de construcción del
    director.

6.  El resultado de la construcción tan solo se puede obtener
    directamente del director si todos los productos siguen la misma
    interfaz. De lo contrario, el cliente deberá extraer el resultado
    del constructor.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes construir objetos paso a paso, aplazar pasos de la
    construcción o ejecutar pasos de forma recursiva.
-    Puedes reutilizar el mismo código de construcción al construir
    varias representaciones de productos.
-    *Principio de responsabilidad única*. Puedes aislar un código de
    construcción complejo de la lógica de negocio del producto.
:::

::: col-sm-6
-    La complejidad general del código aumenta, ya que el patrón exige
    la creación de varias clases nuevas.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Muchos diseños empiezan utilizando el [Factory
    Method](factory-method.html) (menos complicado y más personalizable
    mediante las subclases) y evolucionan hacia [Abstract
    Factory](abstract-factory.html), [Prototype](prototype.html), o
    [Builder](builder.html) (más flexibles, pero más complicados).

-   [Builder](builder.html) se enfoca en construir objetos complejos,
    paso a paso. [Abstract Factory](abstract-factory.html) se
    especializa en crear familias de objetos relacionados. *Abstract
    Factory* devuelve el producto inmediatamente, mientras que *Builder*
    te permite ejecutar algunos pasos adicionales de construcción antes
    de extraer el producto.

-   Puedes utilizar [Builder](builder.html) al crear árboles
    [Composite](composite.html) complejos porque puedes programar sus
    pasos de construcción para que funcionen de forma recursiva.

-   Puedes combinar [Builder](builder.html) con [Bridge](bridge.html):
    la clase *directora* juega el papel de la abstracción, mientras que
    diferentes *constructoras* actúan como *implementaciones*.

-   Los patrones [Abstract Factory](abstract-factory.html),
    [Builder](builder.html) y [Prototype](prototype.html) pueden todos
    ellos implementarse como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Builder en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Builder en C#"){.prog-lang-link}
[![Builder en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Builder en C++"){.prog-lang-link}
[![Builder en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Builder en Go"){.prog-lang-link}
[![Builder en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Builder en Java"){.prog-lang-link}
[![Builder en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Builder en PHP"){.prog-lang-link}
[![Builder en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Builder en Python"){.prog-lang-link}
[![Builder en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Builder en Ruby"){.prog-lang-link}
[![Builder en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Builder en Rust"){.prog-lang-link}
[![Builder en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Builder en Swift"){.prog-lang-link}
[![Builder en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Builder en TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-es" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### ¡Apoya nuestro sitio web gratuito y compra el libro! {#apoya-nuestro-sitio-web-gratuito-y-compra-el-libro .title}

-   22 patrones de diseño y 8 principios explicados en profundidad
-   436 páginas bien estructuradas, fáciles de leer y libres de
    tecnicismos
-   225 ilustraciones y diagramas claros y útiles
-   Un archivo con ejemplos de código en 11 lenguajes
-   Todos los dispositivos soportados: Formatos PDF/EPUB/MOBI/KFX

[ Saber más...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Leer siguiente

[Prototype []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Comparación de
fábricas](factory-comparison.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-es" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-es6b6c.png?id=8220dd8a5470a7d81199673f68532e1b){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-es-2x.png?id=d0ed33274cf5aa89b4987c6abe659376 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Este artículo es parte de nuestro libro eléctronico **Sumérgete en los
patrones de diseño**.

[ Saber más...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
