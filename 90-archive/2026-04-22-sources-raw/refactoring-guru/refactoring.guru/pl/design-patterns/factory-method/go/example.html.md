::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} / [Metoda
wytwórcza](../../factory-method.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Metoda
wytwórcza](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Metoda wytwórcza** w języku Go {#metoda-wytwórcza-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Metoda wytwórcza** jest kreacyjnym wzorcem projektowym rozwiązującym
problem tworzenia obiektów-produktów bez określania ich konkretnych
klas.

Metoda wytwórcza definiuje metodę która ma służyć tworzeniu obiektów bez
bezpośredniego wywoływania konstruktora (poprzez operator `new`).
Podklasy mogą nadpisać tę metodę w celu zmiany klasy tworzonych
obiektów.

> Jeśli masz problem ze zrozumieniem różnicy pomiędzy poszczególnymi
> koncepcjami i wzorcami wytwórczymi, przeczytaj nasze [Porównanie
> fabryk](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Metoda wytwórcza
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Gun](#example-0--iGun-go)
:::

::: en-file
 [gun](#example-0--gun-go)
:::

::: en-file
 [ak47](#example-0--ak47-go)
:::

::: en-file
 [musket](#example-0--musket-go)
:::

::: en-file
 [gun­Factory](#example-0--gunFactory-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

W Go nie da się zaimplementować klasycznego wzorca Metody wytwórczej z
racji braku takich funkcjonalności języków obiektowych jak klasy i
dziedziczenie. Możemy jednak zaimplementować jego okrojoną wersję -
Fabrykę Prostą.

W poniższym przykładzie będziemy produkować różne rodzaje broni stosując
strukturę fabryczną.

Zaczniemy od stworzenia interfejsu `iGun` definiującego wszystkie metody
właściwe broniom. Mamy typ struktury `gun` implementujący interfejs
iGun. Dwie konkretne bronie --- `ak47` i `muszkiet` zawierają strukturę
broni i pośrednio implementują wszystkie metody `iGun`.

Struktura `gunFactory` służy za fabrykę tworzącą bronie wybranego
typu na podstawie przekazanego jej argumentu. Klientem jest tu
*main.go*. Zamiast pracować bezpośrednio z `ak47` lub z `muszkietem`,
zależna jest od `gunFactory` w zakresie generowania instancji różnych
typów broni zależnie od parametrów-łańcuchów znaków.

#### []{#example-0--iGun-go .anchor} **iGun.go:** Interfejs produktu

<figure class="code">
<pre class="code" lang="go"><code>package main

type IGun interface {
    setName(name string)
    setPower(power int)
    getName() string
    getPower() int
}</code></pre>
</figure>

#### []{#example-0--gun-go .anchor} **gun.go:** Konkretny produkt

<figure class="code">
<pre class="code" lang="go"><code>package main

type Gun struct {
    name  string
    power int
}

func (g *Gun) setName(name string) {
    g.name = name
}

func (g *Gun) getName() string {
    return g.name
}

func (g *Gun) setPower(power int) {
    g.power = power
}

func (g *Gun) getPower() int {
    return g.power
}</code></pre>
</figure>

#### []{#example-0--ak47-go .anchor} **ak47.go:** Konkretny produkt

<figure class="code">
<pre class="code" lang="go"><code>package main

type Ak47 struct {
    Gun
}

func newAk47() IGun {
    return &amp;Ak47{
        Gun: Gun{
            name:  &quot;AK47 gun&quot;,
            power: 4,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--musket-go .anchor} **musket.go:** Konkretny produkt

<figure class="code">
<pre class="code" lang="go"><code>package main

type musket struct {
    Gun
}

func newMusket() IGun {
    return &amp;musket{
        Gun: Gun{
            name:  &quot;Musket gun&quot;,
            power: 1,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--gunFactory-go .anchor} **gunFactory.go:** Fabryka

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func getGun(gunType string) (IGun, error) {
    if gunType == &quot;ak47&quot; {
        return newAk47(), nil
    }
    if gunType == &quot;musket&quot; {
        return newMusket(), nil
    }
    return nil, fmt.Errorf(&quot;Wrong gun type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    ak47, _ := getGun(&quot;ak47&quot;)
    musket, _ := getGun(&quot;musket&quot;)

    printDetails(ak47)
    printDetails(musket)
}

func printDetails(g IGun) {
    fmt.Printf(&quot;Gun: %s&quot;, g.getName())
    fmt.Println()
    fmt.Printf(&quot;Power: %d&quot;, g.getPower())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Gun: AK47 gun
Power: 4
Gun: Musket gun
Power: 1</code></pre>
</figure>

::: next
#### Czytaj dalej

[Prototyp w języku Go []{.fa
.fa-arrow-right}](../../prototype/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Budowniczy w języku
Go](../../builder/go/example.html){.btn .btn-default rel="prev"}
:::

## **Metoda wytwórcza** w innych językach

[![Metoda wytwórcza w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Metoda wytwórcza w języku C#"){.prog-lang-link}
[![Metoda wytwórcza w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Metoda wytwórcza w języku C++"){.prog-lang-link}
[![Metoda wytwórcza w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Metoda wytwórcza w języku Java"){.prog-lang-link}
[![Metoda wytwórcza w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Metoda wytwórcza w języku PHP"){.prog-lang-link}
[![Metoda wytwórcza w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Metoda wytwórcza w języku Python"){.prog-lang-link}
[![Metoda wytwórcza w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Metoda wytwórcza w języku Ruby"){.prog-lang-link}
[![Metoda wytwórcza w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Metoda wytwórcza w języku Rust"){.prog-lang-link}
[![Metoda wytwórcza w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Metoda wytwórcza w języku Swift"){.prog-lang-link}
[![Metoda wytwórcza w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Metoda wytwórcza w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
