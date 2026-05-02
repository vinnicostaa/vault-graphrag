:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** em Go {#singleton-em-go .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Singleton** é um padrão de projeto criacional, que garante que
apenas um objeto desse tipo exista e forneça um único ponto de acesso a
ele para qualquer outro código.

O Singleton tem quase os mesmos prós e contras que as variáveis globais.
Embora sejam super úteis, eles quebram a modularidade do seu código.

Você pode usar classes que dependem de singletons em algumas outras
situações. Você terá que levar a classe singleton também. Na maioria das
vezes, essa limitação surge durante a criação de testes de unidade.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Singleton ](../../singleton.html){.btn .btn-lg
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
 [single](#example-0--single-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [Outro exemplo](#example-1)
:::

::: en-file
 [sync­Once](#example-1--syncOnce-go)
:::

::: en-file
 [main](#example-1--main-go)
:::

::: en-file
 [output](#example-1--output-txt)
:::
:::::::::::::
::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Outro exemplo](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Exemplo conceitual {#example-0-title}

Normalmente, uma instância singleton é criada quando a struct é
inicializada pela primeira vez. Para que isso aconteça, definimos o
método `getInstance` na struct. Este método será responsável por criar e
retornar a instância singleton. Uma vez criada, a mesma instância
singleton será retornada toda vez que `getInstance` for chamado.

E as goroutines? A struct singleton deve retornar a mesma instância
sempre que várias goroutines estiverem tentando acessar essa instância.
Por causa disso, é muito fácil implementar incorretamente o padrão de
design singleton. O exemplo abaixo ilustra a maneira correta de criar um
singleton.

Alguns pontos dignos de nota:

-   Há uma verificação `nil` no início para garantir que
    `singleInstance` esteja vazio na primeira vez. Isso evita operações
    de bloqueio caras toda vez que o método `getinstance` é chamado. Se
    esta verificação falhar, significa que o campo `singleInstance` já
    está preenchido.

-   A struct `singleInstance` é criada dentro do bloqueio.

-   Há outra verificação de `nil` após o bloqueio ser adquirido. Isso é
    para garantir que, se mais de uma goroutine ignorar a primeira
    verificação, apenas uma goroutine possa criar a instância singleton.
    Caso contrário, todas as goroutines criarão suas próprias instâncias
    da struct singleton.

#### []{#example-0--single-go .anchor} **single.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var lock = &amp;sync.Mutex{}

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        lock.Lock()
        defer lock.Unlock()
        if singleInstance == nil {
            fmt.Println(&quot;Creating single instance now.&quot;)
            singleInstance = &amp;single{}
        } else {
            fmt.Println(&quot;Single instance already created.&quot;)
        }
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Outro exemplo {#example-1-title}

Existem outros métodos de criação de uma instância singleton no Go:

1.  função `init`

Podemos criar uma única instância dentro da função `init`. Isso só é
aplicável se a inicialização antecipada da instância estiver ok. A
função `init` é chamada apenas uma vez por arquivo em um pacote, então
podemos ter certeza de que apenas uma única instância será criada.

2.  `sync.Once`

O `sync.Once` só executará a operação uma vez. Veja o código abaixo:

#### []{#example-1--syncOnce-go .anchor} **syncOnce.go:** Singleton

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
    &quot;sync&quot;
)

var once sync.Once

type single struct {
}

var singleInstance *single

func getInstance() *single {
    if singleInstance == nil {
        once.Do(
            func() {
                fmt.Println(&quot;Creating single instance now.&quot;)
                singleInstance = &amp;single{}
            })
    } else {
        fmt.Println(&quot;Single instance already created.&quot;)
    }

    return singleInstance
}</code></pre>
</figure>

#### []{#example-1--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

func main() {

    for i := 0; i &lt; 30; i++ {
        go getInstance()
    }

    // Scanln is similar to Scan, but stops scanning at a newline and
    // after the final item there must be a newline or EOF.
    fmt.Scanln()
}</code></pre>
</figure>

#### []{#example-1--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Exemplo conceitual](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Outro exemplo](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### Leia a seguir

[Adapter em Go []{.fa
.fa-arrow-right}](../../adapter/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Prototype em
Go](../../prototype/go/example.html){.btn .btn-default rel="prev"}
:::

## **Singleton** em outras linguagens

[![Singleton em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton em C#"){.prog-lang-link}
[![Singleton em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton em C++"){.prog-lang-link}
[![Singleton em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton em Java"){.prog-lang-link}
[![Singleton em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton em PHP"){.prog-lang-link}
[![Singleton em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton em Python"){.prog-lang-link}
[![Singleton em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton em Ruby"){.prog-lang-link}
[![Singleton em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton em Rust"){.prog-lang-link}
[![Singleton em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton em Swift"){.prog-lang-link}
[![Singleton em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
