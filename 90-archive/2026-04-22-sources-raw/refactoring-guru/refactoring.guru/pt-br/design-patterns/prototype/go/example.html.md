::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** em Go {#prototype-em-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
O **Prototype** é um padrão de projeto criacional que permite a clonagem
de objetos, mesmo complexos, sem acoplamento à suas classes específicas.

Todas as classes de prototypes(protótipos) devem ter uma interface comum
que permita copiar objetos, mesmo que suas classes concretas sejam
desconhecidas. Objetos protótipos podem produzir cópias completas, pois
objetos da mesma classe podem acessar os campos privados um do outro.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Prototype ](../../prototype.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
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

## Exemplo conceitual {#example-0-title}

Vamos tentar descobrir o padrão Prototype usando um exemplo baseado no
sistema de arquivos do sistema operacional. O sistema de arquivos do SO
é recursivo: as pastas contêm arquivos e pastas, que também podem
incluir arquivos e pastas e assim por diante.

Cada arquivo e pasta pode ser representado por uma interface `inode`. A
interface `inode` também possui a função `clone`.

Ambas as structs `file` e `folder` implementam as funções `print` e
`clone`, uma vez que são do tipo `inode`. Além disso, observe a função
`clone` em `file` e `folder`. A função `clone` em ambos retorna uma
cópia do respectivo arquivo ou pasta. Durante a clonagem, acrescentamos
a palavra-chave "\_clone" ao campo de nome.

#### []{#example-0--inode-go .anchor} **inode.go:** Interface do prototype

<figure class="code">
<pre class="code" lang="go"><code>package main

type Inode interface {
    print(string)
    clone() Inode
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Prototype concreto

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

#### []{#example-0--folder-go .anchor} **folder.go:** Prototype concreto

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

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

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
#### Leia a seguir

[Singleton em Go []{.fa
.fa-arrow-right}](../../singleton/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Factory Method em
Go](../../factory-method/go/example.html){.btn .btn-default rel="prev"}
:::

## **Prototype** em outras linguagens

[![Prototype em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Prototype em C#"){.prog-lang-link}
[![Prototype em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype em C++"){.prog-lang-link}
[![Prototype em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype em Java"){.prog-lang-link}
[![Prototype em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototype em PHP"){.prog-lang-link}
[![Prototype em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype em Python"){.prog-lang-link}
[![Prototype em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype em Ruby"){.prog-lang-link}
[![Prototype em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype em Rust"){.prog-lang-link}
[![Prototype em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype em Swift"){.prog-lang-link}
[![Prototype em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype em TypeScript"){.prog-lang-link}
::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
