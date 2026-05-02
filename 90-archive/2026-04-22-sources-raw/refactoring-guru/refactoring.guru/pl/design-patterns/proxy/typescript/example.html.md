:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Pełnomocnik](../../proxy.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Pełnomocnik](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Pełnomocnik** w języku TypeScript {#pełnomocnik-w-języku-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Pełnomocnik** to strukturalny wzorzec projektowy według którego
obiekt-usługodawca używany przez klienta jest zastępowany przez obiekt
zastępczy, zwany pełnomocnikiem. Pełnomocnik przechwytuje żądania od
klienta, wykonuje jakąś pracę (kontrola dostępu, zarządzanie pamięcią
podręczną, itp.) a następnie przekazuje żądanie usługodawcy.

Obiekt będący pełnomocnikiem ma ten sam interfejs co usługodawca, co
czyni go wymienialnym z obiektem usługodawcy dotychczas przekazywanym
klientowi.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Pełnomocnik ](../../proxy.html){.btn .btn-lg
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Pełnomocnik nie jest częstym gościem w aplikacjach
napisanych w TypeScript, ale w niektórych wyjątkowych sytuacjach bardzo
się przydaje. Jest niezastąpiony wszędzie tam, gdzie trzeba dodać jakąś
funkcjonalność obiektowi istniejącej klasy bez zmiany kodu klienta.

**Identyfikacja:** Pełnomocnicy delegują całą faktyczną pracę innemu
obiektowi. Każda metoda pełnomocnika powinna odnosić się do
obiektu-usługodawcy, chyba, że pełnomocnik jest klasą pochodną usługi.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Pełnomocnik** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--index-ts .anchor} **index.ts:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Subject interface declares common operations for both RealSubject and the
 * Proxy. As long as the client works with RealSubject using this interface,
 * you&#39;ll be able to pass it a proxy instead of a real subject.
 */
interface Subject {
    request(): void;
}

/**
 * The RealSubject contains some core business logic. Usually, RealSubjects are
 * capable of doing some useful work which may also be very slow or sensitive -
 * e.g. correcting input data. A Proxy can solve these issues without any
 * changes to the RealSubject&#39;s code.
 */
class RealSubject implements Subject {
    public request(): void {
        console.log(&#39;RealSubject: Handling request.&#39;);
    }
}

/**
 * The Proxy has an interface identical to the RealSubject.
 */
class Proxy implements Subject {
    private realSubject: RealSubject;

    /**
     * The Proxy maintains a reference to an object of the RealSubject class. It
     * can be either lazy-loaded or passed to the Proxy by the client.
     */
    constructor(realSubject: RealSubject) {
        this.realSubject = realSubject;
    }

    /**
     * The most common applications of the Proxy pattern are lazy loading,
     * caching, controlling the access, logging, etc. A Proxy can perform one of
     * these things and then, depending on the result, pass the execution to the
     * same method in a linked RealSubject object.
     */
    public request(): void {
        if (this.checkAccess()) {
            this.realSubject.request();
            this.logAccess();
        }
    }

    private checkAccess(): boolean {
        // Some real checks should go here.
        console.log(&#39;Proxy: Checking access prior to firing a real request.&#39;);

        return true;
    }

    private logAccess(): void {
        console.log(&#39;Proxy: Logging the time of request.&#39;);
    }
}

/**
 * The client code is supposed to work with all objects (both subjects and
 * proxies) via the Subject interface in order to support both real subjects and
 * proxies. In real life, however, clients mostly work with their real subjects
 * directly. In this case, to implement the pattern more easily, you can extend
 * your proxy from the real subject&#39;s class.
 */
function clientCode(subject: Subject) {
    // ...

    subject.request();

    // ...
}

console.log(&#39;Client: Executing the client code with a real subject:&#39;);
const realSubject = new RealSubject();
clientCode(realSubject);

console.log(&#39;&#39;);

console.log(&#39;Client: Executing the same client code with a proxy:&#39;);
const proxy = new Proxy(realSubject);
clientCode(proxy);</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>

::: next
#### Czytaj dalej

[Łańcuch zobowiązań w języku TypeScript []{.fa
.fa-arrow-right}](../../chain-of-responsibility/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pyłek w języku
TypeScript](../../flyweight/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Pełnomocnik** w innych językach

[![Pełnomocnik w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Pełnomocnik w języku C#"){.prog-lang-link}
[![Pełnomocnik w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Pełnomocnik w języku C++"){.prog-lang-link}
[![Pełnomocnik w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Pełnomocnik w języku Go"){.prog-lang-link}
[![Pełnomocnik w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Pełnomocnik w języku Java"){.prog-lang-link}
[![Pełnomocnik w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Pełnomocnik w języku PHP"){.prog-lang-link}
[![Pełnomocnik w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Pełnomocnik w języku Python"){.prog-lang-link}
[![Pełnomocnik w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Pełnomocnik w języku Ruby"){.prog-lang-link}
[![Pełnomocnik w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Pełnomocnik w języku Rust"){.prog-lang-link}
[![Pełnomocnik w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Pełnomocnik w języku Swift"){.prog-lang-link}
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
