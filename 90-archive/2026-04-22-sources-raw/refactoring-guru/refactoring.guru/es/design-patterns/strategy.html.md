::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Strategy {#strategy .title}

::: aka
También llamado: [Estrategia]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Strategy** es un patrón de diseño de comportamiento que te permite
definir una familia de algoritmos, colocar cada uno de ellos en una
clase separada y hacer sus objetos intercambiables.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Strategy" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Un día decidiste crear una aplicación de navegación para viajeros
ocasionales. La aplicación giraba alrededor de un bonito mapa que
ayudaba a los usuarios a orientarse rápidamente en cualquier ciudad.

Una de las funciones más solicitadas para la aplicación era la
planificación automática de rutas. Un usuario debía poder introducir una
dirección y ver la ruta más rápida a ese destino mostrado en el mapa.

La primera versión de la aplicación sólo podía generar las rutas sobre
carreteras. Las personas que viajaban en coche estaban locas de alegría.
Pero, aparentemente, no a todo el mundo le gusta conducir durante sus
vacaciones. De modo que, en la siguiente actualización, añadiste una
opción para crear rutas a pie. Después, añadiste otra opción para
permitir a las personas utilizar el transporte público en sus rutas.

Sin embargo, esto era sólo el principio. Más tarde planeaste añadir la
generación de rutas para ciclistas, y más tarde, otra opción para trazar
rutas por todas las atracciones turísticas de una ciudad.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="El código del navegador se saturó" />
<figcaption><p>El código del navegador se saturó.</p></figcaption>
</figure>

Aunque desde una perspectiva comercial la aplicación era un éxito, la
parte técnica provocaba muchos dolores de cabeza. Cada vez que añadías
un nuevo algoritmo de enrutamiento, la clase principal del navegador
doblaba su tamaño. En cierto momento, la bestia se volvió demasiado
difícil de mantener.

Cualquier cambio en alguno de los algoritmos, ya fuera un sencillo
arreglo de un error o un ligero ajuste de la representación de la calle,
afectaba a toda la clase, aumentando las probabilidades de crear un
error en un código ya funcional.

Además, el trabajo en equipo se volvió ineficiente. Tus compañeros,
contratados tras el exitoso lanzamiento, se quejaban de que dedicaban
demasiado tiempo a resolver conflictos de integración. Implementar una
nueva función te exige cambiar la misma clase enorme, entrando en
conflicto con el código producido por otras personas.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Strategy sugiere que tomes esa clase que hace algo específico
de muchas formas diferentes y extraigas todos esos algoritmos para
colocarlos en clases separadas llamadas *estrategias*.

La clase original, llamada *contexto*, debe tener un campo para
almacenar una referencia a una de las estrategias. El contexto delega el
trabajo a un objeto de estrategia vinculado en lugar de ejecutarlo por
su cuenta.

La clase contexto no es responsable de seleccionar un algoritmo adecuado
para la tarea. En lugar de eso, el cliente pasa la estrategia deseada a
la clase contexto. De hecho, la clase contexto no sabe mucho acerca de
las estrategias. Funciona con todas las estrategias a través de la misma
interfaz genérica, que sólo expone un único método para disparar el
algoritmo encapsulado dentro de la estrategia seleccionada.

De esta forma, el contexto se vuelve independiente de las estrategias
concretas, así que puedes añadir nuevos algoritmos o modificar los
existentes sin cambiar el código de la clase contexto o de otras
estrategias.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Estrategias de planificación de rutas" />
<figcaption><p>Estrategias de planificación de rutas.</p></figcaption>
</figure>

En nuestra aplicación de navegación, cada algoritmo de enrutamiento
puede extraerse y ponerse en su propia clase con un único método
`crearRuta`. El método acepta un origen y un destino y devuelve una
colección de puntos de control de la ruta.

Incluso contando con los mismos argumentos, cada clase de enrutamiento
puede crear una ruta diferente. A la clase navegadora principal no le
importa qué algoritmo se selecciona ya que su labor principal es
representar un grupo de puntos de control en el mapa. La clase tiene un
método para cambiar la estrategia activa de enrutamiento, de modo que
sus clientes, como los botones en la interfaz de usuario, pueden
sustituir el comportamiento seleccionado de enrutamiento por otro.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-es2f14.png?id=1cf442d8c2d5d78f214499bb72dfdc72"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-es.png?id=1cf442d8c2d5d78f214499bb72dfdc72 640w,/images/patterns/content/strategy/strategy-comic-1-es-1.5x.png?id=a6241eecbec495def9828d9b4bc89be9 960w,/images/patterns/content/strategy/strategy-comic-1-es-2x.png?id=0627ee71c0de11348ddaebe6e39915f5 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Varias estrategias de transporte" />
<figcaption><p>Varias estrategias para llegar
al aeropuerto.</p></figcaption>
</figure>

Imagina que tienes que llegar al aeropuerto. Puedes tomar el autobús,
pedir un taxi o ir en bicicleta. Éstas son tus estrategias de
transporte. Puedes elegir una de las estrategias, dependiendo de
factores como el presupuesto o los límites de tiempo.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Estructura del patrón de diseño Strategy" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Estructura del patrón de diseño Strategy" />
</figure>
:::

1.  La clase **Contexto** mantiene una referencia a una de las
    estrategias concretas y se comunica con este objeto únicamente a
    través de la interfaz estrategia.

2.  La interfaz **Estrategia** es común a todas las estrategias
    concretas. Declara un método que la clase contexto utiliza para
    ejecutar una estrategia.

3.  Las **Estrategias Concretas** implementan distintas variaciones de
    un algoritmo que la clase contexto utiliza.

4.  La clase contexto invoca el método de ejecución en el objeto de
    estrategia vinculado cada vez que necesita ejecutar el algoritmo. La
    clase contexto no sabe con qué tipo de estrategia funciona o cómo se
    ejecuta el algoritmo.

5.  El **Cliente** crea un objeto de estrategia específico y lo pasa a
    la clase contexto. La clase contexto expone un modificador *set* que
    permite a los clientes sustituir la estrategia asociada al contexto
    durante el tiempo de ejecución.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el contexto utiliza varias **estrategias** para
ejecutar diversas operaciones aritméticas.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz estrategia declara operaciones comunes a todas
// las versiones soportadas de algún algoritmo. El contexto
// utiliza esta interfaz para invocar el algoritmo definido por
// las estrategias concretas.
interface Strategy is
    method execute(a, b)

// Las estrategias concretas implementan el algoritmo mientras
// siguen la interfaz estrategia base. La interfaz las hace
// intercambiables en el contexto.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// El contexto define la interfaz de interés para los clientes.
class Context is
    // El contexto mantiene una referencia a uno de los objetos
    // de estrategia. El contexto no conoce la clase concreta de
    // una estrategia. Debe trabajar con todas las estrategias a
    // través de la interfaz estrategia.
    private strategy: Strategy

    // Normalmente, el contexto acepta una estrategia a través
    // del constructor y también proporciona un setter
    // (modificador) para poder cambiar la estrategia durante el
    // tiempo de ejecución.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // El contexto delega parte del trabajo al objeto de
    // estrategia en lugar de implementar varias versiones del
    // algoritmo por su cuenta.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// El código cliente elige una estrategia concreta y la pasa al
// contexto. El cliente debe conocer las diferencias entre
// estrategias para elegir la mejor opción.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::::: applicability
::: applicability-problem
Utiliza el patrón Strategy cuando quieras utiliza distintas variantes de
un algoritmo dentro de un objeto y poder cambiar de un algoritmo a otro
durante el tiempo de ejecución.
:::

::: applicability-solution
El patrón Strategy te permite alterar indirectamente el comportamiento
del objeto durante el tiempo de ejecución asociándolo con distintos
subobjetos que pueden realizar subtareas específicas de distintas
maneras.
:::

::: applicability-problem
Utiliza el patrón Strategy cuando tengas muchas clases similares que
sólo se diferencien en la forma en que ejecutan cierto comportamiento.
:::

::: applicability-solution
El patrón Strategy te permite extraer el comportamiento variante para
ponerlo en una jerarquía de clases separada y combinar las clases
originales en una, reduciendo con ello el código duplicado.
:::

::: applicability-problem
Utiliza el patrón para aislar la lógica de negocio de una clase, de los
detalles de implementación de algoritmos que pueden no ser tan
importantes en el contexto de esa lógica.
:::

::: applicability-solution
El patrón Strategy te permite aislar el código, los datos internos y las
dependencias de varios algoritmos, del resto del código. Los diversos
clientes obtienen una interfaz simple para ejecutar los algoritmos y
cambiarlos durante el tiempo de ejecución.
:::

::: applicability-problem
Utiliza el patrón cuando tu clase tenga un enorme operador condicional
que cambie entre distintas variantes del mismo algoritmo.
:::

::: applicability-solution
El patrón Strategy te permite suprimir dicho condicional extrayendo
todos los algoritmos para ponerlos en clases separadas, las cuales
implementan la misma interfaz. El objeto original delega la ejecución a
uno de esos objetos, en lugar de implementar todas las variantes del
algoritmo.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  En la clase contexto, identifica un algoritmo que tienda a sufrir
    cambios frecuentes. También puede ser un enorme condicional que
    seleccione y ejecute una variante del mismo algoritmo durante el
    tiempo de ejecución.

2.  Declara la interfaz estrategia común a todas las variantes del
    algoritmo.

3.  Uno a uno, extrae todos los algoritmos y ponlos en sus propias
    clases. Todas deben implementar la misma interfaz estrategia.

4.  En la clase contexto, añade un campo para almacenar una referencia a
    un objeto de estrategia. Proporciona un modificador *set* para
    sustituir valores de ese campo. La clase contexto debe trabajar con
    el objeto de estrategia únicamente a través de la interfaz
    estrategia. La clase contexto puede definir una interfaz que permita
    a la estrategia acceder a sus datos.

5.  Los clientes de la clase contexto deben asociarla con una estrategia
    adecuada que coincida con la forma en la que esperan que la clase
    contexto realice su trabajo principal.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes intercambiar algoritmos usados dentro de un objeto durante
    el tiempo de ejecución.
-    Puedes aislar los detalles de implementación de un algoritmo del
    código que lo utiliza.
-    Puedes sustituir la herencia por composición.
-    *Principio de abierto/cerrado*. Puedes introducir nuevas
    estrategias sin tener que cambiar el contexto.
:::

::: col-sm-6
-    Si sólo tienes un par de algoritmos que raramente cambian, no hay
    una razón real para complicar el programa en exceso con nuevas
    clases e interfaces que vengan con el patrón.
-    Los clientes deben conocer las diferencias entre estrategias para
    poder seleccionar la adecuada.
-    Muchos lenguajes de programación modernos tienen un soporte de tipo
    funcional que te permite implementar distintas versiones de un
    algoritmo dentro de un grupo de funciones anónimas. Entonces puedes
    utilizar estas funciones exactamente como habrías utilizado los
    objetos de estrategia, pero sin saturar tu código con clases e
    interfaces adicionales.
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

-   [Decorator](decorator.html) te permite cambiar la piel de un objeto,
    mientras que [Strategy](strategy.html) te permite cambiar sus
    entrañas.

-   [Template Method](template-method.html) se basa en la herencia: te
    permite alterar partes de un algoritmo extendiendo esas partes en
    subclases. [Strategy](strategy.html) se basa en la composición:
    puedes alterar partes del comportamiento del objeto suministrándole
    distintas estrategias que se correspondan con ese comportamiento.
    *Template Method* trabaja al nivel de la clase, por lo que es
    estático. *Strategy* trabaja al nivel del objeto, permitiéndote
    cambiar los comportamientos durante el tiempo de ejecución.

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

[![Strategy en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Strategy en C#"){.prog-lang-link}
[![Strategy en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Strategy en C++"){.prog-lang-link}
[![Strategy en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Strategy en Go"){.prog-lang-link}
[![Strategy en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Strategy en Java"){.prog-lang-link}
[![Strategy en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Strategy en PHP"){.prog-lang-link}
[![Strategy en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Strategy en Python"){.prog-lang-link}
[![Strategy en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Strategy en Ruby"){.prog-lang-link}
[![Strategy en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Strategy en Rust"){.prog-lang-link}
[![Strategy en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Strategy en Swift"){.prog-lang-link}
[![Strategy en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Strategy en TypeScript"){.prog-lang-link}
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

[Template Method []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} State](state.html){.btn .btn-default rel="prev"}
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
