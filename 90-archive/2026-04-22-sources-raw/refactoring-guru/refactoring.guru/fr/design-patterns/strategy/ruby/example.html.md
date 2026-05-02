:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Stratégie](../../strategy.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Stratégie](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Stratégie** en Ruby {#stratégie-en-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
La **Stratégie** est un patron de conception comportemental qui
transforme un ensemble de comportements en objets, et les rend
interchangeables à l'intérieur de l'objet du contexte original.

L'objet original, que l'on appelle contexte, garde une référence vers un
objet stratégie et lui délègue l'exécution du comportement. Les autres
objets doivent remplacer l'objet stratégie associé afin de modifier la
manière dont le contexte fonctionne.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Stratégie ](../../strategy.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le patron de conception stratégie est très
répandu en Ruby. Il est souvent utilisé dans divers frameworks pour
fournir aux utilisateurs la possibilité de modifier le comportement
d'une classe sans l'étendre.

**Identification :** La stratégie peut être reconnue par des méthodes
qui laissent un objet imbriqué exécuter les tâches, ainsi que par le
setter qui permet de remplacer cet objet par un autre.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de la **Stratégie**.
Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="ruby"><code># The Context defines the interface of interest to clients.
class Context
  # The Context maintains a reference to one of the Strategy objects. The
  # Context does not know the concrete class of a strategy. It should work with
  # all strategies via the Strategy interface.
  attr_writer :strategy

  # Usually, the Context accepts a strategy through the constructor, but also
  # provides a setter to change it at runtime.
  def initialize(strategy)
    @strategy = strategy
  end

  # Usually, the Context allows replacing a Strategy object at runtime.
  def strategy=(strategy)
    @strategy = strategy
  end

  # The Context delegates some work to the Strategy object instead of
  # implementing multiple versions of the algorithm on its own.
  def do_some_business_logic
    # ...

    puts &#39;Context: Sorting data using the strategy (not sure how it\&#39;ll do it)&#39;
    result = @strategy.do_algorithm(%w[a b c d e])
    print result.join(&#39;,&#39;)

    # ...
  end
end

# The Strategy interface declares operations common to all supported versions of
# some algorithm.
#
# The Context uses this interface to call the algorithm defined by Concrete
# Strategies.
class Strategy
  # @abstract
  #
  # @param [Array] data
  def do_algorithm(_data)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Strategies implement the algorithm while following the base Strategy
# interface. The interface makes them interchangeable in the Context.

class ConcreteStrategyA &lt; Strategy
  # @param [Array] data
  #
  # @return [Array]
  def do_algorithm(data)
    data.sort
  end
end

class ConcreteStrategyB &lt; Strategy
  # @param [Array] data
  #
  # @return [Array]
  def do_algorithm(data)
    data.sort.reverse
  end
end

# The client code picks a concrete strategy and passes it to the context. The
# client should be aware of the differences between strategies in order to make
# the right choice.

context = Context.new(ConcreteStrategyA.new)
puts &#39;Client: Strategy is set to normal sorting.&#39;
context.do_some_business_logic
puts &quot;\n\n&quot;

puts &#39;Client: Strategy is set to reverse sorting.&#39;
context.strategy = ConcreteStrategyB.new
context.do_some_business_logic</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Client: Strategy is set to normal sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
a,b,c,d,e

Client: Strategy is set to reverse sorting.
Context: Sorting data using the strategy (not sure how it&#39;ll do it)
e,d,c,b,a</code></pre>
</figure>

## **Stratégie** dans les autres langues

[![Stratégie en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Stratégie en C#"){.prog-lang-link}
[![Stratégie en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Stratégie en C++"){.prog-lang-link}
[![Stratégie en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Stratégie en Go"){.prog-lang-link}
[![Stratégie en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Stratégie en Java"){.prog-lang-link}
[![Stratégie en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Stratégie en PHP"){.prog-lang-link}
[![Stratégie en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Stratégie en Python"){.prog-lang-link}
[![Stratégie en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Stratégie en Rust"){.prog-lang-link}
[![Stratégie en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Stratégie en Swift"){.prog-lang-link}
[![Stratégie en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Stratégie en TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
