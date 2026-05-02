::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Chain of Responsibility {#chain-of-responsibility .title}

::: aka
También llamado: [Cadena de
responsabilidad, ]{style="display:inline-block"}[CoR, ]{style="display:inline-block"}[Chain
of Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Chain of Responsibility** es un patrón de diseño de comportamiento que
te permite pasar solicitudes a lo largo de una cadena de manejadores. Al
recibir una solicitud, cada manejador decide si la procesa o si la pasa
al siguiente manejador de la cadena.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Chain of Responsibility" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás trabajando en un sistema de pedidos online. Quieres
restringir el acceso al sistema de forma que únicamente los usuarios
autenticados puedan generar pedidos. Además, los usuarios que tengan
permisos administrativos deben tener pleno acceso a todos los pedidos.

Tras planificar un poco, te das cuenta de que estas comprobaciones deben
realizarse secuencialmente. La aplicación puede intentar autenticar a un
usuario en el sistema cuando reciba una solicitud que contenga las
credenciales del usuario. Sin embargo, si esas credenciales no son
correctas y la autenticación falla, no hay razón para proceder con otras
comprobaciones.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-es938e.png?id=933d7c471a3a557d60f07a63c2f61bf4"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-es.png?id=933d7c471a3a557d60f07a63c2f61bf4 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-es-1.5x.png?id=6d5ab3632a4d759ab9f4815a1960e6c9 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-es-2x.png?id=f8f78a1bc776618bf423e1f5989822d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Problema solucionado por el patrón Chain of Responsibility" />
<figcaption><p>La solicitud debe pasar una serie de comprobaciones antes
de que el propio sistema de pedidos pueda gestionarla.</p></figcaption>
</figure>

Durante los meses siguientes, implementas varias de esas comprobaciones
secuenciales.

-   Uno de tus colegas sugiere que no es seguro pasar datos sin procesar
    directamente al sistema de pedidos. De modo que añades un paso
    adicional de validación para sanear los datos de una solicitud.

-   Más tarde, alguien se da cuenta de que el sistema es vulnerable al
    desciframiento de contraseñas por la fuerza. Para evitarlo, añades
    rápidamente una comprobación que filtra las solicitudes fallidas
    repetidas que vengan de la misma dirección IP.

-   Otra persona sugiere que podrías acelerar el sistema devolviendo los
    resultados en caché en solicitudes repetidas que contengan los
    mismos datos, de modo que añades otra comprobación que permite a la
    solicitud pasar por el sistema únicamente cuando no hay una
    respuesta adecuada en caché.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-es8ede.png?id=77abd95312760a426351895db7fff55d"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-es.png?id=77abd95312760a426351895db7fff55d 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-es-1.5x.png?id=23d6a6f240bdf7d3d0c72c9489797e25 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-es-2x.png?id=550bd802c700050a6932eb58d6346eb9 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Con cada nueva comprobación el código crece, se complica y se afea" />
<figcaption><p>Cuanto más crece el código, más
se complica.</p></figcaption>
</figure>

El código de las comprobaciones, que ya se veía desordenado, se vuelve
más y más abotargado cada vez que añades una nueva función. En
ocasiones, un cambio en una comprobación afecta a las demás. Y lo peor
de todo es que, cuando intentas reutilizar las comprobaciones para
proteger otros componentes del sistema, tienes que duplicar parte del
código, ya que esos componentes necesitan parte de las comprobaciones,
pero no todas ellas.

El sistema se vuelve muy difícil de comprender y costoso de mantener.
Luchas con el código durante un tiempo hasta que un día decides
refactorizarlo todo.
:::

::: {.section .solution}
##  Solución {#solution}

Al igual que muchos otros patrones de diseño de comportamiento, el
**Chain of Responsibility** se basa en transformar comportamientos
particulares en objetos autónomos llamados *manejadores*. En nuestro
caso, cada comprobación debe ponerse dentro de su propia clase con un
único método que realice la comprobación. La solicitud, junto con su
información, se pasa a este método como argumento.

El patrón sugiere que vincules esos manejadores en una cadena. Cada
manejador vinculado tiene un campo para almacenar una referencia al
siguiente manejador de la cadena. Además de procesar una solicitud, los
manejadores la pasan a lo largo de la cadena. La solicitud viaja por la
cadena hasta que todos los manejadores han tenido la oportunidad de
procesarla.

Y ésta es la mejor parte: un manejador puede decidir no pasar la
solicitud más allá por la cadena y detener con ello el procesamiento.

En nuestro ejemplo de los sistemas de pedidos, un manejador realiza el
procesamiento y después decide si pasa la solicitud al siguiente eslabón
de la cadena. Asumiendo que la solicitud contiene la información
correcta, todos los manejadores pueden ejecutar su comportamiento
principal, ya sean comprobaciones de autenticación o almacenamiento en
la memoria caché.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-esbfec.png?id=122092268f688aa2015b2f20dabafb89"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-es.png?id=122092268f688aa2015b2f20dabafb89 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-es-1.5x.png?id=0edbc910c0d4c34d2efdd9f20b049bcf 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-es-2x.png?id=3689c7f432e4ac52adca66105f885ae3 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Los manejadores se alinean uno tras otro, formando una cadena" />
<figcaption><p>Los manejadores se alinean uno tras otro, formando
una cadena.</p></figcaption>
</figure>

No obstante, hay una solución ligeramente diferente (y un poco más
estandarizada) en la que, al recibir una solicitud, un manejador decide
si puede procesarla. Si puede, no pasa la solicitud más allá. De modo
que un único manejador procesa la solicitud o no lo hace ninguno en
absoluto. Esta solución es muy habitual cuando tratamos con eventos en
pilas de elementos dentro de una interfaz gráfica de usuario (GUI).

Por ejemplo, cuando un usuario hace clic en un botón, el evento se
propaga por la cadena de elementos GUI que comienza en el botón, recorre
sus contenedores (como formularios o paneles) y acaba en la ventana
principal de la aplicación. El evento es procesado por el primer
elemento de la cadena que es capaz de gestionarlo. Este ejemplo también
es destacable porque muestra que siempre se puede extraer una cadena de
un árbol de objetos.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-es7aef.png?id=899eb13cebd9b76e5e8c2583f7a8b2f0"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-es.png?id=899eb13cebd9b76e5e8c2583f7a8b2f0 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-es-1.5x.png?id=6c77931cdce7ed8f4414fc7967b477a6 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-es-2x.png?id=dae29aba464b4baa61eae7e806bdf0d5 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Una cadena puede formarse a partir de una rama de un árbol de objetos" />
<figcaption><p>Una cadena puede formarse a partir de una rama de un
árbol de objetos.</p></figcaption>
</figure>

Es fundamental que todas las clases manejadoras implementen la misma
interfaz. Cada manejadora concreta solo debe preocuparse por la
siguiente que cuente con el método `ejecutar`. De esta forma puedes
componer cadenas durante el tiempo de ejecución, utilizando varios
manejadores sin acoplar tu código a sus clases concretas.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-es197d.png?id=a57afa94531c4b0395559c4d2d675967"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-es.png?id=a57afa94531c4b0395559c4d2d675967 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-es-1.5x.png?id=16c115a091d83236a996f00af3171dbf 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-es-2x.png?id=e3bef85a490debda074d2f11dc0f1520 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Hablar con soporte técnico a veces es difícil" />
<figcaption><p>Una llamada al soporte técnico puede pasar por
muchos operadores.</p></figcaption>
</figure>

Acabas de comprar e instalar una nueva pieza de hardware en tu
computadora. Como eres un fanático de la informática, la computadora
tiene varios sistemas operativos instalados. Intentas arrancarlos todos
para ver si soportan el hardware. Windows detecta y habilita el hardware
automáticamente. Sin embargo, tu querido Linux se niega a funcionar con
el nuevo hardware. Ligeramente esperanzado, decides llamar al número de
teléfono de soporte técnico escrito en la caja.

Lo primero que oyes es la voz robótica del contestador automático. Te
sugiere nueve soluciones populares a varios problemas, pero ninguna de
ellas es relevante a tu caso. Después de un rato, el robot te conecta
con un operador humano.

Por desgracia, el operador tampoco consigue sugerirte nada específico.
Se dedica a recitar largos pasajes del manual, negándose a escuchar tus
comentarios. Cuando escuchas por enésima vez la frase "¿has intentado
apagar y encender la computadora?", exiges que te pasen con un ingeniero
de verdad.

Por fin, el operador pasa tu llamada a unos de los ingenieros, que
probablemente ansiaba una conversación humana desde hacía tiempo,
sentado en la solitaria sala del servidor del oscuro sótano de un
edificio de oficinas. El ingeniero te indica dónde descargar los drivers
adecuados para tu nuevo hardware y cómo instalarlos en Linux. Por fin,
¡la solución! Acabas la llamada dando saltos de alegría.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Estructura del patrón de diseño Chain Of Responsibility" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Estructura del patrón de diseño Chain Of Responsibility" />
</figure>
:::

1.  La clase **Manejadora** declara la interfaz común a todos los
    manejadores concretos. Normalmente contiene un único método para
    manejar solicitudes, pero en ocasiones también puede contar con otro
    método para establecer el siguiente manejador de la cadena.

2.  La clase **Manejadora Base** es opcional y es donde puedes colocar
    el código boilerplate (segmentos de código que suelen no alterarse)
    común para todas las clases manejadoras.

    Normalmente, esta clase define un campo para almacenar una
    referencia al siguiente manejador. Los clientes pueden crear una
    cadena pasando un manejador al constructor o modificador (*setter*)
    del manejador previo. La clase también puede implementar el
    comportamiento de gestión por defecto: puede pasar la ejecución al
    siguiente manejador después de comprobar su existencia.

3.  Los **Manejadores Concretos** contienen el código para procesar las
    solicitudes. Al recibir una solicitud, cada manejador debe decidir
    si procesarla y, además, si la pasa a lo largo de la cadena.

    Habitualmente los manejadores son autónomos e inmutables, y aceptan
    toda la información necesaria únicamente a través del constructor.

4.  El **Cliente** puede componer cadenas una sola vez o componerlas
    dinámicamente, dependiendo de la lógica de la aplicación. Observa
    que se puede enviar una solicitud a cualquier manejador de la
    cadena; no tiene por qué ser al primero.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Chain of Responsibility** es responsable de
mostrar información de ayuda contextual para elementos GUI activos.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-esf11a.png?id=8e8e07276030bd650874fa831c53156f"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-es.png?id=8e8e07276030bd650874fa831c53156f 610w,/images/patterns/diagrams/chain-of-responsibility/example-es-1.5x.png?id=9b4fb4ac59061623c4bc904b2cfedcda 915w,/images/patterns/diagrams/chain-of-responsibility/example-es-2x.png?id=708c56c5f322b8c4ae38367386f1021d 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Ejemplo de estructura del patrón Chain of Responsibility" />
<figcaption><p>Las clases GUI se crean con el patrón Composite. Cada
elemento se vincula a su elemento contenedor. En cualquier momento
puedes crear una cadena de elementos que comience con el propio elemento
y recorra todos los elementos contenedores.</p></figcaption>
</figure>

La GUI de la aplicación se estructura normalmente como un árbol de
objetos. Por ejemplo, la clase `Diálogo`, que representa la ventana
principal de la aplicación, es la raíz del árbol de objetos. La clase
diálogo contiene `Paneles`, que pueden contener otros paneles o simples
elementos de bajo nivel, como `Botones` y `CamposdeTexto`.

Un simple componente puede mostrar breves pistas contextuales, siempre y
cuando el componente tenga asignado cierto texto de ayuda. Pero los
componentes más complejos definen su propia forma de mostrar ayuda
contextual, por ejemplo, mostrando un extracto del manual o abriendo una
página en un navegador.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-es0a6c.png?id=0e4121044c422a4912190422c797c853"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-es.png?id=0e4121044c422a4912190422c797c853 250w,/images/patterns/diagrams/chain-of-responsibility/example2-es-1.5x.png?id=6983b6ef59e28c8aa3e98d07ac89d5a7 375w,/images/patterns/diagrams/chain-of-responsibility/example2-es-2x.png?id=fdd402509cd4b6d21bbd6bc45c20c7cd 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Ejemplo de estructura del patrón Chain of Responsibility" />
<figcaption><p>Ésta es la forma en la que las solicitudes de ayuda
recorren objetos GUI.</p></figcaption>
</figure>

Cuando un usuario apunta el cursor del ratón a un elemento y pulsa la
tecla `F1`, la aplicación detecta el componente bajo el puntero y le
envía una solicitud de ayuda. La solicitud emerge por todos los
contenedores del elemento hasta que llega al elemento capaz de mostrar
la información de ayuda.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz manejadora declara un método para ejecutar una
// solicitud.
interface ComponentWithContextualHelp is
    method showHelp()


// La clase base para componentes simples.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // El contenedor del componente actúa como el siguiente
    // eslabón de la cadena de manejadores.
    protected field container: Container

    // El componente muestra una pista si tiene un texto de
    // ayuda asignado. De lo contrario, reenvía la llamada al
    // contenedor, si es que existe.
    method showHelp() is
        if (tooltipText != null)
            // Muestra la pista.
        else
            container.showHelp()


// Los contenedores pueden contener componentes simples y otros
// contenedores como hijos. Las relaciones de la cadena se
// establecen aquí. La clase hereda el comportamiento showHelp
// (mostrarAyuda) de su padre.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Los componentes primitivos pueden estar bien con la
// implementación de la ayuda por defecto...
class Button extends Component is
    // ...

// Pero los componentes complejos pueden sobrescribir la
// implementación por defecto. Si no puede proporcionarse el
// texto de ayuda de una nueva forma, el componente siempre
// puede invocar la implementación base (véase la clase
// Componente).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Muestra una ventana modal con el texto de ayuda.
        else
            super.showHelp()

// ...igual que arriba...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Abre la página de ayuda wiki.
        else
            super.showHelp()


// Código cliente.
class Application is
    // Cada aplicación configura la cadena de forma diferente.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imagina lo que pasa aquí.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Chain of Responsibility cuando tu programa deba
procesar distintos tipos de solicitudes de varias maneras, pero los
tipos exactos de solicitudes y sus secuencias no se conozcan de
antemano.
:::

::: applicability-solution
El patrón te permite encadenar varios manejadores y, al recibir una
solicitud, "preguntar" a cada manejador si puede procesarla. De esta
forma todos los manejadores tienen la oportunidad de procesar la
solicitud.
:::

::: applicability-problem
Utiliza el patrón cuando sea fundamental ejecutar varios manejadores en
un orden específico.
:::

::: applicability-solution
Ya que puedes vincular los manejadores de la cadena en cualquier orden,
todas las solicitudes recorrerán la cadena exactamente como planees.
:::

::: applicability-problem
Utiliza el patrón Chain of Responsibility cuando el grupo de manejadores
y su orden deban cambiar durante el tiempo de ejecución.
:::

::: applicability-solution
Si aportas modificadores (*setters*) para un campo de referencia dentro
de las clases manejadoras, podrás insertar, eliminar o reordenar los
manejadores dinámicamente.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Declara la interfaz manejadora y describe la firma de un método para
    manejar solicitudes.

    Decide cómo pasará el cliente la información de la solicitud dentro
    del método. La forma más flexible consiste en convertir la solicitud
    en un objeto y pasarlo al método de gestión como argumento.

2.  Para eliminar código boilerplate duplicado en manejadores concretos,
    puede merecer la pena crear una clase manejadora abstracta base,
    derivada de la interfaz manejadora.

    Esta clase debe tener un campo para almacenar una referencia al
    siguiente manejador de la cadena. Considera hacer la clase
    inmutable. No obstante, si planeas modificar las cadenas durante el
    tiempo de ejecución, deberás definir un modificador (*setter*) para
    alterar el valor del campo de referencia.

    También puedes implementar el comportamiento por defecto conveniente
    para el método de control, que consiste en reenviar la solicitud al
    siguiente objeto, a no ser que no quede ninguno. Los manejadores
    concretos podrán utilizar este comportamiento invocando al método
    padre.

3.  Una a una, crea subclases manejadoras concretas e implementa los
    métodos de control. Cada manejador debe tomar dos decisiones cuando
    recibe una solicitud:

    -   Si procesa la solicitud.
    -   Si pasa la solicitud al siguiente eslabón de la cadena.

4.  El cliente puede ensamblar cadenas por su cuenta o recibir cadenas
    prefabricadas de otros objetos. En el último caso, debes implementar
    algunas clases fábrica para crear cadenas de acuerdo con los ajustes
    de configuración o de entorno.

5.  El cliente puede activar cualquier manejador de la cadena, no solo
    el primero. La solicitud se pasará a lo largo de la cadena hasta que
    algún manejador se rehúse a pasarlo o hasta que llegue al final de
    la cadena.

6.  Debido a la naturaleza dinámica de la cadena, el cliente debe estar
    listo para gestionar los siguientes escenarios:

    -   La cadena puede consistir en un único vínculo.
    -   Algunas solicitudes pueden no llegar al final de la cadena.
    -   Otras pueden llegar hasta el final de la cadena sin ser
        gestionadas.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes controlar el orden de control de solicitudes.
-    *Principio de responsabilidad única*. Puedes desacoplar las clases
    que invoquen operaciones de las que realicen operaciones.
-    *Principio de abierto/cerrado*. Puedes introducir nuevos
    manejadores en la aplicación sin descomponer el código cliente
    existente.
:::

::: col-sm-6
-    Algunas solicitudes pueden acabar sin ser gestionadas.
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

-   [Chain of Responsibility](chain-of-responsibility.html) se utiliza a
    menudo junto a [Composite](composite.html). En este caso, cuando un
    componente hoja recibe una solicitud, puede pasarla a lo largo de la
    cadena de todos los componentes padre hasta la raíz del árbol de
    objetos.

-   Los manejadores del [Chain of
    Responsibility](chain-of-responsibility.html) se pueden implementar
    como [Comandos](command.html). En este caso, puedes ejecutar muchas
    operaciones diferentes sobre el mismo objeto de contexto,
    representado por una solicitud.

    Sin embargo, hay otra solución en la que la propia solicitud es un
    objeto *Comando*. En este caso, puedes ejecutar la misma operación
    en una serie de contextos diferentes vinculados en una cadena.

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
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Chain of Responsibility en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Chain of Responsibility en C#"){.prog-lang-link}
[![Chain of Responsibility en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Chain of Responsibility en C++"){.prog-lang-link}
[![Chain of Responsibility en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Chain of Responsibility en Go"){.prog-lang-link}
[![Chain of Responsibility en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Chain of Responsibility en Java"){.prog-lang-link}
[![Chain of Responsibility en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Chain of Responsibility en PHP"){.prog-lang-link}
[![Chain of Responsibility en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Chain of Responsibility en Python"){.prog-lang-link}
[![Chain of Responsibility en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Chain of Responsibility en Ruby"){.prog-lang-link}
[![Chain of Responsibility en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Chain of Responsibility en Rust"){.prog-lang-link}
[![Chain of Responsibility en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Chain of Responsibility en Swift"){.prog-lang-link}
[![Chain of Responsibility en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Chain of Responsibility en TypeScript"){.prog-lang-link}
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

[Command []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Patrones de
comportamiento](behavioral-patterns.html){.btn .btn-default rel="prev"}
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
