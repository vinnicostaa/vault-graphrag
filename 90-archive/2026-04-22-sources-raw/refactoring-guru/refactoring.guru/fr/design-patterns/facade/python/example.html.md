:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Façade](../../facade.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Façade](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Façade** en Python {#façade-en-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
La **Façade** est un patron de conception structurel qui fournit une
interface simplifiée (mais limitée) à un système complexe de classes,
bibliothèques ou frameworks.

La façade permet non seulement de diminuer la complexité générale d'une
application, mais elle permet également de rassembler les dépendances
indésirables au même endroit.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Façade ](../../facade.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** La façade est régulièrement utilisée dans
les applications écrites en Python. Elle se révèle très pratique pour
gérer les bibliothèques complexes et les API.

**Identification :** La façade peut être reconnue dans une classe qui a
une interface simple, mais délègue la majorité des tâches à d'autres. En
général, la façade gère le cycle de vie des objets qu'elle utilise.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de la **Façade**. Nous
allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-py .anchor} **main.py:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations


class Facade:
    &quot;&quot;&quot;
    The Facade class provides a simple interface to the complex logic of one or
    several subsystems. The Facade delegates the client requests to the
    appropriate objects within the subsystem. The Facade is also responsible for
    managing their lifecycle. All of this shields the client from the undesired
    complexity of the subsystem.
    &quot;&quot;&quot;

    def __init__(self, subsystem1: Subsystem1, subsystem2: Subsystem2) -&gt; None:
        &quot;&quot;&quot;
        Depending on your application&#39;s needs, you can provide the Facade with
        existing subsystem objects or force the Facade to create them on its
        own.
        &quot;&quot;&quot;

        self._subsystem1 = subsystem1 or Subsystem1()
        self._subsystem2 = subsystem2 or Subsystem2()

    def operation(self) -&gt; str:
        &quot;&quot;&quot;
        The Facade&#39;s methods are convenient shortcuts to the sophisticated
        functionality of the subsystems. However, clients get only to a fraction
        of a subsystem&#39;s capabilities.
        &quot;&quot;&quot;

        results = []
        results.append(&quot;Facade initializes subsystems:&quot;)
        results.append(self._subsystem1.operation1())
        results.append(self._subsystem2.operation1())
        results.append(&quot;Facade orders subsystems to perform the action:&quot;)
        results.append(self._subsystem1.operation_n())
        results.append(self._subsystem2.operation_z())
        return &quot;\n&quot;.join(results)


class Subsystem1:
    &quot;&quot;&quot;
    The Subsystem can accept requests either from the facade or client directly.
    In any case, to the Subsystem, the Facade is yet another client, and it&#39;s
    not a part of the Subsystem.
    &quot;&quot;&quot;

    def operation1(self) -&gt; str:
        return &quot;Subsystem1: Ready!&quot;

    # ...

    def operation_n(self) -&gt; str:
        return &quot;Subsystem1: Go!&quot;


class Subsystem2:
    &quot;&quot;&quot;
    Some facades can work with multiple subsystems at the same time.
    &quot;&quot;&quot;

    def operation1(self) -&gt; str:
        return &quot;Subsystem2: Get ready!&quot;

    # ...

    def operation_z(self) -&gt; str:
        return &quot;Subsystem2: Fire!&quot;


def client_code(facade: Facade) -&gt; None:
    &quot;&quot;&quot;
    The client code works with complex subsystems through a simple interface
    provided by the Facade. When a facade manages the lifecycle of the
    subsystem, the client might not even know about the existence of the
    subsystem. This approach lets you keep the complexity under control.
    &quot;&quot;&quot;

    print(facade.operation(), end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code may have some of the subsystem&#39;s objects already created.
    # In this case, it might be worthwhile to initialize the Facade with these
    # objects instead of letting the Facade create new instances.
    subsystem1 = Subsystem1()
    subsystem2 = Subsystem2()
    facade = Facade(subsystem1, subsystem2)
    client_code(facade)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>

::: next
#### Suivant

[Poids mouche en Python []{.fa
.fa-arrow-right}](../../flyweight/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Décorateur en
Python](../../decorator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Façade** dans les autres langues

[![Façade en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Façade en C#"){.prog-lang-link}
[![Façade en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Façade en C++"){.prog-lang-link}
[![Façade en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Façade en Go"){.prog-lang-link}
[![Façade en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Façade en Java"){.prog-lang-link}
[![Façade en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Façade en PHP"){.prog-lang-link}
[![Façade en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Façade en Ruby"){.prog-lang-link}
[![Façade en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Façade en Rust"){.prog-lang-link}
[![Façade en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Façade en Swift"){.prog-lang-link}
[![Façade en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Façade en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
