:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** em Ruby {#prototype-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Prototype** é um padrão de projeto criacional que permite a clonagem
de objetos, mesmo complexos, sem acoplamento à suas classes específicas.

Todas as classes de prototypes(protótipos) devem ter uma interface comum
que permita copiar objetos, mesmo que suas classes concretas sejam
desconhecidas. Objetos protótipos podem produzir cópias completas, pois
objetos da mesma classe podem acessar os campos privados um do outro.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Prototype ](../../prototype.html){.btn .btn-lg
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

**Exemplos de uso:** O padrão Prototype está disponível e pronto para
uso em Ruby com métodos `dup` ou `clone`.

**Identificação:** O prototype pode ser facilmente reconhecido pelos
métodos `clone` ou `copy`, etc.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Prototype**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code># The example class that has cloning ability. We&#39;ll see how the values of field
# with different types will be cloned.
class Prototype
  attr_accessor :primitive, :component, :circular_reference

  def initialize
    @primitive = nil
    @component = nil
    @circular_reference = nil
  end

  # @return [Prototype]
  def clone
    @component = deep_copy(@component)

    # Cloning an object that has a nested object with backreference requires
    # special treatment. After the cloning is completed, the nested object
    # should point to the cloned object, instead of the original object.
    @circular_reference = deep_copy(@circular_reference)
    @circular_reference.prototype = self
    deep_copy(self)
  end

  # deep_copy is the usual Marshalling hack to make a deep copy. But it&#39;s rather
  # slow and inefficient, therefore, in real applications, use a special gem.
  private def deep_copy(object)
    Marshal.load(Marshal.dump(object))
  end
end

class ComponentWithBackReference
  attr_accessor :prototype

  # @param [Prototype] prototype
  def initialize(prototype)
    @prototype = prototype
  end
end

# The client code.
p1 = Prototype.new
p1.primitive = 245
p1.component = Time.now
p1.circular_reference = ComponentWithBackReference.new(p1)

p2 = p1.clone

if p1.primitive == p2.primitive
  puts &#39;Primitive field values have been carried over to a clone. Yay!&#39;
else
  puts &#39;Primitive field values have not been copied. Booo!&#39;
end

if p1.component.equal?(p2.component)
  puts &#39;Simple component has not been cloned. Booo!&#39;
else
  puts &#39;Simple component has been cloned. Yay!&#39;
end

if p1.circular_reference.equal?(p2.circular_reference)
  puts &#39;Component with back reference has not been cloned. Booo!&#39;
else
  puts &#39;Component with back reference has been cloned. Yay!&#39;
end

if p1.circular_reference.prototype.equal?(p2.circular_reference.prototype)
  print &#39;Component with back reference is linked to original object. Booo!&#39;
else
  print &#39;Component with back reference is linked to the clone. Yay!&#39;
end</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Primitive field values have been carried over to a clone. Yay!
Simple component has been cloned. Yay!
Component with back reference has been cloned. Yay!
Component with back reference is linked to the clone. Yay!</code></pre>
</figure>

## **Prototype** em outras linguagens

[![Prototype em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype em C#"){.prog-lang-link}
[![Prototype em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype em C++"){.prog-lang-link}
[![Prototype em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype em Go"){.prog-lang-link}
[![Prototype em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype em Java"){.prog-lang-link}
[![Prototype em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototype em PHP"){.prog-lang-link}
[![Prototype em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype em Python"){.prog-lang-link}
[![Prototype em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype em Rust"){.prog-lang-link}
[![Prototype em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype em Swift"){.prog-lang-link}
[![Prototype em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype em TypeScript"){.prog-lang-link}
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
