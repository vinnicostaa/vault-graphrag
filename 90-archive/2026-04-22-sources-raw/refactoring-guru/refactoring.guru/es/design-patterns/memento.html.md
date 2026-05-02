::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Memento {#memento .title}

::: aka
También llamado:
[Recuerdo, ]{style="display:inline-block"}[Instantánea, ]{style="display:inline-block"}[Snapshot]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Memento** es un patrón de diseño de comportamiento que te permite
guardar y restaurar el estado previo de un objeto sin revelar los
detalles de su implementación.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-es6165.png?id=425d7fafd404116e99e93c3d8a04ec89"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-es.png?id=425d7fafd404116e99e93c3d8a04ec89 640w,/images/patterns/content/memento/memento-es-1.5x.png?id=6353c15d1fdddf98ac2d2e21c826356f 960w,/images/patterns/content/memento/memento-es-2x.png?id=f8e6f144581c43f7c27fbe03d86f7cc9 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Memento" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás creando una aplicación de edición de texto. Además de
editar texto, tu programa puede formatearlo, asi como insertar imágenes
en línea, etc.

En cierto momento, decides permitir a los usuarios deshacer cualquier
operación realizada en el texto. Esta función se ha vuelto tan habitual
en los últimos años que hoy en día todo el mundo espera que todas las
aplicaciones la tengan. Para la implementación eliges la solución
directa. Antes de realizar cualquier operación, la aplicación registra
el estado de todos los objetos y lo guarda en un almacenamiento. Más
tarde, cuando un usuario decide revertir una acción, la aplicación
extrae la última *instantánea* del historial y la utiliza para restaurar
el estado de todos los objetos.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-es9c64.png?id=a68a8b5d7acbd2851ac4a2475f7705c2"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-es.png?id=a68a8b5d7acbd2851ac4a2475f7705c2 470w,/images/patterns/diagrams/memento/problem1-es-1.5x.png?id=227112d1efa8d3346b8108cf113f9329 705w,/images/patterns/diagrams/memento/problem1-es-2x.png?id=398af7adf9c1a0b046157a3a9d3c1eec 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Revertir operaciones en el editor" />
<figcaption><p>Antes de ejecutar una operación, la aplicación guarda una
instantánea del estado de los objetos, que más tarde se puede utilizar
para restaurar objetos a su estado previo.</p></figcaption>
</figure>

Pensemos en estas instantáneas de estado. ¿Cómo producirías una,
exactamente? Probablemente tengas que recorrer todos los campos de un
objeto y copiar sus valores en el almacenamiento. Sin embargo, esto sólo
funcionará si el objeto tiene unas restricciones bastante laxas al
acceso a sus contenidos. Lamentablemente, la mayoría de objetos reales
no permite a otros asomarse a su interior fácilmente, y esconden todos
los datos significativos en campos privados.

Ignora ese problema por ahora y asumamos que nuestros objetos se
comportan como hippies: prefieren relaciones abiertas y mantienen su
estado público. Aunque esta solución resolvería el problema inmediato y
te permitiría producir instantáneas de estados de objetos a voluntad,
sigue teniendo algunos inconvenientes serios. En el futuro, puede que
decidas refactorizar algunas de las clases editoras, o añadir o eliminar
algunos de los campos. Parece fácil, pero esto también exige cambiar las
clases responsables de copiar el estado de los objetos afectados.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-es5949.png?id=5a709b06d4a59000713828bba37b8231"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-es.png?id=5a709b06d4a59000713828bba37b8231 300w,/images/patterns/diagrams/memento/problem2-es-1.5x.png?id=7af1df2e1217cc8d6d69f0ba56df69d5 450w,/images/patterns/diagrams/memento/problem2-es-2x.png?id=c57b571687fea34a556b3c8a9a161a1a 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="¿Cómo hacer una copia del estado privado del objeto?" />
<figcaption><p>¿Cómo hacer una copia del estado privado
del objeto?</p></figcaption>
</figure>

Pero aún hay más. Pensemos en las propias "instantáneas" del estado del
editor. ¿Qué datos contienen? Como mínimo, deben contener el texto, las
coordenadas del cursor, la posición actual de desplazamiento, etc. Para
realizar una instantánea debes recopilar estos valores y meterlos en
algún tipo de contenedor.

Probablemente almacenarás muchos de estos objetos de contenedor dentro
de una lista que represente el historial. Por lo tanto, probablemente
los contenedores acaben siendo objetos de una clase. La clase no tendrá
apenas métodos, pero sí muchos campos que reflejen el estado del editor.
Para permitir que otros objetos escriban y lean datos a y desde una
instantánea, es probable que tengas que hacer sus campos públicos. Esto
expondrá todos los estados del editor, privados o no. Otras clases se
volverán dependientes de cada pequeño cambio en la clase de la
instantánea, que de otra forma ocurriría dentro de campos y métodos
privados sin afectar a clases externas.

Parece que hemos llegado a un callejón sin salida: o bien expones todos
los detalles internos de las clases, haciéndolas demasiado frágiles, o
restringes el acceso a su estado, haciendo imposible producir
instantáneas. ¿Hay alguna otra forma de implementar el \"deshacer\"?
:::

::: {.section .solution}
##  Solución {#solution}

Todos los problemas que hemos experimentado han sido provocados por una
encapsulación fragmentada. Algunos objetos intentan hacer más de lo que
deben. Para recopilar los datos necesarios para realizar una acción,
invaden el espacio privado de otros objetos en lugar de permitir a esos
objetos realizar la propia acción.

El patrón Memento delega la creación de instantáneas de estado al
propietario de ese estado, el objeto *originador*. Por lo tanto, en
lugar de que haya otros objetos intentando copiar el estado del editor
desde el "exterior", la propia clase editora puede hacer la instantánea,
ya que tiene pleno acceso a su propio estado.

El patrón sugiere almacenar la copia del estado del objeto en un objeto
especial llamado *memento*. Los contenidos del memento no son accesibles
para ningún otro objeto excepto el que lo produjo. Otros objetos deben
comunicarse con mementos utilizando una interfaz limitada que pueda
permitir extraer los metadatos de la instantánea (tiempo de creación, el
nombre de la operación realizada, etc.), pero no el estado del objeto
original contenido en la instantánea.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-es70a1.png?id=67b003877de5f9de9b2c73452139fdda"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-es.png?id=67b003877de5f9de9b2c73452139fdda 610w,/images/patterns/diagrams/memento/solution-es-1.5x.png?id=4bc1a5dc672010d99e1507772621b3b8 915w,/images/patterns/diagrams/memento/solution-es-2x.png?id=535b0464f3d49c0586fa1792588791b6 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="El originador tiene pleno acceso al memento, mientras que el cuidador sólo puede acceder a los metadatos" />
<figcaption><p>El originador tiene pleno acceso al memento, mientras que
el cuidador sólo puede acceder a los metadatos.</p></figcaption>
</figure>

Una política tan restrictiva te permite almacenar mementos dentro de
otros objetos, normalmente llamados *cuidadores*. Debido a que el
cuidador trabaja con el memento únicamente a través de la interfaz
limitada, no puede manipular el estado almacenado dentro del memento. Al
mismo tiempo, el originador tiene acceso a todos los campos dentro del
memento, permitiéndole restaurar su estado previo a voluntad.

En nuestro ejemplo del editor de texto, podemos crear una clase separada
de historial que actúe como cuidadora. Una pila de mementos almacenados
dentro de la cuidadora crecerá cada vez que el editor vaya a ejecutar
una operación. Puedes incluso presentar esta pila dentro de la UI de la
aplicación, mostrando a un usuario el historial de operaciones
previamente realizadas.

Cuando un usuario activa la función Deshacer, el historial toma el
memento más reciente de la pila y lo pasa de vuelta al editor,
solicitando una restauración. Debido a que el editor tiene pleno acceso
al memento, cambia su propio estado con los valores tomados del memento.
:::

:::::::::: {.section .structure-container}
##  Estructura {#structure}

::::::::: structure
#### Implementación basada en clases anidadas

La implementación clásica del patrón se basa en el soporte de clases
anidadas, disponible en varios lenguajes de programación populares (como
C++, C# y Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento basado en clases anidadas" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento basado en clases anidadas" />
</figure>
:::
::::

1.  La clase **Originadora** puede producir instantáneas de su propio
    estado, así como restaurar su estado a partir de instantáneas cuando
    lo necesita.

2.  El **Memento** es un objeto de valor que actúa como instantánea del
    estado del originador. Es práctica común hacer el memento inmutable
    y pasarle los datos solo una vez, a través del constructor.

3.  La **Cuidadora** sabe no solo "cuándo" y "por qué" capturar el
    estado de la originadora, sino también cuándo debe restaurarse el
    estado.

    Una cuidadora puede rastrear el historial de la originadora
    almacenando una pila de mementos. Cuando la originadora deba
    retroceder en el historial, la cuidadora extraerá el memento de más
    arriba de la pila y lo pasará al método de restauración de la
    originadora.

4.  En esta implementación, la clase memento se anida dentro de la
    originadora. Esto permite a la originadora acceder a los campos y
    métodos de la clase memento, aunque se declaren privados. Por otro
    lado, la cuidadora tiene un acceso muy limitado a los campos y
    métodos de la clase memento, lo que le permite almacenar mementos en
    una pila pero no alterar su estado.

#### Implementación basada en una interfaz intermedia

Existe una implementación alternativa, adecuada para lenguajes de
programación que no soportan clases anidadas (sí, PHP, estoy hablando de
ti).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento sin clases anidadas" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento sin clases anidadas" />
</figure>
:::
::::

1.  En ausencia de clases anidadas, puedes restringir el acceso a los
    campos de la clase memento estableciendo una convención de que las
    cuidadoras sólo pueden trabajar con una memento a través de una
    interfaz intermediaria explícitamente declarada, que sólo declarará
    métodos relacionados con los metadatos del memento.

2.  Por otro lado, las originadoras pueden trabajar directamente con un
    objeto memento, accediendo a campos y métodos declarados en la clase
    memento. El inconveniente de esta solución es que debes declarar
    públicos todos los miembros de la clase memento.

#### Implementación con una encapsulación más estricta

Existe otra implementación que resulta útil cuando no queremos dejar la
más mínima opción a que otras clases accedan al estado de la originadora
a través del memento.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento con encapsulación estricta" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento con encapsulación estricta" />
</figure>
:::
::::

1.  Esta implementación permite tener varios tipos de originadoras y
    mementos. Cada originadora trabaja con una clase memento
    correspondiente. Ninguna de las dos expone su estado a nadie.

2.  Las cuidadoras tienen ahora explícitamente restringido cambiar el
    estado almacenado en los mementos. Además, la clase cuidadora se
    vuelve independiente de la originadora porque el método de
    restauración se define ahora en la clase memento.

3.  Cada memento queda vinculado a la originadora que lo produce. La
    originadora se pasa al constructor del memento, junto con los
    valores de su estado. Gracias a la estrecha relación entre estas
    clases, un memento puede restaurar el estado de su originadora,
    siempre que esta última haya definido los modificadores (setters)
    adecuados.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo utiliza el patrón Memento junto al patrón
[Command](command.html) para almacenar instantáneas del estado complejo
del editor de texto y restaurar un estado previo a partir de estas
instantáneas cuando sea necesario.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Ejemplo de estructura del Memento" />
<figcaption><p>Guardar instantáneas del estado del editor
de texto.</p></figcaption>
</figure>

Los objetos de comando actúan como cuidadores. Buscan el memento del
editor antes de ejecutar operaciones relacionadas con los comandos.
Cuando un usuario intenta deshacer el comando más reciente, el editor
puede utilizar el memento almacenado en ese comando para revertirse a sí
mismo al estado previo.

La clase memento no declara ningún campo, consultor (getter) o
modificador (setter) como público. Por lo tanto, ningún objeto puede
alterar sus contenidos. Los mementos se vinculan al objeto del editor
que los creó. Esto permite a un memento restaurar el estado del editor
vinculado pasando los datos a través de modificadores en el objeto
editor. Ya que los mementos están vinculados a objetos de editor
específicos, puedes hacer que tu aplicación soporte varias ventanas de
editor independientes con una pila centralizada para deshacer.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// El originador contiene información importante que puede
// cambiar con el paso del tiempo. También define un método para
// guardar su estado dentro de un memento, y otro método para
// restaurar el estado a partir de él.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Guarda el estado actual dentro de un memento.
    method createSnapshot():Snapshot is
        // El memento es un objeto inmutable; ese es el motivo
        // por el que el originador pasa su estado a los
        // parámetros de su constructor.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// La clase memento almacena el estado pasado del editor.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // En cierto punto, puede restaurarse un estado previo del
    // editor utilizando un objeto memento.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Un objeto de comando puede actuar como cuidador. En este
// caso, el comando obtiene un memento justo antes de cambiar el
// estado del originador. Cuando se solicita deshacer, restaura
// el estado del originador a partir del memento.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Memento cuando quieras producir instantáneas del
estado del objeto para poder restaurar un estado previo del objeto.
:::

::: applicability-solution
El patrón Memento te permite realizar copias completas del estado de un
objeto, incluyendo campos privados, y almacenarlos independientemente
del objeto. Aunque la mayoría de la gente recuerda este patrón gracias
al caso de la función Deshacer, también es indispensable a la hora de
tratar con transacciones (por ejemplo, si debes volver atrás sobre un
error en una operación).
:::

::: applicability-problem
Utiliza el patrón cuando el acceso directo a los campos, consultores o
modificadores del objeto viole su encapsulación.
:::

::: applicability-solution
El Memento hace al propio objeto responsable de la creación de una
instantánea de su estado. Ningún otro objeto puede leer la instantánea,
lo que hace que los datos del estado del objeto original queden seguros.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Determina qué clase jugará el papel de la originadora. Es importante
    saber si el programa utiliza un objeto central de este tipo o varios
    más pequeños.

2.  Crea la clase memento. Uno a uno, declara un grupo de campos que
    reflejen los campos declarados dentro de la clase originadora.

3.  Haz la clase memento inmutable. Una clase memento debe aceptar los
    datos sólo una vez, a través del constructor. La clase no debe tener
    modificadores.

4.  Si tu lenguaje de programación soporta clases anidadas, anida la
    clase memento dentro de la originadora. Si no es así, extrae una
    interfaz en blanco de la clase memento y haz que el resto de objetos
    la utilicen para remitirse a ella. Puedes añadir operaciones de
    metadatos a la interfaz, pero nada que exponga el estado de la
    originadora.

5.  Añade un método para producir mementos a la clase originadora. La
    originadora debe pasar su estado a la clase memento a través de uno
    o varios argumentos del constructor del memento.

    El tipo de retorno del método debe ser del mismo que la interfaz que
    extrajiste en el paso anterior (asumiendo que lo hiciste).
    Básicamente, el método productor del memento debe trabajar
    directamente con la clase memento.

6.  Añade un método para restaurar el estado del originador a su clase.
    Debe aceptar un objeto memento como argumento. Si extrajiste una
    interfaz en el paso previo, haz que sea el tipo del parámetro. En
    este caso, debes especificar el tipo del objeto entrante al de la
    clase memento, ya que la originadora necesita pleno acceso a ese
    objeto.

7.  La cuidadora, independientemente de que represente un objeto de
    comando, un historial, o algo totalmente diferente, debe saber
    cuándo solicitar nuevos mementos de la originadora, cómo
    almacenarlos y cuándo restaurar la originadora con un memento
    particular.

8.  El vínculo entre cuidadoras y originadoras puede moverse dentro de
    la clase memento. En este caso, cada memento debe conectarse a la
    originadora que lo creó. El método de restauración también se moverá
    a la clase memento. No obstante, todo esto sólo tendrá sentido si la
    clase memento está anidada dentro de la originadora o la clase
    originadora proporciona suficientes modificadores para sobrescribir
    su estado.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes producir instantáneas del estado del objeto sin violar su
    encapsulación.
-    Puedes simplificar el código de la originadora permitiendo que la
    cuidadora mantenga el historial del estado de la originadora.
:::

::: col-sm-6
-    La aplicación puede consumir mucha memoria RAM si los clientes
    crean mementos muy a menudo.
-    Las cuidadoras deben rastrear el ciclo de vida de la originadora
    para poder destruir mementos obsoletos.
-    La mayoría de los lenguajes de programación dinámicos, como PHP,
    Python y JavaScript, no pueden garantizar que el estado dentro del
    memento se mantenga intacto.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Puedes utilizar [Command](command.html) y [Memento](memento.html)
    juntos cuando implementes "deshacer". En este caso, los comandos son
    responsables de realizar varias operaciones sobre un objeto destino,
    mientras que los mementos guardan el estado de ese objeto justo
    antes de que se ejecute el comando.

-   Puedes usar [Memento](memento.html) junto con
    [Iterator](iterator.html) para capturar el estado de la iteración
    actual y reanudarla si fuera necesario.

-   En ocasiones, [Prototype](prototype.html) puede ser una alternativa
    más simple al patrón [Memento](memento.html). Esto funciona si el
    objeto cuyo estado quieres almacenar en el historial es
    suficientemente sencillo y no tiene enlaces a recursos externos, o
    estos son fáciles de restablecer.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Memento en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Memento en C#"){.prog-lang-link}
[![Memento en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Memento en C++"){.prog-lang-link}
[![Memento en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Memento en Go"){.prog-lang-link}
[![Memento en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Memento en Java"){.prog-lang-link}
[![Memento en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Memento en PHP"){.prog-lang-link}
[![Memento en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Memento en Python"){.prog-lang-link}
[![Memento en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Memento en Ruby"){.prog-lang-link}
[![Memento en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Memento en Rust"){.prog-lang-link}
[![Memento en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Memento en Swift"){.prog-lang-link}
[![Memento en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Memento en TypeScript"){.prog-lang-link}
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

[Observer []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Mediator](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
