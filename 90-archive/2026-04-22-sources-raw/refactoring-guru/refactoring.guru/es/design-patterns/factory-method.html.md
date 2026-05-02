::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Factory Method {#factory-method .title}

::: aka
También llamado: [Método
fábrica, ]{style="display:inline-block"}[Constructor
virtual]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Factory Method** es un patrón de diseño creacional que proporciona una
interfaz para crear objetos en una superclase, mientras permite a las
subclases alterar el tipo de objetos que se crearán.

<figure class="image">
<img
src="../../images/patterns/content/factory-method/factory-method-es2ab1.png?id=4040e2911292e5a623f10e36c380459d"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-es.png?id=4040e2911292e5a623f10e36c380459d 640w,/images/patterns/content/factory-method/factory-method-es-1.5x.png?id=a7e0ef817c21485d237946fe68934e78 960w,/images/patterns/content/factory-method/factory-method-es-2x.png?id=4d995e1b565cbd4bc4a0647274bcdceb 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón Factory Method" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás creando una aplicación de gestión logística. La
primera versión de tu aplicación sólo es capaz de manejar el transporte
en camión, por lo que la mayor parte de tu código se encuentra dentro de
la clase `Camión`.

Al cabo de un tiempo, tu aplicación se vuelve bastante popular. Cada día
recibes decenas de peticiones de empresas de transporte marítimo para
que incorpores la logística por mar a la aplicación.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/problem1-es3df1.png?id=9eb28d90ae098ed7a8a1e7b81d2a9d2c"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-es.png?id=9eb28d90ae098ed7a8a1e7b81d2a9d2c 600w,/images/patterns/diagrams/factory-method/problem1-es-1.5x.png?id=ec9f887b2cc844ac7172c9be8c4b6df6 900w,/images/patterns/diagrams/factory-method/problem1-es-2x.png?id=43a57d1fe740ed44ac77a879fe778236 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Añadir una nueva clase de transporte al programa provoca un problema" />
<figcaption><p>Añadir una nueva clase al programa no es tan sencillo si
el resto del código ya está acoplado a
clases existentes.</p></figcaption>
</figure>

Estupendo, ¿verdad? Pero, ¿qué pasa con el código? En este momento, la
mayor parte de tu código está acoplado a la clase `Camión`. Para añadir
barcos a la aplicación habría que hacer cambios en toda la base del
código. Además, si más tarde decides añadir otro tipo de transporte a la
aplicación, probablemente tendrás que volver a hacer todos estos
cambios.

Al final acabarás con un código bastante sucio, plagado de condicionales
que cambian el comportamiento de la aplicación dependiendo de la clase
de los objetos de transporte.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Factory Method sugiere que, en lugar de llamar al operador
`new` para construir objetos directamente, se invoque a un método
*fábrica* especial. No te preocupes: los objetos se siguen creando a
través del operador `new`, pero se invocan desde el método fábrica. Los
objetos devueltos por el método fábrica a menudo se denominan
*productos*.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="La estructura de las clases creadoras" />
<figcaption><p>Las subclases pueden alterar la clase de los objetos
devueltos por el método fábrica.</p></figcaption>
</figure>

A simple vista, puede parecer que este cambio no tiene sentido, ya que
tan solo hemos cambiado el lugar desde donde invocamos al constructor.
Sin embargo, piensa en esto: ahora puedes sobrescribir el método fábrica
en una subclase y cambiar la clase de los productos creados por el
método.

No obstante, hay una pequeña limitación: las subclases sólo pueden
devolver productos de distintos tipos si dichos productos tienen una
clase base o interfaz común. Además, el método fábrica en la clase base
debe tener su tipo de retorno declarado como dicha interfaz.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution2-es39df.png?id=a8ba5847ee9e34e57c6090afce86673a"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-es.png?id=a8ba5847ee9e34e57c6090afce86673a 490w,/images/patterns/diagrams/factory-method/solution2-es-1.5x.png?id=3aecd9e2062a7aaca2e1de0b04faf858 735w,/images/patterns/diagrams/factory-method/solution2-es-2x.png?id=6b9634803fb095534f3240b7a7f08b8c 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="La estructura de la jerarquía de productos" />
<figcaption><p>Todos los productos deben seguir la
misma interfaz.</p></figcaption>
</figure>

Por ejemplo, tanto la clase `Camión` como la clase `Barco` deben
implementar la interfaz `Transporte`, que declara un método llamado
`entrega`. Cada clase implementa este método de forma diferente: los
camiones entregan su carga por tierra, mientras que los barcos lo hacen
por mar. El método fábrica dentro de la clase `LogísticaTerrestre`
devuelve objetos de tipo camión, mientras que el método fábrica de la
clase `LogísticaMarítima` devuelve barcos.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/solution3-esef25.png?id=5bdaeaf51cf95732da54938184ac5a9e"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-es.png?id=5bdaeaf51cf95732da54938184ac5a9e 640w,/images/patterns/diagrams/factory-method/solution3-es-1.5x.png?id=4b1c8c2bbd6e7b1c89c4c376357f8e44 960w,/images/patterns/diagrams/factory-method/solution3-es-2x.png?id=4bd2eb285b3906b813bfa93ab10cf6aa 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="La estructura del código tras aplicar el patrón Factory Method" />
<figcaption><p>Siempre y cuando todas las clases de producto implementen
una interfaz común, podrás pasar sus objetos al código cliente
sin descomponerlo.</p></figcaption>
</figure>

El código que utiliza el método fábrica (a menudo denominado código
*cliente*) no encuentra diferencias entre los productos devueltos por
varias subclases, y trata a todos los productos como la clase abstracta
`Transporte`. El cliente sabe que todos los objetos de transporte deben
tener el método `entrega`, pero no necesita saber cómo funciona
exactamente.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="La estructura del patrón Factory Method" /><img
src="../../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="La estructura del patrón Factory Method" />
</figure>
:::

1.  El **Producto** declara la interfaz, que es común a todos los
    objetos que puede producir la clase creadora y sus subclases.

2.  Los **Productos Concretos** son distintas implementaciones de la
    interfaz de producto.

3.  La clase **Creadora** declara el método fábrica que devuelve nuevos
    objetos de producto. Es importante que el tipo de retorno de este
    método coincida con la interfaz de producto.

    Puedes declarar el patrón Factory Method como abstracto para forzar
    a todas las subclases a implementar sus propias versiones del
    método. Como alternativa, el método fábrica base puede devolver
    algún tipo de producto por defecto.

    Observa que, a pesar de su nombre, la creación de producto **no** es
    la principal responsabilidad de la clase creadora. Normalmente, esta
    clase cuenta con alguna lógica de negocios central relacionada con
    los productos. El patrón Factory Method ayuda a desacoplar esta
    lógica de las clases concretas de producto. Aquí tienes una
    analogía: una gran empresa de desarrollo de software puede contar
    con un departamento de formación de programadores. Sin embargo, la
    principal función de la empresa sigue siendo escribir código, no
    preparar programadores.

4.  Los **Creadores Concretos** sobrescriben el Factory Method base, de
    modo que devuelva un tipo diferente de producto.

    Observa que el método fábrica no tiene que **crear** nuevas
    instancias todo el tiempo. También puede devolver objetos existentes
    de una memoria caché, una agrupación de objetos, u otra fuente.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

Este ejemplo ilustra cómo puede utilizarse el patrón **Factory Method**
para crear elementos de interfaz de usuario (UI) multiplataforma sin
acoplar el código cliente a clases UI concretas.

<figure class="image">
<img
src="../../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Ejemplo de la estructura del patrón Factory Method" />
<figcaption><p>Ejemplo del diálogo multiplataforma.</p></figcaption>
</figure>

La clase base de diálogo utiliza distintos elementos UI para representar
su ventana. En varios sistemas operativos, estos elementos pueden tener
aspectos diferentes, pero su comportamiento debe ser consistente. Un
botón en Windows sigue siendo un botón en Linux.

Cuando entra en juego el patrón Factory Method no hace falta reescribir
la lógica del diálogo para cada sistema operativo. Si declaramos un
patrón Factory Method que produce botones dentro de la clase base de
diálogo, más tarde podremos crear una subclase de diálogo que devuelva
botones al estilo de Windows desde el Factory Method. Entonces la
subclase hereda la mayor parte del código del diálogo de la clase base,
pero, gracias al Factory Method, puede representar botones al estilo de
Windows en pantalla.

Para que este patrón funcione, la clase base de diálogo debe funcionar
con botones abstractos, es decir, una clase base o una interfaz que
sigan todos los botones concretos. De este modo, el código sigue siendo
funcional, independientemente del tipo de botones con el que trabaje.

Por supuesto, también se puede aplicar este sistema a otros elementos
UI. Sin embargo, con cada nuevo método de fábrica que añadas al diálogo,
más te acercarás al patrón [Abstract Factory](abstract-factory.html). No
temas, más adelante hablaremos sobre este patrón.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase creadora declara el método fábrica que debe devolver
// un objeto de una clase de producto. Normalmente, las
// subclases de la creadora proporcionan la implementación de
// este método.
class Dialog is
    // La creadora también puede proporcionar cierta
    // implementación por defecto del método fábrica.
    abstract method createButton():Button

    // Observa que, a pesar de su nombre, la principal
    // responsabilidad de la creadora no es crear productos.
    // Normalmente contiene cierta lógica de negocio que depende
    // de los objetos de producto devueltos por el método
    // fábrica. Las subclases pueden cambiar indirectamente esa
    // lógica de negocio sobrescribiendo el método fábrica y
    // devolviendo desde él un tipo diferente de producto.
    method render() is
        // Invoca el método fábrica para crear un objeto de
        // producto.
        Button okButton = createButton()
        // Ahora utiliza el producto.
        okButton.onClick(closeDialog)
        okButton.render()


// Los creadores concretos sobrescriben el método fábrica para
// cambiar el tipo de producto resultante.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// La interfaz de producto declara las operaciones que todos los
// productos concretos deben implementar.
interface Button is
    method render()
    method onClick(f)

// Los productos concretos proporcionan varias implementaciones
// de la interfaz de producto.

class WindowsButton implements Button is
    method render(a, b) is
        // Representa un botón en estilo Windows.
    method onClick(f) is
        // Vincula un evento clic de OS nativo.

class HTMLButton implements Button is
    method render(a, b) is
        // Devuelve una representación HTML de un botón.
    method onClick(f) is
        // Vincula un evento clic de navegador web.

class Application is
    field dialog: Dialog

    // La aplicación elige un tipo de creador dependiendo de la
    // configuración actual o los ajustes del entorno.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // El código cliente funciona con una instancia de un
    // creador concreto, aunque a través de su interfaz base.
    // Siempre y cuando el cliente siga funcionando con el
    // creador a través de la interfaz base, puedes pasarle
    // cualquier subclase del creador.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el Método Fábrica cuando no conozcas de antemano las
dependencias y los tipos exactos de los objetos con los que deba
funcionar tu código.
:::

::: applicability-solution
El patrón Factory Method separa el código de construcción de producto
del código que hace uso del producto. Por ello, es más fácil extender el
código de construcción de producto de forma independiente al resto del
código.

Por ejemplo, para añadir un nuevo tipo de producto a la aplicación, sólo
tendrás que crear una nueva subclase creadora y sobrescribir el Factory
Method que contiene.
:::

::: applicability-problem
Utiliza el Factory Method cuando quieras ofrecer a los usuarios de tu
biblioteca o framework, una forma de extender sus componentes internos.
:::

::: applicability-solution
La herencia es probablemente la forma más sencilla de extender el
comportamiento por defecto de una biblioteca o un framework. Pero, ¿cómo
reconoce el framework si debe utilizar tu subclase en lugar de un
componente estándar?

La solución es reducir el código que construye componentes en todo el
framework a un único patrón Factory Method y permitir que cualquiera
sobrescriba este método además de extender el propio componente.

Veamos cómo funcionaría. Imagina que escribes una aplicación utilizando
un framework de UI de código abierto. Tu aplicación debe tener botones
redondos, pero el framework sólo proporciona botones cuadrados.
Extiendes la clase estándar `Botón` con una maravillosa subclase
`BotónRedondo`, pero ahora tienes que decirle a la clase principal
`FrameworkUI` que utilice la nueva subclase de botón en lugar de la
clase por defecto.

Para conseguirlo, creamos una subclase `UIConBotonesRedondos` a partir
de una clase base del framework y sobrescribimos su método `crearBotón`.
Si bien este método devuelve objetos `Botón` en la clase base, haces que
tu subclase devuelva objetos `BotónRedondo`. Ahora, utiliza la clase
`UIConBotonesRedondos` en lugar de `FrameworkUI`. ¡Eso es todo!
:::

::: applicability-problem
Utiliza el Factory Method cuando quieras ahorrar recursos del sistema
mediante la reutilización de objetos existentes en lugar de
reconstruirlos cada vez.
:::

::: applicability-solution
A menudo experimentas esta necesidad cuando trabajas con objetos grandes
y que consumen muchos recursos, como conexiones de bases de datos,
sistemas de archivos y recursos de red.

Pensemos en lo que hay que hacer para reutilizar un objeto existente:

1.  Primero, debemos crear un almacenamiento para llevar un registro de
    todos los objetos creados.
2.  Cuando alguien necesite un objeto, el programa deberá buscar un
    objeto disponible dentro de ese agrupamiento.
3.  ... y devolverlo al código cliente.
4.  Si no hay objetos disponibles, el programa deberá crear uno nuevo (y
    añadirlo al agrupamiento).

¡Eso es mucho código! Y hay que ponerlo todo en un mismo sitio para no
contaminar el programa con código duplicado.

Es probable que el lugar más evidente y cómodo para colocar este código
sea el constructor de la clase cuyos objetos intentamos reutilizar. Sin
embargo, un constructor siempre debe devolver **nuevos objetos** por
definición. No puede devolver instancias existentes.

Por lo tanto, necesitas un método regular capaz de crear nuevos objetos,
además de reutilizar los existentes. Eso suena bastante a lo que hace un
patrón Factory Method.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Haz que todos los productos sigan la misma interfaz. Esta interfaz
    deberá declarar métodos que tengan sentido en todos los productos.

2.  Añade un patrón Factory Method vacío dentro de la clase creadora. El
    tipo de retorno del método deberá coincidir con la interfaz común de
    los productos.

3.  Encuentra todas las referencias a constructores de producto en el
    código de la clase creadora. Una a una, sustitúyelas por
    invocaciones al Factory Method, mientras extraes el código de
    creación de productos para colocarlo dentro del Factory Method.

    Puede ser que tengas que añadir un parámetro temporal al Factory
    Method para controlar el tipo de producto devuelto.

    A estas alturas, es posible que el aspecto del código del Factory
    Method luzca bastante desagradable. Puede ser que tenga un operador
    `switch` largo que elige qué clase de producto instanciar. Pero, no
    te preocupes, lo arreglaremos enseguida.

4.  Ahora, crea un grupo de subclases creadoras para cada tipo de
    producto enumerado en el Factory Method. Sobrescribe el Factory
    Method en las subclases y extrae las partes adecuadas de código
    constructor del método base.

5.  Si hay demasiados tipos de producto y no tiene sentido crear
    subclases para todos ellos, puedes reutilizar el parámetro de
    control de la clase base en las subclases.

    Por ejemplo, imagina que tienes la siguiente jerarquía de clases: la
    clase base `Correo` con las subclases `CorreoAéreo` y
    `CorreoTerrestre` y la clase `Transporte` con `Avión`, `Camión` y
    `Tren`. La clase `CorreoAéreo` sólo utiliza objetos `Avión`, pero
    `CorreoTerrestre` puede funcionar tanto con objetos `Camión`, como
    con objetos `Tren`. Puedes crear una nueva subclase (digamos,
    `CorreoFerroviario`) que gestione ambos casos, pero hay otra opción.
    El código cliente puede pasar un argumento al Factory Method de la
    clase `CorreoTerrestre` para controlar el producto que quiere
    recibir.

6.  Si, tras todas las extracciones, el Factory Method base queda vacío,
    puedes hacerlo abstracto. Si queda algo dentro, puedes convertirlo
    en un comportamiento por defecto del método.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Evitas un acoplamiento fuerte entre el creador y los productos
    concretos.
-    *Principio de responsabilidad única*. Puedes mover el código de
    creación de producto a un lugar del programa, haciendo que el código
    sea más fácil de mantener.
-    *Principio de abierto/cerrado*. Puedes incorporar nuevos tipos de
    productos en el programa sin descomponer el código cliente
    existente.
:::

::: col-sm-6
-    Puede ser que el código se complique, ya que debes incorporar una
    multitud de nuevas subclases para implementar el patrón. La
    situación ideal sería introducir el patrón en una jerarquía
    existente de clases creadoras.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Muchos diseños empiezan utilizando el [Factory
    Method](factory-method.html) (menos complicado y más personalizable
    mediante las subclases) y evolucionan hacia [Abstract
    Factory](abstract-factory.html), [Prototype](prototype.html), o
    [Builder](builder.html) (más flexibles, pero más complicados).

-   Las clases del [Abstract Factory](abstract-factory.html) a menudo se
    basan en un grupo de [métodos de fábrica](factory-method.html), pero
    también puedes utilizar [Prototype](prototype.html) para escribir
    los métodos de estas clases.

-   Puedes utilizar el patrón [Factory Method](factory-method.html)
    junto con el [Iterator](iterator.html) para permitir que las
    subclases de la colección devuelvan distintos tipos de iteradores
    que sean compatibles con las colecciones.

-   [Prototype](prototype.html) no se basa en la herencia, por lo que no
    presenta sus inconvenientes. No obstante, *Prototype* requiere de
    una inicialización complicada del objeto clonado. [Factory
    Method](factory-method.html) se basa en la herencia, pero no
    requiere de un paso de inicialización.

-   [Factory Method](factory-method.html) es una especialización del
    [Template Method](template-method.html). Al mismo tiempo, un
    *Factory Method* puede servir como paso de un gran *Template
    Method*.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Factory Method en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Factory Method en C#"){.prog-lang-link}
[![Factory Method en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Factory Method en C++"){.prog-lang-link}
[![Factory Method en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Factory Method en Go"){.prog-lang-link}
[![Factory Method en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Factory Method en Java"){.prog-lang-link}
[![Factory Method en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Factory Method en PHP"){.prog-lang-link}
[![Factory Method en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Factory Method en Python"){.prog-lang-link}
[![Factory Method en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Factory Method en Ruby"){.prog-lang-link}
[![Factory Method en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Factory Method en Rust"){.prog-lang-link}
[![Factory Method en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Factory Method en Swift"){.prog-lang-link}
[![Factory Method en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Factory Method en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Contenido adicional {#extras}

-   Consulta nuestra [Comparación de fábricas](factory-comparison.html)
    si aún no te queda clara la diferencia entre los varios patrones y
    conceptos de la fábrica.
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

[Abstract Factory []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Patrones
creacionales](creational-patterns.html){.btn .btn-default rel="prev"}
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
