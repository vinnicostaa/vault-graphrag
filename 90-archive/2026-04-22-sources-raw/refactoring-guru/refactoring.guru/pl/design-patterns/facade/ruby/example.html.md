:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Fasada](../../facade.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fasada](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Fasada** w języku Ruby {#fasada-w-języku-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Fasada** to strukturalny wzorzec projektowy wyposażający złożony
system klas, bibliotekę lub framework w uproszczony interfejs.

Fasada zmniejsza ogólną złożoność aplikacji oraz pomaga przenieść
niechciane zależności w jedno miejsce w programie.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Fasada ](../../facade.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Fasada jest często używany w aplikacjach
pisanych w Ruby, gdzie przydaje się w pracy ze złożonymi bibliotekami i
API.

**Identyfikacja:** Użycie Fasady można stwierdzić po istnieniu klasy o
uproszczonym interfejsie, która deleguje większość swoich zadań innym
klasom. Zazwyczaj fasady zarządzają całym cyklem życia używanych przez
siebie obiektów.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Fasada** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-rb .anchor} **main.rb:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="ruby"><code># The Facade class provides a simple interface to the complex logic of one or
# several subsystems. The Facade delegates the client requests to the
# appropriate objects within the subsystem. The Facade is also responsible for
# managing their lifecycle. All of this shields the client from the undesired
# complexity of the subsystem.
class Facade
  # Depending on your application&#39;s needs, you can provide the Facade with
  # existing subsystem objects or force the Facade to create them on its own.
  def initialize(subsystem1, subsystem2)
    @subsystem1 = subsystem1 || Subsystem1.new
    @subsystem2 = subsystem2 || Subsystem2.new
  end

  # The Facade&#39;s methods are convenient shortcuts to the sophisticated
  # functionality of the subsystems. However, clients get only to a fraction of
  # a subsystem&#39;s capabilities.
  def operation
    results = []
    results.append(&#39;Facade initializes subsystems:&#39;)
    results.append(@subsystem1.operation1)
    results.append(@subsystem2.operation1)
    results.append(&#39;Facade orders subsystems to perform the action:&#39;)
    results.append(@subsystem1.operation_n)
    results.append(@subsystem2.operation_z)
    results.join(&quot;\n&quot;)
  end
end

# The Subsystem can accept requests either from the facade or client directly.
# In any case, to the Subsystem, the Facade is yet another client, and it&#39;s not
# a part of the Subsystem.
class Subsystem1
  # @return [String]
  def operation1
    &#39;Subsystem1: Ready!&#39;
  end

  # ...

  # @return [String]
  def operation_n
    &#39;Subsystem1: Go!&#39;
  end
end

# Some facades can work with multiple subsystems at the same time.
class Subsystem2
  # @return [String]
  def operation1
    &#39;Subsystem2: Get ready!&#39;
  end

  # ...

  # @return [String]
  def operation_z
    &#39;Subsystem2: Fire!&#39;
  end
end

# The client code works with complex subsystems through a simple interface
# provided by the Facade. When a facade manages the lifecycle of the subsystem,
# the client might not even know about the existence of the subsystem. This
# approach lets you keep the complexity under control.
def client_code(facade)
  print facade.operation
end

# The client code may have some of the subsystem&#39;s objects already created. In
# this case, it might be worthwhile to initialize the Facade with these objects
# instead of letting the Facade create new instances.
subsystem1 = Subsystem1.new
subsystem2 = Subsystem2.new
facade = Facade.new(subsystem1, subsystem2)
client_code(facade)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>

## **Fasada** w innych językach

[![Fasada w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fasada w języku C#"){.prog-lang-link}
[![Fasada w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Fasada w języku C++"){.prog-lang-link}
[![Fasada w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Fasada w języku Go"){.prog-lang-link}
[![Fasada w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Fasada w języku Java"){.prog-lang-link}
[![Fasada w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fasada w języku PHP"){.prog-lang-link}
[![Fasada w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fasada w języku Python"){.prog-lang-link}
[![Fasada w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fasada w języku Rust"){.prog-lang-link}
[![Fasada w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fasada w języku Swift"){.prog-lang-link}
[![Fasada w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fasada w języku TypeScript"){.prog-lang-link}
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
