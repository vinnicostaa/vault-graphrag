:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** を Go で {#command-を-go-で .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}リクエストや簡単な操作をオブジェクトに変換します[。]{.chpule2}[
]{.chpuri2}

変換により[、]{.chpule2}[
]{.chpuri2}コマンドの遅延実行や遠隔実行を可能にしたり[、]{.chpule2}[
]{.chpuri2}コマンドの履歴の保存を可能にしたりできます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Command の詳細 ](../../command.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例](#example-0)
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

## 概念的な例 {#example-0-title}

Command パターンを[、]{.chpule2}[
]{.chpuri2}テレビを例にとってみてみましょう[。]{.chpule2}[
]{.chpuri2}テレビの電源を入れるには[、]{.chpule2}[
]{.chpuri2}以下のどちらかを押します[：]{.chpule2}[ ]{.chpuri2}

-   リモコンのオンのボタン
-   実際のテレビのオンのボタン

[ ]{.chpule1}[「]{.chpuri1}オンにする[」]{.chpule2}[
]{.chpuri2}コマンド・オブジェクトの実装は[、]{.chpule2}[
]{.chpuri2}テレビを受け手として開始します[。]{.chpule2}[
]{.chpuri2}このコマンドの実行メソッドが呼ばれると[、]{.chpule2}[
]{.chpuri2}それは次に `TV.on` 関数を呼び出します[。]{.chpule2}[
]{.chpuri2}次に[、]{.chpule2}[
]{.chpuri2}インボーカーを定義します[。]{.chpule2}[
]{.chpuri2}実際のところ[、]{.chpule2}[
]{.chpuri2}インボーカーは二つあります[。]{.chpule2}[
]{.chpuri2}リモコンとテレビ自身です[。]{.chpule2}[ ]{.chpuri2}両方とも[
]{.chpule1}[「]{.chpuri1}オンにする[」]{.chpule2}[
]{.chpuri2}コマンド・オブジェクトを埋め込んでいます[。]{.chpule2}[
]{.chpuri2}

同じリクエストを複数のインボーカーでラップしていることに注目してください[。]{.chpule2}[
]{.chpuri2}他のコマンドでも同様です[。]{.chpule2}[
]{.chpuri2}別々のコマンド・オブジェクトを作成する利点は[、]{.chpule2}[
]{.chpuri2}UI
ロジックをビジネス・ロジックから切り離すことです[。]{.chpule2}[
]{.chpuri2}インボーカーごとに異なるハンドラーを開発する必要はありません[。]{.chpule2}[
]{.chpuri2}コマンド・オブジェクトには[、]{.chpule2}[
]{.chpuri2}実行に必要なすべての情報が含まれています[。]{.chpule2}[
]{.chpuri2}そのため[、]{.chpule2}[
]{.chpuri2}遅延実行にも使用できます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--button-go .anchor} **button.go:** インボーカー

<figure class="code">
<pre class="code" lang="go"><code>package main

type Button struct {
    command Command
}

func (b *Button) press() {
    b.command.execute()
}</code></pre>
</figure>

#### []{#example-0--command-go .anchor} **command.go:** コマンド・インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type Command interface {
    execute()
}</code></pre>
</figure>

#### []{#example-0--onCommand-go .anchor} **onCommand.go:** 具象コマンド

<figure class="code">
<pre class="code" lang="go"><code>package main

type OnCommand struct {
    device Device
}

func (c *OnCommand) execute() {
    c.device.on()
}</code></pre>
</figure>

#### []{#example-0--offCommand-go .anchor} **offCommand.go:** 具象コマンド

<figure class="code">
<pre class="code" lang="go"><code>package main

type OffCommand struct {
    device Device
}

func (c *OffCommand) execute() {
    c.device.off()
}</code></pre>
</figure>

#### []{#example-0--device-go .anchor} **device.go:** 受け手インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type Device interface {
    on()
    off()
}</code></pre>
</figure>

#### []{#example-0--tv-go .anchor} **tv.go:** 具象受け手

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

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Turning tv on
Turning tv off</code></pre>
</figure>

::: next
#### 次を読む

[Iterator を Go で []{.fa
.fa-arrow-right}](../../iterator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Chain of Responsibility を Go
で](../../chain-of-responsibility/go/example.html){.btn .btn-default
rel="prev"}
:::

## 他言語での **Command**

[![Command を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command を C# で"){.prog-lang-link}
[![Command を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command を C++ で"){.prog-lang-link}
[![Command を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Command を Java で"){.prog-lang-link}
[![Command を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command を PHP で"){.prog-lang-link}
[![Command を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command を Python で"){.prog-lang-link}
[![Command を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command を Ruby で"){.prog-lang-link}
[![Command を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Command を Rust で"){.prog-lang-link}
[![Command を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command を Swift で"){.prog-lang-link}
[![Command を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
