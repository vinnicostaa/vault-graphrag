:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** en Python {#factory-method-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Factory method** es un patrón de diseño creacional que resuelve el
problema de crear objetos de producto sin especificar sus clases
concretas.

El patrón Factory Method define un método que debe utilizarse para crear
objetos, en lugar de una llamada directa al constructor (operador
`new`). Las subclases pueden sobrescribir este método para cambiar las
clases de los objetos que se crearán.

> Si no sabes la diferencia entre varios patrones y conceptos de la
> fábrica, lee nuestra [Comparación de
> fábricas](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Factory Method
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Factory Method se utiliza mucho en el
código Python. Resulta muy útil cuando necesitas proporcionar un alto
nivel de flexibilidad a tu código.

**Identificación:** Los métodos fábrica pueden ser reconocidos por
métodos de creación, que crean objetos de clases concretas, pero los
devuelven como objetos del tipo abstracto o interfaz.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Factory
Method**. Se centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-py .anchor} **main.py:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Creator(ABC):
    &quot;&quot;&quot;
    The Creator class declares the factory method that is supposed to return an
    object of a Product class. The Creator&#39;s subclasses usually provide the
    implementation of this method.
    &quot;&quot;&quot;

    @abstractmethod
    def factory_method(self):
        &quot;&quot;&quot;
        Note that the Creator may also provide some default implementation of
        the factory method.
        &quot;&quot;&quot;
        pass

    def some_operation(self) -&gt; str:
        &quot;&quot;&quot;
        Also note that, despite its name, the Creator&#39;s primary responsibility
        is not creating products. Usually, it contains some core business logic
        that relies on Product objects, returned by the factory method.
        Subclasses can indirectly change that business logic by overriding the
        factory method and returning a different type of product from it.
        &quot;&quot;&quot;

        # Call the factory method to create a Product object.
        product = self.factory_method()

        # Now, use the product.
        result = f&quot;Creator: The same creator&#39;s code has just worked with {product.operation()}&quot;

        return result


&quot;&quot;&quot;
Concrete Creators override the factory method in order to change the resulting
product&#39;s type.
&quot;&quot;&quot;


class ConcreteCreator1(Creator):
    &quot;&quot;&quot;
    Note that the signature of the method still uses the abstract product type,
    even though the concrete product is actually returned from the method. This
    way the Creator can stay independent of concrete product classes.
    &quot;&quot;&quot;

    def factory_method(self) -&gt; Product:
        return ConcreteProduct1()


class ConcreteCreator2(Creator):
    def factory_method(self) -&gt; Product:
        return ConcreteProduct2()


class Product(ABC):
    &quot;&quot;&quot;
    The Product interface declares the operations that all concrete products
    must implement.
    &quot;&quot;&quot;

    @abstractmethod
    def operation(self) -&gt; str:
        pass


&quot;&quot;&quot;
Concrete Products provide various implementations of the Product interface.
&quot;&quot;&quot;


class ConcreteProduct1(Product):
    def operation(self) -&gt; str:
        return &quot;{Result of the ConcreteProduct1}&quot;


class ConcreteProduct2(Product):
    def operation(self) -&gt; str:
        return &quot;{Result of the ConcreteProduct2}&quot;


def client_code(creator: Creator) -&gt; None:
    &quot;&quot;&quot;
    The client code works with an instance of a concrete creator, albeit through
    its base interface. As long as the client keeps working with the creator via
    the base interface, you can pass it any creator&#39;s subclass.
    &quot;&quot;&quot;

    print(f&quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
          f&quot;{creator.some_operation()}&quot;, end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    print(&quot;App: Launched with the ConcreteCreator1.&quot;)
    client_code(ConcreteCreator1())
    print(&quot;\n&quot;)

    print(&quot;App: Launched with the ConcreteCreator2.&quot;)
    client_code(ConcreteCreator2())</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

::: next
#### Leer siguiente

[Prototype en Python []{.fa
.fa-arrow-right}](../../prototype/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Builder en
Python](../../builder/python/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** en otros lenguajes

[![Factory Method en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method en C#"){.prog-lang-link}
[![Factory Method en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method en C++"){.prog-lang-link}
[![Factory Method en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method en Go"){.prog-lang-link}
[![Factory Method en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method en Java"){.prog-lang-link}
[![Factory Method en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method en PHP"){.prog-lang-link}
[![Factory Method en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method en Ruby"){.prog-lang-link}
[![Factory Method en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method en Rust"){.prog-lang-link}
[![Factory Method en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method en Swift"){.prog-lang-link}
[![Factory Method en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
