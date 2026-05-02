:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** em Ruby {#iterator-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Iterador** é um padrão de projeto comportamental que permite a
passagem sequencial através de uma estrutura de dados complexa sem expor
seus detalhes internos.

Graças ao Iterator, os clientes podem examinar elementos de diferentes
coleções de maneira semelhante usando uma única interface iterador.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Iterator ](../../iterator.html){.btn .btn-lg
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão é muito comum no código Ruby. Muitos
frameworks e bibliotecas o usam para fornecer uma maneira padrão de
percorrer suas coleções.

**Identificação:** O iterador é fácil de reconhecer pelos métodos de
navegação (como `next`, `previous` e outros). O código cliente que usa
iteradores pode não ter acesso direto à coleção que está sendo
percorrida.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Iterator**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code>class AlphabeticalOrderIterator
  # In Ruby, the Enumerable mixin provides classes with several traversal and
  # searching methods, and with the ability to sort. The class must provide a
  # method each, which yields successive members of the collection.
  include Enumerable

  # This attribute indicates the traversal direction.
  attr_accessor :reverse
  private :reverse

  # @return [Array]
  attr_accessor :collection
  private :collection

  # @param [Array] collection
  # @param [Boolean] reverse
  def initialize(collection, reverse: false)
    @collection = collection
    @reverse = reverse
  end

  def each(&amp;block)
    return @collection.reverse.each(&amp;block) if reverse

    @collection.each(&amp;block)
  end
end

class WordsCollection
  # @return [Array]
  attr_accessor :collection
  private :collection

  def initialize(collection = [])
    @collection = collection
  end

  # The `iterator` method returns the iterator object itself, by default we
  # return the iterator in ascending order.
  def iterator
    AlphabeticalOrderIterator.new(@collection)
  end

  # @return [AlphabeticalOrderIterator]
  def reverse_iterator
    AlphabeticalOrderIterator.new(@collection, reverse: true)
  end

  # @param [String] item
  def add_item(item)
    @collection &lt;&lt; item
  end
end

# The client code may or may not know about the Concrete Iterator or Collection
# classes, depending on the level of indirection you want to keep in your
# program.
collection = WordsCollection.new
collection.add_item(&#39;First&#39;)
collection.add_item(&#39;Second&#39;)
collection.add_item(&#39;Third&#39;)

puts &#39;Straight traversal:&#39;
collection.iterator.each { |item| puts item }
puts &quot;\n&quot;

puts &#39;Reverse traversal:&#39;
collection.reverse_iterator.each { |item| puts item }</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First</code></pre>
</figure>

## **Iterator** em outras linguagens

[![Iterator em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator em C#"){.prog-lang-link}
[![Iterator em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Iterator em C++"){.prog-lang-link}
[![Iterator em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator em Go"){.prog-lang-link}
[![Iterator em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator em Java"){.prog-lang-link}
[![Iterator em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator em PHP"){.prog-lang-link}
[![Iterator em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator em Python"){.prog-lang-link}
[![Iterator em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator em Rust"){.prog-lang-link}
[![Iterator em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator em Swift"){.prog-lang-link}
[![Iterator em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator em TypeScript"){.prog-lang-link}
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
