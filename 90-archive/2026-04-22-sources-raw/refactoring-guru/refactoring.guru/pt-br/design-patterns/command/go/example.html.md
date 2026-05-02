:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** em Go {#command-em-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Command** é um padrão de projeto comportamental que converte
solicitações ou operações simples em objetos.

A conversão permite a execução adiada ou remota de comandos,
armazenamento do histórico de comandos, etc.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Command ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [button](#example-0--button-go)
:::

::: en-file
 [command](#example-0--command-go)
:::

::: en-file
 [on­Command](#example-0--onCommand-go)
:::

::: en-file
 [off­Command](#example-0--offCommand-go)
:::

::: en-file
 [device](#example-0--device-go)
:::

::: en-file
 [tv](#example-0--tv-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Vejamos o padrão Command com o caso de uma TV. Uma TV pode ser LIGADA
por:

-   Botão LIGAR no controle remoto;
-   Botão LIGAR na própria TV.

Podemos começar implementando o objeto de comando LIGAR com a TV como um
receptor. Quando o método `execute` é chamado neste comando, ele, por
sua vez, chama a função `TV.on`. A última parte é definir um invocador.
Na verdade, teremos dois invocadores: o controle remoto e a própria TV.
Ambos irão incorporar o objeto de comando LIGAR.

Observe como empacotamos a mesma solicitação em vários invocadores. Da
mesma forma que podemos fazer com outros comandos. A vantagem de criar
um objeto de comando separado é que separamos a lógica da IU da lógica
do negócio subjacente. Não há necessidade de desenvolver handlers
diferentes para cada um dos invocadores. O objeto de comando contém
todas as informações de que precisa para ser executado. Portanto, também
pode ser usado para uma execução atrasada.

#### []{#example-0--button-go .anchor} **button.go:** Invocador

<figure class="code">
<pre class="code" lang="go"><code>package main

type Button struct {
    command Command
}

func (b *Button) press() {
    b.command.execute()
}</code></pre>
</figure>

#### []{#example-0--command-go .anchor} **command.go:** Interface do command

<figure class="code">
<pre class="code" lang="go"><code>package main

type Command interface {
    execute()
}</code></pre>
</figure>

#### []{#example-0--onCommand-go .anchor} **onCommand.go:** Command concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type OnCommand struct {
    device Device
}

func (c *OnCommand) execute() {
    c.device.on()
}</code></pre>
</figure>

#### []{#example-0--offCommand-go .anchor} **offCommand.go:** Command concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

type OffCommand struct {
    device Device
}

func (c *OffCommand) execute() {
    c.device.off()
}</code></pre>
</figure>

#### []{#example-0--device-go .anchor} **device.go:** Interface do receptor

<figure class="code">
<pre class="code" lang="go"><code>package main

type Device interface {
    on()
    off()
}</code></pre>
</figure>

#### []{#example-0--tv-go .anchor} **tv.go:** Receptor concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Tv struct {
    isRunning bool
}

func (t *Tv) on() {
    t.isRunning = true
    fmt.Println(&quot;Turning tv on&quot;)
}

func (t *Tv) off() {
    t.isRunning = false
    fmt.Println(&quot;Turning tv off&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    tv := &amp;Tv{}

    onCommand := &amp;OnCommand{
        device: tv,
    }

    offCommand := &amp;OffCommand{
        device: tv,
    }

    onButton := &amp;Button{
        command: onCommand,
    }
    onButton.press()

    offButton := &amp;Button{
        command: offCommand,
    }
    offButton.press()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Turning tv on
Turning tv off</code></pre>
</figure>

::: next
#### Leia a seguir

[Iterator em Go []{.fa
.fa-arrow-right}](../../iterator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Chain of Responsibility em
Go](../../chain-of-responsibility/go/example.html){.btn .btn-default
rel="prev"}
:::

## **Command** em outras linguagens

[![Command em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command em C#"){.prog-lang-link}
[![Command em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command em C++"){.prog-lang-link}
[![Command em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Command em Java"){.prog-lang-link}
[![Command em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command em PHP"){.prog-lang-link}
[![Command em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command em Python"){.prog-lang-link}
[![Command em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command em Ruby"){.prog-lang-link}
[![Command em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command em Rust"){.prog-lang-link}
[![Command em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command em Swift"){.prog-lang-link}
[![Command em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
