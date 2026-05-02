::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Prototype {#prototype .title}

::: aka
También llamado:
[Prototipo, ]{style="display:inline-block"}[Clon, ]{style="display:inline-block"}[Clone]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Prototype** es un patrón de diseño creacional que nos permite copiar
objetos existentes sin que el código dependa de sus clases.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Prototype" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Digamos que tienes un objeto y quieres crear una copia exacta de él.
¿Cómo lo harías? En primer lugar, debes crear un nuevo objeto de la
misma clase. Después debes recorrer todos los campos del objeto original
y copiar sus valores en el nuevo objeto.

¡Bien! Pero hay una trampa. No todos los objetos se pueden copiar de
este modo, porque algunos de los campos del objeto pueden ser privados e
invisibles desde fuera del propio objeto.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-1-ese215.png?id=f7b66c55b0d72a95499d460239d253d6"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-es.png?id=f7b66c55b0d72a95499d460239d253d6 600w,/images/patterns/content/prototype/prototype-comic-1-es-1.5x.png?id=993bd95413a210a4847b939f9cf39ffb 900w,/images/patterns/content/prototype/prototype-comic-1-es-2x.png?id=503d13e8b69703e80dcfc3a59368a7d6 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="¿Qué puede salir mal cuando copiamos cosas &quot;desde fuera&quot;?" />
<figcaption><p><a href="https://vimeo.com/120816425">No siempre</a> es
posible copiar un objeto “desde fuera”.</p></figcaption>
</figure>

Hay otro problema con el enfoque directo. Dado que debes conocer la
clase del objeto para crear un duplicado, el código se vuelve
dependiente de esa clase. Si esta dependencia adicional no te da miedo,
todavía hay otra trampa. En ocasiones tan solo conocemos la interfaz que
sigue el objeto, pero no su clase concreta, cuando, por ejemplo, un
parámetro de un método acepta cualquier objeto que siga cierta interfaz.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Prototype delega el proceso de clonación a los propios objetos
que están siendo clonados. El patrón declara una interfaz común para
todos los objetos que soportan la clonación. Esta interfaz nos permite
clonar un objeto sin acoplar el código a la clase de ese objeto.
Normalmente, dicha interfaz contiene un único método `clonar`.

La implementación del método `clonar` es muy parecida en todas las
clases. El método crea un objeto a partir de la clase actual y lleva
todos los valores de campo del viejo objeto, al nuevo. Se puede incluso
copiar campos privados, porque la mayoría de los lenguajes de
programación permite a los objetos acceder a campos privados de otros
objetos que pertenecen a la misma clase.

Un objeto que soporta la clonación se denomina *prototipo*. Cuando tus
objetos tienen decenas de campos y miles de configuraciones posibles, la
clonación puede servir como alternativa a la creación de subclases.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-2-es19a3.png?id=84c7c097d4127b1ae7f6bda14ee9b24b"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-es.png?id=84c7c097d4127b1ae7f6bda14ee9b24b 343w,/images/patterns/content/prototype/prototype-comic-2-es-1.5x.png?id=7a1593729c0a1a7eca31ff85da02ac0f 515w,/images/patterns/content/prototype/prototype-comic-2-es-2x.png?id=e9ac65cc8bfa333a5212f97b138836c4 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Prototipos prefabricados" />
<figcaption><p>Los prototipos prefabricados pueden ser una alternativa a
las subclases.</p></figcaption>
</figure>

Funciona así: se crea un grupo de objetos configurados de maneras
diferentes. Cuando necesites un objeto como el que has configurado,
clonas un prototipo en lugar de construir un nuevo objeto desde cero.
:::

## Analogía del mundo real

En la vida real, los prototipos se utilizan para realizar pruebas de
todo tipo antes de comenzar con la producción en masa de un producto.
Sin embargo, en este caso, los prototipos no forman parte de una
producción real, sino que juegan un papel pasivo.

<figure class="image">
<img
src="../../images/patterns/content/prototype/prototype-comic-3-esdf1c.png?id=3262eeb6f8e8f6e06f060fef253b3270"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-es.png?id=3262eeb6f8e8f6e06f060fef253b3270 600w,/images/patterns/content/prototype/prototype-comic-3-es-1.5x.png?id=b109c492e90df806c772cf9a8c3cbc3b 900w,/images/patterns/content/prototype/prototype-comic-3-es-2x.png?id=80c158cc60776f790de4a191e63fdde0 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="La división de una célula" />
<figcaption><p>La división de una célula.</p></figcaption>
</figure>

Ya que los prototipos industriales en realidad no se copian a sí mismos,
una analogía más precisa del patrón es el proceso de la división
mitótica de una célula (biología, ¿recuerdas?). Tras la división
mitótica, se forma un par de células idénticas. La célula original actúa
como prototipo y asume un papel activo en la creación de la copia.

:::::::: {.section .structure-container}
##  Estructura {#structure}

::::::: structure
#### Implementación básica

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="La estructura del patrón de diseño Prototype" /><img
src="../../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="La estructura del patrón de diseño Prototype" />
</figure>
:::
::::

1.  La interfaz **Prototipo** declara los métodos de clonación. En la
    mayoría de los casos, se trata de un único método `clonar`.

2.  La clase **Prototipo Concreto** implementa el método de clonación.
    Además de copiar la información del objeto original al clon, este
    método también puede gestionar algunos casos extremos del proceso de
    clonación, como, por ejemplo, clonar objetos vinculados, deshacer
    dependencias recursivas, etc.

3.  El **Cliente** puede producir una copia de cualquier objeto que siga
    la interfaz del prototipo.

#### Implementación del registro de prototipos

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="El registro de prototipos" /><img
src="../../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="El registro de prototipos" />
</figure>
:::
::::

1.  El **Registro de Prototipos** ofrece una forma sencilla de acceder a
    prototipos de uso frecuente. Almacena un grupo de objetos
    prefabricados listos para ser copiados. El registro de prototipos
    más sencillo es una tabla *hash* con los pares `name → prototype`.
    No obstante, si necesitas un criterio de búsqueda más preciso que un
    simple nombre, puedes crear una versión mucho más robusta del
    registro.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Prototype** nos permite producir copias
exactas de objetos geométricos sin acoplar el código a sus clases.

<figure class="image">
<img
src="../../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Ejemplo de la estructura del patrón Prototype" />
<figcaption><p>Clonación de un grupo de objetos que pertenece a una
jerarquía de clase.</p></figcaption>
</figure>

Todas las clases de forma siguen la misma interfaz, que proporciona un
método de clonación. Una subclase puede invocar el método de clonación
padre antes de copiar sus propios valores de campo al objeto resultante.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Prototipo base.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // Un constructor normal.
    constructor Shape() is
        // ...

    // El constructor prototipo. Un nuevo objeto se inicializa
    // con valores del objeto existente.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // La operación clonar devuelve una de las subclases de
    // Shape (Forma).
    abstract method clone():Shape


// Prototipo concreto. El método de clonación crea un nuevo
// objeto y lo pasa al constructor. Hasta que el constructor
// termina, tiene una referencia a un nuevo clon. De este modo
// nadie tiene acceso a un clon a medio terminar. Esto garantiza
// la consistencia del resultado de la clonación.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // Para copiar campos privados definidos en la clase
        // padre es necesaria una llamada a un constructor
        // padre.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// En alguna parte del código cliente.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // La variable `anotherCircle` (otroCírculo) contiene
        // una copia exacta del objeto `circle`.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Prototype es genial porque te permite producir una
        // copia de un objeto sin conocer nada de su tipo.
        Array shapesCopy = new Array of Shapes.

        // Por ejemplo, no conocemos los elementos exactos de la
        // matriz de formas. Lo único que sabemos es que son
        // todas formas. Pero, gracias al polimorfismo, cuando
        // invocamos el método `clonar` en una forma, el
        // programa comprueba su clase real y ejecuta el método
        // de clonación adecuado definido en dicha clase. Por
        // eso obtenemos los clones adecuados en lugar de un
        // grupo de simples objetos Shape.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // La matriz `shapesCopy` contiene copias exactas del
        // hijo de la matriz `shape`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Prototype cuando tu código no deba depender de las
clases concretas de objetos que necesites copiar.
:::

::: applicability-solution
Esto sucede a menudo cuando tu código funciona con objetos pasados por
código de terceras personas a través de una interfaz. Las clases
concretas de estos objetos son desconocidas y no podrías depender de
ellas aunque quisieras.

El patrón Prototype proporciona al código cliente una interfaz general
para trabajar con todos los objetos que soportan la clonación. Esta
interfaz hace que el código cliente sea independiente de las clases
concretas de los objetos que clona.
:::

::: applicability-problem
Utiliza el patrón cuando quieras reducir la cantidad de subclases que
solo se diferencian en la forma en que inicializan sus respectivos
objetos. Puede ser que alguien haya creado estas subclases para poder
crear objetos con una configuración específica.
:::

::: applicability-solution
El patrón Prototype te permite utilizar como prototipos un grupo de
objetos prefabricados, configurados de maneras diferentes.

En lugar de instanciar una subclase que coincida con una configuración,
el cliente puede, sencillamente, buscar el prototipo adecuado y
clonarlo.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Crea la interfaz del prototipo y declara el método `clonar` en ella,
    o, simplemente, añade el método a todas las clases de una jerarquía
    de clase existente, si la tienes.

2.  Una clase de prototipo debe definir el constructor alternativo que
    acepta un objeto de dicha clase como argumento. El constructor debe
    copiar los valores de todos los campos definidos en la clase del
    objeto que se le pasa a la instancia recién creada. Si deseas
    cambiar una subclase, debes invocar al constructor padre para
    permitir que la superclase gestione la clonación de sus campos
    privados.

    Si el lenguaje de programación que utilizas no soporta la sobrecarga
    de métodos, puedes definir un método especial para copiar la
    información del objeto. El constructor es el lugar más adecuado para
    hacerlo, porque entrega el objeto resultante justo después de
    invocar el operador `new`.

3.  Normalmente, el método de clonación consiste en una sola línea que
    ejecuta un operador `new` con la versión prototípica del
    constructor. Observa que todas las clases deben sobreescribir
    explícitamente el método de clonación y utilizar su propio nombre de
    clase junto al operador `new`. De lo contrario, el método de
    clonación puede producir un objeto a partir de una clase madre.

4.  Opcionalmente, puedes crear un registro de prototipos centralizado
    para almacenar un catálogo de prototipos de uso frecuente.

    Puedes implementar el registro como una nueva clase de fábrica o
    colocarlo en la clase base de prototipo con un método estático para
    buscar el prototipo. Este método debe buscar un prototipo con base
    en el criterio de búsqueda que el código cliente pase al método. El
    criterio puede ser una etiqueta tipo *string* o un grupo complejo de
    parámetros de búsqueda. Una vez encontrado el prototipo adecuado, el
    registro deberá clonarlo y devolver la copia al cliente.

    Por último, sustituye las llamadas directas a los constructores de
    las subclases por llamadas al método de fábrica del registro de
    prototipos.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes clonar objetos sin acoplarlos a sus clases concretas.
-    Puedes evitar un código de inicialización repetido clonando
    prototipos prefabricados.
-    Puedes crear objetos complejos con más facilidad.
-    Obtienes una alternativa a la herencia al tratar con preajustes de
    configuración para objetos complejos.
:::

::: col-sm-6
-    Clonar objetos complejos con referencias circulares puede resultar
    complicado.
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

-   Las clases del [Abstract Factory](abstract-factory.html) a menudo se
    basan en un grupo de [métodos de fábrica](factory-method.html), pero
    también puedes utilizar [Prototype](prototype.html) para escribir
    los métodos de estas clases.

-   [Prototype](prototype.html) puede ayudar a cuando necesitas guardar
    copias de [Comandos](command.html) en un historial.

-   Los diseños que hacen un uso amplio de [Composite](composite.html) y
    [Decorator](decorator.html) a menudo pueden beneficiarse del uso del
    [Prototype](prototype.html). Aplicar el patrón te permite clonar
    estructuras complejas en lugar de reconstruirlas desde cero.

-   [Prototype](prototype.html) no se basa en la herencia, por lo que no
    presenta sus inconvenientes. No obstante, *Prototype* requiere de
    una inicialización complicada del objeto clonado. [Factory
    Method](factory-method.html) se basa en la herencia, pero no
    requiere de un paso de inicialización.

-   En ocasiones, [Prototype](prototype.html) puede ser una alternativa
    más simple al patrón [Memento](memento.html). Esto funciona si el
    objeto cuyo estado quieres almacenar en el historial es
    suficientemente sencillo y no tiene enlaces a recursos externos, o
    estos son fáciles de restablecer.

-   Los patrones [Abstract Factory](abstract-factory.html),
    [Builder](builder.html) y [Prototype](prototype.html) pueden todos
    ellos implementarse como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Prototype en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototype en C#"){.prog-lang-link}
[![Prototype en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototype en C++"){.prog-lang-link}
[![Prototype en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototype en Go"){.prog-lang-link}
[![Prototype en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototype en Java"){.prog-lang-link}
[![Prototype en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototype en PHP"){.prog-lang-link}
[![Prototype en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototype en Python"){.prog-lang-link}
[![Prototype en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototype en Ruby"){.prog-lang-link}
[![Prototype en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototype en Rust"){.prog-lang-link}
[![Prototype en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototype en Swift"){.prog-lang-link}
[![Prototype en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototype en TypeScript"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Builder](builder.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
