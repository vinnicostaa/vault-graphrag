:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Mediator {#mediator .title}

::: aka
También llamado:
[Mediador, ]{style="display:inline-block"}[Intermediary, ]{style="display:inline-block"}[Controller]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Mediator** es un patrón de diseño de comportamiento que te permite
reducir las dependencias caóticas entre objetos. El patrón restringe las
comunicaciones directas entre los objetos, forzándolos a colaborar
únicamente a través de un objeto mediador.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Mediator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Digamos que tienes un diálogo para crear y editar perfiles de cliente.
Consiste en varios controles de formulario, como campos de texto,
casillas, botones, etc.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-esd0a2.png?id=fe36c1b47a1f57ebe216fc98ca87416f"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-es.png?id=fe36c1b47a1f57ebe216fc98ca87416f 600w,/images/patterns/diagrams/mediator/problem1-es-1.5x.png?id=403b9a4c950ce2fbaf30c9220044eb98 900w,/images/patterns/diagrams/mediator/problem1-es-2x.png?id=2ab573e503b55518b2b948133d10abaa 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Relaciones caóticas entre elementos de la interfaz de usuario" />
<figcaption><p>Las relaciones entre los elementos de la interfaz de
usuario pueden volverse caóticas cuando la
aplicación crece.</p></figcaption>
</figure>

Algunos de los elementos del formulario pueden interactuar con otros.
Por ejemplo, al seleccionar la casilla "tengo un perro" puede aparecer
un campo de texto oculto para introducir el nombre del perro. Otro
ejemplo es el botón de envío que tiene que validar los valores de todos
los campos antes de guardar la información.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Los elementos de la UI son interdependientes" />
<figcaption><p>Los elementos pueden tener muchas relaciones con otros
elementos. Por eso, los cambios en algunos elementos pueden afectar a
los demás.</p></figcaption>
</figure>

Al implementar esta lógica directamente dentro del código de los
elementos del formulario, haces que las clases de estos elementos sean
mucho más difíciles de reutilizar en otros formularios de la aplicación.
Por ejemplo, no podrás utilizar la clase de la casilla dentro de otro
formulario porque está acoplada al campo de texto del perro. O bien
podrás utilizar todas las clases implicadas en representar el formulario
de perfil, o no podrás usar ninguna en absoluto.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Mediator sugiere que detengas toda comunicación directa entre
los componentes que quieres hacer independientes entre sí. En lugar de
ello, estos componentes deberán colaborar indirectamente, invocando un
objeto mediador especial que redireccione las llamadas a los componentes
adecuados. Como resultado, los componentes dependen únicamente de una
sola clase mediadora, en lugar de estar acoplados a decenas de sus
colegas.

En nuestro ejemplo del formulario de edición de perfiles, la propia
clase de diálogo puede actuar como mediadora. Lo más probable es que la
clase de diálogo conozca ya todos sus subelementos, por lo que ni
siquiera será necesario que introduzcas nuevas dependencias en esta
clase.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-es0789.png?id=a78d5ff6638c97f56d73bdf99a99554e"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-es.png?id=a78d5ff6638c97f56d73bdf99a99554e 600w,/images/patterns/diagrams/mediator/solution1-es-1.5x.png?id=9ebbbb231db21c5f03f2e986f88d23ec 900w,/images/patterns/diagrams/mediator/solution1-es-2x.png?id=9dd5f2a2d50bf12454322424719b62f3 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Los elementos UI deben comunicarse a través del mediador" />
<figcaption><p>Los elementos UI deben comunicarse indirectamente, a
través del objeto mediador.</p></figcaption>
</figure>

El cambio más significativo lo sufren los propios elementos del
formulario. Pensemos en el botón de envío. Antes, cada vez que un
usuario hacía clic en el botón, tenía que validar los valores de todos
los elementos individuales del formulario. Ahora su único trabajo
consiste en notificar al diálogo acerca del clic. Al recibir esta
notificación, el propio diálogo realiza las validaciones o pasa la tarea
a los elementos individuales. De este modo, en lugar de estar ligado a
una docena de elementos del formulario, el botón solo es dependiente de
la clase diálogo.

Puedes ir más lejos y reducir en mayor medida la dependencia extrayendo
la interfaz común para todos los tipos de diálogo. La interfaz declarará
el método de notificación que pueden utilizar todos los elementos del
formulario para notificar al diálogo sobre los eventos que le suceden a
estos elementos. Por lo tanto, ahora nuestro botón de envío debería
poder funcionar con cualquier diálogo que implemente esa interfaz.

De este modo, el patrón Mediator te permite encapsular una compleja red
de relaciones entre varios objetos dentro de un único objeto mediador.
Cuantas menos dependencias tenga una clase, más fácil es modificar,
extender o reutilizar esa clase.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Torre de control del tráfico aéreo" />
<figcaption><p>Los pilotos de aviones no hablan directamente entre sí
para decidir quién es el siguiente en aterrizar su avión. Todas las
comunicaciones pasan por la torre de control.</p></figcaption>
</figure>

Los pilotos de los aviones que llegan o salen del área de control del
aeropuerto no se comunican directamente entre sí. En lugar de eso,
hablan con un controlador de tráfico aéreo, que está sentado en una
torre alta cerca de la pista de aterrizaje. Sin el controlador de
tráfico aéreo, los pilotos tendrían que ser conscientes de todos los
aviones en las proximidades del aeropuerto y discutir las prioridades de
aterrizaje con un comité de decenas de otros pilotos. Probablemente,
esto provocaría que las estadísticas de accidentes aéreos se dispararan.

La torre no necesita controlar el vuelo completo. Sólo existe para
imponer límites en el área de la terminal porque el número de actores
implicados puede resultar difícil de gestionar para un piloto.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estructura del patrón de diseño Mediator" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estructura del patrón de diseño Mediator" />
</figure>
:::

1.  Los **Componentes** son varias clases que contienen cierta lógica de
    negocio. Cada componente tiene una referencia a una interfaz
    mediadora, declarada con su tipo. El componente no conoce la clase
    de la interfaz mediadora, por lo que puedes reutilizarlo en otros
    programas vinculándolo a una mediadora diferente.

2.  La interfaz **Mediadora** declara métodos de comunicación con los
    componentes, que normalmente incluyen un único método de
    notificación. Los componentes pueden pasar cualquier contexto como
    argumentos de este método, incluyendo sus propios objetos, pero sólo
    de tal forma que no haya acoplamiento entre un componente receptor y
    la clase del emisor.

3.  Los **Mediadores Concretos** encapsulan las relaciones entre varios
    componentes. Los mediadores concretos a menudo mantienen referencias
    a todos los componentes que gestionan y en ocasiones gestionan
    incluso su ciclo de vida.

4.  Los componentes no deben conocer otros componentes. Si le sucede
    algo importante a un componente, o dentro de él, sólo debe notificar
    a la interfaz mediadora. Cuando la mediadora recibe la notificación,
    puede identificar fácilmente al emisor, lo cual puede ser suficiente
    para decidir qué componente debe activarse en respuesta.

    Desde la perspectiva de un componente, todo parece una caja negra.
    El emisor no sabe quién acabará gestionando su solicitud, y el
    receptor no sabe quién envió la solicitud.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Mediator** te ayuda a eliminar dependencias
mutuas entre varias clases UI: botones, casillas y etiquetas de texto.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Ejemplo de estructura del patrón Mediator" />
<figcaption><p>Estructura de las clases de diálogo UI.</p></figcaption>
</figure>

Un elemento activado por un usuario, no se comunica directamente con
otros elementos, aunque parezca que debería. En lugar de eso, el
elemento solo necesita dar a conocer el evento al mediador, pasando la
información contextual junto a la notificación.

En este ejemplo, el diálogo de autenticación actúa como mediador. Sabe
cómo deben colaborar los elementos concretos y facilita su comunicación
indirecta. Al recibir una notificación sobre un evento, el diálogo
decide qué elemento debe encargarse del evento y redirige la llamada en
consecuencia.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz mediadora declara un método utilizado por los
// componentes para notificar al mediador sobre varios eventos.
// El mediador puede reaccionar a estos eventos y pasar la
// ejecución a otros componentes.
interface Mediator is
    method notify(sender: Component, event: string)


// La clase concreta mediadora. La red entrecruzada de
// conexiones entre componentes individuales se ha desenredado y
// se ha colocado dentro de la mediadora.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Crea todos los objetos del componente y pasa el
        // mediador actual a sus constructores para establecer
        // vínculos.

    // Cuando sucede algo con un componente, notifica al
    // mediador, que al recibir la notificación, puede hacer
    // algo por su cuenta o pasar la solicitud a otro
    // componente.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Muestra los componentes del formulario de
                // inicio de sesión.
                // 2. Esconde los componentes del formulario de
                // registro.
            else
                title = &quot;Register&quot;
                // 1. Muestra los componentes del formulario de
                // registro.
                // 2. Esconde los componentes del formulario de
                // inicio de sesión.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Intenta encontrar un usuario utilizando las
                // credenciales de inicio de sesión.
                if (!found)
                    // Muestra un mensaje de error sobre el
                    // campo de inicio de sesión.
            else
                // 1. Crea una cuenta de usuario utilizando
                // información de los campos de registro.
                // 2. Ingresa a ese usuario.
                // ...


// Los componentes se comunican con un mediador utilizando la
// interfaz mediadora. Gracias a ello, puedes utilizar los
// mismos componentes en otros contextos vinculándolos con
// diferentes objetos mediadores.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Los componentes concretos no hablan entre sí. Sólo tienen un
// canal de comunicación, que es el envío de notificaciones al
// mediador.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ...</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Mediator cuando resulte difícil cambiar algunas de las
clases porque están fuertemente acopladas a un puñado de otras clases.
:::

::: applicability-solution
El patrón te permite extraer todas las relaciones entre clases dentro de
una clase separada, aislando cualquier cambio en un componente
específico, del resto de los componentes.
:::

::: applicability-problem
Utiliza el patrón cuando no puedas reutilizar un componente en un
programa diferente porque sea demasiado dependiente de otros
componentes.
:::

::: applicability-solution
Una vez aplicado el patrón Mediator, los componentes individuales no
conocen los otros componentes. Todavía pueden comunicarse entre sí,
aunque indirectamente, a través del objeto mediador. Para reutilizar un
componente en una aplicación diferente, debes darle una nueva clase
mediadora.
:::

::: applicability-problem
Utiliza el patrón Mediator cuando te encuentres creando cientos de
subclases de componente sólo para reutilizar un comportamiento básico en
varios contextos.
:::

::: applicability-solution
Debido a que todas las relaciones entre componentes están contenidas
dentro del mediador, resulta fácil definir formas totalmente nuevas de
colaboración entre estos componentes introduciendo nuevas clases
mediadoras, sin tener que cambiar los propios componentes.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Identifica un grupo de clases fuertemente acopladas que se
    beneficiarían de ser más independientes (p. ej., para un
    mantenimiento más sencillo o una reutilización más simple de esas
    clases).

2.  Declara la interfaz mediadora y describe el protocolo de
    comunicación deseado entre mediadores y otros varios componentes. En
    la mayoría de los casos, un único método para recibir notificaciones
    de los componentes es suficiente.

    Esta interfaz es fundamental cuando quieras reutilizar las clases
    del componente en distintos contextos. Siempre y cuando el
    componente trabaje con su mediador a través de la interfaz genérica,
    podrás vincular el componente con una implementación diferente del
    mediador.

3.  Implementa la clase concreta mediadora. Esta clase se beneficiará de
    almacenar referencias a todos los componentes que gestiona.

4.  Puedes ir más lejos y hacer la interfaz mediadora responsable de la
    creación y destrucción de objetos del componente. Tras esto, la
    mediadora puede parecerse a una [fábrica](abstract-factory.html) o
    una [fachada](facade.html).

5.  Los componentes deben almacenar una referencia al objeto mediador.
    La conexión se establece normalmente en el constructor del
    componente, donde un objeto mediador se pasa como argumento.

6.  Cambia el código de los componentes de forma que invoquen el método
    de notificación del mediador en lugar de los métodos de otros
    componentes. Extrae el código que implique llamar a otros
    componentes dentro de la clase mediadora. Ejecuta este código cuando
    el mediador reciba notificaciones de ese componente.
:::

##  Pros y contras {#pros-cons}

-    *Principio de responsabilidad única*. Puedes extraer las
    comunicaciones entre varios componentes dentro de un único sitio,
    haciéndolo más fácil de comprender y mantener.

-    *Principio de abierto/cerrado*. Puedes introducir nuevos mediadores
    sin tener que cambiar los propios componentes.

-    Puedes reducir el acoplamiento entre varios componentes de un
    programa.

-    Puedes reutilizar componentes individuales con mayor facilidad.

-   Con el tiempo, un mediador puede evolucionar a un [objeto
    todopoderoso](https://es.wikipedia.org/wiki/Objeto_todopoderoso).

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Mediator en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Mediator en C#"){.prog-lang-link}
[![Mediator en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Mediator en C++"){.prog-lang-link}
[![Mediator en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Mediator en Go"){.prog-lang-link}
[![Mediator en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Mediator en Java"){.prog-lang-link}
[![Mediator en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Mediator en PHP"){.prog-lang-link}
[![Mediator en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Mediator en Python"){.prog-lang-link}
[![Mediator en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Mediator en Ruby"){.prog-lang-link}
[![Mediator en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Mediator en Rust"){.prog-lang-link}
[![Mediator en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Mediator en Swift"){.prog-lang-link}
[![Mediator en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Mediator en TypeScript"){.prog-lang-link}
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

[Memento []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Iterator](iterator.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
