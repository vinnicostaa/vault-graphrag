:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Template
Method](../../template-method.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Template
Method](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Template Method** em Python {#template-method-em-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Template Method** é um padrão de projeto comportamental que permite
definir o esqueleto de um algoritmo em uma classe base e permitir que as
subclasses substituam as etapas sem alterar a estrutura geral do
algoritmo.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Template Method ](../../template-method.html){.btn
.btn-lg .btn-primary}
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

**Exemplos de uso:** O padrão Template Method é bastante comum nos
frameworks Python. Os desenvolvedores costumam usá-lo para fornecer aos
usuários do framework um meio simples de estender a funcionalidade
padrão usando herança.

**Identificação:** O Template Method pode ser reconhecido por métodos
comportamentais que já possuem um comportamento "padrão" definido pela
classe base.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Template
Method**. Ele se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-py .anchor} **main.py:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="python"><code>from abc import ABC, abstractmethod


class AbstractClass(ABC):
    &quot;&quot;&quot;
    The Abstract Class defines a template method that contains a skeleton of
    some algorithm, composed of calls to (usually) abstract primitive
    operations.

    Concrete subclasses should implement these operations, but leave the
    template method itself intact.
    &quot;&quot;&quot;

    def template_method(self) -&gt; None:
        &quot;&quot;&quot;
        The template method defines the skeleton of an algorithm.
        &quot;&quot;&quot;

        self.base_operation1()
        self.required_operations1()
        self.base_operation2()
        self.hook1()
        self.required_operations2()
        self.base_operation3()
        self.hook2()

    # These operations already have implementations.

    def base_operation1(self) -&gt; None:
        print(&quot;AbstractClass says: I am doing the bulk of the work&quot;)

    def base_operation2(self) -&gt; None:
        print(&quot;AbstractClass says: But I let subclasses override some operations&quot;)

    def base_operation3(self) -&gt; None:
        print(&quot;AbstractClass says: But I am doing the bulk of the work anyway&quot;)

    # These operations have to be implemented in subclasses.

    @abstractmethod
    def required_operations1(self) -&gt; None:
        pass

    @abstractmethod
    def required_operations2(self) -&gt; None:
        pass

    # These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
    # since the hooks already have default (but empty) implementation. Hooks
    # provide additional extension points in some crucial places of the
    # algorithm.

    def hook1(self) -&gt; None:
        pass

    def hook2(self) -&gt; None:
        pass


class ConcreteClass1(AbstractClass):
    &quot;&quot;&quot;
    Concrete classes have to implement all abstract operations of the base
    class. They can also override some operations with a default implementation.
    &quot;&quot;&quot;

    def required_operations1(self) -&gt; None:
        print(&quot;ConcreteClass1 says: Implemented Operation1&quot;)

    def required_operations2(self) -&gt; None:
        print(&quot;ConcreteClass1 says: Implemented Operation2&quot;)


class ConcreteClass2(AbstractClass):
    &quot;&quot;&quot;
    Usually, concrete classes override only a fraction of base class&#39;
    operations.
    &quot;&quot;&quot;

    def required_operations1(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Implemented Operation1&quot;)

    def required_operations2(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Implemented Operation2&quot;)

    def hook1(self) -&gt; None:
        print(&quot;ConcreteClass2 says: Overridden Hook1&quot;)


def client_code(abstract_class: AbstractClass) -&gt; None:
    &quot;&quot;&quot;
    The client code calls the template method to execute the algorithm. Client
    code does not have to know the concrete class of an object it works with, as
    long as it works with objects through the interface of their base class.
    &quot;&quot;&quot;

    # ...
    abstract_class.template_method()
    # ...


if __name__ == &quot;__main__&quot;:
    print(&quot;Same client code can work with different subclasses:&quot;)
    client_code(ConcreteClass1())
    print(&quot;&quot;)

    print(&quot;Same client code can work with different subclasses:&quot;)
    client_code(ConcreteClass2())</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass1 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass1 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway

Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass2 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass2 says: Overridden Hook1
ConcreteClass2 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway</code></pre>
</figure>

::: next
#### Leia a seguir

[Visitor em Python []{.fa
.fa-arrow-right}](../../visitor/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Strategy em
Python](../../strategy/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Template Method** em outras linguagens

[![Template Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Template Method em C#"){.prog-lang-link}
[![Template Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Template Method em C++"){.prog-lang-link}
[![Template Method em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Template Method em Go"){.prog-lang-link}
[![Template Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Template Method em Java"){.prog-lang-link}
[![Template Method em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Template Method em PHP"){.prog-lang-link}
[![Template Method em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Template Method em Ruby"){.prog-lang-link}
[![Template Method em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Template Method em Rust"){.prog-lang-link}
[![Template Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Template Method em Swift"){.prog-lang-link}
[![Template Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Template Method em TypeScript"){.prog-lang-link}
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
