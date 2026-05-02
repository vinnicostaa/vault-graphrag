:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Bridge {#bridge .title}

::: aka
También llamado: [Puente]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Bridge** es un patrón de diseño estructural que te permite dividir una
clase grande, o un grupo de clases estrechamente relacionadas, en dos
jerarquías separadas (abstracción e implementación) que pueden
desarrollarse independientemente la una de la otra.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Bridge" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

¿*Abstracción?* ¿*Implementación*? ¿Asusta? Mantengamos la calma y
veamos un ejemplo sencillo.

Digamos que tienes una clase geométrica `Forma` con un par de subclases:
`Círculo` y `Cuadrado`. Deseas extender esta jerarquía de clase para que
incorpore colores, por lo que planeas crear las subclases de forma
`Rojo` y `Azul`. Sin embargo, como ya tienes dos subclases, tienes que
crear cuatro combinaciones de clase, como `CírculoAzul` y
`CuadradoRojo`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-es3026.png?id=15670d9922988494a1fa39591a205297"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-es.png?id=15670d9922988494a1fa39591a205297 480w,/images/patterns/diagrams/bridge/problem-es-1.5x.png?id=63c41453c278fb86c560e2c0c319e3e4 720w,/images/patterns/diagrams/bridge/problem-es-2x.png?id=8f3101d70657909039a68d5693bd56a8 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Problema del patrón Bridge" />
<figcaption><p>El número de combinaciones de clase crece en
progresión geométrica.</p></figcaption>
</figure>

Añadir nuevos tipos de forma y color a la jerarquía hará que ésta crezca
exponencialmente. Por ejemplo, para añadir una forma de triángulo
deberás introducir dos subclases, una para cada color. Y, después, para
añadir un nuevo color habrá que crear tres subclases, una para cada tipo
de forma. Cuanto más avancemos, peor será.
:::

::: {.section .solution}
##  Solución {#solution}

Este problema se presenta porque intentamos extender las clases de forma
en dos dimensiones independientes: por forma y por color. Es un problema
muy habitual en la herencia de clases.

El patrón Bridge intenta resolver este problema pasando de la herencia a
la composición del objeto. Esto quiere decir que se extrae una de las
dimensiones a una jerarquía de clases separada, de modo que las clases
originales referencian un objeto de la nueva jerarquía, en lugar de
tener todo su estado y sus funcionalidades dentro de una clase.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-esd2b6.png?id=ea5b0ca5eb04228838c3079bd7d877d6"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-es.png?id=ea5b0ca5eb04228838c3079bd7d877d6 460w,/images/patterns/diagrams/bridge/solution-es-1.5x.png?id=221d3ec9412a5c4d73f26b0032c87141 690w,/images/patterns/diagrams/bridge/solution-es-2x.png?id=44e038fd90cd45eb6b9474d3798ecfd0 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Solución sugerida por el patrón Bridge" />
<figcaption><p>Puedes evitar la explosión de una jerarquía de clase
transformándola en varias jerarquías relacionadas.</p></figcaption>
</figure>

Con esta solución, podemos extraer el código relacionado con el color y
colocarlo dentro de su propia clase, con dos subclases: `Rojo` y `Azul`.
La clase `Forma` obtiene entonces un campo de referencia que apunta a
uno de los objetos de color. Ahora la forma puede delegar cualquier
trabajo relacionado con el color al objeto de color vinculado. Esa
referencia actuará como un puente entre las clases `Forma` y `Color`. En
adelante, añadir nuevos colores no exigirá cambiar la jerarquía de forma
y viceversa.

#### Abstracción e implementación

El libro de la GoF []{.pop-i}["Gang of Four" (banda de los cuatro) es el
sobrenombre de los cuatro autores del primer libro sobre patrones de
diseño: *Patrones de diseño*
[https://refactoring.guru/es/gof-book](https://www.amazon.com/-/dp/8478290591).]{.pop-content}
introduce los términos *Abstracción* e *Implementación* como parte de la
definición del patrón Bridge. En mi opinión, los términos suenan
demasiado académicos y provocan que el patrón parezca más complicado de
lo que es en realidad. Una vez leído el sencillo ejemplo con las formas
y los colores, vamos a descifrar el significado que esconden las
temibles palabras del libro de esta banda de cuatro.

La *Abstracción* (también llamada *interfaz*) es una capa de control de
alto nivel para una entidad. Esta capa no tiene que hacer ningún trabajo
real por su cuenta, sino que debe delegar el trabajo a la capa de
*implementación* (también llamada *plataforma*).

Ten en cuenta que no estamos hablando de las *interfaces* o las *clases
abstractas* de tu lenguaje de programación. Son cosas diferentes.

Cuando hablamos de aplicación reales, la abstracción puede representarse
por una interfaz gráfica de usuario (GUI), y la implementación puede ser
el código del sistema operativo subyacente (API) a la que la capa GUI
llama en respuesta a las interacciones del usuario.

En términos generales, puedes extender esa aplicación en dos direcciones
independientes:

-   Tener varias GUI diferentes (por ejemplo, personalizadas para
    clientes regulares o administradores).
-   Soportar varias API diferentes (por ejemplo, para poder lanzar la
    aplicación con Windows, Linux y macOS).

En el peor de los casos, esta aplicación podría asemejarse a un plato
gigante de espagueti, en el que cientos de condicionales conectan
distintos tipos de GUI con varias API por todo el código.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-esc347.png?id=adab9e8324ca6f2db4a3af44616d0af3"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-es.png?id=adab9e8324ca6f2db4a3af44616d0af3 600w,/images/patterns/content/bridge/bridge-3-es-1.5x.png?id=0b3b79d6ebc0e2f273988da22a361301 900w,/images/patterns/content/bridge/bridge-3-es-2x.png?id=51c0396446b451fd772a56421d7c071d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Es mucho más sencillo gestionar cambios en código modular" />
<figcaption><p>Realizar incluso un cambio sencillo en una base de código
monolítica es bastante difícil porque debes comprender <em>todo el
asunto</em> muy bien. Es mucho más sencillo realizar cambios en módulos
más pequeños y bien definidos.</p></figcaption>
</figure>

Puedes poner orden en este caos metiendo el código relacionado con
combinaciones específicas interfaz-plataforma dentro de clases
independientes. Sin embargo, pronto descubrirás que hay *muchas* de
estas clases. La jerarquía de clase crecerá exponencialmente porque
añadir una nueva GUI o soportar una API diferente exigirá que se creen
más y más clases.

Intentemos resolver este problema con el patrón Bridge, que nos sugiere
que dividamos las clases en dos jerarquías:

-   Abstracción: la capa GUI de la aplicación.
-   Implementación: las API de los sistemas operativos.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-es6fc7.png?id=aa4df8805e6cae965df234d7dcd3d477"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-es.png?id=aa4df8805e6cae965df234d7dcd3d477 640w,/images/patterns/content/bridge/bridge-2-es-1.5x.png?id=2fd1ca41674135b87964960a84acecf4 960w,/images/patterns/content/bridge/bridge-2-es-2x.png?id=549b6e0899b90e12dd20ecf764af1e91 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Arquitectura multiplataforma" />
<figcaption><p>Una de las formas de estructurar una
aplicación multiplataforma.</p></figcaption>
</figure>

El objeto de la abstracción controla la apariencia de la aplicación,
delegando el trabajo real al objeto de la implementación vinculado. Las
distintas implementaciones son intercambiables siempre y cuando sigan
una interfaz común, permitiendo a la misma GUI funcionar con Windows y
Linux.

En consecuencia, puedes cambiar las clases de la GUI sin tocar las
clases relacionadas con la API. Además, añadir soporte para otro sistema
operativo sólo requiere crear una subclase en la jerarquía de
implementación.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-es2121.png?id=864aabf2d8a5e95525968e5a1bc49f15"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-es.png?id=864aabf2d8a5e95525968e5a1bc49f15 560w,/images/patterns/diagrams/bridge/structure-es-1.5x.png?id=f276a6477a99e77fb20b5720ac78443d 840w,/images/patterns/diagrams/bridge/structure-es-2x.png?id=fc28523953d3d8bf0f9c807df2c0dd7d 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Patrón de diseño Bridge" /><img
src="../../images/patterns/diagrams/bridge/structure-es-indexedd922.png?id=53438bb41db049e24b4ce8fab5289b51"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-es-indexed.png?id=53438bb41db049e24b4ce8fab5289b51 560w,/images/patterns/diagrams/bridge/structure-es-indexed-1.5x.png?id=55ded4dfc91f6107fa0ae9fc7591ca75 840w,/images/patterns/diagrams/bridge/structure-es-indexed-2x.png?id=adcb081baa92ed45668a801c25833107 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Patrón de diseño Bridge" />
</figure>
:::

1.  La **Abstracción** ofrece lógica de control de alto nivel. Depende
    de que el objeto de la implementación haga el trabajo de bajo nivel.

2.  La **Implementación** declara la interfaz común a todas las
    implementaciones concretas. Una abstracción sólo se puede comunicar
    con un objeto de implementación a través de los métodos que se
    declaren aquí.

    La abstracción puede enumerar los mismos métodos que la
    implementación, pero normalmente la abstracción declara
    funcionalidades complejas que dependen de una amplia variedad de
    operaciones primitivas declaradas por la implementación.

3.  Las **Implementaciones Concretas** contienen código específico de
    plataforma.

4.  Las **Abstracciones Refinadas** proporcionan variantes de lógica de
    control. Como sus padres, trabajan con distintas implementaciones a
    través de la interfaz general de implementación.

5.  Normalmente, el **Cliente** sólo está interesado en trabajar con la
    abstracción. No obstante, el cliente tiene que vincular el objeto de
    la abstracción con uno de los objetos de la implementación.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo ilustra cómo puede ayudar el patrón **Bridge** a dividir el
código monolítico de una aplicación que gestiona dispositivos y sus
controles remotos. Las clases `Dispositivo` actúan como implementación,
mientras que las clases `Remoto` actúan como abstracción.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-esab39.png?id=89c406a189c45885004d7fa094f616b1"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-es.png?id=89c406a189c45885004d7fa094f616b1 640w,/images/patterns/diagrams/bridge/example-es-1.5x.png?id=bb870a109edc52920a51b3fc9110e7f4 960w,/images/patterns/diagrams/bridge/example-es-2x.png?id=19bb8272b4082c5f47c4d047e6cb9967 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Ejemplo de estructura del patrón Bridge" />
<figcaption><p>La jerarquía de clase original se divide en dos partes:
dispositivos y controles remotos.</p></figcaption>
</figure>

La clase base de control remoto declara un campo de referencia que la
vincula con un objeto de dispositivo. Todos los controles remotos
funcionan con los dispositivos a través de la interfaz general de
dispositivos, que permite al mismo remoto soportar varios tipos de
dispositivos.

Puedes desarrollar las clases de control remoto independientemente de
las clases de dispositivo. Lo único necesario es crear una nueva
subclase de control remoto. Por ejemplo, puede ser que un control remoto
básico cuente tan solo con dos botones, pero puedes extenderlo
añadiéndole funciones, como una batería adicional o pantalla táctil.

El código cliente vincula el tipo deseado de control remoto con un
objeto específico de dispositivo a través del constructor del control
remoto.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La &quot;abstracción&quot; define la interfaz para la parte de
// &quot;control&quot; de las dos jerarquías de clase. Mantiene una
// referencia a un objeto de la jerarquía de &quot;implementación&quot; y
// delega todo el trabajo real a este objeto.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Puedes extender clases de la jerarquía de abstracción
// independientemente de las clases de dispositivo.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// La interfaz de &quot;implementación&quot; declara métodos comunes a
// todas las clases concretas de implementación. No tiene por
// qué coincidir con la interfaz de la abstracción. De hecho,
// las dos interfaces pueden ser completamente diferentes.
// Normalmente, la interfaz de implementación únicamente
// proporciona operaciones primitivas, mientras que la
// abstracción define operaciones de más alto nivel con base en
// las primitivas.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Todos los dispositivos siguen la misma interfaz.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// En algún lugar del código cliente.
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Bridge cuando quieras dividir y organizar una clase
monolítica que tenga muchas variantes de una sola funcionalidad (por
ejemplo, si la clase puede trabajar con diversos servidores de bases de
datos).
:::

::: applicability-solution
Conforme más crece una clase, más difícil resulta entender cómo funciona
y más tiempo se tarda en realizar un cambio. Cambiar una de las
variaciones de funcionalidad puede exigir realizar muchos cambios a toda
la clase, lo que a menudo provoca que se cometan errores o no se aborden
algunos de los efectos colaterales críticos.

El patrón Bridge te permite dividir la clase monolítica en varias
jerarquías de clase. Después, puedes cambiar las clases de cada
jerarquía independientemente de las clases de las otras. Esta solución
simplifica el mantenimiento del código y minimiza el riesgo de
descomponer el código existente.
:::

::: applicability-problem
Utiliza el patrón cuando necesites extender una clase en varias
dimensiones ortogonales (independientes).
:::

::: applicability-solution
El patrón Bridge sugiere que extraigas una jerarquía de clase separada
para cada una de las dimensiones. La clase original delega el trabajo
relacionado a los objetos pertenecientes a dichas jerarquías, en lugar
de hacerlo todo por su cuenta.
:::

::: applicability-problem
Utiliza el patrón Bridge cuando necesites poder cambiar implementaciones
durante el tiempo de ejecución.
:::

::: applicability-solution
Aunque es opcional, el patrón Bridge te permite sustituir el objeto de
implementación dentro de la abstracción. Es tan sencillo como asignar un
nuevo valor a un campo.

*Por cierto, este último punto es la razón principal por la que tanta
gente confunde el patrón Bridge con el patrón [Strategy](strategy.html).
Recuerda que un patrón es algo más que un cierto modo de estructurar tus
clases. También puede comunicar intención y el tipo de problema que se
está abordando.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Identifica las dimensiones ortogonales de tus clases. Estos
    conceptos independientes pueden ser: abstracción/plataforma,
    dominio/infraestructura, *front end*/*back end*, o
    interfaz/implementación.

2.  Comprueba qué operaciones necesita el cliente y defínelas en la
    clase base de abstracción.

3.  Determina las operaciones disponibles en todas las plataformas.
    Declara aquellas que necesite la abstracción en la interfaz general
    de implementación.

4.  Crea clases concretas de implementación para todas las plataformas
    de tu dominio, pero asegúrate de que todas sigan la interfaz de
    implementación.

5.  Dentro de la clase de abstracción añade un campo de referencia para
    el tipo de implementación. La abstracción delega la mayor parte del
    trabajo al objeto de la implementación referenciado en ese campo.

6.  Si tienes muchas variantes de lógica de alto nivel, crea
    abstracciones refinadas para cada variante extendiendo la clase base
    de abstracción.

7.  El código cliente debe pasar un objeto de implementación al
    constructor de la abstracción para asociar el uno con el otro.
    Después, el cliente puede ignorar la implementación y trabajar solo
    con el objeto de la abstracción.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes crear clases y aplicaciones independientes de plataforma.
-    El código cliente funciona con abstracciones de alto nivel. No está
    expuesto a los detalles de la plataforma.
-    *Principio de abierto/cerrado*. Puedes introducir nuevas
    abstracciones e implementaciones independientes entre sí.
-    *Principio de responsabilidad única*. Puedes centrarte en la lógica
    de alto nivel en la abstracción y en detalles de la plataforma en la
    implementación.
:::

::: col-sm-6
-    Puede ser que el código se complique si aplicas el patrón a una
    clase muy cohesionada.
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

-   [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (y, hasta cierto punto,
    [Adapter](adapter.html)) tienen estructuras muy similares. De hecho,
    todos estos patrones se basan en la composición, que consiste en
    delegar trabajo a otros objetos. Sin embargo, todos ellos solucionan
    problemas diferentes. Un patrón no es simplemente una receta para
    estructurar tu código de una forma específica. También puede
    comunicar a otros desarrolladores el problema que resuelve.

-   Puedes utilizar [Abstract Factory](abstract-factory.html) junto a
    [Bridge](bridge.html). Este emparejamiento resulta útil cuando
    algunas abstracciones definidas por *Bridge* sólo pueden funcionar
    con implementaciones específicas. En este caso, *Abstract Factory*
    puede encapsular estas relaciones y esconder la complejidad al
    código cliente.

-   Puedes combinar [Builder](builder.html) con [Bridge](bridge.html):
    la clase *directora* juega el papel de la abstracción, mientras que
    diferentes *constructoras* actúan como *implementaciones*.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Bridge en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Bridge en C#"){.prog-lang-link}
[![Bridge en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Bridge en C++"){.prog-lang-link}
[![Bridge en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Bridge en Go"){.prog-lang-link}
[![Bridge en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Bridge en Java"){.prog-lang-link}
[![Bridge en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Bridge en PHP"){.prog-lang-link}
[![Bridge en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Bridge en Python"){.prog-lang-link}
[![Bridge en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Bridge en Ruby"){.prog-lang-link}
[![Bridge en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Bridge en Rust"){.prog-lang-link}
[![Bridge en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Bridge en Swift"){.prog-lang-link}
[![Bridge en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Bridge en TypeScript"){.prog-lang-link}
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

[Composite []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Adapter](adapter.html){.btn .btn-default
rel="prev"}
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
