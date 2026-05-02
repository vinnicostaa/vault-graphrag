::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Facade {#facade .title}

::: aka
También llamado: [Fachada]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Facade** es un patrón de diseño estructural que proporciona una
interfaz simplificada a una biblioteca, un framework o cualquier otro
grupo complejo de clases.

<figure class="image">
<img
src="../../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Facade" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que debes lograr que tu código trabaje con un amplio grupo de
objetos que pertenecen a una sofisticada biblioteca o *framework*.
Normalmente, debes inicializar todos esos objetos, llevar un registro de
las dependencias, ejecutar los métodos en el orden correcto y así
sucesivamente.

Como resultado, la lógica de negocio de tus clases se vería
estrechamente acoplada a los detalles de implementación de las clases de
terceros, haciéndola difícil de comprender y mantener.
:::

::: {.section .solution}
##  Solución {#solution}

Una fachada es una clase que proporciona una interfaz simple a un
subsistema complejo que contiene muchas partes móviles. Una fachada
puede proporcionar una funcionalidad limitada en comparación con
trabajar directamente con el subsistema. Sin embargo, tan solo incluye
las funciones realmente importantes para los clientes.

Tener una fachada resulta útil cuando tienes que integrar tu aplicación
con una biblioteca sofisticada con decenas de funciones, de la cual sólo
necesitas una pequeña parte.

Por ejemplo, una aplicación que sube breves vídeos divertidos de gatos a
las redes sociales, podría potencialmente utilizar una biblioteca de
conversión de vídeo profesional. Sin embargo, lo único que necesita en
realidad es una clase con el método simple
`codificar(nombreDelArchivo, formato)`. Una vez que crees dicha clase y
la conectes con la biblioteca de conversión de vídeo, tendrás tu primera
fachada.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/live-example-esc5e7.png?id=ff6a4ba4b2ee5f00e3a721aba0481a4a"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-es.png?id=ff6a4ba4b2ee5f00e3a721aba0481a4a 490w,/images/patterns/diagrams/facade/live-example-es-1.5x.png?id=8309e2d69891ee15b84e8256fe7fbc89 735w,/images/patterns/diagrams/facade/live-example-es-2x.png?id=d4aa87ddee92e7c4b570b8ece7c9c319 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Un ejemplo de recepción de un pedido por teléfono" />
<figcaption><p>Haciendo pedidos por teléfono.</p></figcaption>
</figure>

Cuando llamas a una tienda para hacer un pedido por teléfono, un
operador es tu fachada a todos los servicios y departamentos de la
tienda. El operador te proporciona una sencilla interfaz de voz al
sistema de pedidos, pasarelas de pago y varios servicios de entrega.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Estructura del patrón de diseño Facade" /><img
src="../../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Estructura del patrón de diseño Facade" />
</figure>
:::

1.  El patrón **Facade** proporciona un práctico acceso a una parte
    específica de la funcionalidad del subsistema. Sabe a dónde dirigir
    la petición del cliente y cómo operar todas las partes móviles.

2.  Puede crearse una clase **Fachada Adicional** para evitar contaminar
    una única fachada con funciones no relacionadas que podrían
    convertirla en otra estructura compleja. Las fachadas adicionales
    pueden utilizarse por clientes y por otras fachadas.

3.  El **Subsistema Complejo** consiste en decenas de objetos diversos.
    Para lograr que todos hagan algo significativo, debes profundizar en
    los detalles de implementación del subsistema, que pueden incluir
    inicializar objetos en el orden correcto y suministrarles datos en
    el formato adecuado.

    Las clases del subsistema no conocen la existencia de la fachada.
    Operan dentro del sistema y trabajan entre sí directamente.

4.  El **Cliente** utiliza la fachada en lugar de invocar directamente
    los objetos del subsistema.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Facade** simplifica la interacción con un
framework complejo de conversión de vídeo.

<figure class="image">
<img
src="../../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Ejemplo de la estructura del patrón Facade" />
<figcaption><p>Un ejemplo de aislamiento de múltiples dependencias
dentro de una única clase fachada.</p></figcaption>
</figure>

En lugar de hacer que tu código trabaje con decenas de las clases del
framework directamente, creas una clase fachada que encapsula esa
funcionalidad y la esconde del resto del código. Esta estructura también
te ayuda a minimizar el esfuerzo de actualizar a futuras versiones del
framework o de sustituirlo por otro. Lo único que tendrías que cambiar
en la aplicación es la implementación de los métodos de la fachada.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Estas son algunas de las clases de un framework de conversión
// de video de un tercero. No controlamos ese código, por lo que
// no podemos simplificarlo.

class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// Creamos una clase fachada para esconder la complejidad del
// framework tras una interfaz simple. Es una solución de
// equilibrio entre funcionalidad y simplicidad.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// Las clases Application no dependen de un millón de clases
// proporcionadas por el complejo framework. Además, si decides
// cambiar los frameworks, sólo tendrás de volver a escribir la
// clase fachada.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Facade cuando necesites una interfaz limitada pero
directa a un subsistema complejo.
:::

::: applicability-solution
A menudo los subsistemas se vuelven más complejos con el tiempo. Incluso
la aplicación de patrones de diseño suele conducir a la creación de un
mayor número de clases. Un subsistema puede hacerse más flexible y más
fácil de reutilizar en varios contextos, pero la cantidad de código de
configuración que exige de un cliente, crece aún más. El patrón Facade
intenta solucionar este problema proporcionando un atajo a las funciones
más utilizadas del subsistema que mejor encajan con los requisitos del
cliente.
:::

::: applicability-problem
Utiliza el patrón Facade cuando quieras estructurar un subsistema en
capas.
:::

::: applicability-solution
Crea fachadas para definir puntos de entrada a cada nivel de un
subsistema. Puedes reducir el acoplamiento entre varios subsistemas
exigiéndoles que se comuniquen únicamente mediante fachadas.

Por ejemplo, regresemos a nuestro framework de conversión de vídeo.
Puede dividirse en dos capas: la relacionada con el vídeo y la
relacionada con el audio. Puedes crear una fachada para cada capa y
hacer que las clases de cada una de ellas se comuniquen entre sí a
través de esas fachadas. Este procedimiento es bastante similar al
patrón [Mediator](mediator.html).
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Comprueba si es posible proporcionar una interfaz más simple que la
    que está proporcionando un subsistema existente. Estás bien
    encaminado si esta interfaz hace que el código cliente sea
    independiente de muchas de las clases del subsistema.

2.  Declara e implementa esta interfaz en una nueva clase fachada. La
    fachada deberá redireccionar las llamadas desde el código cliente a
    los objetos adecuados del subsistema. La fachada deberá ser
    responsable de inicializar el subsistema y gestionar su ciclo de
    vida, a no ser que el código cliente ya lo haga.

3.  Para aprovechar el patrón al máximo, haz que todo el código cliente
    se comunique con el subsistema únicamente a través de la fachada.
    Ahora el código cliente está protegido de cualquier cambio en el
    código del subsistema. Por ejemplo, cuando se actualice un
    subsistema a una nueva versión, sólo tendrás que modificar el código
    de la fachada.

4.  Si la fachada se vuelve [demasiado
    grande](../smells/large-class.html), piensa en extraer parte de su
    comportamiento y colocarlo dentro de una nueva clase fachada
    refinada.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes aislar tu código de la complejidad de un subsistema.
:::

::: col-sm-6
-    Una fachada puede convertirse en [un objeto
    todopoderoso](https://es.wikipedia.org/wiki/Objeto_todopoderoso)
    acoplado a todas las clases de una aplicación.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Facade](facade.html) define una nueva interfaz para objetos
    existentes, mientras que [Adapter](adapter.html) intenta hacer que
    la interfaz existente sea utilizable. Normalmente *Adapter* sólo
    envuelve un objeto, mientras que *Facade* trabaja con todo un
    subsistema de objetos.

-   [Abstract Factory](abstract-factory.html) puede servir como
    alternativa a [Facade](facade.html) cuando tan solo deseas esconder
    la forma en que se crean los objetos del subsistema a partir del
    código cliente.

-   [Flyweight](flyweight.html) muestra cómo crear muchos pequeños
    objetos, mientras que [Facade](facade.html) muestra cómo crear un
    único objeto que represente un subsistema completo.

-   [Facade](facade.html) y [Mediator](mediator.html) tienen trabajos
    similares: ambos intentan organizar la colaboración entre muchas
    clases estrechamente acopladas.

    -   *Facade* define una interfaz simplificada a un subsistema de
        objetos, pero no introduce ninguna nueva funcionalidad. El
        propio subsistema no conoce la fachada. Los objetos del
        subsistema pueden comunicarse directamente.
    -   *Mediator* centraliza la comunicación entre componentes del
        sistema. Los componentes conocen únicamente el objeto mediador y
        no se comunican directamente.

-   Una clase [fachada](facade.html) a menudo puede transformarse en una
    [Singleton](singleton.html), ya que un único objeto fachada es
    suficiente en la mayoría de los casos.

-   [Facade](facade.html) es similar a [Proxy](proxy.html) en el sentido
    de que ambos pueden almacenar temporalmente una entidad compleja e
    inicializarla por su cuenta. Al contrario que *Facade*, *Proxy*
    tiene la misma interfaz que su objeto de servicio, lo que hace que
    sean intercambiables.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Facade en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Facade en C#"){.prog-lang-link}
[![Facade en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Facade en C++"){.prog-lang-link}
[![Facade en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Facade en Go"){.prog-lang-link}
[![Facade en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Facade en Java"){.prog-lang-link}
[![Facade en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Facade en PHP"){.prog-lang-link}
[![Facade en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Facade en Python"){.prog-lang-link}
[![Facade en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Facade en Ruby"){.prog-lang-link}
[![Facade en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Facade en Rust"){.prog-lang-link}
[![Facade en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Facade en Swift"){.prog-lang-link}
[![Facade en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Facade en TypeScript"){.prog-lang-link}
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

[Flyweight []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Decorator](decorator.html){.btn .btn-default
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
