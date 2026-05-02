::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Command {#command .title}

::: aka
También llamado:
[Comando, ]{style="display:inline-block"}[Orden, ]{style="display:inline-block"}[Action, ]{style="display:inline-block"}[Transaction]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Command** es un patrón de diseño de comportamiento que convierte una
solicitud en un objeto independiente que contiene toda la información
sobre la solicitud. Esta transformación te permite parametrizar los
métodos con diferentes solicitudes, retrasar o poner en cola la
ejecución de una solicitud y soportar operaciones que no se
pueden realizar.

<figure class="image">
<img
src="../../images/patterns/content/command/command-esf25e.png?id=aa52c5ce63cbff0adf7dfcc5909f7eb4"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/command/command-es.png?id=aa52c5ce63cbff0adf7dfcc5909f7eb4 640w,/images/patterns/content/command/command-es-1.5x.png?id=20dfc473acecd2fdf7051764b302283b 960w,/images/patterns/content/command/command-es-2x.png?id=bdc52551c69cb9558d3459dd025b7812 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Command" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás trabajando en una nueva aplicación de edición de
texto. Tu tarea actual consiste en crear una barra de herramientas con
unos cuantos botones para varias operaciones del editor. Creaste una
clase `Botón` muy limpia que puede utilizarse para los botones de la
barra de herramientas y también para botones genéricos en diversos
diálogos.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem163e2.png?id=84189315a0e8d91da262792979005ab4"
style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/command/problem1.png?id=84189315a0e8d91da262792979005ab4 230w,/images/patterns/diagrams/command/problem1-1.5x.png?id=77bf0bc58e5c9c9b8369bc4f8dec457f 345w,/images/patterns/diagrams/command/problem1-2x.png?id=af4c4e9c1d1b4fa2c4104c5f6bb18719 460w"
sizes="(max-width: 720px) 100vw, 230px" loading="lazy" width="230"
alt="Problema resuelto por el patrón Command" />
<figcaption><p>Todos los botones de la aplicación provienen de la
misma clase.</p></figcaption>
</figure>

Aunque todos estos botones se parecen, se supone que hacen cosas
diferentes. ¿Dónde pondrías el código para los varios gestores de clics
de estos botones? La solución más simple consiste en crear cientos de
subclases para cada lugar donde se utilice el botón. Estas subclases
contendrán el código que deberá ejecutarse con el clic en un botón.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem2f2f9.png?id=f0e33da1842b3a3ee3b4857de0b6ec93"
style="aspect-ratio: 2.11;"
srcset="/images/patterns/diagrams/command/problem2.png?id=f0e33da1842b3a3ee3b4857de0b6ec93 400w,/images/patterns/diagrams/command/problem2-1.5x.png?id=7ae15e2b07d5d076a878d99c3ed8ac32 600w,/images/patterns/diagrams/command/problem2-2x.png?id=5eea7d0f6247da011150bb63e423f405 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Muchas subclases de botón" />
<figcaption><p>Muchas subclases de botón. ¿Qué puede
salir mal?</p></figcaption>
</figure>

Pronto te das cuenta de que esta solución es muy deficiente. En primer
lugar, tienes una enorme cantidad de subclases, lo cual no supondría un
problema si no corrieras el riesgo de descomponer el código de esas
subclases cada vez que modifiques la clase base `Botón`. Dicho de forma
sencilla, tu código GUI depende torpemente del volátil código de la
lógica de negocio.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/problem3-ese0d9.png?id=60329621ad06a1a84a62328ce281ca8b"
style="aspect-ratio: 4.36;"
srcset="/images/patterns/diagrams/command/problem3-es.png?id=60329621ad06a1a84a62328ce281ca8b 480w,/images/patterns/diagrams/command/problem3-es-1.5x.png?id=bcbadbc34f2b0e9aa6ca4c8bd9385279 720w,/images/patterns/diagrams/command/problem3-es-2x.png?id=f91f74e46951797c12cc37f61eb27d46 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Varias clases implementan la misma funcionalidad" />
<figcaption><p>Varias clases implementan la
misma funcionalidad.</p></figcaption>
</figure>

Y aquí está la parte más desagradable. Algunas operaciones, como
copiar/pegar texto, deben ser invocadas desde varios lugares. Por
ejemplo, un usuario podría hacer clic en un pequeño botón "Copiar" de la
barra de herramientas, o copiar algo a través del menú contextual, o
pulsar `Ctrl+C` en el teclado.

Inicialmente, cuando tu aplicación solo tenía la barra de herramientas,
no había problema en colocar la implementación de varias operaciones
dentro de las subclases de botón. En otras palabras, tener el código
para copiar texto dentro de la subclase `BotónCopiar` estaba bien. Sin
embargo, cuando implementas menús contextuales, atajos y otros
elementos, debes duplicar el código de la operación en muchas clases, o
bien hacer menús dependientes de los botones, lo cual es una opción aún
peor.
:::

::: {.section .solution}
##  Solución {#solution}

El buen diseño de software a menudo se basa en el principio de
separación de responsabilidades, lo que suele tener como resultado la
división de la aplicación en capas. El ejemplo más habitual es tener una
capa para la interfaz gráfica de usuario (GUI) y otra capa para la
lógica de negocio. La capa GUI es responsable de representar una bonita
imagen en pantalla, capturar entradas y mostrar resultados de lo que el
usuario y la aplicación están haciendo. Sin embargo, cuando se trata de
hacer algo importante, como calcular la trayectoria de la luna o
componer un informe anual, la capa GUI delega el trabajo a la capa
subyacente de la lógica de negocio.

El código puede tener este aspecto: un objeto GUI invoca a un método de
un objeto de la lógica de negocio, pasándole algunos argumentos. Este
proceso se describe habitualmente como un objeto que envía a otro una
*solicitud*.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution1-es74de.png?id=cf13c7c5cf3fea65059b417add990515"
style="aspect-ratio: 2.47;"
srcset="/images/patterns/diagrams/command/solution1-es.png?id=cf13c7c5cf3fea65059b417add990515 470w,/images/patterns/diagrams/command/solution1-es-1.5x.png?id=8724f7dacca224fb9d2ea144cdbaa6b3 705w,/images/patterns/diagrams/command/solution1-es-2x.png?id=1c3ee25e79a26ed5f6f2a1c7fc1d2cd0 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="La capa GUI puede acceder a la capa de la lógica de negocio directamente" />
<figcaption><p>Los objetos GUI pueden acceder directamente a los objetos
de la lógica de negocio.</p></figcaption>
</figure>

El patrón Command sugiere que los objetos GUI no envíen estas
solicitudes directamente. En lugar de ello, debes extraer todos los
detalles de la solicitud, como el objeto que está siendo invocado, el
nombre del método y la lista de argumentos, y ponerlos dentro de una
clase *comando* separada con un único método que activa esta solicitud.

Los objetos de comando sirven como vínculo entre varios objetos GUI y de
lógica de negocio. De ahora en adelante, el objeto GUI no tiene que
conocer qué objeto de la lógica de negocio recibirá la solicitud y cómo
la procesará. El objeto GUI activa el comando, que gestiona todos los
detalles.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution2-es7e4f.png?id=a6ea2c1ff0d9c7d3daf9bd26b87e497b"
style="aspect-ratio: 2.89;"
srcset="/images/patterns/diagrams/command/solution2-es.png?id=a6ea2c1ff0d9c7d3daf9bd26b87e497b 550w,/images/patterns/diagrams/command/solution2-es-1.5x.png?id=1a49d8eab70ad6906941afb464f31134 825w,/images/patterns/diagrams/command/solution2-es-2x.png?id=307e12ad0a938dded9be6665a9619236 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Acceso a la capa de lógica de negocio a través de un comando." />
<figcaption><p>Acceso a la capa de lógica de negocio a través de
un comando.</p></figcaption>
</figure>

El siguiente paso es hacer que tus comandos implementen la misma
interfaz. Normalmente tiene un único método de ejecución que no acepta
parámetros. Esta interfaz te permite utilizar varios comandos con el
mismo emisor de la solicitud, sin acoplarla a clases concretas de
comandos. Adicionalmente, ahora puedes cambiar objetos de comando
vinculados al emisor, cambiando efectivamente el comportamiento del
emisor durante el tiempo de ejecución.

Puede que hayas observado que falta una pieza del rompecabezas, que son
los parámetros de la solicitud. Un objeto GUI puede haber proporcionado
al objeto de la capa de negocio algunos parámetros. Ya que el método de
ejecución del comando no tiene parámetros, ¿cómo pasaremos los detalles
de la solicitud al receptor? Resulta que el comando debe estar
preconfigurado con esta información o ser capaz de conseguirla por su
cuenta.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/solution3-esfb9f.png?id=fa9bc757cceb92fd03e9b8a49fd49a93"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/command/solution3-es.png?id=fa9bc757cceb92fd03e9b8a49fd49a93 440w,/images/patterns/diagrams/command/solution3-es-1.5x.png?id=9c8efe3022180a036fd0bb89d1b5e460 660w,/images/patterns/diagrams/command/solution3-es-2x.png?id=b398836186fcdea28e04b915b85f8695 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Los objetos GUI delegan el trabajo a los comandos" />
<figcaption><p>Los objetos GUI delegan el trabajo a
los comandos.</p></figcaption>
</figure>

Regresemos a nuestro editor de textos. Tras aplicar el patrón Command,
ya no necesitamos todas esas subclases de botón para implementar varios
comportamientos de clic. Basta con colocar un único campo dentro de la
clase base `Botón` que almacene una referencia a un objeto de comando y
haga que el botón ejecute ese comando en un clic.

Implementarás un puñado de clases de comando para toda operación posible
y las vincularás con botones particulares, dependiendo del
comportamiento pretendido de los botones.

Otros elementos GUI, como menús, atajos o diálogos enteros, se pueden
implementar del mismo modo. Se vincularán a un comando que se ejecuta
cuando un usuario interactúa con el elemento GUI. Como probablemente ya
habrás adivinado, los elementos relacionados con las mismas operaciones
se vincularán a los mismos comandos, evitando cualquier duplicación de
código.

Como resultado, los comandos se convierten en una conveniente capa
intermedia que reduce el acoplamiento entre las capas de la GUI y la
lógica de negocio. ¡Y esto es tan solo una fracción de las ventajas que
ofrece el patrón Command!
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/command/command-comic-16e6f.png?id=551df832f445080976f3116e0dc120c9"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/command/command-comic-1.png?id=551df832f445080976f3116e0dc120c9 600w,/images/patterns/content/command/command-comic-1-1.5x.png?id=82dc5c38ce372ed4dfd4a37c04faeb72 900w,/images/patterns/content/command/command-comic-1-2x.png?id=47b3c00b2cfbda7157a1ed9da6ce2948 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Realizando un pedido en un restaurante" />
<figcaption><p>Realizando un pedido en un restaurante.</p></figcaption>
</figure>

Tras un largo paseo por la ciudad, entras en un buen restaurante y te
sientas a una mesa junto a la ventana. Un amable camarero se acerca y
toma tu pedido rápidamente, apuntándolo en un papel. El camarero se va a
la cocina y pega el pedido a la pared. Al cabo de un rato, el pedido
llega al chef, que lo lee y prepara la comida. El cocinero coloca la
comida en una bandeja junto al pedido. El camarero descubre la bandeja,
comprueba el pedido para asegurarse de que todo está como lo querías, y
lo lleva todo a tu mesa.

El pedido en papel hace la función de un comando. Permanece en una cola
hasta que el chef está listo para servirlo. Este pedido contiene toda la
información relevante necesaria para preparar la comida. Permite al chef
empezar a cocinar de inmediato, en lugar de tener que correr de un lado
a otro aclarando los detalles del pedido directamente contigo.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/command/structure44ec.png?id=1cd7833638f4c43630f4a84017d31195"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.7;"
srcset="/images/patterns/diagrams/command/structure.png?id=1cd7833638f4c43630f4a84017d31195 630w,/images/patterns/diagrams/command/structure-1.5x.png?id=6e5b68277c7747b22266e408891d5841 945w,/images/patterns/diagrams/command/structure-2x.png?id=1dfaa84354ffe49ef7ad46ce069482d2 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estructura del patrón de diseño Command" /><img
src="../../images/patterns/diagrams/command/structure-indexed49b8.png?id=95529d7282dc7bc1c5bc443423b1cf4f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/command/structure-indexed.png?id=95529d7282dc7bc1c5bc443423b1cf4f 640w,/images/patterns/diagrams/command/structure-indexed-1.5x.png?id=29d6c5c7a06da2747ed756c0ddad6169 960w,/images/patterns/diagrams/command/structure-indexed-2x.png?id=e4cc286a44768c7d060af33497da7df6 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Estructura del patrón de diseño Command" />
</figure>
:::

1.  La clase **Emisora** (o *invocadora*) es responsable de inicializar
    las solicitudes. Esta clase debe tener un campo para almacenar una
    referencia a un objeto de comando. El emisor activa este comando en
    lugar de enviar la solicitud directamente al receptor. Ten en cuenta
    que el emisor no es responsable de crear el objeto de comando.
    Normalmente, obtiene un comando precreado de parte del cliente a
    través del constructor.

2.  La interfaz **Comando** normalmente declara un único método para
    ejecutar el comando.

3.  Los **Comandos Concretos** implementan varios tipos de solicitudes.
    Un comando concreto no se supone que tenga que realizar el trabajo
    por su cuenta, sino pasar la llamada a uno de los objetos de la
    lógica de negocio. Sin embargo, para lograr simplificar el código,
    estas clases se pueden fusionar.

    Los parámetros necesarios para ejecutar un método en un objeto
    receptor pueden declararse como campos en el comando concreto.
    Puedes hacer inmutables los objetos de comando permitiendo la
    inicialización de estos campos únicamente a través del constructor.

4.  La clase **Receptora** contiene cierta lógica de negocio. Casi
    cualquier objeto puede actuar como receptor. La mayoría de los
    comandos solo gestiona los detalles sobre cómo se pasa una solicitud
    al receptor, mientras que el propio receptor hace el trabajo real.

5.  El **Cliente** crea y configura los objetos de comando concretos. El
    cliente debe pasar todos los parámetros de la solicitud, incluyendo
    una instancia del receptor, dentro del constructor del comando.
    Después de eso, el comando resultante puede asociarse con uno o
    varios emisores.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Command** ayuda a rastrear el historial de
operaciones ejecutadas y hace posible revertir una operación si es
necesario.

<figure class="image">
<img
src="../../images/patterns/diagrams/command/example31bf.png?id=1f42c8395fe54d0e409026b91881e2a0"
style="aspect-ratio: 1.07;"
srcset="/images/patterns/diagrams/command/example.png?id=1f42c8395fe54d0e409026b91881e2a0 640w,/images/patterns/diagrams/command/example-1.5x.png?id=435055a78a82c005eebb1bef869998ae 960w,/images/patterns/diagrams/command/example-2x.png?id=ed44dfd9b02eccf1ae7e2714d018ed36 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Ejemplo de estructura del patrón Command" />
<figcaption><p>Operaciones que no se pueden realizar en un editor
de texto.</p></figcaption>
</figure>

Los comandos que resultan en cambiar el estado del editor (por ejemplo,
cortar y pegar) realizan una copia de seguridad del estado del editor
antes de ejecutar una operación asociada con el comando. Una vez que un
comando es ejecutado, se coloca en el historial del comando (una pila de
objetos de comando) junto a la copia de seguridad del estado del editor
en ese momento. Más tarde, si el usuario necesita revertir la operación,
la aplicación puede tomar el comando más reciente del historial, leer la
copia asociada del estado del editor, y restaurarla.

El código cliente (elementos GUI, historial de comando, etc.) no se
acopla a clases concretas de comando porque trabaja con los comandos a
través de la interfaz de comando. Esta solución te permite introducir
nuevos comandos en la aplicación sin descomponer el código existente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase base comando define la interfaz común a todos los
// comandos concretos.
abstract class Command is
    protected field app: Application
    protected field editor: Editor
    protected field backup: text

    constructor Command(app: Application, editor: Editor) is
        this.app = app
        this.editor = editor

    // Realiza una copia de seguridad del estado del editor.
    method saveBackup() is
        backup = editor.text

    // Restaura el estado del editor.
    method undo() is
        editor.text = backup

    // El método de ejecución se declara abstracto para forzar a
    // todos los comandos concretos a proporcionar sus propias
    // implementaciones. El método debe devolver verdadero o
    // falso dependiendo de si el comando cambia el estado del
    // editor.
    abstract method execute()


// Los comandos concretos van aquí.
class CopyCommand extends Command is
    // El comando copiar no se guarda en el historial ya que no
    // cambia el estado del editor.
    method execute() is
        app.clipboard = editor.getSelection()
        return false

class CutCommand extends Command is
    // El comando cortar no cambia el estado del editor, por lo
    // que debe guardarse en el historial. Y se guardará siempre
    // y cuando el método devuelva verdadero.
    method execute() is
        saveBackup()
        app.clipboard = editor.getSelection()
        editor.deleteSelection()
        return true

class PasteCommand extends Command is
    method execute() is
        saveBackup()
        editor.replaceSelection(app.clipboard)
        return true

// La operación deshacer también es un comando.
class UndoCommand extends Command is
    method execute() is
        app.undo()
        return false


// El historial global de comandos tan solo es una pila.
class CommandHistory is
    private field history: array of Command

    // El último dentro...
    method push(c: Command) is
        // Empuja el comando al final de la matriz del
        // historial.

    // ...el primero fuera.
    method pop():Command is
        // Obtiene el comando más reciente del historial.


// La clase editora tiene operaciones reales de edición de
// texto. Juega el papel de un receptor: todos los comandos
// acaban delegando la ejecución a los métodos del editor.
class Editor is
    field text: string

    method getSelection() is
        // Devuelve el texto seleccionado.

    method deleteSelection() is
        // Borra el texto seleccionado.

    method replaceSelection(text) is
        // Inserta los contenidos del portapapeles en la
        // posición actual.


// La clase Aplicación establece relaciones entre objetos. Actúa
// como un emisor: cuando algo debe hacerse, crea un objeto de
// comando y lo ejecuta.
class Application is
    field clipboard: string
    field editors: array of Editors
    field activeEditor: Editor
    field history: CommandHistory

    // El código que asigna comandos a objetos UI puede tener
    // este aspecto.
    method createUI() is
        // ...
        copy = function() { executeCommand(
            new CopyCommand(this, activeEditor)) }
        copyButton.setCommand(copy)
        shortcuts.onKeyPress(&quot;Ctrl+C&quot;, copy)

        cut = function() { executeCommand(
            new CutCommand(this, activeEditor)) }
        cutButton.setCommand(cut)
        shortcuts.onKeyPress(&quot;Ctrl+X&quot;, cut)

        paste = function() { executeCommand(
            new PasteCommand(this, activeEditor)) }
        pasteButton.setCommand(paste)
        shortcuts.onKeyPress(&quot;Ctrl+V&quot;, paste)

        undo = function() { executeCommand(
            new UndoCommand(this, activeEditor)) }
        undoButton.setCommand(undo)
        shortcuts.onKeyPress(&quot;Ctrl+Z&quot;, undo)

    // Ejecuta un comando y comprueba si debe añadirse al
    // historial.
    method executeCommand(command) is
        if (command.execute())
            history.push(command)

    // Toma el comando más reciente del historial y ejecuta su
    // método deshacer. Observa que no conocemos la clase de ese
    // comando. Pero no tenemos por qué, ya que el comando sabe
    // cómo deshacer su propia acción.
    method undo() is
        command = history.pop()
        if (command != null)
            command.undo()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Command cuando quieras parametrizar objetos con
operaciones.
:::

::: applicability-solution
El patrón Command puede convertir una llamada a un método específico en
un objeto autónomo. Este cambio abre la puerta a muchos usos
interesantes: puedes pasar comandos como argumentos de método,
almacenarlos dentro de otros objetos, cambiar comandos vinculados
durante el tiempo de ejecución, etc.

Aquí tienes un ejemplo: estás desarrollando un componente GUI, como un
menú contextual, y quieres que los usuarios puedan configurar opciones
del menú que activen operaciones cuando un usuario final haga clic sobre
ellos.
:::

::: applicability-problem
Utiliza el patrón Command cuando quieras poner operaciones en cola,
programar su ejecución, o ejecutarlas de forma remota.
:::

::: applicability-solution
Como pasa con cualquier otro objeto, un comando se pueden serializar, lo
cual implica convertirlo en una cadena que pueda escribirse fácilmente a
un archivo o una base de datos. Más tarde, la cadena puede restaurarse
como el objeto de comando inicial. De este modo, puedes retardar y
programar la ejecución del comando. ¡Pero aún hay más! Del mismo modo,
puedes poner comandos en cola, así como registrarlos o enviarlos por la
red.
:::

::: applicability-problem
Utiliza el patrón Command cuando quieras implementar operaciones
reversibles.
:::

::: applicability-solution
Aunque hay muchas formas de implementar deshacer/rehacer, el patrón
Command es quizá la más popular de todas.

Para poder revertir operaciones, debes implementar el historial de las
operaciones realizadas. El historial de comando es una pila que contiene
todos los objetos de comando ejecutados junto a copias de seguridad
relacionadas del estado de la aplicación.

Este método tiene dos desventajas. Primero, no es tan fácil guardar el
estado de una aplicación, porque parte de ella puede ser privada. Este
problema puede mitigarse con el patrón [Memento](memento.html).

Segundo, las copias de seguridad de estado pueden consumir mucha memoria
RAM. Por lo tanto, en ocasiones puedes recurrir a una implementación
alternativa: en lugar de restaurar el estado pasado, el comando realiza
la operación inversa, aunque ésta también tiene un precio, ya que puede
resultar difícil o incluso imposible de implementar.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Declara la interfaz de comando con un único método de ejecución.

2.  Empieza extrayendo solicitudes y poniéndolas dentro de clases
    concretas de comando que implementen la interfaz de comando. Cada
    clase debe contar con un grupo de campos para almacenar los
    argumentos de las solicitudes junto con referencias al objeto
    receptor. Todos estos valores deben inicializarse a través del
    constructor del comando.

3.  Identifica clases que actúen como *emisoras*. Añade los campos para
    almacenar comandos dentro de estas clases. Las emisoras deberán
    comunicarse con sus comandos tan solo a través de la interfaz de
    comando. Normalmente las emisoras no crean objetos de comando por su
    cuenta, sino que los obtienen del código cliente.

4.  Cambia las emisoras de forma que ejecuten el comando en lugar de
    enviar directamente una solicitud al receptor.

5.  El cliente debe inicializar objetos en el siguiente orden:

    -   Crear receptores.
    -   Crear comandos y asociarlos con receptores si es necesario.
    -   Crear emisores y asociarlos con comandos específicos.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de responsabilidad única*. Puedes desacoplar las clases
    que invocan operaciones de las que realizan esas operaciones.
-    *Principio de abierto/cerrado*. Puedes introducir nuevos comandos
    en la aplicación sin descomponer el código cliente existente.
-    Puedes implementar deshacer/rehacer.
-    Puedes implementar la ejecución diferida de operaciones.
-    Puedes ensamblar un grupo de comandos simples para crear uno
    complejo.
:::

::: col-sm-6
-    El código puede complicarse, ya que estás introduciendo una nueva
    capa entre emisores y receptores.
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

-   Los manejadores del [Chain of
    Responsibility](chain-of-responsibility.html) se pueden implementar
    como [Comandos](command.html). En este caso, puedes ejecutar muchas
    operaciones diferentes sobre el mismo objeto de contexto,
    representado por una solicitud.

    Sin embargo, hay otra solución en la que la propia solicitud es un
    objeto *Comando*. En este caso, puedes ejecutar la misma operación
    en una serie de contextos diferentes vinculados en una cadena.

-   Puedes utilizar [Command](command.html) y [Memento](memento.html)
    juntos cuando implementes "deshacer". En este caso, los comandos son
    responsables de realizar varias operaciones sobre un objeto destino,
    mientras que los mementos guardan el estado de ese objeto justo
    antes de que se ejecute el comando.

-   [Command](command.html) y [Strategy](strategy.html) pueden resultar
    similares porque puedes usar ambos para parametrizar un objeto con
    cierta acción. No obstante, tienen propósitos muy diferentes.

    -   Puedes utilizar *Command* para convertir cualquier operación en
        un objeto. Los parámetros de la operación se convierten en
        campos de ese objeto. La conversión te permite aplazar la
        ejecución de la operación, ponerla en cola, almacenar el
        historial de comandos, enviar comandos a servicios remotos, etc.

    -   Por su parte, *Strategy* normalmente describe distintas formas
        de hacer lo mismo, permitiéndote intercambiar estos algoritmos
        dentro de una única clase contexto.

-   [Prototype](prototype.html) puede ayudar a cuando necesitas guardar
    copias de [Comandos](command.html) en un historial.

-   Puedes tratar a [Visitor](visitor.html) como una versión potente del
    patrón [Command](command.html). Sus objetos pueden ejecutar
    operaciones sobre varios objetos de distintas clases.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Command en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](command/csharp/example.html "Command en C#"){.prog-lang-link}
[![Command en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](command/cpp/example.html "Command en C++"){.prog-lang-link}
[![Command en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](command/go/example.html "Command en Go"){.prog-lang-link}
[![Command en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](command/java/example.html "Command en Java"){.prog-lang-link}
[![Command en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](command/php/example.html "Command en PHP"){.prog-lang-link}
[![Command en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](command/python/example.html "Command en Python"){.prog-lang-link}
[![Command en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](command/ruby/example.html "Command en Ruby"){.prog-lang-link}
[![Command en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](command/rust/example.html "Command en Rust"){.prog-lang-link}
[![Command en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](command/swift/example.html "Command en Swift"){.prog-lang-link}
[![Command en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](command/typescript/example.html "Command en TypeScript"){.prog-lang-link}
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

[Iterator []{.fa .fa-arrow-right}](iterator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Chain of
Responsibility](chain-of-responsibility.html){.btn .btn-default
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
