:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Adapter {#adapter .title}

::: aka
También llamado:
[Adaptador, ]{style="display:inline-block"}[Envoltorio, ]{style="display:inline-block"}[Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Adapter** es un patrón de diseño estructural que permite la
colaboración entre objetos con interfaces incompatibles.

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-es4055.png?id=5b877b3bdab93ef57848e3d6426064f1"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/adapter/adapter-es.png?id=5b877b3bdab93ef57848e3d6426064f1 640w,/images/patterns/content/adapter/adapter-es-1.5x.png?id=950f68cbd2962e37867388078378c294 960w,/images/patterns/content/adapter/adapter-es-2x.png?id=0627cb0c14b241c13833dc6d96b0df5c 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Adapter" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás creando una aplicación de monitoreo del mercado de
valores. La aplicación descarga la información de bolsa desde varias
fuentes en formato XML para presentarla al usuario con bonitos gráficos
y diagramas.

En cierto momento, decides mejorar la aplicación integrando una
inteligente biblioteca de análisis de una tercera persona. Pero hay una
trampa: la biblioteca de análisis solo funciona con datos en formato
JSON.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/problem-esd261.png?id=7a6a0991c2df691e77ba316d4652ed3c"
style="aspect-ratio: 2.41;"
srcset="/images/patterns/diagrams/adapter/problem-es.png?id=7a6a0991c2df691e77ba316d4652ed3c 530w,/images/patterns/diagrams/adapter/problem-es-1.5x.png?id=39f55f7903ff98083b95571f3eabcfe2 795w,/images/patterns/diagrams/adapter/problem-es-2x.png?id=0375841dd491633a0e02a67332ed4247 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="La estructura de la aplicación antes de la integración con la biblioteca de análisis" />
<figcaption><p>No puedes utilizar la biblioteca de análisis “tal cual”
porque ésta espera los datos en un formato que es incompatible con
tu aplicación.</p></figcaption>
</figure>

Podrías cambiar la biblioteca para que funcione con XML. Sin embargo,
esto podría descomponer parte del código existente que depende de la
biblioteca. Y, lo que es peor, podrías no tener siquiera acceso al
código fuente de la biblioteca, lo que hace imposible esta solución.
:::

::: {.section .solution}
##  Solución {#solution}

Puedes crear un *adaptador*. Se trata de un objeto especial que
convierte la interfaz de un objeto, de forma que otro objeto pueda
comprenderla.

Un adaptador envuelve uno de los objetos para esconder la complejidad de
la conversión que tiene lugar tras bambalinas. El objeto envuelto ni
siquiera es consciente de la existencia del adaptador. Por ejemplo,
puedes envolver un objeto que opera con metros y kilómetros con un
adaptador que convierte todos los datos al sistema anglosajón, es decir,
pies y millas.

Los adaptadores no solo convierten datos a varios formatos, sino que
también ayudan a objetos con distintas interfaces a colaborar. Funciona
así:

1.  El adaptador obtiene una interfaz compatible con uno de los objetos
    existentes.
2.  Utilizando esta interfaz, el objeto existente puede invocar con
    seguridad los métodos del adaptador.
3.  Al recibir una llamada, el adaptador pasa la solicitud al segundo
    objeto, pero en un formato y orden que ese segundo objeto espera.

En ocasiones se puede incluso crear un adaptador de dos direcciones que
pueda convertir las llamadas en ambos sentidos.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/solution-es131b.png?id=2b1c418b5b85e53a37ea4bdad18064d6"
style="aspect-ratio: 1.51;"
srcset="/images/patterns/diagrams/adapter/solution-es.png?id=2b1c418b5b85e53a37ea4bdad18064d6 530w,/images/patterns/diagrams/adapter/solution-es-1.5x.png?id=efb1bbcd0f9f98e544bec71135168538 795w,/images/patterns/diagrams/adapter/solution-es-2x.png?id=ee482c83d2aeab7ccd3158ffc3970f7f 1060w"
sizes="(max-width: 720px) 100vw, 530px" loading="lazy" width="530"
alt="Solución del patrón Adapter" />
</figure>

Regresemos a nuestra aplicación del mercado de valores. Para resolver el
dilema de los formatos incompatibles, puedes crear adaptadores de XML a
JSON para cada clase de la biblioteca de análisis con la que trabaje tu
código directamente. Después ajustas tu código para que se comunique con
la biblioteca únicamente a través de estos adaptadores. Cuando un
adaptador recibe una llamada, traduce los datos XML entrantes a una
estructura JSON y pasa la llamada a los métodos adecuados de un objeto
de análisis envuelto.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/adapter/adapter-comic-1-esf415.png?id=8e18327b405f5590443a97a5a6ae6877"
style="aspect-ratio: 1.78;"
srcset="/images/patterns/content/adapter/adapter-comic-1-es.png?id=8e18327b405f5590443a97a5a6ae6877 533w,/images/patterns/content/adapter/adapter-comic-1-es-1.5x.png?id=1ea0d97bbd26ace735bb07b2abc3c335 800w,/images/patterns/content/adapter/adapter-comic-1-es-2x.png?id=6c3831488916aa7b4ea632659be8a368 1067w"
sizes="(max-width: 720px) 100vw, 533px" loading="lazy" width="533"
alt="Ejemplo del patrón Adapter" />
<figcaption><p>Una maleta antes y después de un viaje
al extranjero.</p></figcaption>
</figure>

Cuando viajas de Europa a Estados Unidos por primera vez, puede ser que
te lleves una sorpresa cuanto intentes cargar tu computadora portátil.
Los tipos de enchufe son diferentes en cada país, por lo que un enchufe
español no sirve en Estados Unidos. El problema puede solucionarse
utilizando un adaptador que incluya el enchufe americano y el europeo.
:::

:::::::: {.section .structure-container}
##  Estructura {#structure}

::::::: structure
#### Adaptador de objetos

Esta implementación utiliza el principio de composición de objetos: el
adaptador implementa la interfaz de un objeto y envuelve el otro. Puede
implementarse en todos los lenguajes de programación populares.

:::: adapter-object
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-object-adaptera83b.png?id=33dffbe3aece294162440c7ddd3d5d4f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.81;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter.png?id=33dffbe3aece294162440c7ddd3d5d4f 580w,/images/patterns/diagrams/adapter/structure-object-adapter-1.5x.png?id=c1b8a87b74fc4ce5639212fe19ee06fe 870w,/images/patterns/diagrams/adapter/structure-object-adapter-2x.png?id=03e8052e168c962d6bc369bbb13b0945 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Estructura del patrón de diseño Adapter (el adaptador de objetos)" /><img
src="../../images/patterns/diagrams/adapter/structure-object-adapter-indexedd6ac.png?id=a20b311948b361a058097e5bcdbf067a"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.76;"
srcset="/images/patterns/diagrams/adapter/structure-object-adapter-indexed.png?id=a20b311948b361a058097e5bcdbf067a 600w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-1.5x.png?id=72a1c8fde064c4915379fb34a1a66881 900w,/images/patterns/diagrams/adapter/structure-object-adapter-indexed-2x.png?id=759771515f08d74d53cf4fe500f814a3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Estructura del patrón de diseño Adapter (el adaptador de objetos)" />
</figure>
:::
::::

1.  La clase **Cliente** contiene la lógica de negocio existente del
    programa.

2.  La **Interfaz con el Cliente** describe un protocolo que otras
    clases deben seguir para poder colaborar con el código cliente.

3.  **Servicio** es alguna clase útil (normalmente de una tercera parte
    o heredada). El cliente no puede utilizar directamente esta clase
    porque tiene una interfaz incompatible.

4.  La clase **Adaptadora** es capaz de trabajar tanto con la clase
    cliente como con la clase de servicio: implementa la interfaz con el
    cliente, mientras envuelve el objeto de la clase de servicio. La
    clase adaptadora recibe llamadas del cliente a través de la interfaz
    de cliente y las traduce en llamadas al objeto envuelto de la clase
    de servicio, pero en un formato que pueda comprender.

5.  El código cliente no se acopla a la clase adaptadora concreta
    siempre y cuando funcione con la clase adaptadora a través de la
    interfaz con el cliente. Gracias a esto, puedes introducir nuevos
    tipos de adaptadores en el programa sin descomponer el código
    cliente existente. Esto puede resultar útil cuando la interfaz de la
    clase de servicio se cambia o sustituye, ya que puedes crear una
    nueva clase adaptadora sin cambiar el código cliente.

#### Clase adaptadora

Esta implementación utiliza la herencia, porque la clase adaptadora
hereda interfaces de ambos objetos al mismo tiempo. Ten en cuenta que
esta opción sólo puede implementarse en lenguajes de programación que
soporten la herencia múltiple, como C++.

:::: adapter-class
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/structure-class-adapterf043.png?id=e1c60240508146ed3b98ac562cc8e510"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter.png?id=e1c60240508146ed3b98ac562cc8e510 550w,/images/patterns/diagrams/adapter/structure-class-adapter-1.5x.png?id=299a79bdfa10ac53213ba02408255430 825w,/images/patterns/diagrams/adapter/structure-class-adapter-2x.png?id=ddca3e3e4d972b7c58207daba8d24866 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Patrón de diseño Adapter (clase adaptadora)" /><img
src="../../images/patterns/diagrams/adapter/structure-class-adapter-indexedd30e.png?id=250b5c485a7dfba7c16b89a9201538fb"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.72;"
srcset="/images/patterns/diagrams/adapter/structure-class-adapter-indexed.png?id=250b5c485a7dfba7c16b89a9201538fb 550w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-1.5x.png?id=b56d736f8076d34b1896de0a2b22a219 825w,/images/patterns/diagrams/adapter/structure-class-adapter-indexed-2x.png?id=9ae1182ef2a34d2ea65f4b4f94a4019e 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Patrón de diseño Adapter (clase adaptadora)" />
</figure>
:::
::::

1.  La **Clase adaptadora** no necesita envolver objetos porque hereda
    comportamientos tanto de la clase cliente como de la clase de
    servicio. La adaptación tiene lugar dentro de los métodos
    sobrescritos. La clase adaptadora resultante puede utilizarse en
    lugar de una clase cliente existente.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo del patrón **Adapter** se basa en el clásico conflicto
entre piezas cuadradas y agujeros redondos.

<figure class="image">
<img
src="../../images/patterns/diagrams/adapter/examplef4c9.png?id=9d2b6857ce256f2c669383ce4df3d0aa"
style="aspect-ratio: 1.68;"
srcset="/images/patterns/diagrams/adapter/example.png?id=9d2b6857ce256f2c669383ce4df3d0aa 640w,/images/patterns/diagrams/adapter/example-1.5x.png?id=76e68b9cea3b371e465e81587e57e5d9 960w,/images/patterns/diagrams/adapter/example-2x.png?id=0ac62d1bc151e8ce6abad8e8502756cf 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Ejemplo de estructura del patrón Adapter" />
<figcaption><p>Adaptando piezas cuadradas a
agujeros redondos.</p></figcaption>
</figure>

El patrón Adapter finge ser una pieza redonda con un radio igual a la
mitad del diámetro del cuadrado (en otras palabras, el radio del círculo
más pequeño en el que quepa la pieza cuadrada).

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Digamos que tienes dos clases con interfaces compatibles:
// RoundHole (HoyoRedondo) y RoundPeg (PiezaRedonda).
class RoundHole is
    constructor RoundHole(radius) { ... }

    method getRadius() is
        // Devuelve el radio del agujero.

    method fits(peg: RoundPeg) is
        return this.getRadius() &gt;= peg.getRadius()

class RoundPeg is
    constructor RoundPeg(radius) { ... }

    method getRadius() is
        // Devuelve el radio de la pieza.


// Pero hay una clase incompatible: SquarePeg (PiezaCuadrada).
class SquarePeg is
    constructor SquarePeg(width) { ... }

    method getWidth() is
        // Devuelve la anchura de la pieza cuadrada.


// Una clase adaptadora te permite encajar piezas cuadradas en
// hoyos redondos. Extiende la clase RoundPeg para permitir a
// los objetos adaptadores actuar como piezas redondas.
class SquarePegAdapter extends RoundPeg is
    // En realidad, el adaptador contiene una instancia de la
    // clase SquarePeg.
    private field peg: SquarePeg

    constructor SquarePegAdapter(peg: SquarePeg) is
        this.peg = peg

    method getRadius() is
        // El adaptador simula que es una pieza redonda con un
        // radio que pueda albergar la pieza cuadrada que el
        // adaptador envuelve.
        return peg.getWidth() * Math.sqrt(2) / 2


// En algún punto del código cliente.
hole = new RoundHole(5)
rpeg = new RoundPeg(5)
hole.fits(rpeg) // verdadero

small_sqpeg = new SquarePeg(5)
large_sqpeg = new SquarePeg(10)
hole.fits(small_sqpeg) // esto no compila (tipos incompatibles)

small_sqpeg_adapter = new SquarePegAdapter(small_sqpeg)
large_sqpeg_adapter = new SquarePegAdapter(large_sqpeg)
hole.fits(small_sqpeg_adapter) // verdadero
hole.fits(large_sqpeg_adapter) // falso</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza la clase adaptadora cuando quieras usar una clase existente,
pero cuya interfaz no sea compatible con el resto del código.
:::

::: applicability-solution
El patrón Adapter te permite crear una clase intermedia que sirva como
traductora entre tu código y una clase heredada, una clase de un tercero
o cualquier otra clase con una interfaz extraña.
:::

::: applicability-problem
Utiliza el patrón cuando quieras reutilizar varias subclases existentes
que carezcan de alguna funcionalidad común que no pueda añadirse a la
superclase.
:::

::: applicability-solution
Puedes extender cada subclase y colocar la funcionalidad que falta,
dentro de las nuevas clases hijas. No obstante, deberás duplicar el
código en todas estas nuevas clases, lo cual [huele muy
mal](../smells/duplicate-code.html).

Una solución mucho más elegante sería colocar la funcionalidad que falta
dentro de una clase adaptadora. Después puedes envolver objetos a los
que les falten funciones, dentro de la clase adaptadora, obteniendo esas
funciones necesarias de un modo dinámico. Para que esto funcione, las
clases en cuestión deben tener una interfaz común y el campo de la clase
adaptadora debe seguir dicha interfaz. Este procedimiento es muy similar
al del patrón [Decorator](decorator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Asegúrate de que tienes al menos dos clases con interfaces
    incompatibles:

    -   Una útil clase *servicio* que no puedes cambiar (a menudo de un
        tercero, heredada o con muchas dependencias existentes).
    -   Una o varias clases *cliente* que se beneficiarían de contar con
        una clase de servicio.

2.  Declara la interfaz con el cliente y describe el modo en que las
    clases cliente se comunican con la clase de servicio.

3.  Crea la clase adaptadora y haz que siga la interfaz con el cliente.
    Deja todos los métodos vacíos por ahora.

4.  Añade un campo a la clase adaptadora para almacenar una referencia
    al objeto de servicio. La práctica común es inicializar este campo a
    través del constructor, pero en ocasiones es adecuado pasarlo al
    adaptador cuando se invocan sus métodos.

5.  Uno por uno, implementa todos los métodos de la interfaz con el
    cliente en la clase adaptadora. La clase adaptadora deberá delegar
    la mayor parte del trabajo real al objeto de servicio, gestionando
    tan solo la interfaz o la conversión de formato de los datos.

6.  Las clases cliente deberán utilizar la clase adaptadora a través de
    la interfaz con el cliente. Esto te permitirá cambiar o extender las
    clases adaptadoras sin afectar al código cliente.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de responsabilidad única*. Puedes separar la interfaz o
    el código de conversión de datos de la lógica de negocio primaria
    del programa.
-    *Principio de abierto/cerrado*. Puedes introducir nuevos tipos de
    adaptadores al programa sin descomponer el código cliente existente,
    siempre y cuando trabajen con los adaptadores a través de la
    interfaz con el cliente.
:::

::: col-sm-6
-    La complejidad general del código aumenta, ya que debes introducir
    un grupo de nuevas interfaces y clases. En ocasiones resulta más
    sencillo cambiar la clase de servicio de modo que coincida con el
    resto de tu código.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Bridge](bridge.html) suele diseñarse por anticipado, lo que te
    permite desarrollar partes de una aplicación de forma independiente
    entre sí. Por otro lado, [Adapter](adapter.html) se utiliza
    habitualmente con una aplicación existente para hacer que unas
    clases que de otro modo serían incompatibles, trabajen juntas sin
    problemas.

-   [Adapter](adapter.html) proporciona una interfaz completamente
    diferente para acceder a un objeto existente. Por otro lado, con el
    patrón [Decorator](decorator.html) la interfaz permanece igual o se
    amplía. Además, *Decorator* admite la composición recursiva, que no
    es posible cuando se utiliza *Adapter*.

-   Con [Adapter](adapter.html) se accede a un objeto existente a través
    de una interfaz diferente. Con [Proxy](proxy.html), la interfaz
    sigue siendo la misma. Con [Decorator](decorator.html) se accede al
    objeto a través de una interfaz mejorada.

-   [Facade](facade.html) define una nueva interfaz para objetos
    existentes, mientras que [Adapter](adapter.html) intenta hacer que
    la interfaz existente sea utilizable. Normalmente *Adapter* sólo
    envuelve un objeto, mientras que *Facade* trabaja con todo un
    subsistema de objetos.

-   [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (y, hasta cierto punto,
    [Adapter](adapter.html)) tienen estructuras muy similares. De hecho,
    todos estos patrones se basan en la composición, que consiste en
    delegar trabajo a otros objetos. Sin embargo, todos ellos solucionan
    problemas diferentes. Un patrón no es simplemente una receta para
    estructurar tu código de una forma específica. También puede
    comunicar a otros desarrolladores el problema que resuelve.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Adapter en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](adapter/csharp/example.html "Adapter en C#"){.prog-lang-link}
[![Adapter en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](adapter/cpp/example.html "Adapter en C++"){.prog-lang-link}
[![Adapter en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](adapter/go/example.html "Adapter en Go"){.prog-lang-link}
[![Adapter en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](adapter/java/example.html "Adapter en Java"){.prog-lang-link}
[![Adapter en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](adapter/php/example.html "Adapter en PHP"){.prog-lang-link}
[![Adapter en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](adapter/python/example.html "Adapter en Python"){.prog-lang-link}
[![Adapter en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](adapter/ruby/example.html "Adapter en Ruby"){.prog-lang-link}
[![Adapter en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](adapter/rust/example.html "Adapter en Rust"){.prog-lang-link}
[![Adapter en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](adapter/swift/example.html "Adapter en Swift"){.prog-lang-link}
[![Adapter en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](adapter/typescript/example.html "Adapter en TypeScript"){.prog-lang-link}
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

[Bridge []{.fa .fa-arrow-right}](bridge.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Patrones
estructurales](structural-patterns.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
