::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** en Go {#composite-en-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
Le **Composite** est un patron de conception structurel qui permet
d'agencer les objets dans une structure ressemblant à une arborescence,
afin de pouvoir la traiter comme un objet individuel.

Le composite est devenu la solution la plus populaire pour régler les
problèmes d'une structure arborescente. Il offre une fonctionnalité très
pratique qui permet de parcourir récursivement toute l'arborescence et
d'additionner les résultats.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Composite ](../../composite.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [component](#example-0--component-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [file](#example-0--file-go)
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

## Exemple conceptuel {#example-0-title}

Essayons de comprendre le patron de conception composite en utilisant un
exemple basé sur le système de fichiers d'un système d'exploitation.
Dans le système de fichiers, nous trouverons deux types d'objets : des
fichiers et des dossiers. Dans certains cas, les fichiers et dossiers
seront traités de la même manière. C'est ici que le composite nous vient
en aide.

Imaginez que vous devez lancer une recherche sur un mot-clef particulier
dans votre système de fichiers. Ce traitement va concerner les fichiers
et les dossiers. Pour un fichier, il regarde juste son contenu et pour
un dossier, il parcourt tous ses fichiers pour trouver le mot-clef.

#### []{#example-0--component-go .anchor} **component.go:** Interface du composant

<figure class="code">
<pre class="code" lang="go"><code>package main

type Component interface {
    search(string)
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** Composite

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    components []Component
    name       string
}

func (f *Folder) search(keyword string) {
    fmt.Printf(&quot;Serching recursively for keyword %s in folder %s\n&quot;, keyword, f.name)
    for _, composite := range f.components {
        composite.search(keyword)
    }
}

func (f *Folder) add(c Component) {
    f.components = append(f.components, c)
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Feuille

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) search(keyword string) {
    fmt.Printf(&quot;Searching for keyword %s in file %s\n&quot;, keyword, f.name)
}

func (f *File) getName() string {
    return f.name
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Code client

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        name: &quot;Folder1&quot;,
    }

    folder1.add(file1)

    folder2 := &amp;Folder{
        name: &quot;Folder2&quot;,
    }
    folder2.add(file2)
    folder2.add(file3)
    folder2.add(folder1)

    folder2.search(&quot;rose&quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Serching recursively for keyword rose in folder Folder2
Searching for keyword rose in file File2
Searching for keyword rose in file File3
Serching recursively for keyword rose in folder Folder1
Searching for keyword rose in file File1</code></pre>
</figure>

::: next
#### Suivant

[Décorateur en Go []{.fa
.fa-arrow-right}](../../decorator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Pont en Go](../../bridge/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Composite** dans les autres langues

[![Composite en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite en C#"){.prog-lang-link}
[![Composite en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite en C++"){.prog-lang-link}
[![Composite en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite en Java"){.prog-lang-link}
[![Composite en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Composite en PHP"){.prog-lang-link}
[![Composite en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Composite en Python"){.prog-lang-link}
[![Composite en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite en Ruby"){.prog-lang-link}
[![Composite en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Composite en Rust"){.prog-lang-link}
[![Composite en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Composite en Swift"){.prog-lang-link}
[![Composite en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite en TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
