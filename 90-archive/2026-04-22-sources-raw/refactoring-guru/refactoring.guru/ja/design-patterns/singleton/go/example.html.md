:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Singleton](../../singleton.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Singleton](../../../../images/patterns/cards/singleton-minif559.png?id=914e1565dfdf15f240e766163bd303ec){srcset="/images/patterns/cards/singleton-mini-2x.png?id=4a7348083d7719d02709a5c9ff878b9f 2x"}
:::

# **Singleton** を Go で {#singleton-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Singleton** は[、]{.chpule2}[
]{.chpuri2}生成に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}この種類のオブジェクトがただ一つだけ存在することを保証し[、]{.chpule2}[
]{.chpuri2}他のコードに対して唯一のアクセス・ポイントを提供します[。]{.chpule2}[
]{.chpuri2}

Singleton には[、]{.chpule2}[
]{.chpuri2}大域変数とほぼ同じ長所と短所があります[。]{.chpule2}[
]{.chpuri2}両方とも随分と便利ですが[、]{.chpule2}[
]{.chpuri2}コードのモジュール性を犠牲にしています[。]{.chpule2}[
]{.chpuri2}

シングルトンのクラスに依存しているあるクラスを使う場合[、]{.chpule2}[
]{.chpuri2}シングルトンのクラスも一緒に使う必要があります[。]{.chpule2}[
]{.chpuri2}ほとんどの場合[、]{.chpule2}[
]{.chpuri2}この制限は[、]{.chpule2}[
]{.chpuri2}ユニット・テストの作成で問題となります[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Singleton の詳細 ](../../singleton.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [single](#example-0--single-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::

::: en-intro
 [もう一つの例](#example-1)
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
[概念的な例](#example-0){#example-0-tab .nav-item .nav-link .active
toggle="tab" role="tab" aria-controls="example-0" aria-selected="true"}
[もう一つの例](#example-1){#example-1-tab .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 {#example-0-title}

通常[、]{.chpule2}[
]{.chpuri2}構造体が最初に初期化される時にシングルトンのインスタンスが生成されます[。]{.chpule2}[
]{.chpuri2}これを実現するために[、]{.chpule2}[ ]{.chpuri2}構造体に
`get­Instance` メソッドを定義します[。]{.chpule2}[
]{.chpuri2}このメソッドは[、]{.chpule2}[
]{.chpuri2}シングルトンのインスタンスを生成して返します[。]{.chpule2}[
]{.chpuri2}一度生成されたシングルトンのインスタンスは[、]{.chpule2}[
]{.chpuri2}`get­Instance`
が呼ばれるたびに同じものが返されます[。]{.chpule2}[ ]{.chpuri2}

ゴルーチン[ ]{.chpule1}[（]{.chpuri1}goroutine[）]{.chpule2}[
]{.chpuri2}では[、]{.chpule2}[ ]{.chpuri2}どうでしょう[？]{.chpule2}[
]{.chpuri2}シングルトンの構造体は[、]{.chpule2}[
]{.chpuri2}複数のゴルーチンがそのインスタンスにアクセスしようとするたびに[、]{.chpule2}[
]{.chpuri2}同じインスタンスを返す必要があります[。]{.chpule2}[
]{.chpuri2}このため[、]{.chpule2}[ ]{.chpuri2}Singleton
パターンを誤って実装することがよく起きます[。]{.chpule2}[
]{.chpuri2}下記の例は[、]{.chpule2}[
]{.chpuri2}正しいシングルトンの生成方法です[。]{.chpule2}[ ]{.chpuri2}

注目すべき点[：]{.chpule2}[ ]{.chpuri2}

-   初回は `single­Instance` であることを確認するための `nil`
    の検査が冒頭にあります[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[ ]{.chpuri2}`getinstance`
    メソッドの呼び出しのたびに高価なロック操作を行うことを避けるためです[。]{.chpule2}[
    ]{.chpuri2}この検査が陰性ならば[、]{.chpule2}[ ]{.chpuri2}それは
    `single­Instance`
    フィールドはすでに値を持っているということです[。]{.chpule2}[
    ]{.chpuri2}

-   `single­Instance` 構造体は[、]{.chpule2}[
    ]{.chpuri2}ロックの範囲内で生成されます[。]{.chpule2}[ ]{.chpuri2}

-   ロック獲得後にもう一度 `nil` 検査があります[。]{.chpule2}[
    ]{.chpuri2}これは[、]{.chpule2}[
    ]{.chpuri2}二つ以上のゴルーチンが最初の検査をすり抜けた場合に[、]{.chpule2}[
    ]{.chpuri2}ゴルーチン一つだけがシングルトンのインスタンスを作成できるようにするためです[。]{.chpule2}[
    ]{.chpuri2}そしないと[、]{.chpule2}[
    ]{.chpuri2}全部のゴルーチンがそれぞれ専用のシングルトン構造体を作成してしまいます[。]{.chpule2}[
    ]{.chpuri2}

#### []{#example-0--single-go .anchor} **single.go:** シングルトン

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

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

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
## もう一つの例 {#example-1-title}

Go には[、]{.chpule2}[
]{.chpuri2}シングルトンのインスタンスを作成する他のメソッドがあります[。]{.chpule2}[
]{.chpuri2}

1.  `init` 関数

`init`
関数の内部で単一のインスタンスを作成することが可能です[。]{.chpule2}[
]{.chpuri2}これは[、]{.chpule2}[
]{.chpuri2}インスタンスの早期初期化が許されている場合のみ適用できます[。]{.chpule2}[
]{.chpuri2}`init` 関数は[、]{.chpule2}[
]{.chpuri2}パッケージ中のファイルあたり 1
回だけ呼ばれます[。]{.chpule2}[ ]{.chpuri2}ですので[、]{.chpule2}[
]{.chpuri2}ただ一つのインスタンスが作成されることを保証できます[。]{.chpule2}[
]{.chpuri2}

2.  `sync.Once`

`sync.Once` は[、]{.chpule2}[ ]{.chpuri2}1
回だけ実行されます[。]{.chpule2}[
]{.chpuri2}下記のコードをご覧ください[：]{.chpule2}[ ]{.chpuri2}

#### []{#example-1--syncOnce-go .anchor} **syncOnce.go:** シングルトン

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

#### []{#example-1--main-go .anchor} **main.go:** クライアント・コード

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

#### []{#example-1--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Creating single instance now.
Single instance already created.
Single instance already created.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[概念的な例](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [もう一つの例](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Adapter を Go で []{.fa
.fa-arrow-right}](../../adapter/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Prototype を Go
で](../../prototype/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Singleton**

[![Singleton を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Singleton を C# で"){.prog-lang-link}
[![Singleton を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Singleton を C++ で"){.prog-lang-link}
[![Singleton を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Singleton を Java で"){.prog-lang-link}
[![Singleton を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Singleton を PHP で"){.prog-lang-link}
[![Singleton を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Singleton を Python で"){.prog-lang-link}
[![Singleton を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Singleton を Ruby で"){.prog-lang-link}
[![Singleton を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Singleton を Rust で"){.prog-lang-link}
[![Singleton を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Singleton を Swift で"){.prog-lang-link}
[![Singleton を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Singleton を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
