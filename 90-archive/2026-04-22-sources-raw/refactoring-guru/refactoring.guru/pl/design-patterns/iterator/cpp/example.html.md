:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** w języku C++ {#iterator-w-języku-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** to behawioralny wzorzec projektowy pozwalający sekwencyjnie
przechodzić od elementu do elementu jakiegoś zbioru bez konieczności
eksponowania jego formy.

Dzięki Iteratorowi klienci mogą przeglądać kolejne elementy różnych
kolekcji w podobny sposób, za pośrednictwem jednego interfejsu.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Iterator ](../../iterator.html){.btn .btn-lg
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

**Przykłady użycia:** Wzorzec Iterator jest bardzo rozpowszechniony w
kodzie C++. Wiele frameworków i bibliotek pozwala za jego pomocą
poruszać się po elementach ich kolekcji.

**Identyfikacja:** Iterator łatwo rozpoznać po obecności metod
nawigacyjnych (takich jak `następny`, `poprzedni` i innych). Kod klienta
stosujący iteratory może nie mieć bezpośredniego dostępu do badanej
kolekcji.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Iterator** ze szczególnym
naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--main-cc .anchor} **main.cc:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Iterator Design Pattern
 *
 * Intent: Lets you traverse elements of a collection without exposing its
 * underlying representation (list, stack, tree, etc.).
 */

#include &lt;iostream&gt;
#include &lt;string&gt;
#include &lt;vector&gt;

/**
 * C++ has its own implementation of iterator that works with a different
 * generics containers defined by the standard library.
 */

template &lt;typename T, typename U&gt;
class Iterator {
 public:
  typedef typename std::vector&lt;T&gt;::iterator iter_type;
  Iterator(U *p_data, bool reverse = false) : m_p_data_(p_data) {
    m_it_ = m_p_data_-&gt;m_data_.begin();
  }

  void First() {
    m_it_ = m_p_data_-&gt;m_data_.begin();
  }

  void Next() {
    m_it_++;
  }

  bool IsDone() {
    return (m_it_ == m_p_data_-&gt;m_data_.end());
  }

  iter_type Current() {
    return m_it_;
  }

 private:
  U *m_p_data_;
  iter_type m_it_;
};

/**
 * Generic Collections/Containers provides one or several methods for retrieving
 * fresh iterator instances, compatible with the collection class.
 */

template &lt;class T&gt;
class Container {
  friend class Iterator&lt;T, Container&gt;;

 public:
  void Add(T a) {
    m_data_.push_back(a);
  }

  Iterator&lt;T, Container&gt; *CreateIterator() {
    return new Iterator&lt;T, Container&gt;(this);
  }

 private:
  std::vector&lt;T&gt; m_data_;
};

class Data {
 public:
  Data(int a = 0) : m_data_(a) {}

  void set_data(int a) {
    m_data_ = a;
  }

  int data() {
    return m_data_;
  }

 private:
  int m_data_;
};

/**
 * The client code may or may not know about the Concrete Iterator or Collection
 * classes, for this implementation the container is generic so you can used
 * with an int or with a custom class.
 */
void ClientCode() {
  std::cout &lt;&lt; &quot;________________Iterator with int______________________________________&quot; &lt;&lt; std::endl;
  Container&lt;int&gt; cont;

  for (int i = 0; i &lt; 10; i++) {
    cont.Add(i);
  }

  Iterator&lt;int, Container&lt;int&gt;&gt; *it = cont.CreateIterator();
  for (it-&gt;First(); !it-&gt;IsDone(); it-&gt;Next()) {
    std::cout &lt;&lt; *it-&gt;Current() &lt;&lt; std::endl;
  }

  Container&lt;Data&gt; cont2;
  Data a(100), b(1000), c(10000);
  cont2.Add(a);
  cont2.Add(b);
  cont2.Add(c);

  std::cout &lt;&lt; &quot;________________Iterator with custom Class______________________________&quot; &lt;&lt; std::endl;
  Iterator&lt;Data, Container&lt;Data&gt;&gt; *it2 = cont2.CreateIterator();
  for (it2-&gt;First(); !it2-&gt;IsDone(); it2-&gt;Next()) {
    std::cout &lt;&lt; it2-&gt;Current()-&gt;data() &lt;&lt; std::endl;
  }
  delete it;
  delete it2;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>________________Iterator with int______________________________________
0
1
2
3
4
5
6
7
8
9
________________Iterator with custom Class______________________________
100
1000
10000</code></pre>
</figure>

::: next
#### Czytaj dalej

[Mediator w języku C++ []{.fa
.fa-arrow-right}](../../mediator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Polecenie w języku
C++](../../command/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** w innych językach

[![Iterator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator w języku C#"){.prog-lang-link}
[![Iterator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator w języku Go"){.prog-lang-link}
[![Iterator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator w języku Java"){.prog-lang-link}
[![Iterator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator w języku PHP"){.prog-lang-link}
[![Iterator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator w języku Python"){.prog-lang-link}
[![Iterator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator w języku Ruby"){.prog-lang-link}
[![Iterator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator w języku Rust"){.prog-lang-link}
[![Iterator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator w języku Swift"){.prog-lang-link}
[![Iterator w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator w języku TypeScript"){.prog-lang-link}
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
