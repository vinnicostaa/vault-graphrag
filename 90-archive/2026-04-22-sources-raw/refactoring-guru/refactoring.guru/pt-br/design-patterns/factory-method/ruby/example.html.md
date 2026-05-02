:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** em Ruby {#factory-method-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Factory method** é um padrão de projeto criacional, que resolve o
problema de criar objetos de produtos sem especificar suas classes
concretas.

O Factory Method define um método, que deve ser usado para criar objetos
em vez da chamada direta ao construtor (operador `new`). As subclasses
podem substituir esse método para alterar a classe de objetos que serão
criados.

> Se você não conseguir descobrir a diferença entre os padrões
> **Factory**, **Factory Method** e **Abstract Factory**, leia nossa
> [Comparação Factory](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Factory Method ](../../factory-method.html){.btn
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Factory Method é amplamente utilizado no
código Ruby. É muito útil quando você precisa fornecer um alto nível de
flexibilidade para seu código.

**Identificação:** Os métodos fábrica podem ser reconhecidos por métodos
de criação, que criam objetos de classes concretas, mas os retornam como
objetos de tipo ou interface abstrata.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Factory
Method**. Ele se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code># The Creator class declares the factory method that is supposed to return an
# object of a Product class. The Creator&#39;s subclasses usually provide the
# implementation of this method.
class Creator
  # Note that the Creator may also provide some default implementation of the
  # factory method.
  def factory_method
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Also note that, despite its name, the Creator&#39;s primary responsibility is
  # not creating products. Usually, it contains some core business logic that
  # relies on Product objects, returned by the factory method. Subclasses can
  # indirectly change that business logic by overriding the factory method and
  # returning a different type of product from it.
  def some_operation
    # Call the factory method to create a Product object.
    product = factory_method

    # Now, use the product.
    &quot;Creator: The same creator&#39;s code has just worked with #{product.operation}&quot;
  end
end

# Concrete Creators override the factory method in order to change the resulting
# product&#39;s type.
class ConcreteCreator1 &lt; Creator
  # Note that the signature of the method still uses the abstract product type,
  # even though the concrete product is actually returned from the method. This
  # way the Creator can stay independent of concrete product classes.
  def factory_method
    ConcreteProduct1.new
  end
end

class ConcreteCreator2 &lt; Creator
  # @return [ConcreteProduct2]
  def factory_method
    ConcreteProduct2.new
  end
end

# The Product interface declares the operations that all concrete products must
# implement.
class Product
  # return [String]
  def operation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Products provide various implementations of the Product interface.
class ConcreteProduct1 &lt; Product
  # @return [String]
  def operation
    &#39;{Result of the ConcreteProduct1}&#39;
  end
end

class ConcreteProduct2 &lt; Product
  # @return [String]
  def operation
    &#39;{Result of the ConcreteProduct2}&#39;
  end
end

# The client code works with an instance of a concrete creator, albeit through
# its base interface. As long as the client keeps working with the creator via
# the base interface, you can pass it any creator&#39;s subclass.
def client_code(creator)
  print &quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;\
        &quot;#{creator.some_operation}&quot;
end

puts &#39;App: Launched with the ConcreteCreator1.&#39;
client_code(ConcreteCreator1.new)
puts &quot;\n\n&quot;

puts &#39;App: Launched with the ConcreteCreator2.&#39;
client_code(ConcreteCreator2.new)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

## **Factory Method** em outras linguagens

[![Factory Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method em C#"){.prog-lang-link}
[![Factory Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method em C++"){.prog-lang-link}
[![Factory Method em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Factory Method em Go"){.prog-lang-link}
[![Factory Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method em Java"){.prog-lang-link}
[![Factory Method em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method em PHP"){.prog-lang-link}
[![Factory Method em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method em Python"){.prog-lang-link}
[![Factory Method em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method em Rust"){.prog-lang-link}
[![Factory Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method em Swift"){.prog-lang-link}
[![Factory Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method em TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
