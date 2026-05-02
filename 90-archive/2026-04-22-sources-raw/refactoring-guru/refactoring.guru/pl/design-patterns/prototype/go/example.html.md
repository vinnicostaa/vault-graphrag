::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Prototyp](../../prototype.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototyp](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototyp** w języku Go {#prototyp-w-języku-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Prototyp** To kreacyjny wzorzec projektowy pozwalający klonować
obiekty --- również te złożone --- bez konieczności sprzęgania z ich
klasami.

Wszystkie klasy prototyp powinny mieć wspólny interfejs który pozwoli
kopiować ich obiekty nawet gdy nie zna się ich konkretnych klas.
Obiekty-prototypy mogą tworzyć kompletne kopie, ponieważ pola prywatne
danej klasy są dostępne dla innych obiektów tej samej klasy.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Prototyp ](../../prototype.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [inode](#example-0--inode-go)
:::

::: en-file
 [file](#example-0--file-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Spróbujmy pojąć wzorzec Prototyp na przykładzie odnoszącym się do
systemu plików systemu operacyjnego. System plików jest rekursywny:
foldery zawierają pliki i inne foldery, które również mogą zawierać
pliki i foldery i tak dalej.

Każdy plik i folder można przedstawić stosując interfejs `inode`.
Interfejs `inode` zawiera także funkcję `clone`.

Zarówno struktury `file` jak i `folder` implementują funkcje `print` i
`clone` gdyż są one typu `inode`. Zwróćmy też uwagę, że funkcja
`clone` w `file` i w `folder` zwraca kopię --- odpowiednio --- pliku lub
folderu. W momencie klonowania do pola nazwy dołączamy słowo kluczowe
"\_clone".

#### []{#example-0--inode-go .anchor} **inode.go:** Prototype interface

<figure class="code">
<pre class="code" lang="go"><code>package main

type Inode interface {
    print(string)
    clone() Inode
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Concrete prototype

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) print(indentation string) {
    fmt.Println(indentation + f.name)
}

func (f *File) clone() Inode {
    return &amp;File{name: f.name + &quot;_clone&quot;}
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** Concrete prototype

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    children []Inode
    name     string
}

func (f *Folder) print(indentation string) {
    fmt.Println(indentation + f.name)
    for _, i := range f.children {
        i.print(indentation + indentation)
    }
}

func (f *Folder) clone() Inode {
    cloneFolder := &amp;Folder{name: f.name + &quot;_clone&quot;}
    var tempChildren []Inode
    for _, i := range f.children {
        copy := i.clone()
        tempChildren = append(tempChildren, copy)
    }
    cloneFolder.children = tempChildren
    return cloneFolder
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Kod klienta

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        children: []Inode{file1},
        name:     &quot;Folder1&quot;,
    }

    folder2 := &amp;Folder{
        children: []Inode{folder1, file2, file3},
        name:     &quot;Folder2&quot;,
    }
    fmt.Println(&quot;\nPrinting hierarchy for Folder2&quot;)
    folder2.print(&quot;  &quot;)

    cloneFolder := folder2.clone()
    fmt.Println(&quot;\nPrinting hierarchy for clone Folder&quot;)
    cloneFolder.print(&quot;  &quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Wynik działania

<figure class="code">
<pre class="code" lang="output"><code>Printing hierarchy for Folder2
  Folder2
    Folder1
        File1
    File2
    File3

Printing hierarchy for clone Folder
  Folder2_clone
    Folder1_clone
        File1_clone
    File2_clone
    File3_clone</code></pre>
</figure>

::: next
#### Czytaj dalej

[Singleton w języku Go []{.fa
.fa-arrow-right}](../../singleton/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Metoda wytwórcza w języku
Go](../../factory-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Prototyp** w innych językach

[![Prototyp w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototyp w języku C#"){.prog-lang-link}
[![Prototyp w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototyp w języku C++"){.prog-lang-link}
[![Prototyp w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototyp w języku Java"){.prog-lang-link}
[![Prototyp w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototyp w języku PHP"){.prog-lang-link}
[![Prototyp w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototyp w języku Python"){.prog-lang-link}
[![Prototyp w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototyp w języku Ruby"){.prog-lang-link}
[![Prototyp w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototyp w języku Rust"){.prog-lang-link}
[![Prototyp w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototyp w języku Swift"){.prog-lang-link}
[![Prototyp w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototyp w języku TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
