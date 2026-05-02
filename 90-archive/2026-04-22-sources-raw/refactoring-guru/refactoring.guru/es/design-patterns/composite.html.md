::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Composite {#composite .title}

::: aka
También llamado: [Objeto
compuesto, ]{style="display:inline-block"}[Object
Tree]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Composite** es un patrón de diseño estructural que te permite componer
objetos en estructuras de árbol y trabajar con esas estructuras como si
fueran objetos individuales.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Composite" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

El uso del patrón Composite sólo tiene sentido cuando el modelo central
de tu aplicación puede representarse en forma de árbol.

Por ejemplo, imagina que tienes dos tipos de objetos: `Productos` y
`Cajas`. Una `Caja` puede contener varios `Productos` así como cierto
número de `Cajas` más pequeñas. Estas `Cajas` pequeñas también pueden
contener algunos `Productos` o incluso `Cajas` más pequeñas, y así
sucesivamente.

Digamos que decides crear un sistema de pedidos que utiliza estas
clases. Los pedidos pueden contener productos sencillos sin envolver,
así como cajas llenas de productos\... y otras cajas. ¿Cómo determinarás
el precio total de ese pedido?

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-es2c78.png?id=3b02eaf4b7744eb05b261ac48d3d3e4a"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-es.png?id=3b02eaf4b7744eb05b261ac48d3d3e4a 370w,/images/patterns/diagrams/composite/problem-es-1.5x.png?id=eaf132ceaca9d91b75f59ef895d7dcbc 555w,/images/patterns/diagrams/composite/problem-es-2x.png?id=36dce86ec18de08f859c8f1a7342e9ca 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Estructura de un pedido complejo" />
<figcaption><p>Un pedido puede incluir varios productos empaquetados en
cajas, que a su vez están empaquetados en cajas más grandes y así
sucesivamente. La estructura se asemeja a un árbol
boca abajo.</p></figcaption>
</figure>

Puedes intentar la solución directa: desenvolver todas las cajas,
repasar todos los productos y calcular el total. Esto sería viable en el
mundo real; pero en un programa no es tan fácil como ejecutar un bucle.
Tienes que conocer de antemano las clases de `Productos` y `Cajas` a
iterar, el nivel de anidación de las cajas y otros detalles
desagradables. Todo esto provoca que la solución directa sea demasiado
complicada, o incluso imposible.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Composite sugiere que trabajes con `Productos` y `Cajas` a
través de una interfaz común que declara un método para calcular el
precio total.

¿Cómo funcionaría este método? Para un producto, sencillamente devuelve
el precio del producto. Para una caja, recorre cada artículo que
contiene la caja, pregunta su precio y devuelve un total por la caja. Si
uno de esos artículos fuera una caja más pequeña, esa caja también
comenzaría a repasar su contenido y así sucesivamente, hasta que se
calcule el precio de todos los componentes internos. Una caja podría
incluso añadir costos adicionales al precio final, como costos de
empaquetado.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-esf3fa.png?id=7925738e1b255eacd97b27da3a88f50e"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-es.png?id=7925738e1b255eacd97b27da3a88f50e 600w,/images/patterns/content/composite/composite-comic-1-es-1.5x.png?id=ff332db7de204764e47f51b339dd3cfd 900w,/images/patterns/content/composite/composite-comic-1-es-2x.png?id=4f48f8d11fd2082b3f60fbc4d5d2fc16 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Solution sugerida por el patrón Composite" />
<figcaption><p>El patrón Composite te permite ejecutar un comportamiento
de forma recursiva sobre todos los componentes de un árbol
de objetos.</p></figcaption>
</figure>

La gran ventaja de esta solución es que no tienes que preocuparte por
las clases concretas de los objetos que componen el árbol. No tienes que
saber si un objeto es un producto simple o una sofisticada caja. Puedes
tratarlos a todos por igual a través de la interfaz común. Cuando
invocas un método, los propios objetos pasan la solicitud a lo largo del
árbol.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Un ejemplo de estructura militar" />
<figcaption><p>Un ejemplo de estructura militar.</p></figcaption>
</figure>

Los ejércitos de la mayoría de países se estructuran como jerarquías. Un
ejército está formado por varias divisiones; una división es un grupo de
brigadas y una brigada está formada por pelotones, que pueden dividirse
en escuadrones. Por último, un escuadrón es un pequeño grupo de soldados
reales. Las órdenes se dan en la parte superior de la jerarquía y se
pasan hacia abajo por cada nivel hasta que todos los soldados saben lo
que hay que hacer.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-es5711.png?id=8ca4da7e11a04c3e6d8d8c038364a893"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-es.png?id=8ca4da7e11a04c3e6d8d8c038364a893 360w,/images/patterns/diagrams/composite/structure-es-1.5x.png?id=748eea4237c6471c5f0a036d333a5dc2 540w,/images/patterns/diagrams/composite/structure-es-2x.png?id=ecfb776298ef331f15c33a6979f5a9b8 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Estructura del patrón de diseño Composite" /><img
src="../../images/patterns/diagrams/composite/structure-es-indexed13d5.png?id=33bb9d4f3056f2f523111a90c92c7af1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-es-indexed.png?id=33bb9d4f3056f2f523111a90c92c7af1 400w,/images/patterns/diagrams/composite/structure-es-indexed-1.5x.png?id=9a6ced45b3c2781c81495fd1cc9eb2a8 600w,/images/patterns/diagrams/composite/structure-es-indexed-2x.png?id=5a3681df3cabc9cde45de0bfd07b166d 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Estructura del patrón de diseño Composite" />
</figure>
:::

1.  La interfaz **Componente** describe operaciones que son comunes a
    elementos simples y complejos del árbol.

2.  La **Hoja** es un elemento básico de un árbol que no tiene
    subelementos.

    Normalmente, los componentes de la hoja acaban realizando la mayoría
    del trabajo real, ya que no tienen a nadie a quien delegarle el
    trabajo.

3.  El **Contenedor** (también llamado *compuesto*) es un elemento que
    tiene subelementos: hojas u otros contenedores. Un contenedor no
    conoce las clases concretas de sus hijos. Funciona con todos los
    subelementos únicamente a través de la interfaz componente.

    Al recibir una solicitud, un contenedor delega el trabajo a sus
    subelementos, procesa los resultados intermedios y devuelve el
    resultado final al cliente.

4.  El **Cliente** funciona con todos los elementos a través de la
    interfaz componente. Como resultado, el cliente puede funcionar de
    la misma manera tanto con elementos simples como complejos del
    árbol.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Composite** te permite implementar el
apilamiento (*stacking*) de formas geométricas en un editor gráfico.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Ejemplo de estructura del patrón Composite" />
<figcaption><p>Ejemplo del editor de
formas geométricas.</p></figcaption>
</figure>

La clase `GráficoCompuesto` es un contenedor que puede incluir cualquier
cantidad de subformas, incluyendo otras formas compuestas. Una forma
compuesta tiene los mismos métodos que una forma simple. Sin embargo, en
lugar de hacer algo por su cuenta, una forma compuesta pasa la solicitud
de forma recursiva a todos sus hijos y "suma" el resultado.

El código cliente trabaja con todas las formas a través de la interfaz
común a todas las clases de forma. De este modo, el cliente no sabe si
está trabajando con una forma simple o una compuesta. El cliente puede
trabajar con estructuras de objetos muy complejas sin acoplarse a las
clases concretas que forman esa estructura.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz componente declara operaciones comunes para
// objetos simples y complejos de una composición.
interface Graphic is
    method move(x, y)
    method draw()

// La clase hoja representa objetos finales de una composición.
// Un objeto hoja no puede tener ningún subobjeto. Normalmente,
// son los objetos hoja los que hacen el trabajo real, mientras
// que los objetos compuestos se limitan a delegar a sus
// subcomponentes.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Dibuja un punto en X e Y.

// Todas las clases de componente pueden extender otros
// componentes.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Dibuja un círculo en X y Y con radio R.

// La clase compuesta representa componentes complejos que
// pueden tener hijos. Normalmente los objetos compuestos
// delegan el trabajo real a sus hijos y después &quot;recapitulan&quot;
// el resultado.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // Un objeto compuesto puede añadir o eliminar otros
    // componentes (tanto simples como complejos) a o desde su
    // lista hija.
    method add(child: Graphic) is
        // Añade un hijo a la matriz de hijos.

    method remove(child: Graphic) is
        // Elimina un hijo de la matriz de hijos.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // Un compuesto ejecuta su lógica primaria de una forma
    // particular. Recorre recursivamente todos sus hijos,
    // recopilando y recapitulando sus resultados. Debido a que
    // los hijos del compuesto pasan esas llamadas a sus propios
    // hijos y así sucesivamente, se recorre todo el árbol de
    // objetos como resultado.
    method draw() is
        // 1. Para cada componente hijo:
        //     - Dibuja el componente.
        //     - Actualiza el rectángulo delimitador.
        // 2. Dibuja un rectángulo de línea punteada utilizando
        // las coordenadas de delimitación.


// El código cliente trabaja con todos los componentes a través
// de su interfaz base. De esta forma el código cliente puede
// soportar componentes de hoja simples así como compuestos
// complejos.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Combina componentes seleccionados para formar un
    // componente compuesto complejo.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Se dibujarán todos los componentes.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Composite cuando tengas que implementar una estructura
de objetos con forma de árbol.
:::

::: applicability-solution
El patrón Composite te proporciona dos tipos de elementos básicos que
comparten una interfaz común: hojas simples y contenedores complejos. Un
contenedor puede estar compuesto por hojas y por otros contenedores.
Esto te permite construir una estructura de objetos recursivos anidados
parecida a un árbol.
:::

::: applicability-problem
Utiliza el patrón cuando quieras que el código cliente trate elementos
simples y complejos de la misma forma.
:::

::: applicability-solution
Todos los elementos definidos por el patrón Composite comparten una
interfaz común. Utilizando esta interfaz, el cliente no tiene que
preocuparse por la clase concreta de los objetos con los que funciona.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Asegúrate de que el modelo central de tu aplicación pueda
    representarse como una estructura de árbol. Intenta dividirlo en
    elementos simples y contenedores. Recuerda que los contenedores
    deben ser capaces de contener tanto elementos simples como otros
    contenedores.

2.  Declara la interfaz componente con una lista de métodos que tengan
    sentido para componentes simples y complejos.

3.  Crea una clase hoja para representar elementos simples. Un programa
    puede tener varias clases hoja diferentes.

4.  Crea una clase contenedora para representar elementos complejos.
    Incluye un campo matriz en esta clase para almacenar referencias a
    subelementos. La matriz debe poder almacenar hojas y contenedores,
    así que asegúrate de declararla con el tipo de la interfaz
    componente.

    Al implementar los métodos de la interfaz componente, recuerda que
    un contenedor debe delegar la mayor parte del trabajo a los
    subelementos.

5.  Por último, define los métodos para añadir y eliminar elementos
    hijos dentro del contenedor.

    Ten en cuenta que estas operaciones se pueden declarar en la
    interfaz componente. Esto violaría el *Principio de segregación de
    la interfaz* porque los métodos de la clase hoja estarían vacíos. No
    obstante, el cliente podrá tratar a todos los elementos de la misma
    manera, incluso al componer el árbol.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes trabajar con estructuras de árbol complejas con mayor
    comodidad: utiliza el polimorfismo y la recursión en tu favor.
-    *Principio de abierto/cerrado*. Puedes introducir nuevos tipos de
    elemento en la aplicación sin descomponer el código existente, que
    ahora funciona con el árbol de objetos.
:::

::: col-sm-6
-    Puede resultar difícil proporcionar una interfaz común para clases
    cuya funcionalidad difiere demasiado. En algunos casos, tendrás que
    generalizar en exceso la interfaz componente, provocando que sea más
    difícil de comprender.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Puedes utilizar [Builder](builder.html) al crear árboles
    [Composite](composite.html) complejos porque puedes programar sus
    pasos de construcción para que funcionen de forma recursiva.

-   [Chain of Responsibility](chain-of-responsibility.html) se utiliza a
    menudo junto a [Composite](composite.html). En este caso, cuando un
    componente hoja recibe una solicitud, puede pasarla a lo largo de la
    cadena de todos los componentes padre hasta la raíz del árbol de
    objetos.

-   Puedes utilizar [Iteradores](iterator.html) para recorrer árboles
    [Composite](composite.html).

-   Puedes utilizar el patrón [Visitor](visitor.html) para ejecutar una
    operación sobre un árbol [Composite](composite.html) entero.

-   Puedes implementar nodos de hoja compartidos del árbol
    [Composite](composite.html) como [Flyweights](flyweight.html) para
    ahorrar memoria RAM.

-   [Composite](composite.html) y [Decorator](decorator.html) tienen
    diagramas de estructura similares ya que ambos se basan en la
    composición recursiva para organizar un número indefinido de
    objetos.

    Un *Decorator* es como un *Composite* pero sólo tiene un componente
    hijo. Hay otra diferencia importante: *Decorator* añade
    responsabilidades adicionales al objeto envuelto, mientras que
    *Composite* se limita a "recapitular" los resultados de sus hijos.

    No obstante, los patrones también pueden colaborar: puedes utilizar
    el *Decorator* para extender el comportamiento de un objeto
    específico del árbol *Composite*.

-   Los diseños que hacen un uso amplio de [Composite](composite.html) y
    [Decorator](decorator.html) a menudo pueden beneficiarse del uso del
    [Prototype](prototype.html). Aplicar el patrón te permite clonar
    estructuras complejas en lugar de reconstruirlas desde cero.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Composite en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Composite en C#"){.prog-lang-link}
[![Composite en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Composite en C++"){.prog-lang-link}
[![Composite en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Composite en Go"){.prog-lang-link}
[![Composite en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Composite en Java"){.prog-lang-link}
[![Composite en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Composite en PHP"){.prog-lang-link}
[![Composite en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Composite en Python"){.prog-lang-link}
[![Composite en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Composite en Ruby"){.prog-lang-link}
[![Composite en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Composite en Rust"){.prog-lang-link}
[![Composite en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Composite en Swift"){.prog-lang-link}
[![Composite en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Composite en TypeScript"){.prog-lang-link}
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

[Decorator []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Bridge](bridge.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
