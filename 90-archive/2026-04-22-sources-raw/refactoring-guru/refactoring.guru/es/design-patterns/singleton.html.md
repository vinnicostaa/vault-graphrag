::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: aka
También llamado: [Instancia única]{style="display:inline-block"}
:::

::: {.section .intent}
##  Propósito {#intent}

**Singleton** es un patrón de diseño creacional que nos permite
asegurarnos de que una clase tenga una única instancia, a la vez que
proporciona un punto de acceso global a dicha instancia.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Patrón Singleton" />
</figure>
:::

::: {.section .problem}
##  Problema {#problem}

El patrón Singleton resuelve dos problemas al mismo tiempo, vulnerando
el *Principio de responsabilidad única*:

1.  **Garantizar que una clase tenga una única instancia**. ¿Por qué
    querría alguien controlar cuántas instancias tiene una clase? El
    motivo más habitual es controlar el acceso a algún recurso
    compartido, por ejemplo, una base de datos o un archivo.

    Funciona así: imagina que has creado un objeto y al cabo de un
    tiempo decides crear otro nuevo. En lugar de recibir un objeto
    nuevo, obtendrás el que ya habías creado.

    Ten en cuenta que este comportamiento es imposible de implementar
    con un constructor normal, ya que una llamada al constructor siempre
    **debe** devolver un nuevo objeto por diseño.

<figure class="image">
<img
src="../../images/patterns/content/singleton/singleton-comic-1-esc5e1.png?id=cc859f28938dcdbd30d5149a9d916060"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-es.png?id=cc859f28938dcdbd30d5149a9d916060 600w,/images/patterns/content/singleton/singleton-comic-1-es-1.5x.png?id=1a22c57c6e6d8d9c6f70619886c1e5b2 900w,/images/patterns/content/singleton/singleton-comic-1-es-2x.png?id=17c35e1810803a004b3c0d0c92390eb1 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="El acceso global a un objeto" />
<figcaption><p>Puede ser que los clientes ni siquiera se den cuenta de
que trabajan con el mismo objeto todo el tiempo.</p></figcaption>
</figure>

2.  **Proporcionar un punto de acceso global a dicha instancia**.
    ¿Recuerdas esas variables globales que utilizaste (bueno, sí, fui
    yo) para almacenar objetos esenciales? Aunque son muy útiles,
    también son poco seguras, ya que cualquier código podría
    sobrescribir el contenido de esas variables y descomponer la
    aplicación.

    Al igual que una variable global, el patrón Singleton nos permite
    acceder a un objeto desde cualquier parte del programa. No obstante,
    también evita que otro código sobreescriba esa instancia.

    Este problema tiene otra cara: no queremos que el código que
    resuelve el primer problema se encuentre disperso por todo el
    programa. Es mucho más conveniente tenerlo dentro de una clase,
    sobre todo si el resto del código ya depende de ella.

Hoy en día el patrón Singleton se ha popularizado tanto que la gente
suele llamar *singleton* a cualquier patrón, incluso si solo resuelve
uno de los problemas antes mencionados.
:::

::: {.section .solution}
##  Solución {#solution}

Todas las implementaciones del patrón Singleton tienen estos dos pasos
en común:

-   Hacer privado el constructor por defecto para evitar que otros
    objetos utilicen el operador `new` con la clase Singleton.
-   Crear un método de creación estático que actúe como constructor.
    Tras bambalinas, este método invoca al constructor privado para
    crear un objeto y lo guarda en un campo estático. Las siguientes
    llamadas a este método devuelven el objeto almacenado en caché.

Si tu código tiene acceso a la clase Singleton, podrá invocar su método
estático. De esta manera, cada vez que se invoque este método, siempre
se devolverá el mismo objeto.
:::

::: {.section .analogy}
##  Analogía en el mundo real {#analogy}

El gobierno es un ejemplo excelente del patrón Singleton. Un país sólo
puede tener un gobierno oficial. Independientemente de las identidades
personales de los individuos que forman el gobierno, el título "Gobierno
de X" es un punto de acceso global que identifica al grupo de personas a
cargo.
:::

::::: {.section .structure-container}
##  Estructura {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/singleton/structure-esf207.png?id=5c4efb4d96f6c36272407bf076902e92"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-es.png?id=5c4efb4d96f6c36272407bf076902e92 430w,/images/patterns/diagrams/singleton/structure-es-1.5x.png?id=355b36ab6a860e1953cd8429c0317738 645w,/images/patterns/diagrams/singleton/structure-es-2x.png?id=29076977d3b61d6a345c4cd982ac180b 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="La estructura del patrón Singleton" /><img
src="../../images/patterns/diagrams/singleton/structure-es-indexedb37a.png?id=50bd72d8b40c167970ab2174d9b1b266"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-es-indexed.png?id=50bd72d8b40c167970ab2174d9b1b266 430w,/images/patterns/diagrams/singleton/structure-es-indexed-1.5x.png?id=34d23f475fadcbdbd233c5b82d902056 645w,/images/patterns/diagrams/singleton/structure-es-indexed-2x.png?id=25e94bfadc47aa9560915d215292c553 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="La estructura del patrón Singleton" />
</figure>
:::

1.  La clase **Singleton** declara el método estático `obtenerInstancia`
    que devuelve la misma instancia de su propia clase.

    El constructor del Singleton debe ocultarse del código cliente. La
    llamada al método `obtenerInstancia` debe ser la única manera de
    obtener el objeto de Singleton.
::::
:::::

::: {.section .pseudocode}
##  Pseudocódigo {#pseudocode}

En este ejemplo, la clase de conexión de la base de datos actúa como
**Singleton**. Esta clase no tiene un constructor público, por lo que la
única manera de obtener su objeto es invocando el método
`obtenerInstancia`. Este método almacena en caché el primer objeto
creado y lo devuelve en todas las llamadas siguientes.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// La clase Base de datos define el método `obtenerInstancia`
// que permite a los clientes acceder a la misma instancia de
// una conexión de la base de datos a través del programa.
class Database is
    // El campo para almacenar la instancia singleton debe
    // declararse estático.
    private static field instance: Database

    // El constructor del singleton siempre debe ser privado
    // para evitar llamadas de construcción directas con el
    // operador `new`.
    private constructor Database() is
        // Algún código de inicialización, como la propia
        // conexión al servidor de una base de datos.
        // ...

    // El método estático que controla el acceso a la instancia
    // singleton.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Garantiza que la instancia aún no se ha
                // inicializado por otro hilo mientras ésta ha
                // estado esperando el desbloqueo.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Por último, cualquier singleton debe definir cierta
    // lógica de negocio que pueda ejecutarse en su instancia.
    public method query(sql) is
        // Por ejemplo, todas las consultas a la base de datos
        // de una aplicación pasan por este método. Por lo
        // tanto, aquí puedes colocar lógica de regularización
        // (throttling) o de envío a la memoria caché.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // La variable `bar` contendrá el mismo objeto que la
        // variable `foo`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Aplicabilidad {#applicability}

::::::: applicability
::: applicability-problem
Utiliza el patrón Singleton cuando una clase de tu programa tan solo
deba tener una instancia disponible para todos los clientes; por
ejemplo, un único objeto de base de datos compartido por distintas
partes del programa.
:::

::: applicability-solution
El patrón Singleton deshabilita el resto de las maneras de crear objetos
de una clase, excepto el método especial de creación. Este método crea
un nuevo objeto, o bien devuelve uno existente si ya ha sido creado.
:::

::: applicability-problem
Utiliza el patrón Singleton cuando necesites un control más estricto de
las variables globales.
:::

::: applicability-solution
Al contrario que las variables globales, el patrón Singleton garantiza
que haya una única instancia de una clase. A excepción de la propia
clase Singleton, nada puede sustituir la instancia en caché.

Ten en cuenta que siempre podrás ajustar esta limitación y permitir la
creación de cierto número de instancias Singleton. La única parte del
código que requiere cambios es el cuerpo del método `getInstance`.
:::
:::::::
::::::::

::: {.section .checklist}
##  Cómo implementarlo {#checklist}

1.  Añade un campo estático privado a la clase para almacenar la
    instancia Singleton.

2.  Declara un método de creación estático público para obtener la
    instancia Singleton.

3.  Implementa una inicialización diferida dentro del método estático.
    Debe crear un nuevo objeto en su primera llamada y colocarlo dentro
    del campo estático. El método deberá devolver siempre esa instancia
    en todas las llamadas siguientes.

4.  Declara el constructor de clase como privado. El método estático de
    la clase seguirá siendo capaz de invocar al constructor, pero no a
    los otros objetos.

5.  Repasa el código cliente y sustituye todas las llamadas directas al
    constructor de la instancia Singleton por llamadas a su método de
    creación estático.
:::

:::::: {.section .pros-cons}
##  Pros y contras {#pros-cons}

::::: row
::: col-sm-6
-    Puedes tener la certeza de que una clase tiene una única instancia.
-    Obtienes un punto de acceso global a dicha instancia.
-    El objeto Singleton solo se inicializa cuando se requiere por
    primera vez.
:::

::: col-sm-6
-    Vulnera el *Principio de responsabilidad única*. El patrón resuelve
    dos problemas al mismo tiempo.
-    El patrón Singleton puede enmascarar un mal diseño, por ejemplo,
    cuando los componentes del programa saben demasiado los unos sobre
    los otros.
-    El patrón requiere de un tratamiento especial en un entorno con
    múltiples hilos de ejecución, para que varios hilos no creen un
    objeto Singleton varias veces.
-    Puede resultar complicado realizar la prueba unitaria del código
    cliente del Singleton porque muchos *frameworks* de prueba dependen
    de la herencia a la hora de crear objetos simulados (mock objects).
    Debido a que la clase Singleton es privada y en la mayoría de los
    lenguajes resulta imposible sobrescribir métodos estáticos, tendrás
    que pensar en una manera original de simular el Singleton. O,
    simplemente, no escribas las pruebas. O no utilices el patrón
    Singleton.
:::
:::::
::::::

::: {.section .relations}
##  Relaciones con otros patrones {#relations}

-   Una clase [fachada](facade.html) a menudo puede transformarse en una
    [Singleton](singleton.html), ya que un único objeto fachada es
    suficiente en la mayoría de los casos.

-   [Flyweight](flyweight.html) podría asemejarse a
    [Singleton](singleton.html) si de algún modo pudieras reducir todos
    los estados compartidos de los objetos a un único objeto flyweight.
    Pero existen dos diferencias fundamentales entre estos patrones:

    1.  Solo debe haber una instancia Singleton, mientras que una clase
        *Flyweight* puede tener varias instancias con distintos estados
        intrínsecos.
    2.  El objeto *Singleton* puede ser mutable. Los objetos flyweight
        son inmutables.

-   Los patrones [Abstract Factory](abstract-factory.html),
    [Builder](builder.html) y [Prototype](prototype.html) pueden todos
    ellos implementarse como [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Ejemplos de código {#implementations}

[![Singleton en
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton en C#"){.prog-lang-link}
[![Singleton en
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton en C++"){.prog-lang-link}
[![Singleton en
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton en Go"){.prog-lang-link}
[![Singleton en
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton en Java"){.prog-lang-link}
[![Singleton en
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton en PHP"){.prog-lang-link}
[![Singleton en
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton en Python"){.prog-lang-link}
[![Singleton en
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton en Ruby"){.prog-lang-link}
[![Singleton en
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton en Rust"){.prog-lang-link}
[![Singleton en
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton en Swift"){.prog-lang-link}
[![Singleton en
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton en TypeScript"){.prog-lang-link}
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

[Patrones estructurales []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Prototype](prototype.html){.btn .btn-default
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
