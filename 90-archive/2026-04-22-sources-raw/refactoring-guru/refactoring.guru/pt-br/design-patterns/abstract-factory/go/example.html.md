::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Abstract
Factory](../../abstract-factory.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Abstract
Factory](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Abstract Factory** em Go {#abstract-factory-em-go .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Abstract Factory** é um padrão de projeto criacional, que resolve o
problema de criar famílias inteiras de produtos sem especificar suas
classes concretas.

O Abstract Factory define uma interface para criar todos os produtos
distintos, mas deixa a criação real do produto para classes fábrica
concretas. Cada tipo de fábrica corresponde a uma determinada variedade
de produtos.

O código cliente chama os métodos de criação de um objeto fábrica em vez
de criar produtos diretamente com uma chamada de construtor (usando
operador `new`). Como uma fábrica corresponde a uma única variante de
produto, todos os seus produtos serão compatíveis.

O código cliente trabalha com fábricas e produtos somente através de
suas interfaces abstratas. Ele permite que o mesmo código cliente
funcione com produtos diferentes. Você apenas cria uma nova classe
fábrica concreta e a passa para o código cliente.

> Se você não conseguir descobrir a diferença entre os padrões
> **Factory**, **Factory Method** e **Abstract Factory**, leia nossa
> [Comparação Factory](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Abstract Factory
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Sports­Factory](#example-0--iSportsFactory-go)
:::

::: en-file
 [adidas](#example-0--adidas-go)
:::

::: en-file
 [nike](#example-0--nike-go)
:::

::: en-file
 [i­Shoe](#example-0--iShoe-go)
:::

::: en-file
 [adidas­Shoe](#example-0--adidasShoe-go)
:::

::: en-file
 [nike­Shoe](#example-0--nikeShoe-go)
:::

::: en-file
 [i­Shirt](#example-0--iShirt-go)
:::

::: en-file
 [adidas­Shirt](#example-0--adidasShirt-go)
:::

::: en-file
 [nike­Shirt](#example-0--nikeShirt-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Digamos que você precise comprar um kit esportivo, um conjunto de dois
produtos diferentes: um par de sapatos e uma camisa. Você gostaria de
comprar um kit esportivo completo da mesma marca para combinar com todos
os itens.

Se tentarmos transformar isso em código, o abstract factory nos ajudará
a criar conjuntos de produtos para que sempre correspondam uns aos
outros.

#### []{#example-0--iSportsFactory-go .anchor} **iSportsFactory.go:** Interface do abstract factory

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ISportsFactory interface {
    makeShoe() IShoe
    makeShirt() IShirt
}

func GetSportsFactory(brand string) (ISportsFactory, error) {
    if brand == &quot;adidas&quot; {
        return &amp;Adidas{}, nil
    }

    if brand == &quot;nike&quot; {
        return &amp;Nike{}, nil
    }

    return nil, fmt.Errorf(&quot;Wrong brand type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--adidas-go .anchor} **adidas.go:** Factory concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Adidas struct {
}

func (a *Adidas) makeShoe() IShoe {
    return &amp;AdidasShoe{
        Shoe: Shoe{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}

func (a *Adidas) makeShirt() IShirt {
    return &amp;AdidasShirt{
        Shirt: Shirt{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--nike-go .anchor} **nike.go:** Factory concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nike struct {
}

func (n *Nike) makeShoe() IShoe {
    return &amp;NikeShoe{
        Shoe: Shoe{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}

func (n *Nike) makeShirt() IShirt {
    return &amp;NikeShirt{
        Shirt: Shirt{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--iShoe-go .anchor} **iShoe.go:** Produto abstrato

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shoe struct {
    logo string
    size int
}

func (s *Shoe) setLogo(logo string) {
    s.logo = logo
}

func (s *Shoe) getLogo() string {
    return s.logo
}

func (s *Shoe) setSize(size int) {
    s.size = size
}

func (s *Shoe) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShoe-go .anchor} **adidasShoe.go:** Produto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--nikeShoe-go .anchor} **nikeShoe.go:** Produto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--iShirt-go .anchor} **iShirt.go:** Produto abstrato

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShirt interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shirt struct {
    logo string
    size int
}

func (s *Shirt) setLogo(logo string) {
    s.logo = logo
}

func (s *Shirt) getLogo() string {
    return s.logo
}

func (s *Shirt) setSize(size int) {
    s.size = size
}

func (s *Shirt) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShirt-go .anchor} **adidasShirt.go:** Produto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--nikeShirt-go .anchor} **nikeShirt.go:** Produto concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    adidasFactory, _ := GetSportsFactory(&quot;adidas&quot;)
    nikeFactory, _ := GetSportsFactory(&quot;nike&quot;)

    nikeShoe := nikeFactory.makeShoe()
    nikeShirt := nikeFactory.makeShirt()

    adidasShoe := adidasFactory.makeShoe()
    adidasShirt := adidasFactory.makeShirt()

    printShoeDetails(nikeShoe)
    printShirtDetails(nikeShirt)

    printShoeDetails(adidasShoe)
    printShirtDetails(adidasShirt)
}

func printShoeDetails(s IShoe) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}

func printShirtDetails(s IShirt) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Logo: nike
Size: 14
Logo: nike
Size: 14
Logo: adidas
Size: 14
Logo: adidas
Size: 14</code></pre>
</figure>

::: next
#### Leia a seguir

[Builder em Go []{.fa
.fa-arrow-right}](../../builder/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Padrões de Projeto em Go](../../go.html){.btn
.btn-default rel="prev"}
:::

## **Abstract Factory** em outras linguagens

[![Abstract Factory em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Abstract Factory em C#"){.prog-lang-link}
[![Abstract Factory em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Abstract Factory em C++"){.prog-lang-link}
[![Abstract Factory em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Abstract Factory em Java"){.prog-lang-link}
[![Abstract Factory em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Abstract Factory em PHP"){.prog-lang-link}
[![Abstract Factory em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Abstract Factory em Python"){.prog-lang-link}
[![Abstract Factory em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Abstract Factory em Ruby"){.prog-lang-link}
[![Abstract Factory em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Abstract Factory em Rust"){.prog-lang-link}
[![Abstract Factory em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Abstract Factory em Swift"){.prog-lang-link}
[![Abstract Factory em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Abstract Factory em TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
