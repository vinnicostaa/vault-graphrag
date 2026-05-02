::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Iterator {#iterator .title}

::: aka
También llamado: [Iterador]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Iterator** es un patrón de diseño de comportamiento que te permite
recorrer elementos de una colección sin exponer su representación
subyacente (lista, pila, árbol, etc.).

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-es4d17.png?id=79d47b82a1e72adaaa70d8e1a3b10a4e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-es.png?id=79d47b82a1e72adaaa70d8e1a3b10a4e 640w,/images/patterns/content/iterator/iterator-es-1.5x.png?id=13aedcc4daa6843e6d4525b84064d8fa 960w,/images/patterns/content/iterator/iterator-es-2x.png?id=866e4ff07d4c47b09e834b5196da9226 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Iterator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Las colecciones son de los tipos de datos más utilizados en
programación. Sin embargo, una colección tan solo es un contenedor para
un grupo de objetos.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Varios tipos de colecciones" />
<figcaption><p>Varios tipos de colecciones.</p></figcaption>
</figure>

La mayoría de las colecciones almacena sus elementos en simples listas,
pero algunas de ellas se basan en pilas, árboles, grafos y otras
estructuras complejas de datos.

Independientemente de cómo se estructure una colección, debe aportar una
forma de acceder a sus elementos de modo que otro código pueda utilizar
dichos elementos. Debe haber una forma de recorrer cada elemento de la
colección sin acceder a los mismos elementos una y otra vez.

Esto puede parecer un trabajo sencillo si tienes una colección basada en
una lista. En este caso sólo tienes que recorrer en bucle todos sus
elementos. Pero, ¿cómo recorres secuencialmente elementos de una
estructura compleja de datos, como un árbol? Por ejemplo, un día puede
bastarte con un recorrido de profundidad de un árbol, pero, al día
siguiente, quizá necesites un recorrido en anchura. Y, la semana
siguiente, puedes necesitar otra cosa, como un acceso aleatorio a los
elementos del árbol.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Varios algoritmos de recorrido" />
<figcaption><p>La misma colección puede recorrerse de varias
formas diferentes.</p></figcaption>
</figure>

Añadir más y más algoritmos de recorrido a la colección nubla
gradualmente su responsabilidad principal, que es el almacenamiento
eficiente de la información. Además, puede que algunos algoritmos estén
personalizados para una aplicación específica, por lo que incluirlos en
una clase genérica de colección puede resultar extraño.

Por otro lado, el código cliente que debe funcionar con varias
colecciones puede no saber cómo éstas almacenan sus elementos. No
obstante, ya que todas las colecciones proporcionan formas diferentes de
acceder a sus elementos, no tienes otra opción más que acoplar tu código
a las clases de la colección específica.
:::

::: {.section .solution}
##  Solución {#solution}

La idea central del patrón Iterator es extraer el comportamiento de
recorrido de una colección y colocarlo en un objeto independiente
llamado *iterador*.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Los iteradores implementan varios algoritmos de recorrido" />
<figcaption><p>Los iteradores implementan varios algoritmos de
recorrido. Varios objetos iteradores pueden recorrer la misma colección
al mismo tiempo.</p></figcaption>
</figure>

Además de implementar el propio algoritmo, un objeto iterador encapsula
todos los detalles del recorrido, como la posición actual y cuántos
elementos quedan hasta el final. Debido a esto, varios iteradores pueden
recorrer la misma colección al mismo tiempo, independientemente los unos
de los otros.

Normalmente, los iteradores aportan un método principal para extraer
elementos de la colección. El cliente puede continuar ejecutando este
método hasta que no devuelva nada, lo que significa que el iterador ha
recorrido todos los elementos.

Todos los iteradores deben implementar la misma interfaz. Esto hace que
el código cliente sea compatible con cualquier tipo de colección o
cualquier algoritmo de recorrido, siempre y cuando exista un iterador
adecuado. Si necesitas una forma particular de recorrer una colección,
creas una nueva clase iteradora sin tener que cambiar la colección o el
cliente.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-ese35a.png?id=0ceb64477a16210f039bc8c9650029c3"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-es.png?id=0ceb64477a16210f039bc8c9650029c3 640w,/images/patterns/content/iterator/iterator-comic-1-es-1.5x.png?id=1d807987033dadf8c10db6bc6ca15b32 960w,/images/patterns/content/iterator/iterator-comic-1-es-2x.png?id=7f7146cadef5f7660feae7682078bf6a 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Varias formas de pasear por Roma" />
<figcaption><p>Varias formas de pasear por Roma.</p></figcaption>
</figure>

Planeas visitar Roma por unos días y ver todas sus atracciones y puntos
de interés. Pero, una vez allí, podrías perder mucho tiempo dando
vueltas, incapaz de encontrar siquiera el Coliseo.

En lugar de eso, podrías comprar una aplicación de guía virtual para tu
smartphone y utilizarla para moverte. Es buena y barata y puedes
quedarte en sitios interesantes todo el tiempo que quieras.

Una tercera alternativa sería dedicar parte del presupuesto del viaje a
contratar un guía local que conozca la ciudad como la palma de su mano.
El guía podría adaptar la visita a tus gustos, mostrarte las atracciones
y contarte un montón de emocionantes historias. Eso sería más divertido
pero, lamentablemente, también más caro.

Todas estas opciones ---las direcciones aleatorias en tu cabeza, el
navegador del smartphone o el guía humano---, actúan como iteradores
sobre la amplia colección de visitas y atracciones de Roma.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Estructura del patrón de diseño Iterator" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estructura del patrón de diseño Iterator" />
</figure>
:::

1.  La interfaz **Iteradora** declara las operaciones necesarias para
    recorrer una colección: extraer el siguiente elemento, recuperar la
    posición actual, reiniciar la iteración, etc.

2.  Los **Iteradores Concretos** implementan algoritmos específicos para
    recorrer una colección. El objeto iterador debe controlar el
    progreso del recorrido por su cuenta. Esto permite a varios
    iteradores recorrer la misma colección con independencia entre sí.

3.  La interfaz **Colección** declara uno o varios métodos para obtener
    iteradores compatibles con la colección. Observa que el tipo de
    retorno de los métodos debe declararse como la interfaz iteradora de
    forma que las colecciones concretas puedan devolver varios tipos de
    iteradores.

4.  Las **Colecciones Concretas** devuelven nuevas instancias de una
    clase iteradora concreta particular cada vez que el cliente solicita
    una. Puede que te estés preguntando: ¿dónde está el resto del código
    de la colección? No te preocupes, debe estar en la misma clase. Lo
    que pasa es que estos detalles no son fundamentales para el patrón
    en sí, por eso los omitimos.

5.  El **Cliente** debe funcionar con colecciones e iteradores a través
    de sus interfaces. De este modo, el cliente no se acopla a clases
    concretas, permitiéndote utilizar varias colecciones e iteradores
    con el mismo código cliente.

    Normalmente, los clientes no crean iteradores por su cuenta, en
    lugar de eso, los obtienen de las colecciones. Sin embargo, en
    algunos casos, el cliente puede crear uno directamente, como cuando
    define su propio iterador especial.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Iterator** se utiliza para recorrer un tipo
especial de colección que encapsula el acceso al grafo social de
Facebook. La colección proporciona varios iteradores que recorren
perfiles de distintas formas.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Ejemplo de estructura del patrón Iterator" />
<figcaption><p>Ejemplo de iteración de
perfiles sociales.</p></figcaption>
</figure>

El iterador 'amigos' puede utilizarse para recorrer los amigos de un
perfil dado. El iterador 'colegas' hace lo mismo, excepto que omite
amigos que no trabajen en la misma empresa que la persona objetivo.
Ambos iteradores implementan una interfaz común que permite a los
clientes extraer perfiles sin profundizar en los detalles de la
implementación, como la autenticación y el envío de solicitudes REST.

El código cliente no está acoplado a clases concretas porque sólo
trabaja con colecciones e iteradores a través de interfaces. Si decides
conectar tu aplicación a una nueva red social, sólo necesitas
proporcionar nuevas clases de colección e iteradoras, sin cambiar el
código existente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz de colección debe declarar un método fábrica para
// producir iteradores. Puedes declarar varios métodos si hay
// distintos tipos de iteración disponibles en tu programa.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Cada colección concreta está acoplada a un grupo de clases
// iteradoras concretas que devuelve, pero el cliente no lo
// está, ya que la firma de estos métodos devuelve interfaces
// iteradoras.
class Facebook implements SocialNetwork is
    // ... El grueso del código de la colección debe ir aquí ...
    // Código de creación del iterador.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// La interfaz común a todos los iteradores.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// La clase iteradora concreta.
class FacebookIterator implements ProfileIterator is
    // El iterador necesita una referencia a la colección que
    // recorre.
    private field facebook: Facebook
    private field profileId, type: string

    // Un objeto iterador recorre la colección
    // independientemente de otro iterador, por eso debe
    // almacenar el estado de iteración.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Cada clase iteradora concreta tiene su propia
    // implementación de la interfaz iteradora común.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Aquí tienes otro truco útil: puedes pasar un iterador a una
// clase cliente en lugar de darle acceso a una colección
// completa. De esta forma, no expones la colección al cliente.
//
// Y hay otra ventaja: puedes cambiar la forma en la que el
// cliente trabaja con la colección durante el tiempo de
// ejecución pasándole un iterador diferente. Esto es posible
// porque el código cliente no está acoplado a clases iteradoras
// concretas.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// La clase Aplicación configura colecciones e iteradores y
// después los pasa al código cliente.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Iterator cuando tu colección tenga una estructura de
datos compleja a nivel interno, pero quieras ocultar su complejidad a
los clientes (ya sea por conveniencia o por razones de seguridad).
:::

::: applicability-solution
El iterador encapsula los detalles del trabajo con una estructura de
datos compleja, proporcionando al cliente varios métodos simples para
acceder a los elementos de la colección. Esta solución, además de ser
muy conveniente para el cliente, también protege la colección frente a
acciones descuidadas o maliciosas que el cliente podría realizar si
trabajara con la colección directamente.
:::

::: applicability-problem
Utiliza el patrón para reducir la duplicación en el código de recorrido
a lo largo de tu aplicación.
:::

::: applicability-solution
El código de los algoritmos de iteración no triviales tiende a ser muy
voluminoso. Cuando se coloca dentro de la lógica de negocio de una
aplicación, puede nublar la responsabilidad del código original y
hacerlo más difícil de mantener. Mover el código de recorrido a
iteradores designados puede ayudarte a hacer el código de la aplicación
más breve y limpio.
:::

::: applicability-problem
Utiliza el patrón Iterator cuando quieras que tu código pueda recorrer
distintas estructuras de datos, o cuando los tipos de estas estructuras
no se conozcan de antemano.
:::

::: applicability-solution
El patrón proporciona un par de interfaces genéricas para colecciones e
iteradores. Teniendo en cuenta que ahora tu código utiliza estas
interfaces, seguirá funcionando si le pasas varios tipos de colecciones
e iteradores que implementen esas interfaces.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Declara la interfaz iteradora. Como mínimo, debe tener un método
    para extraer el siguiente elemento de una colección. Por
    conveniencia, puedes añadir un par de métodos distintos, como para
    extraer el elemento previo, localizar la posición actual o comprobar
    el final de la iteración.

2.  Declara la interfaz de colección y describe un método para buscar
    iteradores. El tipo de retorno debe ser igual al de la interfaz
    iteradora. Puedes declarar métodos similares si planeas tener varios
    grupos distintos de iteradores.

3.  Implementa clases iteradoras concretas para las colecciones que
    quieras que sean recorridas por iteradores. Un objeto iterador debe
    estar vinculado a una única instancia de la colección. Normalmente,
    este vínculo se establece a través del constructor del iterador.

4.  Implementa la interfaz de colección en tus clases de colección. La
    idea principal es proporcionar al cliente un atajo para crear
    iteradores personalizados para una clase de colección particular. El
    objeto de colección debe pasarse a sí mismo al constructor del
    iterador para establecer un vínculo entre ellos.

5.  Repasa el código cliente para sustituir todo el código de recorrido
    de la colección por el uso de iteradores. El cliente busca un nuevo
    objeto iterador cada vez que necesita recorrer los elementos de la
    colección.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de responsabilidad única*. Puedes limpiar el código
    cliente y las colecciones extrayendo algoritmos de recorrido
    voluminosos y colocándolos en clases independientes.
-    *Principio de abierto/cerrado*. Puedes implementar nuevos tipos de
    colecciones e iteradores y pasarlos al código existente sin
    descomponer nada.
-    Puedes recorrer la misma colección en paralelo porque cada objeto
    iterador contiene su propio estado de iteración.
-    Por la misma razón, puedes retrasar una iteración y continuar
    cuando sea necesario.
:::

::: col-sm-6
-    Aplicar el patrón puede resultar excesivo si tu aplicación funciona
    únicamente con colecciones sencillas.
-    Utilizar un iterador puede ser menos eficiente que recorrer
    directamente los elementos de algunas colecciones especializadas.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Puedes utilizar [Iteradores](iterator.html) para recorrer árboles
    [Composite](composite.html).

-   Puedes utilizar el patrón [Factory Method](factory-method.html)
    junto con el [Iterator](iterator.html) para permitir que las
    subclases de la colección devuelvan distintos tipos de iteradores
    que sean compatibles con las colecciones.

-   Puedes usar [Memento](memento.html) junto con
    [Iterator](iterator.html) para capturar el estado de la iteración
    actual y reanudarla si fuera necesario.

-   Puedes utilizar [Visitor](visitor.html) junto con
    [Iterator](iterator.html) para recorrer una estructura de datos
    compleja y ejecutar alguna operación sobre sus elementos, incluso
    aunque todos tengan clases distintas.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Iterator en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Iterator en C#"){.prog-lang-link}
[![Iterator en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Iterator en C++"){.prog-lang-link}
[![Iterator en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Iterator en Go"){.prog-lang-link}
[![Iterator en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Iterator en Java"){.prog-lang-link}
[![Iterator en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Iterator en PHP"){.prog-lang-link}
[![Iterator en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Iterator en Python"){.prog-lang-link}
[![Iterator en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Iterator en Ruby"){.prog-lang-link}
[![Iterator en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Iterator en Rust"){.prog-lang-link}
[![Iterator en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Iterator en Swift"){.prog-lang-link}
[![Iterator en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Iterator en TypeScript"){.prog-lang-link}
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

[Mediator []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Command](command.html){.btn .btn-default
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
