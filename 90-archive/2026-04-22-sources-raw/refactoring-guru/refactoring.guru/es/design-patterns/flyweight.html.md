:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
estructurales](structural-patterns.html){.type}
:::

# Flyweight {#flyweight .title}

::: aka
También llamado: [Peso mosca, ]{style="display:inline-block"}[Peso
ligero, ]{style="display:inline-block"}[Cache]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Flyweight** es un patrón de diseño estructural que te permite mantener
más objetos dentro de la cantidad disponible de RAM compartiendo las
partes comunes del estado entre varios objetos en lugar de mantener toda
la información en cada objeto.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Flyweight" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Para divertirte un poco después de largas horas de trabajo, decides
crear un sencillo videojuego en el que los jugadores se tienen que mover
por un mapa disparándose entre sí. Decides implementar un sistema de
partículas realistas que lo distinga de otros juegos. Grandes cantidades
de balas, misiles y metralla de las explosiones volarán por todo el
mapa, ofreciendo una apasionante experiencia al jugador.

Al terminarlo, subes el último cambio, compilas el juego y se lo envias
a un amigo para una partida de prueba. Aunque el juego funcionaba sin
problemas en tu máquina, tu amigo no logró jugar durante mucho tiempo.
En su computadora el juego se paraba a los pocos minutos de empezar.
Tras dedicar varias horas a revisar los registros de depuración,
descubres que el juego se paraba debido a una cantidad insuficiente de
RAM. Resulta que el equipo de tu amigo es mucho menos potente que tu
computadora, y esa es la razón por la que el problema surgió tan rápido
en su máquina.

El problema estaba relacionado con tu sistema de partículas. Cada
partícula, como una bala, un misil o un trozo de metralla, estaba
representada por un objeto separado que contenía gran cantidad de datos.
En cierto momento, cuando la masacre alcanzaba su punto culminante en la
pantalla del jugador, las partículas recién creadas ya no cabían en el
resto de RAM, provocando que el programa fallara.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-esdef6.png?id=b3ccf686490551f109abed49fdf787d8"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-es.png?id=b3ccf686490551f109abed49fdf787d8 660w,/images/patterns/diagrams/flyweight/problem-es-1.5x.png?id=8ef7d63d42f69da950eb5cc0251916bc 990w,/images/patterns/diagrams/flyweight/problem-es-2x.png?id=5b3eea59f13106d4db9adc13a27f265e 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Problema del patrón Flyweight" />
</figure>
:::

::: {.section .solution}
##  Solución {#solution}

Observando más atentamente la clase `Partícula`, puede ser que te hayas
dado cuenta de que los campos de color y *sprite* consumen mucha más
memoria que otros campos. Lo que es peor, esos dos campos almacenan
información casi idéntica de todas las partículas. Por ejemplo, todas
las balas tienen el mismo color y sprite.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-es2261.png?id=502f62598cdec561435f06760525242e"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-es.png?id=502f62598cdec561435f06760525242e 640w,/images/patterns/diagrams/flyweight/solution1-es-1.5x.png?id=0ef954881aca7bbefe57fa9fb71290e5 960w,/images/patterns/diagrams/flyweight/solution1-es-2x.png?id=e39dc9b9afef0da20d2f60d4d1dbc14d 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Solución del patrón Flyweight" />
</figure>

Otras partes del estado de una partícula, como las coordenadas, vector
de movimiento y velocidad, son únicas en cada partícula. Después de
todo, los valores de estos campos cambian a lo largo del tiempo. Estos
datos representan el contexto siempre cambiante en el que existe la
partícula, mientras que el color y el sprite se mantienen constantes.

Esta información constante de un objeto suele denominarse su *estado
intrínseco*. Existe dentro del objeto y otros objetos únicamente pueden
leerla, no cambiarla. El resto del estado del objeto, a menudo alterado
"desde el exterior" por otros objetos, se denomina el *estado
extrínseco*.

El patrón Flyweight sugiere que dejemos de almacenar el estado
extrínseco dentro del objeto. En lugar de eso, debes pasar este estado a
métodos específicos que dependen de él. Tan solo el estado intrínseco se
mantiene dentro del objeto, permitiendo que lo reutilices en distintos
contextos. Como resultado, necesitarás menos de estos objetos, ya que
sólo se diferencian en el estado intrínseco, que cuenta con muchas menos
variaciones que el extrínseco.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-es5e88.png?id=7ae27d6241d94a02832d10cba1494113"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-es.png?id=7ae27d6241d94a02832d10cba1494113 424w,/images/patterns/diagrams/flyweight/solution3-es-1.5x.png?id=d53c567ca43323dc87595b8666327147 636w,/images/patterns/diagrams/flyweight/solution3-es-2x.png?id=b7fa410c9cbe70c23c60ffc4486a1451 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Solución del patrón Flyweight" />
</figure>

Regresemos a nuestro juego. Dando por hecho que hemos extraído el estado
extrínseco de la clase de nuestra partícula, únicamente tres objetos
diferentes serán suficientes para representar todas las partículas del
juego: una bala, un misil y un trozo de metralla. Como probablemente
habrás adivinado, un objeto que sólo almacena el estado intrínseco se
denomina *Flyweight* (peso mosca).

#### Almacenamiento del estado extrínseco

¿A dónde se mueve el estado extrínseco? Alguna clase tendrá que
almacenarlo, ¿verdad? En la mayoría de los casos, se mueve al objeto
contenedor, que reúne objetos antes de que apliquemos el patrón.

En nuestro caso, se trata del objeto principal `Juego`, que almacena
todas las partículas en su campo `partículas`. Para mover el estado
extrínseco a esta clase, debes crear varios campos matriz para almacenar
coordenadas, vectores y velocidades de cada partícula individual. Pero
eso no es todo. Necesitas otra matriz para almacenar referencias a un
objeto *flyweight* específico que represente una partícula. Estas
matrices deben estar sincronizadas para que puedas acceder a toda la
información de una partícula utilizando el mismo índice.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-es4dbe.png?id=75f4a87b985532b4c4ff42b9b6b237a1"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-es.png?id=75f4a87b985532b4c4ff42b9b6b237a1 640w,/images/patterns/diagrams/flyweight/solution2-es-1.5x.png?id=f4db4a6833accfbb0a6375136435c678 960w,/images/patterns/diagrams/flyweight/solution2-es-2x.png?id=af248bf78d7df7f36cd63f95b1ff5a32 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Solución del patrón Flyweight" />
</figure>

Una solución más elegante sería crear una clase de contexto separada que
almacene el estado extrínseco junto con la referencia al objeto
flyweight. Esta solución únicamente exigiría tener una matriz en la
clase contenedora.

¡Espera un momento! ¿No deberíamos tener tantos de estos objetos
contextuales como teníamos al principio? Técnicamente, sí. Pero el caso
es que estos objetos son mucho más pequeños que antes. Los campos que
consumen más memoria se han movido a unos pocos objetos flyweight.
Ahora, cientos de pequeños objetos contextuales pueden reutilizar un
único objeto flyweight pesado, en lugar de almacenar cientos de copias
de sus datos.

#### Flyweight y la inmutabilidad

Debido a que el mismo objeto flyweight puede utilizarse en distintos
contextos, debes asegurarte de que su estado no se pueda modificar. Un
objeto flyweight debe inicializar su estado una sola vez a través de
parámetros del constructor. No debe exponer ningún método *set*
(modificador) o campo público a otros objetos.

#### Fábrica flyweight

Para un acceso más cómodo a varios objetos flyweight, puedes crear un
método fábrica que gestione un grupo de objetos flyweight existentes. El
método acepta el estado intrínseco del flyweight deseado por un cliente,
busca un objeto flyweight existente que coincida con este estado y lo
devuelve si lo encuentra. Si no, crea un nuevo objeto flyweight y lo
añade al grupo.

Existen muchas opciones para colocar este método. El lugar más obvio es
un contenedor flyweight. Alternativamente, podrías crea un nueva clase
fábrica y hacer estático el método fábrica para colocarlo dentro de una
clase flyweight real.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Estructura del patrón de diseño Flyweight" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Estructura del patrón de diseño Flyweight" />
</figure>
:::

1.  El patrón Flyweight es simplemente una optimización. Antes de
    aplicarlo, asegúrate de que tu programa tenga un problema de consumo
    de RAM provocado por tener una gran cantidad de objetos similares en
    la memoria al mismo tiempo. Asegúrate de que este problema no se
    pueda solucionar de otra forma sensata.

2.  La clase **Flyweight** contiene la parte del estado del objeto
    original que pueden compartir varios objetos. El mismo objeto
    flyweight puede utilizarse en muchos contextos diferentes. El estado
    almacenado dentro de un objeto flyweight se denomina *intrínseco*,
    mientras que al que se pasa a sus métodos se le llama *extrínseco*.

3.  La clase **Contexto** contiene el estado extrínseco, único en todos
    los objetos originales. Cuando un contexto se empareja con uno de
    los objetos flyweight, representa el estado completo del objeto
    original.

4.  Normalmente, el comportamiento del objeto original permanece en la
    clase flyweight. En este caso, quien invoque un método del objeto
    flyweight debe también pasar las partes adecuadas del estado
    extrínseco dentro de los parámetros del método. Por otra parte, el
    comportamiento se puede mover a la clase de contexto, que utilizará
    el objeto flyweight vinculado como mero objeto de datos.

5.  El **Cliente** calcula o almacena el estado extrínseco de los
    objetos flyweight. Desde la perspectiva del cliente, un flyweight es
    un objeto plantilla que puede configurarse durante el tiempo de
    ejecución pasando información contextual dentro de los parámetros de
    sus métodos.

6.  La **Fábrica flyweight** gestiona un grupo de objetos flyweight
    existentes. Con la fábrica, los clientes no crean objetos flyweight
    directamente. En lugar de eso, invocan a la fábrica, pasándole
    partes del estado intrínseco del objeto flyweight deseado. La
    fábrica revisa objetos flyweight creados previamente y devuelve uno
    existente que coincida con los criterios de búsqueda, o bien crea
    uno nuevo si no encuentra nada.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Flyweight** ayuda a reducir el uso de
memoria a la hora de representar millones de objetos de árbol en un
lienzo.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Ejemplo del patrón Flyweight" />
</figure>

El patrón extrae el estado intrínseco repetido de una clase principal
`Árbol` y la mueve dentro de la clase flyweight `TipodeÁrbol`.

Ahora, en lugar de almacenar la misma información en varios objetos, se
mantiene en unos pocos objetos flyweight vinculados a los objetos de
`Árbol` adecuados que actúan como contexto. El código cliente crea
nuevos objetos árbol utilizando la fábrica flyweight, que encapsula la
complejidad de buscar el objeto adecuado y reutilizarlo si es necesario.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase flyweight contiene una parte del estado de un árbol.
// Estos campos almacenan valores que son únicos para cada árbol
// en particular. Por ejemplo, aquí no encontrarás las
// coordenadas del árbol. Pero la textura y los colores que
// comparten muchos árboles sí están aquí. Ya que esta cantidad
// de datos suele ser GRANDE, dedicarás mucha memoria a
// mantenerla en cada objeto árbol. En lugar de eso, podemos
// extraer la textura, el color y otros datos repetidos y
// colocarlos en un objeto independiente que muchos objetos
// individuales del árbol pueden referenciar.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Crea un mapa de bits de un tipo, color y textura
        // concretos.
        // 2. Dibuja el mapa de bits en el lienzo con las
        // coordenadas X y Y.


// La fábrica flyweight decide si reutiliza el flyweight
// existente o si crea un nuevo objeto.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// El objeto contextual contiene la parte extrínseca del estado
// del árbol. Una aplicación puede crear millones de ellas, ya
// que son muy pequeñas: dos coordenadas en números enteros y un
// campo de referencia.
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Las clases Tree y Forest son los clientes de flyweight.
// Puedes fusionarlas si no tienes la intención de desarrollar
// más la clase Tree.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::: applicability
::: applicability-problem
Utiliza el patrón Flyweight únicamente cuando tu programa deba soportar
una enorme cantidad de objetos que apenas quepan en la RAM disponible.
:::

::: applicability-solution
La ventaja de aplicar el patrón depende en gran medida de cómo y dónde
se utiliza. Resulta más útil cuando:

-   la aplicación necesita generar una cantidad enorme de objetos
    similares
-   esto consume toda la RAM disponible de un dispositivo objetivo
-   los objetos contienen estados duplicados que se pueden extraer y
    compartir entre varios objetos
:::
:::::
::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Divide los campos de una clase que se convertirá en flyweight en dos
    partes:

    -   el estado intrínseco: los campos que contienen información
        invariable duplicada a través de varios objetos
    -   el estado extrínseco: los campos que contienen información
        contextual única de cada objeto

2.  Deja los campos que representan el estado intrínseco en la clase,
    pero asegúrate de que sean inmutables. Deben llevar sus valores
    iniciales únicamente dentro del constructor.

3.  Repasa los métodos que utilizan campos del estado extrínseco. Para
    cada campo utilizado en el método, introduce un nuevo parámetro y
    utilízalo en lugar del campo.

4.  Opcionalmente, crea una clase fábrica para gestionar el grupo de
    objetos flyweight, buscando uno existente antes de crear uno nuevo.
    Una vez que la fábrica esté en su sitio, los clientes sólo deberán
    solicitar objetos flyweight a través de ella. Deberán describir el
    flyweight deseado pasando su estado intrínseco a la fábrica.

5.  El cliente deberá almacenar o calcular valores del estado extrínseco
    (contexto) para poder invocar métodos de objetos flyweight. Por
    comodidad, el estado extrínseco puede moverse a una clase contexto
    separada junto con el campo referenciador del flyweight.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes ahorrar mucha RAM, siempre que tu programa tenga toneladas
    de objetos similares.
:::

::: col-sm-6
-    Puede que estés cambiando RAM por ciclos CPU cuando deba calcularse
    de nuevo parte de la información de contexto cada vez que alguien
    invoque un método flyweight.
-    El código se complica mucho. Los nuevos miembros del equipo siempre
    estarán preguntándose por qué el estado de una entidad se separó de
    tal manera.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Puedes implementar nodos de hoja compartidos del árbol
    [Composite](composite.html) como [Flyweights](flyweight.html) para
    ahorrar memoria RAM.

-   [Flyweight](flyweight.html) muestra cómo crear muchos pequeños
    objetos, mientras que [Facade](facade.html) muestra cómo crear un
    único objeto que represente un subsistema completo.

-   [Flyweight](flyweight.html) podría asemejarse a
    [Singleton](singleton.html) si de algún modo pudieras reducir todos
    los estados compartidos de los objetos a un único objeto flyweight.
    Pero existen dos diferencias fundamentales entre estos patrones:

    1.  Solo debe haber una instancia Singleton, mientras que una clase
        *Flyweight* puede tener varias instancias con distintos estados
        intrínsecos.
    2.  El objeto *Singleton* puede ser mutable. Los objetos flyweight
        son inmutables.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Flyweight en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Flyweight en C#"){.prog-lang-link}
[![Flyweight en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Flyweight en C++"){.prog-lang-link}
[![Flyweight en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Flyweight en Go"){.prog-lang-link}
[![Flyweight en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Flyweight en Java"){.prog-lang-link}
[![Flyweight en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Flyweight en PHP"){.prog-lang-link}
[![Flyweight en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Flyweight en Python"){.prog-lang-link}
[![Flyweight en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Flyweight en Ruby"){.prog-lang-link}
[![Flyweight en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Flyweight en Rust"){.prog-lang-link}
[![Flyweight en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Flyweight en Swift"){.prog-lang-link}
[![Flyweight en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Flyweight en TypeScript"){.prog-lang-link}
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

[Proxy []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Facade](facade.html){.btn .btn-default
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
