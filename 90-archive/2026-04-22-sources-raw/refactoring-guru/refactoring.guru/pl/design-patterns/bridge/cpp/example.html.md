:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Most](../../bridge.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Most](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Most** w języku C++ {#most-w-języku-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Most** jest strukturalnym wzorcem projektowym zakładającym podział
logiki biznesowej lub dużej klasy na osobne hierarchie klas które
następnie można rozwijać niezależnie od siebie.

Jedna z takich hierarchii (zwana często Abstrakcją) posiada
referencję do obiektu drugiej hierarchii (zwanej Implementacją) i
deleguje mu część (czasem większość) wywołań. Ponieważ wszystkie
implementacje mają wspólny interfejs, z punktu widzenia abstrakcji są
wymienialne.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Most ](../../bridge.html){.btn .btn-lg
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Most jest szczególnie przydatny gdy trzeba
wspierać obsługę wielu typów serwerów bazodanowych lub interfejsów
programowania aplikacji danego typu (na przykład chmura, platformy
społecznościowe, itd.)

**Identyfikacja:** Most można rozpoznać po wyraźnym rozdzieleniu na
część kontrolującą i wiele różnych platform od których ta część zależy.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Most** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-cc .anchor} **main.cc:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Implementation defines the interface for all implementation classes. It
 * doesn&#39;t have to match the Abstraction&#39;s interface. In fact, the two
 * interfaces can be entirely different. Typically the Implementation interface
 * provides only primitive operations, while the Abstraction defines higher-
 * level operations based on those primitives.
 */

class Implementation {
 public:
  virtual ~Implementation() {}
  virtual std::string OperationImplementation() const = 0;
};

/**
 * Each Concrete Implementation corresponds to a specific platform and
 * implements the Implementation interface using that platform&#39;s API.
 */
class ConcreteImplementationA : public Implementation {
 public:
  std::string OperationImplementation() const override {
    return &quot;ConcreteImplementationA: Here&#39;s the result on the platform A.\n&quot;;
  }
};
class ConcreteImplementationB : public Implementation {
 public:
  std::string OperationImplementation() const override {
    return &quot;ConcreteImplementationB: Here&#39;s the result on the platform B.\n&quot;;
  }
};

/**
 * The Abstraction defines the interface for the &quot;control&quot; part of the two class
 * hierarchies. It maintains a reference to an object of the Implementation
 * hierarchy and delegates all of the real work to this object.
 */

class Abstraction {
  /**
   * @var Implementation
   */
 protected:
  Implementation* implementation_;

 public:
  Abstraction(Implementation* implementation) : implementation_(implementation) {
  }

  virtual ~Abstraction() {
  }

  virtual std::string Operation() const {
    return &quot;Abstraction: Base operation with:\n&quot; +
           this-&gt;implementation_-&gt;OperationImplementation();
  }
};
/**
 * You can extend the Abstraction without changing the Implementation classes.
 */
class ExtendedAbstraction : public Abstraction {
 public:
  ExtendedAbstraction(Implementation* implementation) : Abstraction(implementation) {
  }
  std::string Operation() const override {
    return &quot;ExtendedAbstraction: Extended operation with:\n&quot; +
           this-&gt;implementation_-&gt;OperationImplementation();
  }
};

/**
 * Except for the initialization phase, where an Abstraction object gets linked
 * with a specific Implementation object, the client code should only depend on
 * the Abstraction class. This way the client code can support any abstraction-
 * implementation combination.
 */
void ClientCode(const Abstraction&amp; abstraction) {
  // ...
  std::cout &lt;&lt; abstraction.Operation();
  // ...
}
/**
 * The client code should be able to work with any pre-configured abstraction-
 * implementation combination.
 */

int main() {
  Implementation* implementation = new ConcreteImplementationA;
  Abstraction* abstraction = new Abstraction(implementation);
  ClientCode(*abstraction);
  std::cout &lt;&lt; std::endl;
  delete implementation;
  delete abstraction;

  implementation = new ConcreteImplementationB;
  abstraction = new ExtendedAbstraction(implementation);
  ClientCode(*abstraction);

  delete implementation;
  delete abstraction;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Abstraction: Base operation with:
ConcreteImplementationA: Here&#39;s the result on the platform A.

ExtendedAbstraction: Extended operation with:
ConcreteImplementationB: Here&#39;s the result on the platform B.</code></pre>
</figure>

::: next
#### Czytaj dalej

[Kompozyt w języku C++ []{.fa
.fa-arrow-right}](../../composite/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Adapter w języku
C++](../../adapter/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Most** w innych językach

[![Most w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Most w języku C#"){.prog-lang-link}
[![Most w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Most w języku Go"){.prog-lang-link}
[![Most w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Most w języku Java"){.prog-lang-link}
[![Most w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Most w języku PHP"){.prog-lang-link}
[![Most w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Most w języku Python"){.prog-lang-link}
[![Most w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Most w języku Ruby"){.prog-lang-link}
[![Most w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Most w języku Rust"){.prog-lang-link}
[![Most w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Most w języku Swift"){.prog-lang-link}
[![Most w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Most w języku TypeScript"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
