:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Pont](../../bridge.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pont](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Pont** en Ruby {#pont-en-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **Pont** est un patron de conception structurel qui scinde la logique
métier ou divise de grandes classes dans des hiérarchies de classes
séparées qui vont ensuite évoluer indépendamment.

Une de ces hiérarchies (souvent appelée l'abstraction) gardera une
référence vers un objet de la seconde hiérarchie (l'implémentation).
L'abstraction pourra déléguer certains (parfois la majorité) de ses
appels aux objets de l'implémentation. Puisque toutes les
implémentations ont une interface commune, elles sont interchangeables à
l'intérieur de l'abstraction.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Pont ](../../bridge.html){.btn .btn-lg
.btn-primary}
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

**Exemples d'utilisation :** Le patron de conception pont est très utile
pour gérer les applications multiplateformes, prendre en charge
différents types de serveurs de bases de données ou travailler avec
plusieurs fournisseurs d'API d'un certain genre (par exemple les
plateformes de cloud, réseaux sociaux, etc.).

**Identification :** Le pont peut être identifié grâce à une distinction
très nette entre une entité de contrôle et plusieurs plateformes
différentes dont elle dépend.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du patron de conception
**Pont**. Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quel rôle jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemple conceptuel

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

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

## **Pont** dans les autres langues

[![Pont en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pont en C#"){.prog-lang-link}
[![Pont en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pont en C++"){.prog-lang-link}
[![Pont en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Pont en Go"){.prog-lang-link}
[![Pont en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pont en Java"){.prog-lang-link}
[![Pont en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pont en PHP"){.prog-lang-link}
[![Pont en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pont en Python"){.prog-lang-link}
[![Pont en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pont en Rust"){.prog-lang-link}
[![Pont en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pont en Swift"){.prog-lang-link}
[![Pont en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Pont en TypeScript"){.prog-lang-link}
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
