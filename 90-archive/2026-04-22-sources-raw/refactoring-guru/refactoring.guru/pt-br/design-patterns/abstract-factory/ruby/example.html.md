:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** em Ruby {#abstract-factory-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Abstract Factory** é um padrão de projeto criacional, que resolve o
problema de criar famílias inteiras de produtos sem especificar suas
classes concretas.

O Abstract Factory define uma interface para criar todos os produtos
distintos, mas deixa a criação real do produto para classes fábrica
concretas. Cada tipo de fábrica corresponde a uma determinada variedade
de produtos.

O código cliente chama os métodos de criação de um objeto fábrica em vez
de criar produtos diretamente com uma chamada de construtor (usando
operador `new`). Como uma fábrica corresponde a uma única variante de
produto, todos os seus produtos serão compatíveis.

O código cliente trabalha com fábricas e produtos somente através de
suas interfaces abstratas. Ele permite que o mesmo código cliente
funcione com produtos diferentes. Você apenas cria uma nova classe
fábrica concreta e a passa para o código cliente.

> Se você não conseguir descobrir a diferença entre os padrões
> **Factory**, **Factory Method** e **Abstract Factory**, leia nossa
> [Comparação Factory](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Abstract Factory
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
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

**Exemplos de uso:** O padrão Abstract Factory é bastante comum no
código Ruby. Muitas frameworks e bibliotecas o utilizam para fornecer
uma maneira de estender e personalizar seus componentes padrão.

**Identificação:** O padrão é fácil de reconhecer pelos seus métodos,
que retornam um objeto fárica. Em seguida, a fábrica é usado para criar
subcomponentes específicos.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Abstract
Factory**. Ele se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code># The Abstract Factory interface declares a set of methods that return different
# abstract products. These products are called a family and are related by a
# high-level theme or concept. Products of one family are usually able to
# collaborate among themselves. A family of products may have several variants,
# but the products of one variant are incompatible with products of another.
class AbstractFactory
  # @abstract
  def create_product_a
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def create_product_b
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Factories produce a family of products that belong to a single
# variant. The factory guarantees that resulting products are compatible. Note
# that signatures of the Concrete Factory&#39;s methods return an abstract product,
# while inside the method a concrete product is instantiated.
class ConcreteFactory1 &lt; AbstractFactory
  def create_product_a
    ConcreteProductA1.new
  end

  def create_product_b
    ConcreteProductB1.new
  end
end

# Each Concrete Factory has a corresponding product variant.
class ConcreteFactory2 &lt; AbstractFactory
  def create_product_a
    ConcreteProductA2.new
  end

  def create_product_b
    ConcreteProductB2.new
  end
end

# Each distinct product of a product family should have a base interface. All
# variants of the product must implement this interface.
class AbstractProductA
  # @abstract
  #
  # @return [String]
  def useful_function_a
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Products are created by corresponding Concrete Factories.
class ConcreteProductA1 &lt; AbstractProductA
  def useful_function_a
    &#39;The result of the product A1.&#39;
  end
end

class ConcreteProductA2 &lt; AbstractProductA
  def useful_function_a
    &#39;The result of the product A2.&#39;
  end
end

# Here&#39;s the the base interface of another product. All products can interact
# with each other, but proper interaction is possible only between products of
# the same concrete variant.
class AbstractProductB
  # Product B is able to do its own thing...
  def useful_function_b
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # ...but it also can collaborate with the ProductA.
  #
  # The Abstract Factory makes sure that all products it creates are of the same
  # variant and thus, compatible.
  def another_useful_function_b(_collaborator)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Products are created by corresponding Concrete Factories.
class ConcreteProductB1 &lt; AbstractProductB
  # @return [String]
  def useful_function_b
    &#39;The result of the product B1.&#39;
  end

  # The variant, Product B1, is only able to work correctly with the variant,
  # Product A1. Nevertheless, it accepts any instance of AbstractProductA as an
  # argument.
  def another_useful_function_b(collaborator)
    result = collaborator.useful_function_a
    &quot;The result of the B1 collaborating with the (#{result})&quot;
  end
end

class ConcreteProductB2 &lt; AbstractProductB
  # @return [String]
  def useful_function_b
    &#39;The result of the product B2.&#39;
  end

  # The variant, Product B2, is only able to work correctly with the variant,
  # Product A2. Nevertheless, it accepts any instance of AbstractProductA as an
  # argument.
  def another_useful_function_b(collaborator)
    result = collaborator.useful_function_a
    &quot;The result of the B2 collaborating with the (#{result})&quot;
  end
end

# The client code works with factories and products only through abstract types:
# AbstractFactory and AbstractProduct. This lets you pass any factory or product
# subclass to the client code without breaking it.
def client_code(factory)
  product_a = factory.create_product_a
  product_b = factory.create_product_b

  puts product_b.useful_function_b
  puts product_b.another_useful_function_b(product_a)
end

# The client code can work with any concrete factory class.
puts &#39;Client: Testing client code with the first factory type:&#39;
client_code(ConcreteFactory1.new)

puts &quot;\n&quot;

puts &#39;Client: Testing the same client code with the second factory type:&#39;
client_code(ConcreteFactory2.new)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Client: Testing client code with the first factory type:
The result of the product B1.
The result of the B1 collaborating with the (The result of the product A1.)

Client: Testing the same client code with the second factory type:
The result of the product B2.
The result of the B2 collaborating with the (The result of the product A2.)</code></pre>
</figure>

## **Abstract Factory** em outras linguagens

[![Abstract Factory em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory em C#"){.prog-lang-link}
[![Abstract Factory em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory em C++"){.prog-lang-link}
[![Abstract Factory em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Abstract Factory em Go"){.prog-lang-link}
[![Abstract Factory em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory em Java"){.prog-lang-link}
[![Abstract Factory em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory em PHP"){.prog-lang-link}
[![Abstract Factory em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory em Python"){.prog-lang-link}
[![Abstract Factory em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory em Rust"){.prog-lang-link}
[![Abstract Factory em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory em Swift"){.prog-lang-link}
[![Abstract Factory em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory em TypeScript"){.prog-lang-link}
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
