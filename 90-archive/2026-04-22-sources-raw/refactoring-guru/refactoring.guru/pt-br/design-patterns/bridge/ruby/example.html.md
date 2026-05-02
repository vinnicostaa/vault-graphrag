:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** em Ruby {#bridge-em-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Bridge** é um padrão de projeto estrutural que divide a lógica de
negócio ou uma enorme classe em hierarquias de classe separadas que
podem ser desenvolvidas independentemente.

Uma dessas hierarquias (geralmente chamada de Abstração) obterá uma
referência a um objeto da segunda hierarquia (Implementação). A
abstração poderá delegar algumas (às vezes, a maioria) de suas chamadas
para o objeto de implementações. Como todas as implementações terão uma
interface comum, elas seriam intercambiáveis dentro da abstração.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Bridge ](../../bridge.html){.btn .btn-lg
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

**Exemplos de uso:** O padrão Bridge é especialmente útil ao lidar com
aplicações multi plataforma, oferecer suporte a vários tipos de
servidores de banco de dados, ou ao trabalhar com vários provedores de
API de um determinado tipo (por exemplo, plataformas em nuvem, redes
sociais etc.)

**Identificação:** O Bridge pode ser reconhecida por uma distinção clara
entre alguma entidade controladora e várias plataformas diferentes nas
quais ela se baseia.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Bridge**. Ele se
concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="ruby"><code># The Abstraction defines the interface for the &quot;control&quot; part of the two class
# hierarchies. It maintains a reference to an object of the Implementation
# hierarchy and delegates all of the real work to this object.
class Abstraction
  # @param [Implementation] implementation
  def initialize(implementation)
    @implementation = implementation
  end

  # @return [String]
  def operation
    &quot;Abstraction: Base operation with:\n&quot;\
    &quot;#{@implementation.operation_implementation}&quot;
  end
end

# You can extend the Abstraction without changing the Implementation classes.
class ExtendedAbstraction &lt; Abstraction
  # @return [String]
  def operation
    &quot;ExtendedAbstraction: Extended operation with:\n&quot;\
    &quot;#{@implementation.operation_implementation}&quot;
  end
end

# The Implementation defines the interface for all implementation classes. It
# doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two interfaces
# can be entirely different. Typically the Implementation interface provides
# only primitive operations, while the Abstraction defines higher-level
# operations based on those primitives.
class Implementation
  # @abstract
  #
  # @return [String]
  def operation_implementation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Each Concrete Implementation corresponds to a specific platform and implements
# the Implementation interface using that platform&#39;s API.
class ConcreteImplementationA &lt; Implementation
  # @return [String]
  def operation_implementation
    &#39;ConcreteImplementationA: Here\&#39;s the result on the platform A.&#39;
  end
end

class ConcreteImplementationB &lt; Implementation
  # @return [String]
  def operation_implementation
    &#39;ConcreteImplementationB: Here\&#39;s the result on the platform B.&#39;
  end
end

# Except for the initialization phase, where an Abstraction object gets linked
# with a specific Implementation object, the client code should only depend on
# the Abstraction class. This way the client code can support any abstraction-
# implementation combination.
def client_code(abstraction)
  # ...

  print abstraction.operation

  # ...
end

# The client code should be able to work with any pre-configured abstraction-
# implementation combination.

implementation = ConcreteImplementationA.new
abstraction = Abstraction.new(implementation)
client_code(abstraction)

puts &quot;\n\n&quot;

implementation = ConcreteImplementationB.new
abstraction = ExtendedAbstraction.new(implementation)
client_code(abstraction)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

## **Bridge** em outras linguagens

[![Bridge em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge em C#"){.prog-lang-link}
[![Bridge em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge em C++"){.prog-lang-link}
[![Bridge em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge em Go"){.prog-lang-link}
[![Bridge em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge em Java"){.prog-lang-link}
[![Bridge em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge em PHP"){.prog-lang-link}
[![Bridge em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Bridge em Python"){.prog-lang-link}
[![Bridge em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge em Rust"){.prog-lang-link}
[![Bridge em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge em Swift"){.prog-lang-link}
[![Bridge em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge em TypeScript"){.prog-lang-link}
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
