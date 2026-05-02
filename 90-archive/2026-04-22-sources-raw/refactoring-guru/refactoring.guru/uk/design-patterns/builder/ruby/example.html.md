:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Будівельник](../../builder.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Будівельник](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Будівельник** на Ruby {#будівельник-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Будівельник** --- це породжуючий патерн проектування, який дозволяє
створювати об'єкти покроково.

На відміну від інших породжуючих патернів, Будівельник дозволяє
виготовляти різні продукти, використовуючи один і той же процес
будівництва.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Будівельник ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн можна часто зустріти в Ruby-коді, особливо там,
де необхідним є покрокове створення продуктів або конфігурація складних
об'єктів.

**Ознаки застосування патерна:** Будівельника можна визначити у класі,
який має один створюючий метод та декілька методів налаштування
створюваного продукту. Зазвичай, для зручності, методи налаштувань
викликають ланцюжком (наприклад,
`someBuilder.setValueA(1).setValueB(2).create()`).
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Будівельник**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-rb .anchor} **main.rb:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="ruby"><code># The Builder interface specifies methods for creating the different parts of
# the Product objects.
class Builder
  # @abstract
  def produce_part_a
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def produce_part_b
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def produce_part_c
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The Concrete Builder classes follow the Builder interface and provide specific
# implementations of the building steps. Your program may have several
# variations of Builders, implemented differently.
class ConcreteBuilder1 &lt; Builder
  # A fresh builder instance should contain a blank product object, which is
  # used in further assembly.
  def initialize
    reset
  end

  def reset
    @product = Product1.new
  end

  # Concrete Builders are supposed to provide their own methods for retrieving
  # results. That&#39;s because various types of builders may create entirely
  # different products that don&#39;t follow the same interface. Therefore, such
  # methods cannot be declared in the base Builder interface (at least in a
  # statically typed programming language).
  #
  # Usually, after returning the end result to the client, a builder instance is
  # expected to be ready to start producing another product. That&#39;s why it&#39;s a
  # usual practice to call the reset method at the end of the `getProduct`
  # method body. However, this behavior is not mandatory, and you can make your
  # builders wait for an explicit reset call from the client code before
  # disposing of the previous result.
  def product
    product = @product
    reset
    product
  end

  def produce_part_a
    @product.add(&#39;PartA1&#39;)
  end

  def produce_part_b
    @product.add(&#39;PartB1&#39;)
  end

  def produce_part_c
    @product.add(&#39;PartC1&#39;)
  end
end

# It makes sense to use the Builder pattern only when your products are quite
# complex and require extensive configuration.
#
# Unlike in other creational patterns, different concrete builders can produce
# unrelated products. In other words, results of various builders may not always
# follow the same interface.
class Product1
  def initialize
    @parts = []
  end

  # @param [String] part
  def add(part)
    @parts &lt;&lt; part
  end

  def list_parts
    print &quot;Product parts: #{@parts.join(&#39;, &#39;)}&quot;
  end
end

# The Director is only responsible for executing the building steps in a
# particular sequence. It is helpful when producing products according to a
# specific order or configuration. Strictly speaking, the Director class is
# optional, since the client can control builders directly.
class Director
  # @return [Builder]
  attr_accessor :builder

  def initialize
    @builder = nil
  end

  # The Director works with any builder instance that the client code passes to
  # it. This way, the client code may alter the final type of the newly
  # assembled product.
  def builder=(builder)
    @builder = builder
  end

  # The Director can construct several product variations using the same
  # building steps.

  def build_minimal_viable_product
    @builder.produce_part_a
  end

  def build_full_featured_product
    @builder.produce_part_a
    @builder.produce_part_b
    @builder.produce_part_c
  end
end

# The client code creates a builder object, passes it to the director and then
# initiates the construction process. The end result is retrieved from the
# builder object.

director = Director.new
builder = ConcreteBuilder1.new
director.builder = builder

puts &#39;Standard basic product: &#39;
director.build_minimal_viable_product
builder.product.list_parts

puts &quot;\n\n&quot;

puts &#39;Standard full featured product: &#39;
director.build_full_featured_product
builder.product.list_parts

puts &quot;\n\n&quot;

# Remember, the Builder pattern can be used without a Director class.
puts &#39;Custom product: &#39;
builder.produce_part_a
builder.produce_part_b
builder.product.list_parts</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Standard basic product: 
Product parts: PartA1

Standard full featured product: 
Product parts: PartA1, PartB1, PartC1

Custom product: 
Product parts: PartA1, PartB1</code></pre>
</figure>

## **Будівельник** іншими мовами програмування

[![Будівельник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Будівельник на C#"){.prog-lang-link}
[![Будівельник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Будівельник на C++"){.prog-lang-link}
[![Будівельник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Будівельник на Go"){.prog-lang-link}
[![Будівельник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Будівельник на Java"){.prog-lang-link}
[![Будівельник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Будівельник на PHP"){.prog-lang-link}
[![Будівельник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Будівельник на Python"){.prog-lang-link}
[![Будівельник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Будівельник на Rust"){.prog-lang-link}
[![Будівельник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Будівельник на Swift"){.prog-lang-link}
[![Будівельник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Будівельник на TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
