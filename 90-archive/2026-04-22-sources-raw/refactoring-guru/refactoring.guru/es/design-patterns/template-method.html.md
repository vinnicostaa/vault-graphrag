::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type}
:::

# Template Method {#template-method .title}

::: aka
También llamado: [Método plantilla]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Template Method** es un patrón de diseño de comportamiento que define
el esqueleto de un algoritmo en la superclase pero permite que las
subclases sobrescriban pasos del algoritmo sin cambiar su estructura.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón de diseño Template method" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

Imagina que estás creando una aplicación de minería de datos que analiza
documentos corporativos. Los usuarios suben a la aplicación documentos
en varios formatos (PDF, DOC, CSV) y ésta intenta extraer la información
relevante de estos documentos en un formato uniforme.

La primera versión de la aplicación sólo funcionaba con archivos DOC. La
siguiente versión podía soportar archivos CSV. Un mes después, le
"enseñaste" a extraer datos de archivos PDF.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Las clases de minería de datos contenían mucho código duplicado" />
<figcaption><p>Las clases de minería de datos contenían mucho
código duplicado.</p></figcaption>
</figure>

En cierto momento te das cuenta de que las tres clases tienen mucho
código similar. Aunque el código para gestionar distintos formatos de
datos es totalmente diferente en todas las clases, el código para
procesar y analizar los datos es casi idéntico. ¿No sería genial
deshacerse de la duplicación de código, dejando intacta la estructura
del algoritmo?

Hay otro problema relacionado con el código cliente que utiliza esas
clases. Tiene muchos condicionales que eligen un curso de acción
adecuado dependiendo de la clase del objeto de procesamiento. Si las
tres clases de procesamiento tienen una interfaz común o una clase base,
puedes eliminar los condicionales en el código cliente y utilizar el
polimorfismo al invocar métodos en un objeto de procesamiento.
:::

::: {.section .solution}
##  Solución {#solution}

El patrón Template Method sugiere que dividas un algoritmo en una serie
de pasos, conviertas estos pasos en métodos y coloques una serie de
llamadas a esos métodos dentro de un único *método plantilla*. Los pasos
pueden ser `abstractos`, o contar con una implementación por defecto.
Para utilizar el algoritmo, el cliente debe aportar su propia subclase,
implementar todos los pasos abstractos y sobrescribir algunos de los
opcionales si es necesario (pero no el propio método plantilla).

Veamos cómo funciona en nuestra aplicación de minería de datos. Podemos
crear una clase base para los tres algoritmos de análisis. Esta clase
define un método plantilla consistente en una serie de llamadas a varios
pasos de procesamiento de documentos.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-es5e49.png?id=8d325dc52eecda5564b3f2766c15b91c"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-es.png?id=8d325dc52eecda5564b3f2766c15b91c 600w,/images/patterns/diagrams/template-method/solution-es-1.5x.png?id=e597be3eed36b96ce90ead9214cf8b82 900w,/images/patterns/diagrams/template-method/solution-es-2x.png?id=08d8d16899e75e62d28f6183e461ece0 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="El método plantilla define el esqueleto del algoritmo" />
<figcaption><p>El método plantilla divide el algoritmo en pasos,
permitiendo a las subclases sobrescribir estos pasos pero no el método
en sí.</p></figcaption>
</figure>

Al principio, podemos declarar todos los pasos como `abstractos`,
forzando a las subclases a proporcionar sus propias implementaciones
para estos métodos. En nuestro caso, las subclases ya cuentan con todas
las implementaciones necesarias, por lo que lo único que tendremos que
hacer es ajustar las firmas de los métodos para que coincidan con los
métodos de la superclase.

Ahora, veamos lo que podemos hacer para deshacernos del código
duplicado. Parece que el código para abrir/cerrar archivos y
extraer/analizar información es diferente para varios formatos de datos,
por lo que no tiene sentido tocar estos métodos. No obstante, la
implementación de otros pasos, como analizar los datos sin procesar y
generar informes, es muy similar, por lo que puede meterse en la clase
base, donde las subclases pueden compartir ese código.

Como puedes ver, tenemos dos tipos de pasos:

-   Los *pasos abstractos* deben ser implementados por todas las
    subclases
-   Los *pasos opcionales* ya tienen cierta implementación por defecto,
    pero aún así pueden sobrescribirse si es necesario

Hay otro tipo de pasos, llamados ganchos (*hooks*). Un gancho es un paso
opcional con un cuerpo vacío. Un método plantilla funcionará aunque el
gancho no se sobrescriba. Normalmente, los ganchos se colocan antes y
después de pasos cruciales de los algoritmos, suministrando a las
subclases puntos adicionales de extensión para un algoritmo.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Construcción de viviendas en masa" />
<figcaption><p>Un plan arquitectónico típico puede alterarse ligeramente
para que encaje mejor con las necesidades del cliente.</p></figcaption>
</figure>

El enfoque del método plantilla puede emplearse en la construcción de
viviendas en masa. El plan arquitectónico para construir una casa
estándar puede contener varios puntos de extensión que permitirán a un
potencial propietario ajustar algunos detalles de la casa resultante.

Cada paso de la construcción, como colocar los cimientos, el armazón,
construir las paredes, instalar las tuberías para el agua y el cableado
para la electricidad, etc., puede cambiarse ligeramente para que la casa
resultante sea un poco diferente de las demás.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Estructura del patrón de diseño Template Method" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Estructura del patrón de diseño Template Method" />
</figure>
:::

1.  La **Clase Abstracta** declara métodos que actúan como pasos de un
    algoritmo, así como el propio método plantilla que invoca estos
    métodos en un orden específico. Los pasos pueden declararse
    `abstractos` o contar con una implementación por defecto.

2.  Las **Clases Concretas** pueden sobrescribir todos los pasos, pero
    no el propio método plantilla.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, el patrón **Template Method** proporciona un
"esqueleto" para varias ramas de inteligencia artificial (IA) en un
sencillo videojuego de estrategia.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Ejemplo de estructura del patrón Template Method" />
<figcaption><p>Clases IA de un sencillo videojuego.</p></figcaption>
</figure>

Todas las razas del juego tienen tipos de unidades y edificios casi
iguales. Por lo tanto, puedes reutilizar la misma estructura IA para
varias de ellas, a la vez que puedes sobrescribir algunos de los
detalles. Con esta solución, puedes sobrescribir la IA de los orcos para
que sean más agresivos, hacer que los humanos tengan una actitud más
defensiva y hacer que los monstruos no puedan construir nada. Para
añadir una nueva raza al juego habría que crear una nueva subclase IA y
sobrescribir los métodos por defecto declarados en la clase IA base.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase abstracta define un método plantilla que contiene un
// esqueleto de algún algoritmo compuesto por llamadas,
// normalmente a operaciones primitivas abstractas. Las
// subclases concretas implementan estas operaciones, pero dejan
// el propio método plantilla intacto.
class GameAI is
    // El método plantilla define el esqueleto de un algoritmo.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Algunos de los pasos se pueden implementar directamente
    // en una clase base.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // Y algunos de ellos pueden definirse como abstractos.
    abstract method buildStructures()
    abstract method buildUnits()

    // Una clase puede tener varios métodos plantilla.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Las clases concretas tienen que implementar todas las
// operaciones abstractas de la clase base, pero no deben
// sobrescribir el propio método plantilla.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Construye granjas, después cuarteles y después
            // fortaleza.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Crea peón y añádelo al grupo de exploradores.
            else
                // Crea soldado, añádelo al grupo de guerreros.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Envía exploradores a posición.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Envía guerreros a posición.

// Las subclases también pueden sobrescribir algunas operaciones
// con una implementación por defecto.
class MonstersAI extends GameAI is
    method collectResources() is
        // Los monstruos no recopilan recursos.

    method buildStructures() is
        // Los monstruos no construyen estructuras.

    method buildUnits() is
        // Los monstruos no construyen unidades.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Template Method cuando quieras permitir a tus clientes
que extiendan únicamente pasos particulares de un algoritmo, pero no
todo el algoritmo o su estructura.
:::

::: applicability-solution
El patrón Template Method te permite convertir un algoritmo monolítico
en una serie de pasos individuales que se pueden extender fácilmente con
subclases, manteniendo intacta la estructura definida en una superclase.
:::

::: applicability-problem
Utiliza el patrón cuando tengas muchas clases que contengan algoritmos
casi idénticos, pero con algunas diferencias mínimas. Como resultado,
puede que tengas que modificar todas las clases cuando el algoritmo
cambie.
:::

::: applicability-solution
Cuando conviertes un algoritmo así en un método plantilla, también
puedes elevar los pasos con implementaciones similares a una superclase,
eliminando la duplicación del código. El código que varía entre
subclases puede permanecer en las subclases.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Analiza el algoritmo objetivo para ver si puedes dividirlo en pasos.
    Considera qué pasos son comunes a todas las subclases y cuáles
    siempre serán únicos.

2.  Crea la clase base abstracta y declara el método plantilla y un
    grupo de métodos abstractos que representen los pasos del algoritmo.
    Perfila la estructura del algoritmo en el método plantilla
    ejecutando los pasos correspondientes. Considera declarar el método
    plantilla como `final` para evitar que las subclases lo
    sobrescriban.

3.  No hay problema en que todos los pasos acaben siendo abstractos. Sin
    embargo, a algunos pasos les vendría bien tener una implementación
    por defecto. Las subclases no tienen que implementar esos métodos.

4.  Piensa en añadir ganchos entre los pasos cruciales del algoritmo.

5.  Para cada variación del algoritmo, crea una nueva subclase concreta.
    Ésta *debe* implementar todos los pasos abstractos, pero también
    *puede* sobrescribir algunos de los opcionales.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes permitir a los clientes que sobrescriban tan solo ciertas
    partes de un algoritmo grande, para que les afecten menos los
    cambios que tienen lugar en otras partes del algoritmo.
-    Puedes colocar el código duplicado dentro de una superclase.
:::

::: col-sm-6
-    Algunos clientes pueden verse limitados por el esqueleto
    proporcionado de un algoritmo.
-    Puede que violes el *principio de sustitución de Liskov*
    suprimiendo una implementación por defecto de un paso a través de
    una subclase.
-    Los métodos plantilla tienden a ser más difíciles de mantener
    cuantos más pasos tengan.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   [Factory Method](factory-method.html) es una especialización del
    [Template Method](template-method.html). Al mismo tiempo, un
    *Factory Method* puede servir como paso de un gran *Template
    Method*.

-   [Template Method](template-method.html) se basa en la herencia: te
    permite alterar partes de un algoritmo extendiendo esas partes en
    subclases. [Strategy](strategy.html) se basa en la composición:
    puedes alterar partes del comportamiento del objeto suministrándole
    distintas estrategias que se correspondan con ese comportamiento.
    *Template Method* trabaja al nivel de la clase, por lo que es
    estático. *Strategy* trabaja al nivel del objeto, permitiéndote
    cambiar los comportamientos durante el tiempo de ejecución.
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Template Method en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Template Method en C#"){.prog-lang-link}
[![Template Method en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Template Method en C++"){.prog-lang-link}
[![Template Method en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Template Method en Go"){.prog-lang-link}
[![Template Method en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Template Method en Java"){.prog-lang-link}
[![Template Method en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Template Method en PHP"){.prog-lang-link}
[![Template Method en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Template Method en Python"){.prog-lang-link}
[![Template Method en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Template Method en Ruby"){.prog-lang-link}
[![Template Method en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Template Method en Rust"){.prog-lang-link}
[![Template Method en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Template Method en Swift"){.prog-lang-link}
[![Template Method en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Template Method en TypeScript"){.prog-lang-link}
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

[Visitor []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Strategy](strategy.html){.btn .btn-default
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
