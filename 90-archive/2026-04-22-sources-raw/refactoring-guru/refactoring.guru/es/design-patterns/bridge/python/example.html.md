:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** en Python {#bridge-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Bridge** es un patrón de diseño estructural que divide la lógica de
negocio o una clase muy grande en jerarquías de clases separadas que se
pueden desarrollar independientemente.

Una de estas jerarquías (a menudo denominada Abstracción) obtendrá una
referencia a un objeto de la segunda jerarquía (Implementación). La
abstracción podrá delegar algunas (en ocasiones, la mayoría) de sus
llamadas al objeto de las implementaciones. Como todas las
implementaciones tendrán una interfaz común, serán intercambiables
dentro de la abstracción.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Bridge ](../../bridge.html){.btn .btn-lg
.btn-primary}
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

**Ejemplos de uso:** El patrón Bridge es de especial utilidad a la hora
de tratar con aplicaciones multiplataforma, soportar varios tipos de
servidores de bases de datos, o trabajar con varios proveedores de API
de un cierto tipo (por ejemplo, plataformas en la nube, redes sociales,
etc.).

**Identificación:** El patrón Bridge se puede reconocer por una
distinción clara entre alguna entidad controladora y varias plataformas
diferentes en las que se basa.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Bridge**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-py .anchor} **main.py:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Abstraction:
    &quot;&quot;&quot;
    The Abstraction defines the interface for the &quot;control&quot; part of the two
    class hierarchies. It maintains a reference to an object of the
    Implementation hierarchy and delegates all of the real work to this object.
    &quot;&quot;&quot;

    def __init__(self, implementation: Implementation) -&gt; None:
        self.implementation = implementation

    def operation(self) -&gt; str:
        return (f&quot;Abstraction: Base operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class ExtendedAbstraction(Abstraction):
    &quot;&quot;&quot;
    You can extend the Abstraction without changing the Implementation classes.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return (f&quot;ExtendedAbstraction: Extended operation with:\n&quot;
                f&quot;{self.implementation.operation_implementation()}&quot;)


class Implementation(ABC):
    &quot;&quot;&quot;
    The Implementation defines the interface for all implementation classes. It
    doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
    interfaces can be entirely different. Typically the Implementation interface
    provides only primitive operations, while the Abstraction defines higher-
    level operations based on those primitives.
    &quot;&quot;&quot;

    @abstractmethod
    def operation_implementation(self) -&gt; str:
        pass


&quot;&quot;&quot;
Each Concrete Implementation corresponds to a specific platform and implements
the Implementation interface using that platform&#39;s API.
&quot;&quot;&quot;


class ConcreteImplementationA(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.&quot;


class ConcreteImplementationB(Implementation):
    def operation_implementation(self) -&gt; str:
        return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.&quot;


def client_code(abstraction: Abstraction) -&gt; None:
    &quot;&quot;&quot;
    Except for the initialization phase, where an Abstraction object gets linked
    with a specific Implementation object, the client code should only depend on
    the Abstraction class. This way the client code can support any abstraction-
    implementation combination.
    &quot;&quot;&quot;

    # ...

    print(abstraction.operation(), end=&quot;&quot;)

    # ...


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code should be able to work with any pre-configured abstraction-
    implementation combination.
    &quot;&quot;&quot;

    implementation = ConcreteImplementationA()
    abstraction = Abstraction(implementation)
    client_code(abstraction)

    print(&quot;\n&quot;)

    implementation = ConcreteImplementationB()
    abstraction = ExtendedAbstraction(implementation)
    client_code(abstraction)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

::: next
#### Leer siguiente

[Composite en Python []{.fa
.fa-arrow-right}](../../composite/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Adapter en
Python](../../adapter/python/example.html){.btn .btn-default rel="prev"}
:::

## **Bridge** en otros lenguajes

[![Bridge en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge en C#"){.prog-lang-link}
[![Bridge en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge en C++"){.prog-lang-link}
[![Bridge en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge en Go"){.prog-lang-link}
[![Bridge en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge en Java"){.prog-lang-link}
[![Bridge en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge en PHP"){.prog-lang-link}
[![Bridge en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge en Ruby"){.prog-lang-link}
[![Bridge en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge en Rust"){.prog-lang-link}
[![Bridge en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge en Swift"){.prog-lang-link}
[![Bridge en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge en TypeScript"){.prog-lang-link}
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
