:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Kompozyt](../../composite.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Kompozyt](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Kompozyt** w języku Ruby {#kompozyt-w-języku-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Kompozyt** to strukturalny wzorzec projektowy umożliwiający
komponowanie struktury drzewiastej z obiektów i traktowanie jej jak
pojedynczy obiekt.

Kompozyt stał się dość popularnym rozwiązaniem wielu problemów gdzie w
grę wchodzi struktura drzewa. Zaletą tego wzorca jest możliwość
uruchamiania metod rekurencyjnie na wszystkich elementach struktury i
sumowanie wyników ich działania.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Kompozyt ](../../composite.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Kompozyt jest dość powszechny w kodzie
Ruby. Często stosuje się go do modelowania hierarchii komponentów
interfejsu użytkownika lub kodu który działa na grafach.

**Identyfikacja:** Jeśli klasy wszystkich obiektów w drzewie należą do
jednej hierarchii to najprawdopodobniej mamy do czynienia z kompozytem.
Jeśli dodatkowo metody tych klas delegują zadania obiektom-dzieciom
wchodzącym w skład tego drzewa i robią to za pośrednictwem klasy bazowej
lub bazowego interfejsu hierarchii, to na pewno jest to kompozyt.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Kompozyt** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-rb .anchor} **main.rb:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="ruby"><code># The base Component class declares common operations for both simple and
# complex objects of a composition.
class Component
  # @return [Component]
  def parent
    @parent
  end

  # Optionally, the base Component can declare an interface for setting and
  # accessing a parent of the component in a tree structure. It can also provide
  # some default implementation for these methods.
  def parent=(parent)
    @parent = parent
  end

  # In some cases, it would be beneficial to define the child-management
  # operations right in the base Component class. This way, you won&#39;t need to
  # expose any concrete component classes to the client code, even during the
  # object tree assembly. The downside is that these methods will be empty for
  # the leaf-level components.
  def add(component)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  #
  # @param [Component] component
  def remove(component)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # You can provide a method that lets the client code figure out whether a
  # component can bear children.
  def composite?
    false
  end

  # The base Component may implement some default behavior or leave it to
  # concrete classes (by declaring the method containing the behavior as
  # &quot;abstract&quot;).
  def operation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The Leaf class represents the end objects of a composition. A leaf can&#39;t have
# any children.
#
# Usually, it&#39;s the Leaf objects that do the actual work, whereas Composite
# objects only delegate to their sub-components.
class Leaf &lt; Component
  # return [String]
  def operation
    &#39;Leaf&#39;
  end
end

# The Composite class represents the complex components that may have children.
# Usually, the Composite objects delegate the actual work to their children and
# then &quot;sum-up&quot; the result.
class Composite &lt; Component
  def initialize
    @children = []
  end

  # A composite object can add or remove other components (both simple or
  # complex) to or from its child list.

  # @param [Component] component
  def add(component)
    @children.append(component)
    component.parent = self
  end

  # @param [Component] component
  def remove(component)
    @children.remove(component)
    component.parent = nil
  end

  # @return [Boolean]
  def composite?
    true
  end

  # The Composite executes its primary logic in a particular way. It traverses
  # recursively through all its children, collecting and summing their results.
  # Since the composite&#39;s children pass these calls to their children and so
  # forth, the whole object tree is traversed as a result.
  def operation
    results = []
    @children.each { |child| results.append(child.operation) }
    &quot;Branch(#{results.join(&#39;+&#39;)})&quot;
  end
end

# The client code works with all of the components via the base interface.
def client_code(component)
  puts &quot;RESULT: #{component.operation}&quot;
end

# Thanks to the fact that the child-management operations are declared in the
# base Component class, the client code can work with any component, simple or
# complex, without depending on their concrete classes.
def client_code2(component1, component2)
  component1.add(component2) if component1.composite?

  print &quot;RESULT: #{component1.operation}&quot;
end

# This way the client code can support the simple leaf components...
simple = Leaf.new
puts &#39;Client: I\&#39;ve got a simple component:&#39;
client_code(simple)
puts &quot;\n&quot;

# ...as well as the complex composites.
tree = Composite.new

branch1 = Composite.new
branch1.add(Leaf.new)
branch1.add(Leaf.new)

branch2 = Composite.new
branch2.add(Leaf.new)

tree.add(branch1)
tree.add(branch2)

puts &#39;Client: Now I\&#39;ve got a composite tree:&#39;
client_code(tree)
puts &quot;\n&quot;

puts &#39;Client: I don\&#39;t need to check the components classes even when managing the tree:&#39;
client_code2(tree, simple)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: Leaf

Client: Now I&#39;ve got a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>

## **Kompozyt** w innych językach

[![Kompozyt w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Kompozyt w języku C#"){.prog-lang-link}
[![Kompozyt w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Kompozyt w języku C++"){.prog-lang-link}
[![Kompozyt w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Kompozyt w języku Go"){.prog-lang-link}
[![Kompozyt w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Kompozyt w języku Java"){.prog-lang-link}
[![Kompozyt w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Kompozyt w języku PHP"){.prog-lang-link}
[![Kompozyt w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Kompozyt w języku Python"){.prog-lang-link}
[![Kompozyt w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Kompozyt w języku Rust"){.prog-lang-link}
[![Kompozyt w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Kompozyt w języku Swift"){.prog-lang-link}
[![Kompozyt w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Kompozyt w języku TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
