::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Observer {#observer .title}

::: aka
También llamado:
[Observador, ]{style="display:inline-block"}[Publicación-Suscripción, ]{style="display:inline-block"}[Modelo-patrón, ]{style="display:inline-block"}[Event-Subscriber, ]{style="display:inline-block"}[Listener]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Observer** es un patrón de diseño de comportamiento que te permite
definir un mecanismo de suscripción para notificar a varios objetos
sobre cualquier evento que le suceda al objeto que están observando.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Observer" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que tienes dos tipos de objetos: un objeto `Cliente` y un objeto
`Tienda`. El cliente está muy interesado en una marca particular de
producto (digamos, un nuevo modelo de iPhone) que estará disponible en
la tienda muy pronto.

El cliente puede visitar la tienda cada día para comprobar la
disponibilidad del producto. Pero, mientras el producto está en camino,
la mayoría de estos viajes serán en vano.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-esacaf.png?id=27b9968d42e009f95c6d598b4029c22c"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-es.png?id=27b9968d42e009f95c6d598b4029c22c 600w,/images/patterns/content/observer/observer-comic-1-es-1.5x.png?id=366034072f72f1cc3d8fd20e825c69d1 900w,/images/patterns/content/observer/observer-comic-1-es-2x.png?id=c73ec70f17c697d50ee0fa56d9d1a6c0 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Visita a la tienda vs. envío de spam" />
<figcaption><p>Visita a la tienda vs. envío de spam</p></figcaption>
</figure>

Por otro lado, la tienda podría enviar cientos de correos (lo cual se
podría considerar spam) a todos los clientes cada vez que hay un nuevo
producto disponible. Esto ahorraría a los clientes los interminables
viajes a la tienda, pero, al mismo tiempo, molestaría a otros clientes
que no están interesados en los nuevos productos.

Parece que nos encontramos ante un conflicto. O el cliente pierde tiempo
comprobando la disponibilidad del producto, o bien la tienda desperdicia
recursos notificando a los clientes equivocados.
:::

::: {.section .solution}
##  Solución {#solution}

El objeto que tiene un estado interesante suele denominarse *sujeto*,
pero, como también va a notificar a otros objetos los cambios en su
estado, le llamaremos *notificador* (en ocasiones también llamado
*publicador*). El resto de los objetos que quieren conocer los cambios
en el estado del notificador, se denominan *suscriptores*.

El patrón Observer sugiere que añadas un mecanismo de suscripción a la
clase notificadora para que los objetos individuales puedan suscribirse
o cancelar su suscripción a un flujo de eventos que proviene de esa
notificadora. ¡No temas! No es tan complicado como parece. En realidad,
este mecanismo consiste en: 1) un campo matriz para almacenar una lista
de referencias a objetos suscriptores y 2) varios métodos públicos que
permiten añadir suscriptores y eliminarlos de esa lista.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-esceeb.png?id=3bcc13f74ff9ed564c42b46b686fd3bb"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-es.png?id=3bcc13f74ff9ed564c42b46b686fd3bb 470w,/images/patterns/diagrams/observer/solution1-es-1.5x.png?id=2825729a424ff76aac3f61ca4b3f75e2 705w,/images/patterns/diagrams/observer/solution1-es-2x.png?id=bd6f84ed83f36ac3c5d40f8c95bca836 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Mecanismo de suscripción" />
<figcaption><p>Un mecanismo de suscripción permite a los objetos
individuales suscribirse a notificaciones de eventos.</p></figcaption>
</figure>

Ahora, cuando le sucede un evento importante al notificador, recorre sus
suscriptores y llama al método de notificación específico de sus
objetos.

Las aplicaciones reales pueden tener decenas de clases suscriptoras
diferentes interesadas en seguir los eventos de la misma clase
notificadora. No querrás acoplar la notificadora a todas esas clases.
Además, puede que no conozcas algunas de ellas de antemano si se supone
que otras personas pueden utilizar tu clase notificadora.

Por eso es fundamental que todos los suscriptores implementen la misma
interfaz y que el notificador únicamente se comunique con ellos a través
de esa interfaz. Esta interfaz debe declarar el método de notificación
junto con un grupo de parámetros que el notificador puede utilizar para
pasar cierta información contextual con la notificación.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-esb7da.png?id=6fb15edffc94b5ba1778326f4005bc33"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-es.png?id=6fb15edffc94b5ba1778326f4005bc33 460w,/images/patterns/diagrams/observer/solution2-es-1.5x.png?id=2b5e5edf4a594bbecbaaa60cd4fe2b6e 690w,/images/patterns/diagrams/observer/solution2-es-2x.png?id=857c7d3fbb8b7ce5f4a0c505de5f8cbb 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Métodos de notificación" />
<figcaption><p>El notificador notifica a los suscriptores invocando el
método de notificación específico de sus objetos.</p></figcaption>
</figure>

Si tu aplicación tiene varios tipos diferentes de notificadores y
quieres hacer a tus suscriptores compatibles con todos ellos, puedes ir
más allá y hacer que todos los notificadores sigan la misma interfaz.
Esta interfaz sólo tendrá que describir algunos métodos de suscripción.
La interfaz permitirá a los suscriptores observar los estados de los
notificadores sin acoplarse a sus clases concretas.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-es914b.png?id=27c5c4513d9c52b4198ef61d32b4e201"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-es.png?id=27c5c4513d9c52b4198ef61d32b4e201 600w,/images/patterns/content/observer/observer-comic-2-es-1.5x.png?id=77b1b4750b2db92d250e5249cee8da81 900w,/images/patterns/content/observer/observer-comic-2-es-2x.png?id=920ad9f043225018c463f57abb848e26 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Suscripciones a revistas y periódicos" />
<figcaption><p>Suscripciones a revistas y periódicos.</p></figcaption>
</figure>

Si te suscribes a un periódico o una revista, ya no necesitarás ir a la
tienda a comprobar si el siguiente número está disponible. En lugar de
eso, el notificador envía nuevos números directamente a tu buzón justo
después de la publicación, o incluso antes.

El notificador mantiene una lista de suscriptores y sabe qué revistas
les interesan. Los suscriptores pueden abandonar la lista en cualquier
momento si quieren que el notificador deje de enviarles nuevos números.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Estructura del patrón de diseño Observer" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Estructura del patrón de diseño Observer" />
</figure>
:::

1.  El **Notificador** envía eventos de interés a otros objetos. Esos
    eventos ocurren cuando el notificador cambia su estado o ejecuta
    algunos comportamientos. Los notificadores contienen una
    infraestructura de suscripción que permite a nuevos y antiguos
    suscriptores abandonar la lista.

2.  Cuando sucede un nuevo evento, el notificador recorre la lista de
    suscripción e invoca el método de notificación declarado en la
    interfaz suscriptora en cada objeto suscriptor.

3.  La interfaz **Suscriptora** declara la interfaz de notificación. En
    la mayoría de los casos, consiste en un único método `actualizar`.
    El método puede tener varios parámetros que permitan al notificador
    pasar algunos detalles del evento junto a la actualización.

4.  Los **Suscriptores Concretos** realizan algunas acciones en
    respuesta a las notificaciones emitidas por el notificador. Todas
    estas clases deben implementar la misma interfaz de forma que el
    notificador no esté acoplado a clases concretas.

5.  Normalmente, los suscriptores necesitan cierta información
    contextual para manejar correctamente la actualización. Por este
    motivo, a menudo los notificadores pasan cierta información de
    contexto como argumentos del método de notificación. El notificador
    puede pasarse a sí mismo como argumento, dejando que los
    suscriptores extraigan la información necesaria directamente.

6.  El **Cliente** crea objetos tipo notificador y suscriptor por
    separado y después registra a los suscriptores para las
    actualizaciones del notificador.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Observer** permite al objeto editor de
texto notificar a otros objetos tipo servicio sobre los cambios en su
estado.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Ejemplo de estructura del patrón Observer" />
<figcaption><p>Notificar a objetos sobre eventos que suceden a
otros objetos.</p></figcaption>
</figure>

La lista de suscriptores se compila dinámicamente: los objetos pueden
empezar o parar de escuchar notificaciones durante el tiempo de
ejecución, dependiendo del comportamiento que desees para tu aplicación.

En esta implementación, la clase editora no mantiene la lista de
suscripción por sí misma. Delega este trabajo al objeto ayudante
especial dedicado justo a eso. Puedes actualizar ese objeto para que
sirva como despachador centralizado de eventos, dejando que cualquier
objeto actúe como notificador.

Añadir nuevos suscriptores al programa no requiere cambios en clases
notificadoras existentes, siempre y cuando trabajen con todos los
suscriptores a través de la misma interfaz.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase notificadora base incluye código de gestión de
// suscripciones y métodos de notificación.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// El notificador concreto contiene lógica de negocio real, de
// interés para algunos suscriptores. Podemos derivar esta clase
// de la notificadora base, pero esto no siempre es posible en
// el mundo real porque puede que la notificadora concreta sea
// ya una subclase. En este caso, puedes modificar la lógica de
// la suscripción con composición, como hicimos aquí.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Los métodos de la lógica de negocio pueden notificar los
    // cambios a los suscriptores.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ...


// Aquí está la interfaz suscriptora. Si tu lenguaje de
// programación soporta tipos funcionales, puedes sustituir toda
// la jerarquía suscriptora por un grupo de funciones.


interface EventListener is
    method update(filename)

// Los suscriptores concretos reaccionan a las actualizaciones
// emitidas por el notificador al que están unidos.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Una  aplicación puede configurar notificadores y suscriptores
// durante el tiempo de ejecución.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Observer cuando los cambios en el estado de un objeto
puedan necesitar cambiar otros objetos y el grupo de objetos sea
desconocido de antemano o cambie dinámicamente.
:::

::: applicability-solution
Puedes experimentar este problema a menudo al trabajar con clases de la
interfaz gráfica de usuario. Por ejemplo, si creaste clases
personalizadas de botón y quieres permitir al cliente colgar código
cliente de tus botones para que se active cuando un usuario pulse un
botón.

El patrón Observer permite que cualquier objeto que implemente la
interfaz suscriptora pueda suscribirse a notificaciones de eventos en
objetos notificadores. Puedes añadir el mecanismo de suscripción a tus
botones, permitiendo a los clientes acoplar su código personalizado a
través de clases suscriptoras personalizadas.
:::

::: applicability-problem
Utiliza el patrón cuando algunos objetos de tu aplicación deban observar
a otros, pero sólo durante un tiempo limitado o en casos específicos.
:::

::: applicability-solution
La lista de suscripción es dinámica, por lo que los suscriptores pueden
unirse o abandonar la lista cuando lo deseen.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Repasa tu lógica de negocio e intenta dividirla en dos partes: la
    funcionalidad central, independiente del resto de código, actuará
    como notificador; el resto se convertirá en un grupo de clases
    suscriptoras.

2.  Declara la interfaz suscriptora. Como mínimo, deberá declarar un
    único método `actualizar`.

3.  Declara la interfaz notificadora y describe un par de métodos para
    añadir y eliminar de la lista un objeto suscriptor. Recuerda que los
    notificadores deben trabajar con suscriptores únicamente a través de
    la interfaz suscriptora.

4.  Decide dónde colocar la lista de suscripción y la implementación de
    métodos de suscripción. Normalmente, este código tiene el mismo
    aspecto para todos los tipos de notificadores, por lo que el lugar
    obvio para colocarlo es en una clase abstracta derivada directamente
    de la interfaz notificadora. Los notificadores concretos extienden
    esa clase, heredando el comportamiento de suscripción.

    Sin embargo, si estás aplicando el patrón a una jerarquía de clases
    existentes, considera una solución basada en la composición: coloca
    la lógica de la suscripción en un objeto separado y haz que todos
    los notificadores reales la utilicen.

5.  Crea clases notificadoras concretas. Cada vez que suceda algo
    importante dentro de una notificadora, deberá notificar a todos sus
    suscriptores.

6.  Implementa los métodos de notificación de actualizaciones en clases
    suscriptoras concretas. La mayoría de las suscriptoras necesitarán
    cierta información de contexto sobre el evento, que puede pasarse
    como argumento del método de notificación.

    Pero hay otra opción. Al recibir una notificación, el suscriptor
    puede extraer la información directamente de ella. En este caso, el
    notificador debe pasarse a sí mismo a través del método de
    actualización. La opción menos flexible es vincular un notificador
    con el suscriptor de forma permanente a través del constructor.

7.  El cliente debe crear todos los suscriptores necesarios y
    registrarlos con los notificadores adecuados.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de abierto/cerrado*. Puedes introducir nuevas clases
    suscriptoras sin tener que cambiar el código de la notificadora (y
    viceversa si hay una interfaz notificadora).
-    Puedes establecer relaciones entre objetos durante el tiempo de
    ejecución.
:::

::: col-sm-6
-    Los suscriptores son notificados en un orden aleatorio.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Chain of Responsibility](chain-of-responsibility.html),
    [Command](command.html), [Mediator](mediator.html) y
    [Observer](observer.html) abordan distintas formas de conectar
    emisores y receptores de solicitudes:

    -   *Chain of Responsibility* pasa una solicitud secuencialmente a
        lo largo de una cadena dinámica de receptores potenciales hasta
        que uno de ellos la gestiona.
    -   *Command* establece conexiones unidireccionales entre emisores y
        receptores.
    -   *Mediator* elimina las conexiones directas entre emisores y
        receptores, forzándolos a comunicarse indirectamente a través de
        un objeto mediador.
    -   *Observer* permite a los receptores suscribirse o darse de baja
        dinámicamente a la recepción de solicitudes.

-   La diferencia entre [Mediator](mediator.html) y
    [Observer](observer.html) a menudo resulta difusa. En la mayoría de
    los casos, puedes implementar uno de estos dos patrones; pero en
    ocasiones puedes aplicarlos ambos a la vez. Veamos cómo podemos
    hacerlo.

    La meta principal del patrón *Mediator* consiste en eliminar las
    dependencias mutuas entre un grupo de componentes del sistema. En su
    lugar, estos componentes se vuelven dependientes de un único objeto
    mediador. La meta del patrón *Observer* es establecer conexiones
    dinámicas de un único sentido entre objetos, donde algunos objetos
    actúan como subordinados de otros.

    Hay una implementación popular del patrón *Mediator* que se basa en
    el *Observer*. El objeto mediador juega el papel de notificador, y
    los componentes actúan como suscriptores que se suscriben o se dan
    de baja de los eventos del mediador. Cuando se implementa el
    *Mediator* de esta forma, puede asemejarse mucho al *Observer*.

    Cuando te sientas confundido, recuerda que puedes implementar el
    patrón Mediator de otras maneras. Por ejemplo, puedes vincular
    permanentemente todos los componentes al mismo objeto mediador. Esta
    implementación no se parece al *Observer*, pero aún así será una
    instancia del patrón Mediator.

    Ahora, imagina un programa en el que todos los componentes se hayan
    convertido en notificadores, permitiendo conexiones dinámicas entre
    sí. No hay un objeto mediador centralizado, tan solo un grupo
    distribuido de observadores.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Observer en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Observer en C#"){.prog-lang-link}
[![Observer en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Observer en C++"){.prog-lang-link}
[![Observer en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Observer en Go"){.prog-lang-link}
[![Observer en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Observer en Java"){.prog-lang-link}
[![Observer en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Observer en PHP"){.prog-lang-link}
[![Observer en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Observer en Python"){.prog-lang-link}
[![Observer en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Observer en Ruby"){.prog-lang-link}
[![Observer en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Observer en Rust"){.prog-lang-link}
[![Observer en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Observer en Swift"){.prog-lang-link}
[![Observer en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Observer en TypeScript"){.prog-lang-link}
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

[State []{.fa .fa-arrow-right}](state.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Memento](memento.html){.btn .btn-default
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
