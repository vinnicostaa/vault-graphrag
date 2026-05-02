::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} / [Factory
Method](../../factory-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Factory
Method](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Factory Method** em Go {#factory-method-em-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Factory method** é um padrão de projeto criacional, que resolve o
problema de criar objetos de produtos sem especificar suas classes
concretas.

O Factory Method define um método, que deve ser usado para criar objetos
em vez da chamada direta ao construtor (operador `new`). As subclasses
podem substituir esse método para alterar a classe de objetos que serão
criados.

> Se você não conseguir descobrir a diferença entre os padrões
> **Factory**, **Factory Method** e **Abstract Factory**, leia nossa
> [Comparação Factory](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Factory Method ](../../factory-method.html){.btn
.btn-lg .btn-primary}
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

## Exemplo conceitual {#example-0-title}

É impossível implementar o padrão Factory Method clássico no Go devido à
falta de recursos OOP, como classes e herança. No entanto, ainda podemos
implementar a versão básica do padrão, o Factory Simples.

Neste exemplo, vamos construir vários tipos de armas usando uma struct
factory.

Primeiro, criamos a interface `iGun`, que define todos os métodos que
uma arma deve ter. Existe um tipo de struct `gun` que implementa a
interface iGun. Duas armas concretas --- `ak47` e `musket` --- ambas
incorporam a struct da arma e indiretamente implementam todos os métodos
`iGun`.

A struct `gunFactory` serve como um factory, que cria armas do tipo
desejado com base em um argumento de entrada. O *main.go* atua como o
cliente. Em vez de interagir diretamente com o `ak47` ou `musket`, ele
conta com o `gunFactory` para criar instâncias de várias armas, usando
apenas parâmetros de tipo string para controlar a produção.

#### []{#example-0--iGun-go .anchor} **iGun.go:** Interface do produto

<figure class="code">
<pre class="code" lang="go"><code>package main

type IGun interface {
    setName(name string)
    setPower(power int)
    getName() string
    getPower() int
}</code></pre>
</figure>

#### []{#example-0--gun-go .anchor} **gun.go:** Produto concreto

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

#### []{#example-0--ak47-go .anchor} **ak47.go:** Produto concreto

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

#### []{#example-0--musket-go .anchor} **musket.go:** Produto concreto

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

#### []{#example-0--gunFactory-go .anchor} **gunFactory.go:** Factory

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

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

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

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Gun: AK47 gun
Power: 4
Gun: Musket gun
Power: 1</code></pre>
</figure>

::: next
#### Leia a seguir

[Prototype em Go []{.fa
.fa-arrow-right}](../../prototype/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Builder em
Go](../../builder/go/example.html){.btn .btn-default rel="prev"}
:::

## **Factory Method** em outras linguagens

[![Factory Method em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Factory Method em C#"){.prog-lang-link}
[![Factory Method em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Factory Method em C++"){.prog-lang-link}
[![Factory Method em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Factory Method em Java"){.prog-lang-link}
[![Factory Method em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Factory Method em PHP"){.prog-lang-link}
[![Factory Method em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Factory Method em Python"){.prog-lang-link}
[![Factory Method em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Factory Method em Ruby"){.prog-lang-link}
[![Factory Method em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Factory Method em Rust"){.prog-lang-link}
[![Factory Method em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Factory Method em Swift"){.prog-lang-link}
[![Factory Method em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Factory Method em TypeScript"){.prog-lang-link}
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
