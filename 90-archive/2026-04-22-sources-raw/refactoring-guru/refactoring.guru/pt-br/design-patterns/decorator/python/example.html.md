:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Decorator](../../decorator.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Decorator](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Decorator** em Python {#decorator-em-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Decorator** é um padrão estrutural que permite adicionar novos
comportamentos aos objetos dinamicamente, colocando-os dentro de objetos
wrapper especiais.

Usando decoradores, você pode agrupar objetos inúmeras vezes, pois os
objetos de destino e os decoradores seguem a mesma interface. O objeto
resultante terá um comportamento de empilhamento de todos os wrappers.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Decorator ](../../decorator.html){.btn .btn-lg
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

**Exemplos de uso:** O Decorator é bastante padrão no código Python,
especialmente nos códigos relacionados a fluxos.

**Identificação:** O Decorator pode ser reconhecido por métodos de
criação ou construtores que aceitam objetos da mesma classe ou interface
que uma classe atual.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Decorator**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-py .anchor} **main.py:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="python"><code>class Component():
    &quot;&quot;&quot;
    The base Component interface defines operations that can be altered by
    decorators.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        pass


class ConcreteComponent(Component):
    &quot;&quot;&quot;
    Concrete Components provide default implementations of the operations. There
    might be several variations of these classes.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return &quot;ConcreteComponent&quot;


class Decorator(Component):
    &quot;&quot;&quot;
    The base Decorator class follows the same interface as the other components.
    The primary purpose of this class is to define the wrapping interface for
    all concrete decorators. The default implementation of the wrapping code
    might include a field for storing a wrapped component and the means to
    initialize it.
    &quot;&quot;&quot;

    _component: Component = None

    def __init__(self, component: Component) -&gt; None:
        self._component = component

    @property
    def component(self) -&gt; Component:
        &quot;&quot;&quot;
        The Decorator delegates all work to the wrapped component.
        &quot;&quot;&quot;

        return self._component

    def operation(self) -&gt; str:
        return self._component.operation()


class ConcreteDecoratorA(Decorator):
    &quot;&quot;&quot;
    Concrete Decorators call the wrapped object and alter its result in some
    way.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        &quot;&quot;&quot;
        Decorators may call parent implementation of the operation, instead of
        calling the wrapped object directly. This approach simplifies extension
        of decorator classes.
        &quot;&quot;&quot;
        return f&quot;ConcreteDecoratorA({self.component.operation()})&quot;


class ConcreteDecoratorB(Decorator):
    &quot;&quot;&quot;
    Decorators can execute their behavior either before or after the call to a
    wrapped object.
    &quot;&quot;&quot;

    def operation(self) -&gt; str:
        return f&quot;ConcreteDecoratorB({self.component.operation()})&quot;


def client_code(component: Component) -&gt; None:
    &quot;&quot;&quot;
    The client code works with all objects using the Component interface. This
    way it can stay independent of the concrete classes of components it works
    with.
    &quot;&quot;&quot;

    # ...

    print(f&quot;RESULT: {component.operation()}&quot;, end=&quot;&quot;)

    # ...


if __name__ == &quot;__main__&quot;:
    # This way the client code can support both simple components...
    simple = ConcreteComponent()
    print(&quot;Client: I&#39;ve got a simple component:&quot;)
    client_code(simple)
    print(&quot;\n&quot;)

    # ...as well as decorated ones.
    #
    # Note how decorators can wrap not only simple components but the other
    # decorators as well.
    decorator1 = ConcreteDecoratorA(simple)
    decorator2 = ConcreteDecoratorB(decorator1)
    print(&quot;Client: Now I&#39;ve got a decorated component:&quot;)
    client_code(decorator2)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: ConcreteComponent

Client: Now I&#39;ve got a decorated component:
RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>

::: next
#### Leia a seguir

[Facade em Python []{.fa
.fa-arrow-right}](../../facade/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Composite em
Python](../../composite/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Decorator** em outras linguagens

[![Decorator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Decorator em C#"){.prog-lang-link}
[![Decorator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Decorator em C++"){.prog-lang-link}
[![Decorator em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Decorator em Go"){.prog-lang-link}
[![Decorator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Decorator em Java"){.prog-lang-link}
[![Decorator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Decorator em PHP"){.prog-lang-link}
[![Decorator em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Decorator em Ruby"){.prog-lang-link}
[![Decorator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Decorator em Rust"){.prog-lang-link}
[![Decorator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Decorator em Swift"){.prog-lang-link}
[![Decorator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Decorator em TypeScript"){.prog-lang-link}
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
