::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Builder](../../builder.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Builder](../../../../images/patterns/cards/builder-mini6d24.png?id=19b95fd05e6469679752c0554b116815){srcset="/images/patterns/cards/builder-mini-2x.png?id=de6d0938678b86903a1426dddfdd13bf 2x"}
:::

# **Builder** em Go {#builder-em-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Builder** é um padrão de projeto criacional, que permite a
construção de objetos complexos passo a passo.

Diferente de outros padrões de criação, o Builder não exige que os
produtos tenham uma interface comum. Isso torna possível produzir
produtos diferentes usando o mesmo processo de construção.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Builder ](../../builder.html){.btn .btn-lg
.btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Builder](#example-0--iBuilder-go)
:::

::: en-file
 [normal­Builder](#example-0--normalBuilder-go)
:::

::: en-file
 [igloo­Builder](#example-0--iglooBuilder-go)
:::

::: en-file
 [house](#example-0--house-go)
:::

::: en-file
 [director](#example-0--director-go)
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

## Exemplo conceitual {#example-0-title}

O padrão Builder também é usado quando o produto desejado é complexo e
requer várias etapas para ser concluído. Nesse caso, vários métodos de
construção seriam mais simples do que um único construtor monstruoso. O
problema potencial com o processo de construção de vários estágios é que
um produto parcialmente construído e instável pode ser exposto ao
cliente. O padrão Builder mantém o produto privado até que seja
totalmente construído.

No código abaixo, vemos diferentes tipos de casas (`igloo` e
`normalHouse`) sendo construídas por `iglooBuilder` e `normalBuilder`.
Cada tipo de casa tem as mesmas etapas de construção. A struct opcional
diretor ajuda a organizar o processo de construção.

#### []{#example-0--iBuilder-go .anchor} **iBuilder.go:** Interface do builder

<figure class="code">
<pre class="code" lang="go"><code>package main

type IBuilder interface {
    setWindowType()
    setDoorType()
    setNumFloor()
    getHouse() House
}

func getBuilder(builderType string) IBuilder {
    if builderType == &quot;normal&quot; {
        return newNormalBuilder()
    }

    if builderType == &quot;igloo&quot; {
        return newIglooBuilder()
    }
    return nil
}</code></pre>
</figure>

#### []{#example-0--normalBuilder-go .anchor} **normalBuilder.go:** Builder concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type NormalBuilder struct {
    windowType string
    doorType   string
    floor      int
}

func newNormalBuilder() *NormalBuilder {
    return &amp;NormalBuilder{}
}

func (b *NormalBuilder) setWindowType() {
    b.windowType = &quot;Wooden Window&quot;
}

func (b *NormalBuilder) setDoorType() {
    b.doorType = &quot;Wooden Door&quot;
}

func (b *NormalBuilder) setNumFloor() {
    b.floor = 2
}

func (b *NormalBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}</code></pre>
</figure>

#### []{#example-0--iglooBuilder-go .anchor} **iglooBuilder.go:** Builder concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type IglooBuilder struct {
    windowType string
    doorType   string
    floor      int
}

func newIglooBuilder() *IglooBuilder {
    return &amp;IglooBuilder{}
}

func (b *IglooBuilder) setWindowType() {
    b.windowType = &quot;Snow Window&quot;
}

func (b *IglooBuilder) setDoorType() {
    b.doorType = &quot;Snow Door&quot;
}

func (b *IglooBuilder) setNumFloor() {
    b.floor = 1
}

func (b *IglooBuilder) getHouse() House {
    return House{
        doorType:   b.doorType,
        windowType: b.windowType,
        floor:      b.floor,
    }
}</code></pre>
</figure>

#### []{#example-0--house-go .anchor} **house.go:** Produto

<figure class="code">
<pre class="code" lang="go"><code>package main

type House struct {
    windowType string
    doorType   string
    floor      int
}</code></pre>
</figure>

#### []{#example-0--director-go .anchor} **director.go:** Diretor

<figure class="code">
<pre class="code" lang="go"><code>package main

type Director struct {
    builder IBuilder
}

func newDirector(b IBuilder) *Director {
    return &amp;Director{
        builder: b,
    }
}

func (d *Director) setBuilder(b IBuilder) {
    d.builder = b
}

func (d *Director) buildHouse() House {
    d.builder.setDoorType()
    d.builder.setWindowType()
    d.builder.setNumFloor()
    return d.builder.getHouse()
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    normalBuilder := getBuilder(&quot;normal&quot;)
    iglooBuilder := getBuilder(&quot;igloo&quot;)

    director := newDirector(normalBuilder)
    normalHouse := director.buildHouse()

    fmt.Printf(&quot;Normal House Door Type: %s\n&quot;, normalHouse.doorType)
    fmt.Printf(&quot;Normal House Window Type: %s\n&quot;, normalHouse.windowType)
    fmt.Printf(&quot;Normal House Num Floor: %d\n&quot;, normalHouse.floor)

    director.setBuilder(iglooBuilder)
    iglooHouse := director.buildHouse()

    fmt.Printf(&quot;\nIgloo House Door Type: %s\n&quot;, iglooHouse.doorType)
    fmt.Printf(&quot;Igloo House Window Type: %s\n&quot;, iglooHouse.windowType)
    fmt.Printf(&quot;Igloo House Num Floor: %d\n&quot;, iglooHouse.floor)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Normal House Door Type: Wooden Door
Normal House Window Type: Wooden Window
Normal House Num Floor: 2

Igloo House Door Type: Snow Door
Igloo House Window Type: Snow Window
Igloo House Num Floor: 1</code></pre>
</figure>

::: next
#### Leia a seguir

[Factory Method em Go []{.fa
.fa-arrow-right}](../../factory-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Abstract Factory em
Go](../../abstract-factory/go/example.html){.btn .btn-default
rel="prev"}
:::

## **Builder** em outras linguagens

[![Builder em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Builder em C#"){.prog-lang-link}
[![Builder em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Builder em C++"){.prog-lang-link}
[![Builder em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Builder em Java"){.prog-lang-link}
[![Builder em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Builder em PHP"){.prog-lang-link}
[![Builder em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Builder em Python"){.prog-lang-link}
[![Builder em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Builder em Ruby"){.prog-lang-link}
[![Builder em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Builder em Rust"){.prog-lang-link}
[![Builder em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Builder em Swift"){.prog-lang-link}
[![Builder em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Builder em TypeScript"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
