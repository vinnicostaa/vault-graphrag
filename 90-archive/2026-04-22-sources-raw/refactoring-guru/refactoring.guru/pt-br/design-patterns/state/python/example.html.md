:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** em Python {#state-em-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **State** é um padrão de projeto comportamental que permite que um
objeto altere o comportamento quando seu estado interno for alterado.

O padrão extrai comportamentos relacionados ao estado em classes
separadas de estado e força o objeto original a delegar o trabalho para
uma instância dessas classes, em vez de agir por conta própria.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o State ](../../state.html){.btn .btn-lg
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

**Exemplos de uso:** O padrão State é comumente usado em Python para
converter enormes máquinas de estado baseadas no `switch` para dentro
dos objetos.

**Identificação:** O padrão State pode ser reconhecido por métodos que
alteram seu comportamento, dependendo do estado dos objetos, controlados
externamente.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **State**. Ele se
concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-py .anchor} **main.py:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Context:
    &quot;&quot;&quot;
    The Context defines the interface of interest to clients. It also maintains
    a reference to an instance of a State subclass, which represents the current
    state of the Context.
    &quot;&quot;&quot;

    _state = None
    &quot;&quot;&quot;
    A reference to the current state of the Context.
    &quot;&quot;&quot;

    def __init__(self, state: State) -&gt; None:
        self.transition_to(state)

    def transition_to(self, state: State):
        &quot;&quot;&quot;
        The Context allows changing the State object at runtime.
        &quot;&quot;&quot;

        print(f&quot;Context: Transition to {type(state).__name__}&quot;)
        self._state = state
        self._state.context = self

    &quot;&quot;&quot;
    The Context delegates part of its behavior to the current State object.
    &quot;&quot;&quot;

    def request1(self):
        self._state.handle1()

    def request2(self):
        self._state.handle2()


class State(ABC):
    &quot;&quot;&quot;
    The base State class declares methods that all Concrete State should
    implement and also provides a backreference to the Context object,
    associated with the State. This backreference can be used by States to
    transition the Context to another State.
    &quot;&quot;&quot;

    @property
    def context(self) -&gt; Context:
        return self._context

    @context.setter
    def context(self, context: Context) -&gt; None:
        self._context = context

    @abstractmethod
    def handle1(self) -&gt; None:
        pass

    @abstractmethod
    def handle2(self) -&gt; None:
        pass


&quot;&quot;&quot;
Concrete States implement various behaviors, associated with a state of the
Context.
&quot;&quot;&quot;


class ConcreteStateA(State):
    def handle1(self) -&gt; None:
        print(&quot;ConcreteStateA handles request1.&quot;)
        print(&quot;ConcreteStateA wants to change the state of the context.&quot;)
        self.context.transition_to(ConcreteStateB())

    def handle2(self) -&gt; None:
        print(&quot;ConcreteStateA handles request2.&quot;)


class ConcreteStateB(State):
    def handle1(self) -&gt; None:
        print(&quot;ConcreteStateB handles request1.&quot;)

    def handle2(self) -&gt; None:
        print(&quot;ConcreteStateB handles request2.&quot;)
        print(&quot;ConcreteStateB wants to change the state of the context.&quot;)
        self.context.transition_to(ConcreteStateA())


if __name__ == &quot;__main__&quot;:
    # The client code.

    context = Context(ConcreteStateA())
    context.request1()
    context.request2()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA</code></pre>
</figure>

::: next
#### Leia a seguir

[Strategy em Python []{.fa
.fa-arrow-right}](../../strategy/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Observer em
Python](../../observer/python/example.html){.btn .btn-default
rel="prev"}
:::

## **State** em outras linguagens

[![State em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State em C#"){.prog-lang-link}
[![State em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State em C++"){.prog-lang-link}
[![State em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "State em Go"){.prog-lang-link}
[![State em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State em Java"){.prog-lang-link}
[![State em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State em PHP"){.prog-lang-link}
[![State em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State em Ruby"){.prog-lang-link}
[![State em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State em Rust"){.prog-lang-link}
[![State em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "State em Swift"){.prog-lang-link}
[![State em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State em TypeScript"){.prog-lang-link}
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
