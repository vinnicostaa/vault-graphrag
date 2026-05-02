::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# State {#state .title}

::: aka
También llamado: [Estado]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**State** es un patrón de diseño de comportamiento que permite a un
objeto alterar su comportamiento cuando su estado interno cambia. Parece
como si el objeto cambiara su clase.

<figure class="image">
<img
src="../../images/patterns/content/state/state-es74f4.png?id=03f2a3a86f4b58cc21b4c8c152d6c0ec"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-es.png?id=03f2a3a86f4b58cc21b4c8c152d6c0ec 640w,/images/patterns/content/state/state-es-1.5x.png?id=dac8436d9e16e083c54dc93ce2d93238 960w,/images/patterns/content/state/state-es-2x.png?id=16ce9b8e21486e08222daa4142117744 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño State" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

El patrón State está estrechamente relacionado con el concepto de la
*Máquina de estados finitos* []{.pop-i}[Máquina de estados finitos:
[https://refactoring.guru/es/fsm](https://es.wikipedia.org/wiki/Aut%C3%B3mata_finito)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Máquina de estados finitos" />
<figcaption><p>Máquina de estados finitos.</p></figcaption>
</figure>

La idea principal es que, en cualquier momento dado, un programa puede
encontrarse en un número *finito* de *estados*. Dentro de cada estado
único, el programa se comporta de forma diferente y puede cambiar de un
estado a otro instantáneamente. Sin embargo, dependiendo de un estado
actual, el programa puede cambiar o no a otros estados. Estas normas de
cambio llamadas *transiciones* también son finitas y predeterminadas.

También puedes aplicar esta solución a los objetos. Imagina que tienes
una clase `Documento`. Un documento puede encontrarse en uno de estos
tres estados: `Borrador`, `Moderación` y `Publicado`. El método
`publicar` del documento funciona de forma ligeramente distinta en cada
estado:

-   En `Borrador`, mueve el documento a moderación.
-   En `Moderación`, hace público el documento, pero sólo si el usuario
    actual es un administrador.
-   En `Publicado`, no hace nada en absoluto.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-es84a3.png?id=e5e71091a0317ca7dfef8b6b8c36c0d0"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-es.png?id=e5e71091a0317ca7dfef8b6b8c36c0d0 560w,/images/patterns/diagrams/state/problem2-es-1.5x.png?id=89d0e61bd6fa553f490a1ba4a0ba7d5c 840w,/images/patterns/diagrams/state/problem2-es-2x.png?id=df3f15a7489b09a28217f64250b7abba 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Posibles estados de un objeto de documento" />
<figcaption><p>Posibles estados y transiciones de un objeto
de documento.</p></figcaption>
</figure>

Las máquinas de estado se implementan normalmente con muchos operadores
condicionales (`if` o `switch`) que seleccionan el comportamiento
adecuado dependiendo del estado actual del objeto. Normalmente, este
"estado" es tan solo un grupo de valores de los campos del objeto.
Aunque nunca hayas oído hablar de máquinas de estados finitos,
probablemente hayas implementado un estado al menos alguna vez. ¿Te
suena esta estructura de código?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // No hacer nada.
                break
    // ...</code></pre>
</figure>

La mayor debilidad de una máquina de estado basada en condicionales se
revela una vez que empezamos a añadir más y más estados y
comportamientos dependientes de estados a la clase `Documento`. La
mayoría de los métodos contendrán condicionales monstruosos que eligen
el comportamiento adecuado de un método de acuerdo con el estado actual.
Un código así es muy difícil de mantener, porque cualquier cambio en la
lógica de transición puede requerir cambiar los condicionales de estado
de cada método.

El problema tiende a empeorar con la evolución del proyecto. Es bastante
difícil predecir todos los estados y transiciones posibles en la etapa
de diseño. Por ello, una máquina de estados esbelta, creada con un grupo
limitado de condicionales, puede crecer hasta convertirse en un
abotargado desastre con el tiempo.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón State sugiere que crees nuevas clases para todos los estados
posibles de un objeto y extraigas todos los comportamientos específicos
del estado para colocarlos dentro de esas clases.

En lugar de implementar todos los comportamientos por su cuenta, el
objeto original, llamado *contexto*, almacena una referencia a uno de
los objetos de estado que representa su estado actual y delega todo el
trabajo relacionado con el estado a ese objeto.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-es1675.png?id=d7ffc69f008c9fdf033604f1968c0fbd"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-es.png?id=d7ffc69f008c9fdf033604f1968c0fbd 490w,/images/patterns/diagrams/state/solution-es-1.5x.png?id=18a1ffd2ff478d3bae41cf644e0a3bb7 735w,/images/patterns/diagrams/state/solution-es-2x.png?id=c672445655831c230ef7c5aef7afde34 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Documento delega el trabajo a un objeto de estado" />
<figcaption><p>Documento delega el trabajo a un objeto
de estado.</p></figcaption>
</figure>

Para la transición del contexto a otro estado, sustituye el objeto de
estado activo por otro objeto que represente ese nuevo estado. Esto sólo
es posible si todas las clases de estado siguen la misma interfaz y el
propio contexto funciona con esos objetos a través de esa interfaz.

Esta estructura puede resultar similar al patrón
[Strategy](strategy.html), pero hay una diferencia clave. En el patrón
State, los estados particulares pueden conocerse entre sí e iniciar
transiciones de un estado a otro, mientras que las estrategias casi
nunca se conocen.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

Los botones e interruptores de tu smartphone se comportan de forma
diferente dependiendo del estado actual del dispositivo:

-   Cuando el teléfono está desbloqueado, al pulsar botones se ejecutan
    varias funciones.
-   Cuando el teléfono está bloqueado, pulsar un botón desbloquea la
    pantalla.
-   Cuando la batería del teléfono está baja, pulsar un botón muestra la
    pantalla de carga.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-es80b3.png?id=33f5737d852408c89f800b9931859958"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.29;"
srcset="/images/patterns/diagrams/state/structure-es.png?id=33f5737d852408c89f800b9931859958 540w,/images/patterns/diagrams/state/structure-es-1.5x.png?id=8f300626bc529dde86deea2ef4021a45 810w,/images/patterns/diagrams/state/structure-es-2x.png?id=6da8774df2e9ed0262b9a04cd114cc9b 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estructura del patrón de diseño State" /><img
src="../../images/patterns/diagrams/state/structure-es-indexed0087.png?id=db51c48ab9ef79acb1debca3a1b3bf90"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/state/structure-es-indexed.png?id=db51c48ab9ef79acb1debca3a1b3bf90 540w,/images/patterns/diagrams/state/structure-es-indexed-1.5x.png?id=489c3b5b8519dbba2333b3c6987ec824 810w,/images/patterns/diagrams/state/structure-es-indexed-2x.png?id=78ab37e2193a385d027bfbd27e395769 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estructura del patrón de diseño State" />
</figure>
:::

1.  La clase **Contexto** almacena una referencia a uno de los objetos
    de estado concreto y le delega todo el trabajo específico del
    estado. El contexto se comunica con el objeto de estado a través de
    la interfaz de estado. El contexto expone un modificador (*setter*)
    para pasarle un nuevo objeto de estado.

2.  La interfaz **Estado** declara los métodos específicos del estado.
    Estos métodos deben tener sentido para todos los estados concretos,
    porque no querrás que uno de tus estados tenga métodos inútiles que
    nunca son invocados.

3.  Los **Estados Concretos** proporcionan sus propias implementaciones
    para los métodos específicos del estado. Para evitar la duplicación
    de código similar a través de varios estados, puedes incluir clases
    abstractas intermedias que encapsulen algún comportamiento común.

    Los objetos de estado pueden almacenar una referencia inversa al
    objeto de contexto. A través de esta referencia, el estado puede
    extraer cualquier información requerida del objeto de contexto, así
    como iniciar transiciones de estado.

4.  Tanto el estado de contexto como el concreto pueden establecer el
    nuevo estado del contexto y realizar la transición de estado
    sustituyendo el objeto de estado vinculado al contexto.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **State** permite a los mismos controles del
reproductor de medios comportarse de forma diferente, dependiendo del
estado actual de reproducción.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Ejemplo de estructura del patrón State" />
<figcaption><p>Ejemplo de cambio del comportamiento de un objeto con
objetos de estado.</p></figcaption>
</figure>

El objeto principal del reproductor siempre está vinculado a un objeto
de estado que realiza la mayor parte del trabajo del reproductor.
Algunas acciones sustituyen el objeto de estado actual del reproductor
por otro, lo cual cambia la forma en la que el reproductor reacciona a
las interacciones del usuario.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase ReproductordeAudio actúa como un contexto. También
// mantiene una referencia a una instancia de una de las clases
// estado que representa el estado actual del reproductor de
// audio.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // El contexto delega la gestión de entradas del usuario
        // a un objeto de estado. Naturalmente, el resultado
        // depende del estado que esté activo ahora, ya que cada
        // estado puede gestionar las entradas de manera
        // diferente.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Otros objetos deben ser capaces de cambiar el estado
    // activo del reproductor.
    method changeState(state: State) is
        this.state = state

    // Los métodos UI delegan la ejecución al estado activo.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Un estado puede invocar algunos métodos del servicio en
    // el contexto.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...


// La clase estado base declara métodos que todos los estados
// concretos deben implementar, y también proporciona una
// referencia inversa al objeto de contexto asociado con el
// estado. Los estados pueden utilizar la referencia inversa
// para dirigir el contexto a otro estado.
abstract class State is
    protected field player: AudioPlayer

    // El contexto se pasa a sí mismo a través del constructor
    // del estado. Esto puede ayudar al estado a extraer
    // información de contexto útil si la necesita.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Los estados concretos implementan varios comportamientos
// asociados a un estado del contexto.
class LockedState extends State is

    // Cuando desbloqueas a un jugador bloqueado, puede asumir
    // uno de dos estados.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Bloqueado, no hace nada.

    method clickNext() is
        // Bloqueado, no hace nada.

    method clickPrevious() is
        // Bloqueado, no hace nada.

// También pueden disparar transiciones de estado en el
// contexto.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón State cuando tengas un objeto que se comporta de forma
diferente dependiendo de su estado actual, el número de estados sea
enorme y el código específico del estado cambie con frecuencia.
:::

::: applicability-solution
El patrón sugiere que extraigas todo el código específico del estado y
lo metas dentro de un grupo de clases específicas. Como resultado,
puedes añadir nuevos estados o cambiar los existentes independientemente
entre sí, reduciendo el costo de mantenimiento.
:::

::: applicability-problem
Utiliza el patrón cuando tengas una clase contaminada con enormes
condicionales que alteran el modo en que se comporta la clase de acuerdo
con los valores actuales de los campos de la clase.
:::

::: applicability-solution
El patrón State te permite extraer ramas de esos condicionales a métodos
de las clases estado correspondientes. Al hacerlo, también puedes
limpiar campos temporales y métodos de ayuda implicados en código
específico del estado de fuera de tu clase principal.
:::

::: applicability-problem
Utiliza el patrón State cuando tengas mucho código duplicado por estados
similares y transiciones de una máquina de estados basada en
condiciones.
:::

::: applicability-solution
El patrón State te permite componer jerarquías de clases de estado y
reducir la duplicación, extrayendo el código común y metiéndolo en
clases abstractas base.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Decide qué clase actuará como contexto. Puede ser una clase
    existente que ya tiene el código dependiente del estado, o una nueva
    clase, si el código específico del estado está distribuido a lo
    largo de varias clases.

2.  Declara la interfaz de estado. Aunque puede replicar todos los
    métodos declarados en el contexto, céntrate en los que pueden
    contener comportamientos específicos del estado.

3.  Para cada estado actual, crea una clase derivada de la interfaz de
    estado. Después repasa los métodos del contexto y extrae todo el
    código relacionado con ese estado para meterlo en tu clase recién
    creada.

    Al mover el código a la clase estado, puede que descubras que
    depende de miembros privados del contexto. Hay varias soluciones
    alternativas:

    -   Haz públicos esos campos o métodos.
    -   Convierte el comportamiento que estás extrayendo para ponerlo en
        un método público en el contexto e invócalo desde la clase de
        estado. Esta forma es desagradable pero rápida y siempre podrás
        arreglarlo más adelante.
    -   Anida las clases de estado en la clase contexto, pero sólo si tu
        lenguaje de programación soporta clases anidadas.

4.  En la clase contexto, añade un campo de referencia del tipo de
    interfaz de estado y un modificador (*setter*) público que permita
    sobrescribir el valor de ese campo.

5.  Vuelve a repasar el método del contexto y sustituye los
    condicionales de estado vacíos por llamadas a métodos
    correspondientes del objeto de estado.

6.  Para cambiar el estado del contexto, crea una instancia de una de
    las clases de estado y pásala a la clase contexto. Puedes hacer esto
    dentro de la propia clase contexto, en distintos estados, o en el
    cliente. Se haga de una forma u otra, la clase se vuelve dependiente
    de la clase de estado concreto que instancia.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de responsabilidad única*. Organiza el código
    relacionado con estados particulares en clases separadas.
-    *Principio de abierto/cerrado*. Introduce nuevos estados sin
    cambiar clases de estado existentes o la clase contexto.
-    Simplifica el código del contexto eliminando voluminosos
    condicionales de máquina de estados.
:::

::: col-sm-6
-    Aplicar el patrón puede resultar excesivo si una máquina de estados
    sólo tiene unos pocos estados o raramente cambia.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (y, hasta cierto punto,
    [Adapter](adapter.html)) tienen estructuras muy similares. De hecho,
    todos estos patrones se basan en la composición, que consiste en
    delegar trabajo a otros objetos. Sin embargo, todos ellos solucionan
    problemas diferentes. Un patrón no es simplemente una receta para
    estructurar tu código de una forma específica. También puede
    comunicar a otros desarrolladores el problema que resuelve.

-   [State](state.html) puede considerarse una extensión de
    [Strategy](strategy.html). Ambos patrones se basan en la
    composición: cambian el comportamiento del contexto delegando parte
    del trabajo a objetos ayudantes. *Strategy* hace que estos objetos
    sean completamente independientes y no se conozcan entre sí. Sin
    embargo, *State* no restringe las dependencias entre estados
    concretos, permitiéndoles alterar el estado del contexto a voluntad.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![State en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "State en C#"){.prog-lang-link}
[![State en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "State en C++"){.prog-lang-link}
[![State en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "State en Go"){.prog-lang-link}
[![State en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "State en Java"){.prog-lang-link}
[![State en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "State en PHP"){.prog-lang-link}
[![State en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "State en Python"){.prog-lang-link}
[![State en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "State en Ruby"){.prog-lang-link}
[![State en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "State en Rust"){.prog-lang-link}
[![State en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "State en Swift"){.prog-lang-link}
[![State en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "State en TypeScript"){.prog-lang-link}
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

[Strategy []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Observer](observer.html){.btn .btn-default
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
