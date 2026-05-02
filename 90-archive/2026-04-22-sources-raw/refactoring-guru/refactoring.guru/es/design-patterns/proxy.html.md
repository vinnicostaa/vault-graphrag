:::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Proxy {#proxy .title}

::: {.section .intent}
##  Propósito {#intent}

**Proxy** es un patrón de diseño estructural que te permite proporcionar
un sustituto o marcador de posición para otro objeto. Un proxy controla
el acceso al objeto original, permitiéndote hacer algo antes o después
de que la solicitud llegue al objeto original.

<figure class="image">
<img
src="../../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Proxy" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

¿Por qué querrías controlar el acceso a un objeto? Imagina que tienes un
objeto enorme que consume una gran cantidad de recursos del sistema. Lo
necesitas de vez en cuando, pero no siempre.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/problem-es763f.png?id=4cedbeef6c6f4eb775ed945059053977"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-es.png?id=4cedbeef6c6f4eb775ed945059053977 510w,/images/patterns/diagrams/proxy/problem-es-1.5x.png?id=f971be7701f2e8cb3f929d3194dc5366 765w,/images/patterns/diagrams/proxy/problem-es-2x.png?id=68c95ce54805b94b831673444c0f3e23 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Problema resuelto por el patrón Proxy" />
<figcaption><p>Las consultas a las bases de datos pueden ser
muy lentas.</p></figcaption>
</figure>

Puedes llevar a cabo una implementación diferida, es decir, crear este
objeto sólo cuando sea realmente necesario. Todos los clientes del
objeto tendrán que ejecutar algún código de inicialización diferida.
Lamentablemente, esto seguramente generará una gran cantidad de código
duplicado.

En un mundo ideal, querríamos meter este código directamente dentro de
la clase de nuestro objeto, pero eso no siempre es posible. Por ejemplo,
la clase puede ser parte de una biblioteca cerrada de un tercero.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Proxy sugiere que crees una nueva clase proxy con la misma
interfaz que un objeto de servicio original. Después actualizas tu
aplicación para que pase el objeto proxy a todos los clientes del objeto
original. Al recibir una solicitud de un cliente, el proxy crea un
objeto de servicio real y le delega todo el trabajo.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/solution-ese1e6.png?id=5d7e37f3cd037c137e8dada58319a9bb"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-es.png?id=5d7e37f3cd037c137e8dada58319a9bb 510w,/images/patterns/diagrams/proxy/solution-es-1.5x.png?id=b8618a6c19b0c9772fe02968f9294d26 765w,/images/patterns/diagrams/proxy/solution-es-2x.png?id=6fb4b08230496c1eb957ecb83887279e 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Solución con el patrón Proxy" />
<figcaption><p>El proxy se camufla como objeto de la base de datos.
Puede gestionar la inicialización diferida y el caché de resultados sin
que el cliente o el objeto real de la base de datos
lo sepan.</p></figcaption>
</figure>

Pero, ¿cuál es la ventaja? Si necesitas ejecutar algo antes o después de
la lógica primaria de la clase, el proxy te permite hacerlo sin cambiar
esa clase. Ya que el proxy implementa la misma interfaz que la clase
original, puede pasarse a cualquier cliente que espere un objeto de
servicio real.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Una tarjeta de crédito es un proxy de un manojo de billetes" />
<figcaption><p>Las tarjetas de crédito pueden utilizarse para realizar
pagos tanto como el efectivo.</p></figcaption>
</figure>

Una tarjeta de crédito es un proxy de una cuenta bancaria, que, a su
vez, es un proxy de un manojo de billetes. Ambos implementan la misma
interfaz, por lo que pueden utilizarse para realizar un pago. El
consumidor se siente genial porque no necesita llevar un montón de
efectivo encima. El dueño de la tienda también está contento porque los
ingresos de la transacción se añaden electrónicamente a la cuenta
bancaria de la tienda sin el riesgo de perder el depósito o sufrir un
robo de camino al banco.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Estructura del patrón de diseño Proxy" /><img
src="../../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Estructura del patrón de diseño Proxy" />
</figure>
:::

1.  La **Interfaz de Servicio** declara la interfaz del Servicio. El
    proxy debe seguir esta interfaz para poder camuflarse como objeto de
    servicio.

2.  **Servicio** es una clase que proporciona una lógica de negocio
    útil.

3.  La clase **Proxy** tiene un campo de referencia que apunta a un
    objeto de servicio. Cuando el proxy finaliza su procesamiento (por
    ejemplo, inicialización diferida, registro, control de acceso,
    almacenamiento en caché, etc.), pasa la solicitud al objeto de
    servicio.

    Normalmente los proxies gestionan el ciclo de vida completo de sus
    objetos de servicio.

4.  El **Cliente** debe funcionar con servicios y proxies a través de la
    misma interfaz. De este modo puedes pasar un proxy a cualquier
    código que espere un objeto de servicio.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo ilustra cómo el patrón **Proxy** puede ayudar a introducir
la inicialización diferida y el almacenamiento en caché a una biblioteca
de integración de YouTube de un tercero.

<figure class="image">
<img
src="../../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Ejemplo de estructura del patrón Proxy" />
<figcaption><p>Resultados del almacenamiento en caché de un servicio con
un proxy.</p></figcaption>
</figure>

La biblioteca nos proporciona la clase de descarga de videos. Sin
embargo, es muy ineficiente. Si la aplicación cliente solicita el mismo
video muchas veces, la biblioteca lo descarga una y otra vez, en lugar
de guardarlo en caché y reutilizar el primer archivo descargado.

La clase proxy implementa la misma interfaz que el descargador original
y le delega todo el trabajo. No obstante, mantiene un seguimiento de los
archivos descargados y devuelve los resultados en caché cuando la
aplicación solicita el mismo video varias veces.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz de un servicio remoto.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// La implementación concreta de un conector de servicio. Los
// métodos de esta clase pueden solicitar información a YouTube.
// La velocidad de la solicitud depende de la conexión a
// internet del usuario y de YouTube. La aplicación se
// ralentizará si se lanzan muchas solicitudes al mismo tiempo,
// incluso aunque todas soliciten la misma información.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Envía una solicitud API a YouTube.

    method getVideoInfo(id) is
        // Obtiene metadatos de algún video.

    method downloadVideo(id) is
        // Descarga un archivo de video de YouTube.

// Para ahorrar ancho de banda, podemos guardar en caché
// resultados de la solicitud durante algún tiempo, pero se
// puede colocar este código directamente dentro de la clase de
// servicio. Por ejemplo, puede haberse proporcionado como parte
// de la biblioteca de un tercero y/o definido como `final`. Por
// eso colocamos el código de almacenamiento en caché dentro de
// una nueva clase proxy que implementa la misma interfaz que la
// clase servicio. Delega al objeto de servicio únicamente
// cuando deben enviarse las solicitudes reales.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// La clase GUI, que solía trabajar directamente con un objeto
// de servicio, permanece sin cambios siempre y cuando trabaje
// con el objeto de servicio a través de una interfaz. Podemos
// pasar sin riesgo un objeto proxy en lugar de un objeto de
// servicio real, ya que ambos implementan la misma interfaz.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Representa la página del video.

    method renderListPanel() is
        list = service.listVideos()
        // Representa la lista de miniaturas de los videos.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// La aplicación puede configurar proxies sobre la marcha.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::::::::: applicability
Hay decenas de formas de utilizar el patrón Proxy. Repasemos los usos
más populares.

::: applicability-problem
Inicialización diferida (proxy virtual). Es cuando tienes un objeto de
servicio muy pesado que utiliza muchos recursos del sistema al estar
siempre funcionando, aunque solo lo necesites de vez en cuando.
:::

::: applicability-solution
En lugar de crear el objeto cuando se lanza la aplicación, puedes
retrasar la inicialización del objeto a un momento en que sea realmente
necesario.
:::

::: applicability-problem
Control de acceso (proxy de protección). Es cuando quieres que
únicamente clientes específicos sean capaces de utilizar el objeto de
servicio, por ejemplo, cuando tus objetos son partes fundamentales de un
sistema operativo y los clientes son varias aplicaciones lanzadas
(incluyendo maliciosas).
:::

::: applicability-solution
El proxy puede pasar la solicitud al objeto de servicio tan sólo si las
credenciales del cliente cumplen ciertos criterios.
:::

::: applicability-problem
Ejecución local de un servicio remoto (proxy remoto). Es cuando el
objeto de servicio se ubica en un servidor remoto.
:::

::: applicability-solution
En este caso, el proxy pasa la solicitud del cliente por la red,
gestionando todos los detalles desagradables de trabajar con la red.
:::

::: applicability-problem
Solicitudes de registro (proxy de registro). Es cuando quieres mantener
un historial de solicitudes al objeto de servicio.
:::

::: applicability-solution
El proxy puede registrar cada solicitud antes de pasarla al servicio.
:::

::: applicability-problem
Resultados de solicitudes en caché (proxy de caché). Es cuando necesitas
guardar en caché resultados de solicitudes de clientes y gestionar el
ciclo de vida de ese caché, especialmente si los resultados son muchos.
:::

::: applicability-solution
El proxy puede implementar el caché para solicitudes recurrentes que
siempre dan los mismos resultados. El proxy puede utilizar los
parámetros de las solicitudes como claves de caché.
:::

::: applicability-problem
Referencia inteligente. Es cuando debes ser capaz de desechar un objeto
pesado una vez que no haya clientes que lo utilicen.
:::

::: applicability-solution
El proxy puede rastrear los clientes que obtuvieron una referencia del
objeto de servicio o sus resultados. De vez en cuando, el proxy puede
recorrer los clientes y comprobar si siguen activos. Si la lista del
cliente se vacía, el proxy puede desechar el objeto de servicio y
liberar los recursos subyacentes del sistema.

El proxy también puede rastrear si el cliente ha modificado el objeto de
servicio. Después, los objetos sin cambios pueden ser reutilizados por
otros clientes.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Si no hay una interfaz de servicio preexistente, crea una para que
    los objetos de proxy y de servicio sean intercambiables. No siempre
    resulta posible extraer la interfaz de la clase servicio, porque
    tienes que cambiar todos los clientes del servicio para utilizar esa
    interfaz. El plan B consiste en convertir el proxy en una subclase
    de la clase servicio, de forma que herede la interfaz del servicio.

2.  Crea la clase proxy. Debe tener un campo para almacenar una
    referencia al servicio. Normalmente los proxies crean y gestionan el
    ciclo de vida completo de sus servicios. En raras ocasiones, el
    cliente pasa un servicio al proxy a través de un constructor.

3.  Implementa los métodos del proxy según sus propósitos. En la mayoría
    de los casos, después de hacer cierta labor, el proxy debería
    delegar el trabajo a un objeto de servicio.

4.  Considera introducir un método de creación que decida si el cliente
    obtiene un proxy o un servicio real. Puede tratarse de un simple
    método estático en la clase proxy o de todo un método de fábrica.

5.  Considera implementar la inicialización diferida para el objeto de
    servicio.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes controlar el objeto de servicio sin que los clientes lo
    sepan.
-    Puedes gestionar el ciclo de vida del objeto de servicio cuando a
    los clientes no les importa.
-    El proxy funciona incluso si el objeto de servicio no está listo o
    no está disponible.
-    *Principio de abierto/cerrado*. Puedes introducir nuevos proxies
    sin cambiar el servicio o los clientes.
:::

::: col-sm-6
-    El código puede complicarse ya que debes introducir gran cantidad
    de clases nuevas.
-    La respuesta del servicio puede retrasarse.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Con [Adapter](adapter.html) se accede a un objeto existente a través
    de una interfaz diferente. Con [Proxy](proxy.html), la interfaz
    sigue siendo la misma. Con [Decorator](decorator.html) se accede al
    objeto a través de una interfaz mejorada.

-   [Facade](facade.html) es similar a [Proxy](proxy.html) en el sentido
    de que ambos pueden almacenar temporalmente una entidad compleja e
    inicializarla por su cuenta. Al contrario que *Facade*, *Proxy*
    tiene la misma interfaz que su objeto de servicio, lo que hace que
    sean intercambiables.

-   [Decorator](decorator.html) y [Proxy](proxy.html) tienen estructuras
    similares, pero propósitos muy distintos. Ambos patrones se basan en
    el principio de composición, por el que un objeto debe delegar parte
    del trabajo a otro. La diferencia es que, normalmente, un *Proxy*
    gestiona el ciclo de vida de su objeto de servicio por su cuenta,
    mientras que la composición de los *Decoradores* siempre está
    controlada por el cliente.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Proxy en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Proxy en C#"){.prog-lang-link}
[![Proxy en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Proxy en C++"){.prog-lang-link}
[![Proxy en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Proxy en Go"){.prog-lang-link}
[![Proxy en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Proxy en Java"){.prog-lang-link}
[![Proxy en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Proxy en PHP"){.prog-lang-link}
[![Proxy en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Proxy en Python"){.prog-lang-link}
[![Proxy en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Proxy en Ruby"){.prog-lang-link}
[![Proxy en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Proxy en Rust"){.prog-lang-link}
[![Proxy en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Proxy en Swift"){.prog-lang-link}
[![Proxy en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Proxy en TypeScript"){.prog-lang-link}
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

[Patrones de comportamiento []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Flyweight](flyweight.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::
