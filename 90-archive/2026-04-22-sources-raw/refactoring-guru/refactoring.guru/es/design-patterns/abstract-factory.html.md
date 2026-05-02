::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Abstract Factory {#abstract-factory .title}

::: aka
También llamado: [Fábrica abstracta]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Abstract Factory** es un patrón de diseño creacional que nos permite
producir familias de objetos relacionados sin especificar sus
clases concretas.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-ese1ae.png?id=0378c9faca39afa20e41a4d37e7e3828"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-es.png?id=0378c9faca39afa20e41a4d37e7e3828 640w,/images/patterns/content/abstract-factory/abstract-factory-es-1.5x.png?id=646bb55e4905a572d9b01c7ef27f3039 960w,/images/patterns/content/abstract-factory/abstract-factory-es-2x.png?id=dd1c7afd35d181ec7de2006cbe25e93d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón Abstract Factory" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás creando un simulador de tienda de muebles. Tu código
está compuesto por clases que representan lo siguiente:

1.  Una familia de productos relacionados, digamos: `Silla` + `Sofá` +
    `Mesilla`.

2.  Algunas variantes de esta familia. Por ejemplo, los productos
    `Silla` + `Sofá` + `Mesilla` están disponibles en estas variantes:
    `Moderna`, `Victoriana`, `ArtDecó`.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/problem-es2777.png?id=0e80a49ec432de0af53ddeac87ce4840"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/problem-es.png?id=0e80a49ec432de0af53ddeac87ce4840 420w,/images/patterns/diagrams/abstract-factory/problem-es-1.5x.png?id=4b52ad2ba8c8efd875420818eca441da 630w,/images/patterns/diagrams/abstract-factory/problem-es-2x.png?id=4f3f5d6a7df37e814cfca21fc47fb186 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Familias de productos y sus variantes." />
<figcaption><p>Familias de productos y sus variantes.</p></figcaption>
</figure>

Necesitamos una forma de crear objetos individuales de mobiliario para
que combinen con otros objetos de la misma familia. Los clientes se
enfadan bastante cuando reciben muebles que no combinan.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-1-es68bc.png?id=426c7300b9895e39cfa7a6440bcc026f"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-es.png?id=426c7300b9895e39cfa7a6440bcc026f 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-es-1.5x.png?id=8d1eacccc9e1806f4e02eba706a90641 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-es-2x.png?id=8c6604594ace05200d4fd73c468199b4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Un sofá de estilo moderno no combina con unas sillas de
estilo victoriano.</p></figcaption>
</figure>

Además, no queremos cambiar el código existente al añadir al programa
nuevos productos o familias de productos. Los comerciantes de muebles
actualizan sus catálogos muy a menudo, y debemos evitar tener que
cambiar el código principal cada vez que esto ocurra.
:::

::: {.section .solution}
##  Solución {#solution}

Lo primero que sugiere el patrón Abstract Factory es que declaremos de
forma explícita interfaces para cada producto diferente de la familia de
productos (por ejemplo, silla, sofá o mesilla). Después podemos hacer
que todas las variantes de los productos sigan esas interfaces. Por
ejemplo, todas las variantes de silla pueden implementar la interfaz
`Silla`, así como todas las variantes de mesilla pueden implementar la
interfaz `Mesilla`, y así sucesivamente.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="La jerarquía de la clase Silla" />
<figcaption><p>Todas las variantes del mismo objeto deben moverse a una
única jerarquía de clase.</p></figcaption>
</figure>

El siguiente paso consiste en declarar la *Fábrica abstracta*: una
interfaz con una lista de métodos de creación para todos los productos
que son parte de la familia de productos (por ejemplo, `crearSilla`,
`crearSofá` y `crearMesilla`). Estos métodos deben devolver productos
**abstractos** representados por las interfaces que extrajimos
previamente: `Silla`, `Sofá`, `Mesilla`, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La jerarquía de clases de _Fábrica_" />
<figcaption><p>Cada fábrica concreta se corresponde con una variante
específica del producto.</p></figcaption>
</figure>

Ahora bien, ¿qué hay de las variantes de los productos? Para cada
variante de una familia de productos, creamos una clase de fábrica
independiente basada en la interfaz `FábricaAbstracta`. Una fábrica es
una clase que devuelve productos de un tipo particular. Por ejemplo, la
`FábricadeMueblesModernos` sólo puede crear objetos de `SillaModerna`,
`SofáModerno` y `MesillaModerna`.

El código cliente tiene que funcionar con fábricas y productos a través
de sus respectivas interfaces abstractas. Esto nos permite cambiar el
tipo de fábrica que pasamos al código cliente, así como la variante del
producto que recibe el código cliente, sin descomponer el propio código
cliente.

<figure class="image">
<img
src="../../images/patterns/content/abstract-factory/abstract-factory-comic-2-es51ca.png?id=595e37e3c91e3c025fcc83ee0094f2c2"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-es.png?id=595e37e3c91e3c025fcc83ee0094f2c2 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-es-1.5x.png?id=cd872b7efea7e8d7c2982add1b6e408d 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-es-2x.png?id=2376a32bd3b2e3d66e7f4093eea4c3f7 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Al cliente no le debe importar la clase concreta de la
fábrica con la que funciona.</p></figcaption>
</figure>

Digamos que el cliente quiere una fábrica para producir una silla. El
cliente no tiene que conocer la clase de la fábrica y tampoco importa el
tipo de silla que obtiene. Ya sea un modelo moderno o una silla de
estilo victoriano, el cliente debe tratar a todas las sillas del mismo
modo, utilizando la interfaz abstracta `Silla`. Con este sistema, lo
único que sabe el cliente sobre la silla es que implementa de algún modo
el método `sentarse`. Además, sea cual sea la variante de silla
devuelta, siempre combinará con el tipo de sofá o mesilla producida por
el mismo objeto de fábrica.

Queda otro punto por aclarar: si el cliente sólo está expuesto a las
interfaces abstractas, ¿cómo se crean los objetos de fábrica?
Normalmente, la aplicación crea un objeto de fábrica concreto en la
etapa de inicialización. Justo antes, la aplicación debe seleccionar el
tipo de fábrica, dependiendo de la configuración o de los ajustes del
entorno.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Patrón de diseño Abstract Factory" /><img
src="../../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Patrón de diseño Abstract Factory" />
</figure>
:::

1.  Los **Productos Abstractos** declaran interfaces para un grupo de
    productos diferentes pero relacionados que forman una familia de
    productos.

2.  Los **Productos Concretos** son implementaciones distintas de
    productos abstractos agrupados por variantes. Cada producto
    abstracto (silla/sofá) debe implementarse en todas las variantes
    dadas (victoriano/moderno).

3.  La interfaz **Fábrica Abstracta** declara un grupo de métodos para
    crear cada uno de los productos abstractos.

4.  Las **Fábricas Concretas** implementan métodos de creación de la
    fábrica abstracta. Cada fábrica concreta se corresponde con una
    variante específica de los productos y crea tan solo dichas
    variantes de los productos.

5.  Aunque las fábricas concretas instancian productos concretos, las
    firmas de sus métodos de creación deben devolver los productos
    *abstractos* correspondientes. De este modo, el código cliente que
    utiliza una fábrica no se acopla a la variante específica del
    producto que obtiene de una fábrica. El **Cliente** puede funcionar
    con cualquier variante fábrica/producto concreta, siempre y cuando
    se comunique con sus objetos a través de interfaces abstractas.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo ilustra cómo puede utilizarse el patrón **Abstract
Factory** para crear elementos de interfaz de usuario (UI)
multiplataforma sin acoplar el código cliente a clases UI concretas,
mientras se mantiene la consistencia de todos los elementos creados
respecto al sistema operativo seleccionado.

<figure class="image">
<img
src="../../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Ejemplo del diagrama de clases del patrón Abstract Factory" />
<figcaption><p>Ejemplo de clases UI multiplataforma.</p></figcaption>
</figure>

Es de esperar que los mismos elementos UI de una aplicación
multiplataforma se comporten de forma parecida, aunque tengan un aspecto
un poco diferente en distintos sistemas operativos. Además, es nuestro
trabajo que los elementos UI coincidan con el estilo del sistema
operativo en cuestión. No queremos que nuestro programa represente
controles de macOS al ejecutarse en Windows.

La interfaz fábrica abstracta declara un grupo de métodos de creación
que el código cliente puede utilizar para producir distintos tipos de
elementos UI. Las fábricas concretas coinciden con sistemas operativos
específicos y crean los elementos UI correspondientes.

Funciona así: cuando se lanza, la aplicación comprueba el tipo de
sistema operativo actual. La aplicación utiliza esta información para
crear un objeto de fábrica a partir de una clase que coincida con el
sistema operativo. El resto del código utiliza esta fábrica para crear
elementos UI. Esto evita que se creen elementos equivocados.

Con este sistema, el código cliente no depende de clases concretas de
fábricas y elementos UI, siempre y cuando trabaje con estos objetos a
través de sus interfaces abstractas. Esto también permite que el código
cliente soporte otras fábricas o elementos UI que pudiéramos añadir más
adelante.

Como consecuencia, no necesitas modificar el código cliente cada vez que
añades una nueva variedad de elementos UI a tu aplicación. Tan solo
debes crear una nueva clase de fábrica que produzca estos elementos y
modifique ligeramente el código de inicialización de la aplicación, de
modo que seleccione esa clase cuando resulte apropiado.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz fábrica abstracta declara un grupo de métodos que
// devuelven distintos productos abstractos. Estos productos se
// denominan familia y están relacionados por un tema o concepto
// de alto nivel. Normalmente, los productos de una familia
// pueden colaborar entre sí. Una familia de productos puede
// tener muchas variantes, pero los productos de una variante
// son incompatibles con los productos de otra.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// Las fábricas concretas producen una familia de productos que
// pertenecen a una única variante. La fábrica garantiza que los
// productos resultantes sean compatibles. Las firmas de los
// métodos de las fábricas concretas devuelven un producto
// abstracto mientras que dentro del método se instancia un
// producto concreto.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Cada fábrica concreta tiene una variante de producto
// correspondiente.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Cada producto individual de una familia de productos debe
// tener una interfaz base. Todas las variantes del producto
// deben implementar esta interfaz.
interface Button is
    method paint()

// Los productos concretos son creados por las fábricas
// concretas correspondientes.
class WinButton implements Button is
    method paint() is
        // Representa un botón en estilo Windows.

class MacButton implements Button is
    method paint() is
        // Representa un botón en estilo macOS.

// Aquí está la interfaz base de otro producto. Todos los
// productos pueden interactuar entre sí, pero sólo entre
// productos de la misma variante concreta es posible una
// interacción adecuada.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Representa una casilla en estilo Windows.

class MacCheckbox implements Checkbox is
    method paint() is
        // Representa una casilla en estilo macOS.


// El código cliente funciona con fábricas y productos
// únicamente a través de tipos abstractos: GUIFactory, Button y
// Checkbox. Esto te permite pasar cualquier subclase fábrica o
// producto al código cliente sin descomponerlo.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// La aplicación elige el tipo de fábrica dependiendo de la
// configuración actual o de los ajustes del entorno y la crea
// durante el tiempo de ejecución (normalmente en la etapa de
// inicialización).
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Abstract Factory cuando tu código deba funcionar con
varias familias de productos relacionados, pero no desees que dependa de
las clases concretas de esos productos, ya que puede ser que no los
conozcas de antemano o sencillamente quieras permitir una futura
extensibilidad.
:::

::: applicability-solution
El patrón Abstract Factory nos ofrece una interfaz para crear objetos a
partir de cada clase de la familia de productos. Mientras tu código cree
objetos a través de esta interfaz, no tendrás que preocuparte por crear
la variante errónea de un producto que no combine con los productos que
ya ha creado tu aplicación.
:::

::: applicability-problem
Considera la implementación del patrón Abstract Factory cuando tengas
una clase con un grupo de [métodos de fábrica](factory-method.html) que
nublen su responsabilidad principal.
:::

::: applicability-solution
En un programa bien diseñado *cada clase es responsable tan solo de una
cosa*. Cuando una clase lidia con varios tipos de productos, puede ser
que valga la pena extraer sus métodos de fábrica para ponerlos en una
clase única de fábrica o una implementación completa del patrón Abstract
Factory.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Mapea una matriz de distintos tipos de productos frente a variantes
    de dichos productos.

2.  Declara interfaces abstractas de producto para todos los tipos de
    productos. Después haz que todas las clases concretas de productos
    implementen esas interfaces.

3.  Declara la interfaz de la fábrica abstracta con un grupo de métodos
    de creación para todos los productos abstractos.

4.  Implementa un grupo de clases concretas de fábrica, una por cada
    variante de producto.

5.  Crea un código de inicialización de la fábrica en algún punto de la
    aplicación. Deberá instanciar una de las clases concretas de la
    fábrica, dependiendo de la configuración de la aplicación o del
    entorno actual. Pasa este objeto de fábrica a todas las clases que
    construyen productos.

6.  Explora el código y encuentra todas las llamadas directas a
    constructores de producto. Sustitúyelas por llamadas al método de
    creación adecuado dentro del objeto de fábrica.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes tener la certeza de que los productos que obtienes de una
    fábrica son compatibles entre sí.
-    Evitas un acoplamiento fuerte entre productos concretos y el código
    cliente.
-    *Principio de responsabilidad única*. Puedes mover el código de
    creación de productos a un solo lugar, haciendo que el código sea
    más fácil de mantener.
-    *Principio de abierto/cerrado*. Puedes introducir nuevas variantes
    de productos sin descomponer el código cliente existente.
:::

::: col-sm-6
-    Puede ser que el código se complique más de lo que debería, ya que
    se introducen muchas nuevas interfaces y clases junto al patrón.
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

-   Las clases del [Abstract Factory](abstract-factory.html) a menudo se
    basan en un grupo de [métodos de fábrica](factory-method.html), pero
    también puedes utilizar [Prototype](prototype.html) para escribir
    los métodos de estas clases.

-   [Abstract Factory](abstract-factory.html) puede servir como
    alternativa a [Facade](facade.html) cuando tan solo deseas esconder
    la forma en que se crean los objetos del subsistema a partir del
    código cliente.

-   Puedes utilizar [Abstract Factory](abstract-factory.html) junto a
    [Bridge](bridge.html). Este emparejamiento resulta útil cuando
    algunas abstracciones definidas por *Bridge* sólo pueden funcionar
    con implementaciones específicas. En este caso, *Abstract Factory*
    puede encapsular estas relaciones y esconder la complejidad al
    código cliente.

-   Los patrones [Abstract Factory](abstract-factory.html),
    [Builder](builder.html) y [Prototype](prototype.html) pueden todos
    ellos implementarse como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Abstract Factory en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Abstract Factory en C#"){.prog-lang-link}
[![Abstract Factory en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Abstract Factory en C++"){.prog-lang-link}
[![Abstract Factory en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Abstract Factory en Go"){.prog-lang-link}
[![Abstract Factory en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Abstract Factory en Java"){.prog-lang-link}
[![Abstract Factory en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Abstract Factory en PHP"){.prog-lang-link}
[![Abstract Factory en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Abstract Factory en Python"){.prog-lang-link}
[![Abstract Factory en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Abstract Factory en Ruby"){.prog-lang-link}
[![Abstract Factory en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Abstract Factory en Rust"){.prog-lang-link}
[![Abstract Factory en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Abstract Factory en Swift"){.prog-lang-link}
[![Abstract Factory en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Abstract Factory en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Contenido adicional {#extras}

-   Lee nuestra [Comparación de fábricas](factory-comparison.html) para
    saber más acerca de las diferencias entre los distintos patrones y
    conceptos de fábrica.
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

[Comparación de fábricas []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Factory Method](factory-method.html){.btn
.btn-default rel="prev"}
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
