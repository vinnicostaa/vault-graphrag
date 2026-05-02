::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Decorator {#decorator .title}

::: aka
También llamado:
[Decorador, ]{style="display:inline-block"}[Envoltorio, ]{style="display:inline-block"}[Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Decorator** es un patrón de diseño estructural que te permite añadir
funcionalidades a objetos colocando estos objetos dentro de objetos
encapsuladores especiales que contienen estas funcionalidades.

<figure class="image">
<img
src="../../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Decorator" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás trabajando en una biblioteca de notificaciones que
permite a otros programas notificar a sus usuarios acerca de eventos
importantes.

La versión inicial de la biblioteca se basaba en la clase `Notificador`
que solo contaba con unos cuantos campos, un constructor y un único
método `send`. El método podía aceptar un argumento de mensaje de un
cliente y enviar el mensaje a una lista de correos electrónicos que se
pasaban a la clase notificadora a través de su constructor. Una
aplicación de un tercero que actuaba como cliente debía crear y
configurar el objeto notificador una vez y después utilizarlo cada vez
que sucediera algo importante.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem1-es8bc5.png?id=2dac64cd411324b4da466d52c11eb149"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-es.png?id=2dac64cd411324b4da466d52c11eb149 540w,/images/patterns/diagrams/decorator/problem1-es-1.5x.png?id=6f4c8a704e6e4b8005b6543bf5a15876 810w,/images/patterns/diagrams/decorator/problem1-es-2x.png?id=5aa03215663b9baf22350d6fb4d8ea3d 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Estructura de la biblioteca antes de aplicar el patrón Decorator" />
<figcaption><p>Un programa puede utilizar la clase notificadora para
enviar notificaciones sobre eventos importantes a un grupo predefinido
de correos electrónicos.</p></figcaption>
</figure>

En cierto momento te das cuenta de que los usuarios de la biblioteca
esperan algo más que unas simples notificaciones por correo. A muchos de
ellos les gustaría recibir mensajes SMS sobre asuntos importantes. Otros
querrían recibir las notificaciones por Facebook y, por supuesto, a los
usuarios corporativos les encantaría recibir notificaciones por Slack.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Estructura de la biblioteca después de implementar otros tipos de notificaciones" />
<figcaption><p>Cada tipo de notificación se implementa como una subclase
de la clase notificadora.</p></figcaption>
</figure>

No puede ser muy complicado ¿verdad? Extendiste la clase `Notificador` y
metiste los métodos adicionales de notificación dentro de nuevas
subclases. Ahora el cliente debería instanciar la clase notificadora
deseada y utilizarla para el resto de notificaciones.

Pero entonces alguien te hace una pregunta razonable: "¿Por qué no se
pueden utilizar varios tipos de notificación al mismo tiempo? Si tu casa
está en llamas, probablemente quieras que te informen a través de todos
los canales".

Intentaste solucionar ese problema creando subclases especiales que
combinaban varios métodos de notificación dentro de una clase. Sin
embargo, enseguida resultó evidente que esta solución inflaría el código
en gran medida, no sólo el de la biblioteca, sino también el código
cliente.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estructura de la biblioteca tras crear combinaciones de clases" />
<figcaption><p>Explosión combinatoria de subclases.</p></figcaption>
</figure>

Debes encontrar alguna otra forma de estructurar las clases de las
notificaciones para no alcanzar cifras que rompan accidentalmente un
récord Guinness.
:::

::: {.section .solution}
##  Solución {#solution}

Cuando tenemos que alterar la funcionalidad de un objeto, lo primero que
se viene a la mente es extender una clase. No obstante, la herencia
tiene varias limitaciones importantes de las que debes ser consciente.

-   La herencia es estática. No se puede alterar la funcionalidad de un
    objeto existente durante el tiempo de ejecución. Sólo se puede
    sustituir el objeto completo por otro creado a partir de una
    subclase diferente.
-   Las subclases sólo pueden tener una clase padre. En la mayoría de
    lenguajes, la herencia no permite a una clase heredar
    comportamientos de varias clases al mismo tiempo.

Una de las formas de superar estas limitaciones es empleando la
*Agregación* o la *Composición* []{.pop-i}[*Agregación*: el objeto A
contiene objetos B; B puede existir sin A.\
*Composición*: el objeto A está compuesto de objetos B; A gestiona el
ciclo vital de B; B no puede existir sin A.]{.pop-content} en lugar de
la *Herencia*. Ambas alternativas funcionan prácticamente del mismo
modo: un objeto *tiene una* referencia a otro y le delega parte del
trabajo, mientras que con la herencia, el propio objeto *puede* realizar
ese trabajo, heredando el comportamiento de su superclase.

Con esta nueva solución puedes sustituir fácilmente el objeto "ayudante"
vinculado por otro, cambiando el comportamiento del contenedor durante
el tiempo de ejecución. Un objeto puede utilizar el comportamiento de
varias clases con referencias a varios objetos, delegándoles todo tipo
de tareas. La agregación/composición es el principio clave que se
esconde tras muchos patrones de diseño, incluyendo el Decorator. A
propósito, regresemos a la discusión sobre el patrón.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution1-esee24.png?id=4ff2a6eff5e33975304cb6d74ff9b682"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-es.png?id=4ff2a6eff5e33975304cb6d74ff9b682 550w,/images/patterns/diagrams/decorator/solution1-es-1.5x.png?id=9181583d3d1ac68e0eb18c400fb74df3 825w,/images/patterns/diagrams/decorator/solution1-es-2x.png?id=32193cacb5b8cdee7f2a2b33721cd01d 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Herencia vs. Agregación" />
<figcaption><p>Herencia vs. Agregación</p></figcaption>
</figure>

"Wrapper" (envoltorio, en inglés) es el sobrenombre alternativo del
patrón Decorator, que expresa claramente su idea principal. Un *wrapper*
es un objeto que puede vincularse con un objeto *objetivo*. El wrapper
contiene el mismo grupo de métodos que el objetivo y le delega todas las
solicitudes que recibe. No obstante, el wrapper puede alterar el
resultado haciendo algo antes o después de pasar la solicitud al
objetivo.

¿Cuándo se convierte un simple wrapper en el verdadero decorador? Como
he mencionado, el wrapper implementa la misma interfaz que el objeto
envuelto. Éste es el motivo por el que, desde la perspectiva del
cliente, estos objetos son idénticos. Haz que el campo de referencia del
wrapper acepte cualquier objeto que siga esa interfaz. Esto te permitirá
*envolver* un objeto en varios wrappers, añadiéndole el comportamiento
combinado de todos ellos.

En nuestro ejemplo de las notificaciones, dejemos la sencilla
funcionalidad de las notificaciones por correo electrónico dentro de la
clase base `Notificador`, pero convirtamos el resto de los métodos de
notificación en decoradores.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La solución con el patrón Decorator" />
<figcaption><p>Varios métodos de notificación se convierten
en decoradores.</p></figcaption>
</figure>

El código cliente debe envolver un objeto notificador básico dentro de
un grupo de decoradores que satisfagan las preferencias del cliente. Los
objetos resultantes se estructurarán como una pila.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/solution3-es87f8.png?id=02d50295045ffd18344d4a33a29aabd8"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-es.png?id=02d50295045ffd18344d4a33a29aabd8 300w,/images/patterns/diagrams/decorator/solution3-es-1.5x.png?id=0bc37fed995d3e394494c1c7449bad42 450w,/images/patterns/diagrams/decorator/solution3-es-2x.png?id=1058bedcc9530b2f7ad3d3cfd3deee24 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Las aplicaciones pueden configurar pilas complejas de decoradores de notificación" />
<figcaption><p>Las aplicaciones pueden configurar pilas complejas de
decoradores de notificación.</p></figcaption>
</figure>

El último decorador de la pila será el objeto con el que el cliente
trabaja. Debido a que todos los decoradores implementan la misma
interfaz que la notificadora base, al resto del código cliente no le
importa si está trabajando con el objeto notificador "puro" o con el
decorado.

Podemos aplicar la misma solución a otras funcionalidades, como el
formateo de mensajes o la composición de una lista de destinatarios. El
cliente puede decorar el objeto con los decoradores personalizados que
desee, siempre y cuando sigan la misma interfaz que los demás.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Ejemplo del patrón Decorator" />
<figcaption><p>Obtienes un efecto combinado vistiendo varias prendas
de ropa.</p></figcaption>
</figure>

Vestir ropa es un ejemplo del uso de decoradores. Cuando tienes frío, te
cubres con un suéter. Si sigues teniendo frío a pesar del suéter, puedes
ponerte una chaqueta encima. Si está lloviendo, puedes ponerte un
impermeable. Todas estas prendas "extienden" tu comportamiento básico
pero no son parte de ti, y puedes quitarte fácilmente cualquier prenda
cuando lo desees.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Estructura del patrón de diseño Decorator" /><img
src="../../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Estructura del patrón de diseño Decorator" />
</figure>
:::

1.  El **Componente** declara la interfaz común tanto para wrappers como
    para objetos envueltos.

2.  **Componente Concreto** es una clase de objetos envueltos. Define el
    comportamiento básico, que los decoradores pueden alterar.

3.  La clase **Decoradora Base** tiene un campo para referenciar un
    objeto envuelto. El tipo del campo debe declararse como la interfaz
    del componente para que pueda contener tanto los componentes
    concretos como los decoradores. La clase decoradora base delega
    todas las operaciones al objeto envuelto.

4.  Los **Decoradores Concretos** definen funcionalidades adicionales
    que se pueden añadir dinámicamente a los componentes. Los
    decoradores concretos sobrescriben métodos de la clase decoradora
    base y ejecutan su comportamiento, ya sea antes o después de invocar
    al método padre.

5.  El **Cliente** puede envolver componentes en varias capas de
    decoradores, siempre y cuando trabajen con todos los objetos a
    través de la interfaz del componente.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Decorator** te permite comprimir y
encriptar información delicada independientemente del código que utiliza
esos datos.

<figure class="image">
<img
src="../../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Ejemplo de estructura del patrón Decorator" />
<figcaption><p>Ejemplo de la encriptación y compresión
de decoradores.</p></figcaption>
</figure>

La aplicación envuelve el objeto de la fuente de datos con un par de
decoradores. Ambos wrappers cambian el modo en que los datos se escriben
y se leen en el disco:

-   Justo antes de que los datos se **escriban en el disco**, los
    decoradores los encriptan y comprimen. La clase original escribe en
    el archivo los datos encriptados y protegidos, sin conocer el
    cambio.

-   Después de que los datos son **leídos del disco**, pasan por los
    mismos decoradores, que los descomprimen y decodifican.

Los decoradores y la clase fuente de datos implementan la misma
interfaz, lo que los hace intercambiables en el código cliente.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz de componente define operaciones que los
// decoradores pueden alterar.
interface DataSource is
    method writeData(data)
    method readData():data

// Los componentes concretos proporcionan implementaciones por
// defecto para las operaciones. En un programa puede haber
// muchas variaciones de estas clases.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Escribe datos en el archivo.

    method readData():data is
        // Lee datos del archivo.

// La clase decoradora base sigue la misma interfaz que los
// demás componentes. El principal propósito de esta clase es
// definir la interfaz de encapsulación para todos los
// decoradores concretos. La implementación por defecto del
// código de encapsulación puede incluir un campo para almacenar
// un componente envuelto y los medios para inicializarlo.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // La decoradora base simplemente delega todo el trabajo al
    // componente envuelto. En los decoradores concretos se
    // pueden añadir comportamientos adicionales.
    method writeData(data) is
        wrappee.writeData(data)

    // Los decoradores concretos pueden invocar la
    // implementación padre de la operación en lugar de invocar
    // directamente al objeto envuelto. Esta solución simplifica
    // la extensión de las clases decoradoras.
    method readData():data is
        return wrappee.readData()

// Los decoradores concretos deben invocar métodos en el objeto
// envuelto, pero pueden añadir algo de su parte al resultado.
// Los decoradores pueden ejecutar el comportamiento añadido
// antes o después de la llamada a un objeto envuelto.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Encripta los datos pasados.
        // 2. Pasa los datos encriptados al método writeData
        // (escribirDatos) del objeto envuelto.

    method readData():data is
        // 1. Obtiene datos del método readData (leerDatos) del
        // objeto envuelto.
        // 2. Intenta descifrarlo si está encriptado.
        // 3. Devuelve el resultado.

// Puedes envolver objetos en varias capas de decoradores.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Comprime los datos pasados.
        // 2. Pasa los datos comprimidos al método writeData del
        // objeto envuelto.

    method readData():data is
        // 1. Obtiene datos del método readData del objeto
        // envuelto.
        // 2. Intenta descomprimirlo si está comprimido.
        // 3. Devuelve el resultado.


// Opción 1. Un ejemplo sencillo del montaje de un decorador.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // El archivo objetivo se ha escrito con datos sin
        // formato.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // El archivo objetivo se ha escrito con datos
        // comprimidos.

        source = new EncryptionDecorator(source)
        // La variable fuente ahora contiene esto:
        // Cifrado &gt; Compresión &gt; FileDataSource
        source.writeData(salaryRecords)
        // El archivo se ha escrito con datos comprimidos y
        // encriptados.


// Opción 2. El código cliente que utiliza una fuente externa de
// datos. Los objetos SalaryManager no conocen ni se preocupan
// por las especificaciones del almacenamiento de datos.
// Trabajan con una fuente de datos preconfigurada recibida del
// configurador de la aplicación.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Otros métodos útiles...


// La aplicación puede montar distintas pilas de decoradores
// durante el tiempo de ejecución, dependiendo de la
// configuración o el entorno.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Decorator cuando necesites asignar funcionalidades
adicionales a objetos durante el tiempo de ejecución sin descomponer el
código que utiliza esos objetos.
:::

::: applicability-solution
El patrón Decorator te permite estructurar tu lógica de negocio en
capas, crear un decorador para cada capa y componer objetos con varias
combinaciones de esta lógica, durante el tiempo de ejecución. El código
cliente puede tratar a todos estos objetos de la misma forma, ya que
todos siguen una interfaz común.
:::

::: applicability-problem
Utiliza el patrón cuando resulte extraño o no sea posible extender el
comportamiento de un objeto utilizando la herencia.
:::

::: applicability-solution
Muchos lenguajes de programación cuentan con la palabra clave `final`
que puede utilizarse para evitar que una clase siga extendiéndose. Para
una clase final, la única forma de reutilizar el comportamiento
existente será envolver la clase con tu propio wrapper, utilizando el
patrón Decorator.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Asegúrate de que tu dominio de negocio puede representarse como un
    componente primario con varias capas opcionales encima.

2.  Decide qué métodos son comunes al componente primario y las capas
    opcionales. Crea una interfaz de componente y declara esos métodos
    en ella.

3.  Crea una clase concreta de componente y define en ella el
    comportamiento base.

4.  Crea una clase base decoradora. Debe tener un campo para almacenar
    una referencia a un objeto envuelto. El campo debe declararse con el
    tipo de interfaz de componente para permitir la vinculación a
    componentes concretos, así como a decoradores. La clase decoradora
    base debe delegar todas las operaciones al objeto envuelto.

5.  Asegúrate de que todas las clases implementan la interfaz de
    componente.

6.  Crea decoradores concretos extendiéndolos a partir de la decoradora
    base. Un decorador concreto debe ejecutar su comportamiento antes o
    después de la llamada al método padre (que siempre delega al objeto
    envuelto).

7.  El código cliente debe ser responsable de crear decoradores y
    componerlos del modo que el cliente necesite.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes extender el comportamiento de un objeto sin crear una nueva
    subclase.
-    Puedes añadir o eliminar responsabilidades de un objeto durante el
    tiempo de ejecución.
-    Puedes combinar varios comportamientos envolviendo un objeto con
    varios decoradores.
-    *Principio de responsabilidad única*. Puedes dividir una clase
    monolítica que implementa muchas variantes posibles de
    comportamiento, en varias clases más pequeñas.
:::

::: col-sm-6
-    Resulta difícil eliminar un wrapper específico de la pila de
    wrappers.
-    Es difícil implementar un decorador de tal forma que su
    comportamiento no dependa del orden en la pila de decoradores.
-    El código de configuración inicial de las capas pueden tener un
    aspecto desagradable.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Adapter](adapter.html) proporciona una interfaz completamente
    diferente para acceder a un objeto existente. Por otro lado, con el
    patrón [Decorator](decorator.html) la interfaz permanece igual o se
    amplía. Además, *Decorator* admite la composición recursiva, que no
    es posible cuando se utiliza *Adapter*.

-   Con [Adapter](adapter.html) se accede a un objeto existente a través
    de una interfaz diferente. Con [Proxy](proxy.html), la interfaz
    sigue siendo la misma. Con [Decorator](decorator.html) se accede al
    objeto a través de una interfaz mejorada.

-   [Chain of Responsibility](chain-of-responsibility.html) y
    [Decorator](decorator.html) tienen estructuras de clase muy
    similares. Ambos patrones se basan en la composición recursiva para
    pasar la ejecución a través de una serie de objetos. Sin embargo,
    existen varias diferencias fundamentales:

    Los manejadores de *CoR* pueden ejecutar operaciones arbitrarias con
    independencia entre sí. También pueden dejar de pasar la solicitud
    en cualquier momento. Por otro lado, varios *decoradores* pueden
    extender el comportamiento del objeto manteniendo su consistencia
    con la interfaz base. Además, los decoradores no pueden romper el
    flujo de la solicitud.

-   [Composite](composite.html) y [Decorator](decorator.html) tienen
    diagramas de estructura similares ya que ambos se basan en la
    composición recursiva para organizar un número indefinido de
    objetos.

    Un *Decorator* es como un *Composite* pero sólo tiene un componente
    hijo. Hay otra diferencia importante: *Decorator* añade
    responsabilidades adicionales al objeto envuelto, mientras que
    *Composite* se limita a "recapitular" los resultados de sus hijos.

    No obstante, los patrones también pueden colaborar: puedes utilizar
    el *Decorator* para extender el comportamiento de un objeto
    específico del árbol *Composite*.

-   Los diseños que hacen un uso amplio de [Composite](composite.html) y
    [Decorator](decorator.html) a menudo pueden beneficiarse del uso del
    [Prototype](prototype.html). Aplicar el patrón te permite clonar
    estructuras complejas en lugar de reconstruirlas desde cero.

-   [Decorator](decorator.html) te permite cambiar la piel de un objeto,
    mientras que [Strategy](strategy.html) te permite cambiar sus
    entrañas.

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

[![Decorator en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Decorator en C#"){.prog-lang-link}
[![Decorator en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Decorator en C++"){.prog-lang-link}
[![Decorator en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Decorator en Go"){.prog-lang-link}
[![Decorator en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Decorator en Java"){.prog-lang-link}
[![Decorator en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Decorator en PHP"){.prog-lang-link}
[![Decorator en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Decorator en Python"){.prog-lang-link}
[![Decorator en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Decorator en Ruby"){.prog-lang-link}
[![Decorator en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Decorator en Rust"){.prog-lang-link}
[![Decorator en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Decorator en Swift"){.prog-lang-link}
[![Decorator en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Decorator en TypeScript"){.prog-lang-link}
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

[Facade []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Composite](composite.html){.btn .btn-default
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
