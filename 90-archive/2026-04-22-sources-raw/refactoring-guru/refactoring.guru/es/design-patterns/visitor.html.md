:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Visitor {#visitor .title}

::: aka
También llamado: [Visitante]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Visitor** es un patrón de diseño de comportamiento que te permite
separar algoritmos de los objetos sobre los que operan.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Visitor" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que tu equipo desarrolla una aplicación que funciona con
información geográfica estructurada como un enorme grafo. Cada nodo del
grafo puede representar una entidad compleja, como una ciudad, pero
también cosas más específicas, como industrias, áreas turísticas, etc.
Los nodos están conectados con otros si hay un camino entre los objetos
reales que representan. Técnicamente, cada tipo de nodo está
representado por su propia clase, mientras que cada nodo específico es
un objeto.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Exportando el grafo a XML" />
<figcaption><p>Exportando el grafo a XML.</p></figcaption>
</figure>

En cierto momento, te surge la tarea de implementar la exportación del
grafo a formato XML. Al principio, el trabajo parece bastante sencillo.
Planificaste añadir un método de exportación a cada clase de nodo y
después aprovechar la recursión para recorrer cada nodo del grafo,
ejecutando el método de exportación. La solución era sencilla y
elegante: gracias al polimorfismo, no acoplabas el código que invocaba
el método de exportación a clases concretas de nodos.

Lamentablemente, el arquitecto del sistema no te permitió alterar las
clases de nodo existentes. Dijo que el código ya estaba en producción y
no quería arriesgarse a que se descompusiera por culpa de un potencial
error en tus cambios.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-es7f25.png?id=b61742300f5cc548317a7c0b58fb9d03"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-es.png?id=b61742300f5cc548317a7c0b58fb9d03 500w,/images/patterns/diagrams/visitor/problem2-es-1.5x.png?id=42fedc5eff862db40457790663fe7c1c 750w,/images/patterns/diagrams/visitor/problem2-es-2x.png?id=98e6efce8906300fec51afc8776f738a 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="El método de exportación XML tuvo que añadirse a todas las clases de nodo" />
<figcaption><p>El método de exportación XML tuvo que añadirse a todas
las clases de nodo, lo que supuso el riesgo de descomponer la aplicación
si se introducía algún error con el cambio.</p></figcaption>
</figure>

Además, cuestionó si tenía sentido tener el código de exportación XML
dentro de las clases de nodo. El trabajo principal de estas clases era
trabajar con geodatos. El comportamiento de la exportación XML
resultaría extraño ahí.

Había otra razón para el rechazo. Era muy probable que, una vez que se
implementara esta función, alguien del departamento de marketing te
pidiera que incluyeras la capacidad de exportar a otros formatos, o te
pidiera alguna otra cosa extraña. Esto te forzaría a cambiar de nuevo
esas preciadas y frágiles clases.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Visitor sugiere que coloques el nuevo comportamiento en una
clase separada llamada *visitante*, en lugar de intentar integrarlo
dentro de clases existentes. El objeto que originalmente tenía que
realizar el comportamiento se pasa ahora a uno de los métodos del
visitante como argumento, de modo que el método accede a toda la
información necesaria contenida dentro del objeto.

Ahora, ¿qué pasa si ese comportamiento puede ejecutarse sobre objetos de
clases diferentes? Por ejemplo, en nuestro caso con la exportación XML,
la implementación real probablemente sería un poco diferente en varias
clases de nodo. Por lo tanto, la clase visitante puede definir un grupo
de métodos en lugar de uno solo, y cada uno de ellos podría tomar
argumentos de distintos tipos, así:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Pero, ¿cómo llamaríamos exactamente a estos métodos, sobre todo al
lidiar con el grafo completo? Estos métodos tienen distintas firmas, por
lo que no podemos utilizar el polimorfismo. Para elegir un método
visitante adecuado que sea capaz de procesar un objeto dado, debemos
revisar su clase. ¿No suena esto como una pesadilla?

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}</code></pre>
</figure>

Puede que te preguntes, ¿por qué no utilizar la sobrecarga de métodos?
Eso es cuando le das a todos los métodos el mismo nombre, incluso cuando
soportan distintos grupos de parámetros. Lamentablemente, incluso
asumiendo que nuestro lenguaje de programación la soportara (como Java y
C#), no nos ayudaría. Debido a que la clase exacta de un objeto tipo
nodo es desconocida de antemano, el mecanismo de sobrecarga no será
capaz de determinar el método correcto a ejecutar. Recurrirá por defecto
al método que toma un objeto de la clase base `Nodo`.

Sin embargo, el patrón Visitor ataja este problema. Utiliza una técnica
llamada [Double Dispatch](visitor-double-dispatch.html), que ayuda a
ejecutar el método adecuado sobre un objeto sin complicados
condicionales. En lugar de permitir al cliente seleccionar una versión
adecuada del método a llamar, ¿qué tal si delegamos esta opción a los
objetos que pasamos al visitante como argumento? Como estos objetos
conocen sus propias clases, podrán elegir un método adecuado en el
visitante más fácilmente. "Aceptan" un visitante y le dicen qué método
visitante debe ejecutarse.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Código cliente
foreach (Node node in graph)
    node.accept(exportVisitor)

// Ciudad
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Industria
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

Lo confieso. Hemos tenido que cambiar las clases de nodo, después de
todo. Pero al menos el cambio es trivial y nos permite añadir más
comportamientos sin alterar el código otra vez.

Ahora, si extraemos una interfaz común para todos los visitantes, todos
los nodos existentes pueden funcionar con cualquier visitante que
introduzcas en la aplicación. Si te encuentras introduciendo un nuevo
comportamiento relacionado con los nodos, todo lo que tienes que hacer
es implementar una nueva clase visitante.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Agente de seguros" />
<figcaption><p>Un buen agente de seguros siempre está listo para ofrecer
pólizas diferentes a los distintos tipos
de organizaciones.</p></figcaption>
</figure>

Imagina un experimentado agente de seguros que está deseoso de conseguir
nuevos clientes. Puede visitar todos los edificios de un barrio,
intentando vender seguros a todo aquel que se va encontrando.
Dependiendo del tipo de organización que ocupe el edificio, puede
ofrecer pólizas de seguro especializadas:

-   Si es un edificio residencial, vende seguros médicos.
-   Si es un banco, vende seguros contra robos.
-   Si es una cafetería, vende seguros contra incendios e inundaciones.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-es974e.png?id=617cb8f5476ab116b73bbbfa7dbf7ec2"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-es.png?id=617cb8f5476ab116b73bbbfa7dbf7ec2 520w,/images/patterns/diagrams/visitor/structure-es-1.5x.png?id=9be251912f8c4fa9cebcd5178fdeaa95 780w,/images/patterns/diagrams/visitor/structure-es-2x.png?id=1f36e20b5238df6c1fefacdd72e41941 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estructura del patrón de diseño Visitor" /><img
src="../../images/patterns/diagrams/visitor/structure-es-indexedd02b.png?id=428f034fd018ea7a337182b8550e3c64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-es-indexed.png?id=428f034fd018ea7a337182b8550e3c64 520w,/images/patterns/diagrams/visitor/structure-es-indexed-1.5x.png?id=6a36c23d9fc49e045b56715b6df8e324 780w,/images/patterns/diagrams/visitor/structure-es-indexed-2x.png?id=1279881214c6f7b4f888bf1fd872ae5f 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Estructura del patrón de diseño Visitor" />
</figure>
:::

1.  La interfaz **Visitante** declara un grupo de métodos visitantes que
    pueden tomar elementos concretos de una estructura de objetos como
    argumentos. Estos métodos pueden tener los mismos nombres si el
    programa está escrito en un lenguaje que soporte la sobrecarga, pero
    los tipos de sus parámetros deben ser diferentes.

2.  Cada **Visitante Concreto** implementa varias versiones de los
    mismos comportamientos, personalizadas para las distintas clases de
    elemento concreto.

3.  La interfaz **Elemento** declara un método para "aceptar"
    visitantes. Este método deberá contar con un parámetro declarado con
    el tipo de la interfaz visitante.

4.  Cada **Elemento Concreto** debe implementar el método de aceptación.
    El propósito de este método es redirigir la llamada al método
    adecuado del visitante correspondiente a la clase de elemento
    actual. Piensa que, aunque una clase base de elemento implemente
    este método, todas las subclases deben sobrescribir este método en
    sus propias clases e invocar el método adecuado en el objeto
    visitante.

5.  El **Cliente** representa normalmente una colección o algún otro
    objeto complejo (por ejemplo, un árbol [Composite](composite.html)).
    A menudo, los clientes no son conscientes de todas las clases de
    elemento concreto porque trabajan con objetos de esa colección a
    través de una interfaz abstracta.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Visitor** añade soporte de exportación XML
a la jerarquía de clases de formas geométricas.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Ejemplo de estructura del patrón Visitor" />
<figcaption><p>Exportar varios tipos de objetos a formato XML a través
de un objeto visitante.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La interfaz elemento declara un método `accept` (aceptar) que
// toma la interfaz visitante base como argumento.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Cada clase de elemento concreto debe implementar el método
// `accept` de tal manera que invoque el método del visitante
// que corresponde a la clase del elemento.
class Dot implements Shape is
    // ...

    // Observa que invocamos `visitDot`, que coincide con el
    // nombre de la clase actual. De esta forma, hacemos saber
    // al visitante la clase del elemento con el que trabaja.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// La interfaz Visitor declara un grupo de métodos de visita que
// se corresponden con clases de elemento. La firma de un método
// de visita permite al visitante identificar la clase exacta
// del elemento con el que trata.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Los visitantes concretos implementan varias versiones del
// mismo algoritmo, que puede funcionar con todas las clases de
// elementos concretos.
//
// Puedes disfrutar de la mayor ventaja del patrón Visitor si lo
// utilizas con una estructura compleja de objetos, como un
// árbol Composite. En este caso, puede ser de ayuda almacenar
// algún estado intermedio del algoritmo mientras ejecutas los
// métodos del visitante sobre varios objetos de la estructura.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Exporta la ID del punto (dot) y centra las
        // coordenadas.

    method visitCircle(c: Circle) is
        // Exporta la ID del círculo y centra las coordenadas y
        // el radio.

    method visitRectangle(r: Rectangle) is
        // Exporta la ID del rectángulo, las coordenadas de
        // arriba a la izquierda, la anchura y la altura.

    method visitCompoundShape(cs: CompoundShape) is
        // Exporta la ID de la forma, así como la lista de las
        // ID de sus hijos.


// El código cliente puede ejecutar operaciones del visitante
// sobre cualquier grupo de elementos sin conocer sus clases
// concretas. La operación `accept` dirige una llamada a la
// operación adecuada del objeto visitante.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Si te preguntas por qué necesitamos el método `aceptar` en este ejemplo,
mi artículo [Visitor y Double Dispatch](visitor-double-dispatch.html)
aborda esta cuestión en detalle.
:::

:::::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::::: applicability
::: applicability-problem
Utiliza el patrón Visitor cuando necesites realizar una operación sobre
todos los elementos de una compleja estructura de objetos (por ejemplo,
un árbol de objetos).
:::

::: applicability-solution
El patrón Visitor te permite ejecutar una operación sobre un grupo de
objetos con diferentes clases, haciendo que un objeto visitante
implemente distintas variantes de la misma operación que correspondan a
todas las clases objetivo.
:::

::: applicability-problem
Utiliza el patrón Visitor para limpiar la lógica de negocio de
comportamientos auxiliares.
:::

::: applicability-solution
El patrón te permite hacer que las clases primarias de tu aplicación
estén más centradas en sus trabajos principales extrayendo el resto de
los comportamientos y poniéndolos dentro de un grupo de clases
visitantes.
:::

::: applicability-problem
Utiliza el patrón cuando un comportamiento solo tenga sentido en algunas
clases de una jerarquía de clases, pero no en otras.
:::

::: applicability-solution
Puedes extraer este comportamiento y ponerlo en una clase visitante
separada e implementar únicamente aquellos métodos visitantes que
acepten objetos de clases relevantes, dejando el resto vacíos.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Declara la interfaz visitante con un grupo de métodos "visitantes",
    uno por cada clase de elemento concreto existente en el programa.

2.  Declara la interfaz de elemento. Si estás trabajando con una
    jerarquía de clases de elemento existente, añade el método abstracto
    de "aceptación" a la clase base de la jerarquía. Este método debe
    aceptar un objeto visitante como argumento.

3.  Implementa los métodos de aceptación en todas las clases de elemento
    concreto. Estos métodos simplemente deben redirigir la llamada a un
    método visitante en el objeto visitante entrante que coincida con la
    clase del elemento actual.

4.  Las clases de elemento sólo deben funcionar con visitantes a través
    de la interfaz visitante. Los visitantes, sin embargo, deben conocer
    todas las clases de elemento concreto, referenciadas como tipos de
    parámetro de los métodos de visita.

5.  Por cada comportamiento que no pueda implementarse dentro de la
    jerarquía de elementos, crea una nueva clase concreta visitante e
    implementa todos los métodos visitantes.

    Puede que te encuentres una situación en la que el visitante
    necesite acceso a algunos miembros privados de la clase elemento. En
    este caso, puedes hacer estos campos o métodos públicos, violando la
    encapsulación del elemento, o anidar la clase visitante en la clase
    elemento. Esto último sólo es posible si tienes la suerte de
    trabajar con un lenguaje de programación que soporte clases
    anidadas.

6.  El cliente debe crear objetos visitantes y pasarlos dentro de
    elementos a través de métodos de "aceptación".
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    *Principio de abierto/cerrado*. Puedes introducir un nuevo
    comportamiento que puede funcionar con objetos de clases diferentes
    sin cambiar esas clases.
-    *Principio de responsabilidad única*. Puedes tomar varias versiones
    del mismo comportamiento y ponerlas en la misma clase.
-    Un objeto visitante puede acumular cierta información útil mientras
    trabaja con varios objetos. Esto puede resultar útil cuando quieras
    atravesar una compleja estructura de objetos, como un árbol de
    objetos, y aplicar el visitante a cada objeto de esa estructura.
:::

::: col-sm-6
-    Debes actualizar todos los visitantes cada vez que una clase se
    añada o elimine de la jerarquía de elementos.
-    Los visitantes pueden carecer del acceso necesario a los campos y
    métodos privados de los elementos con los que se supone que deben
    trabajar.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Puedes tratar a [Visitor](visitor.html) como una versión potente del
    patrón [Command](command.html). Sus objetos pueden ejecutar
    operaciones sobre varios objetos de distintas clases.

-   Puedes utilizar el patrón [Visitor](visitor.html) para ejecutar una
    operación sobre un árbol [Composite](composite.html) entero.

-   Puedes utilizar [Visitor](visitor.html) junto con
    [Iterator](iterator.html) para recorrer una estructura de datos
    compleja y ejecutar alguna operación sobre sus elementos, incluso
    aunque todos tengan clases distintas.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Visitor en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Visitor en C#"){.prog-lang-link}
[![Visitor en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Visitor en C++"){.prog-lang-link}
[![Visitor en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Visitor en Go"){.prog-lang-link}
[![Visitor en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Visitor en Java"){.prog-lang-link}
[![Visitor en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Visitor en PHP"){.prog-lang-link}
[![Visitor en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Visitor en Python"){.prog-lang-link}
[![Visitor en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Visitor en Ruby"){.prog-lang-link}
[![Visitor en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Visitor en Rust"){.prog-lang-link}
[![Visitor en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Visitor en Swift"){.prog-lang-link}
[![Visitor en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Visitor en TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Contenido adicional {#extras}

-   ¿Te extraña que no podamos simplemente sustituir el patrón Visitor
    por la sobrecarga de métodos? Lee mi artículo [Visitor y Double
    Dispatch](visitor-double-dispatch.html) para conocer los
    desagradables detalles.
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

[Visitor y Double Dispatch []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Template Method](template-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
