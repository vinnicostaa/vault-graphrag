:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** em Python {#mediator-em-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Mediator** é um padrão de projeto comportamental que reduz o
acoplamento entre os componentes de um programa, fazendo-os se comunicar
indiretamente, por meio de um objeto mediador especial.

O Mediator facilita a modificação, a extensão e a reutilização de
componentes individuais porque eles não são mais dependentes de dezenas
de outras classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Mediator ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O uso mais popular do padrão Mediator no código
Python é facilitar a comunicação entre os componentes de interface do
usuário de uma aplicação. O sinônimo do Mediator é a parte do
Controlador do padrão MVC.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Mediator**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-py .anchor} **main.py:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC


class Mediator(ABC):
    &quot;&quot;&quot;
    The Mediator interface declares a method used by components to notify the
    mediator about various events. The Mediator may react to these events and
    pass the execution to other components.
    &quot;&quot;&quot;

    def notify(self, sender: object, event: str) -&gt; None:
        pass


class ConcreteMediator(Mediator):
    def __init__(self, component1: Component1, component2: Component2) -&gt; None:
        self._component1 = component1
        self._component1.mediator = self
        self._component2 = component2
        self._component2.mediator = self

    def notify(self, sender: object, event: str) -&gt; None:
        if event == &quot;A&quot;:
            print(&quot;Mediator reacts on A and triggers following operations:&quot;)
            self._component2.do_c()
        elif event == &quot;D&quot;:
            print(&quot;Mediator reacts on D and triggers following operations:&quot;)
            self._component1.do_b()
            self._component2.do_c()


class BaseComponent:
    &quot;&quot;&quot;
    The Base Component provides the basic functionality of storing a mediator&#39;s
    instance inside component objects.
    &quot;&quot;&quot;

    def __init__(self, mediator: Mediator = None) -&gt; None:
        self._mediator = mediator

    @property
    def mediator(self) -&gt; Mediator:
        return self._mediator

    @mediator.setter
    def mediator(self, mediator: Mediator) -&gt; None:
        self._mediator = mediator


&quot;&quot;&quot;
Concrete Components implement various functionality. They don&#39;t depend on other
components. They also don&#39;t depend on any concrete mediator classes.
&quot;&quot;&quot;


class Component1(BaseComponent):
    def do_a(self) -&gt; None:
        print(&quot;Component 1 does A.&quot;)
        self.mediator.notify(self, &quot;A&quot;)

    def do_b(self) -&gt; None:
        print(&quot;Component 1 does B.&quot;)
        self.mediator.notify(self, &quot;B&quot;)


class Component2(BaseComponent):
    def do_c(self) -&gt; None:
        print(&quot;Component 2 does C.&quot;)
        self.mediator.notify(self, &quot;C&quot;)

    def do_d(self) -&gt; None:
        print(&quot;Component 2 does D.&quot;)
        self.mediator.notify(self, &quot;D&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code.
    c1 = Component1()
    c2 = Component2()
    mediator = ConcreteMediator(c1, c2)

    print(&quot;Client triggers operation A.&quot;)
    c1.do_a()

    print(&quot;\n&quot;, end=&quot;&quot;)

    print(&quot;Client triggers operation D.&quot;)
    c2.do_d()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.


Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

::: next
#### Leia a seguir

[Memento em Python []{.fa
.fa-arrow-right}](../../memento/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Iterator em
Python](../../iterator/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Mediator** em outras linguagens

[![Mediator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator em C#"){.prog-lang-link}
[![Mediator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator em C++"){.prog-lang-link}
[![Mediator em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator em Go"){.prog-lang-link}
[![Mediator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator em Java"){.prog-lang-link}
[![Mediator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator em PHP"){.prog-lang-link}
[![Mediator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator em Ruby"){.prog-lang-link}
[![Mediator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator em Rust"){.prog-lang-link}
[![Mediator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator em Swift"){.prog-lang-link}
[![Mediator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator em TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
