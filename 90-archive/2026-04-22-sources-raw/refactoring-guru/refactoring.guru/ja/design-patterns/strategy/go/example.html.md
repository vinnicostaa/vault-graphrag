::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Strategy](../../strategy.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Strategy](../../../../images/patterns/cards/strategy-mini28f6.png?id=d38abee4fb6f2aed909d262bdadca936){srcset="/images/patterns/cards/strategy-mini-2x.png?id=f4e6608561f8e5d18be6927d4620ad29 2x"}
:::

# **Strategy** を Go で {#strategy-を-go-で .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**Strategy** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}一連の振る舞いをオブジェクトに転換し[、]{.chpule2}[
]{.chpuri2}元のコンテキスト・オブジェクト内で交換可能とします[。]{.chpule2}[
]{.chpuri2}

元のオブジェクトは[、]{.chpule2}[
]{.chpuri2}コンテキストと呼ばれ[、]{.chpule2}[
]{.chpuri2}一つのストラテジー・オブジェクトへの参照を保持し[、]{.chpule2}[
]{.chpuri2}それに振る舞いの実行を委任します[。]{.chpule2}[
]{.chpuri2}コンテキストがその作業を実行する方法を変えるために[、]{.chpule2}[
]{.chpuri2}他のオブジェクトが[、]{.chpule2}[
]{.chpuri2}現在リンクされているオブジェクトを違うものと置き換えるかもしれません[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Strategy の詳細 ](../../strategy.html){.btn .btn-lg .btn-primary}
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
 [eviction­Algo](#example-0--evictionAlgo-go)
:::

::: en-file
 [fifo](#example-0--fifo-go)
:::

::: en-file
 [lru](#example-0--lru-go)
:::

::: en-file
 [lfu](#example-0--lfu-go)
:::

::: en-file
 [cache](#example-0--cache-go)
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

## 概念的な例 {#example-0-title}

メモリー内キャッシュを構築するとしましょう[。]{.chpule2}[
]{.chpuri2}メモリーが相手なので[、]{.chpule2}[
]{.chpuri2}サイズは限られています[。]{.chpule2}[
]{.chpuri2}最大サイズに達したら[、]{.chpule2}[
]{.chpuri2}いくつかの項目を追い出して[、]{.chpule2}[
]{.chpuri2}空きスペースを確保する必要があります[。]{.chpule2}[
]{.chpuri2}これを行うアルゴリズムにはいろいろあります[。]{.chpule2}[
]{.chpuri2}人気のあるものをいくつか羅列すると[：]{.chpule2}[ ]{.chpuri2}

-   Least Recently Used (LRU)[：]{.chpule2}[
    ]{.chpuri2}使用後最も時間の経った項目を削除[。]{.chpule2}[
    ]{.chpuri2}
-   First In, First Out (FIFO)[：]{.chpule2}[
    ]{.chpuri2}最初に作られた項目を削除[。]{.chpule2}[ ]{.chpuri2}
-   Least Frequently Used (LFU)[：]{.chpule2}[
    ]{.chpuri2}利用頻度が最低の項目を削除[。]{.chpule2}[ ]{.chpuri2}

実行時にアルゴリズムを変更可能とするために[、]{.chpule2}[
]{.chpuri2}キャッシュ・クラスをいかにこれらのアルゴリズムから切り離すかが問題です[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}新しいアルゴリズムが追加されても[、]{.chpule2}[
]{.chpuri2}キャッシュ・クラスは変更不要であるべきです[。]{.chpule2}[
]{.chpuri2}

こういう時に[、]{.chpule2}[ ]{.chpuri2}Strategy
パターンが便利です[。]{.chpule2}[
]{.chpuri2}アルゴリズムのファミリーを作り[、]{.chpule2}[
]{.chpuri2}各アルゴリズムごとに独自のクラスがあるようにします[。]{.chpule2}[
]{.chpuri2}これらのクラスは[、]{.chpule2}[
]{.chpuri2}同一のインターフェースに従うようにして[、]{.chpule2}[
]{.chpuri2}ファミリー内でのアルゴリズムの交換を可能とします[。]{.chpule2}[
]{.chpuri2}とりあえず[、]{.chpule2}[
]{.chpuri2}共通のインターフェースの名前は[、]{.chpule2}[
]{.chpuri2}`eviction­Algo` としましょう[。]{.chpule2}[ ]{.chpuri2}

我々の主キャッシュ・クラスは[、]{.chpule2}[ ]{.chpuri2}`eviction­Algo`
インターフェースを埋め込んでいます[。]{.chpule2}[
]{.chpuri2}あらゆる種類の追い出しアルゴリズムを実装する代わりに[、]{.chpule2}[
]{.chpuri2}キャッシュ・クラスは[、]{.chpule2}[ ]{.chpuri2}それをすべて
`eviction­Algo` インターフェースに委任します[。]{.chpule2}[
]{.chpuri2}`eviction­Algo` はインターフェースなので[、]{.chpule2}[
]{.chpuri2}アルゴリズムを LRU[、]{.chpule2}[
]{.chpuri2}FIFO[、]{.chpule2}[ ]{.chpuri2}LFU
のどれにでも[、]{.chpule2}[
]{.chpuri2}キャッシュ・クラスの変更なしに[、]{.chpule2}[
]{.chpuri2}実行時変更できます[。]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--evictionAlgo-go .anchor} **evictionAlgo.go:** ストラテジー・インターフェース

<figure class="code">
<pre class="code" lang="go"><code>package main

type EvictionAlgo interface {
    evict(c *Cache)
}</code></pre>
</figure>

#### []{#example-0--fifo-go .anchor} **fifo.go:** 具象ストラテジー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Fifo struct {
}

func (l *Fifo) evict(c *Cache) {
    fmt.Println(&quot;Evicting by fifo strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lru-go .anchor} **lru.go:** 具象ストラテジー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lru struct {
}

func (l *Lru) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lru strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--lfu-go .anchor} **lfu.go:** 具象ストラテジー

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Lfu struct {
}

func (l *Lfu) evict(c *Cache) {
    fmt.Println(&quot;Evicting by lfu strtegy&quot;)
}</code></pre>
</figure>

#### []{#example-0--cache-go .anchor} **cache.go:** コンテキスト

<figure class="code">
<pre class="code" lang="go"><code>package main

type Cache struct {
    storage      map[string]string
    evictionAlgo EvictionAlgo
    capacity     int
    maxCapacity  int
}

func initCache(e EvictionAlgo) *Cache {
    storage := make(map[string]string)
    return &amp;Cache{
        storage:      storage,
        evictionAlgo: e,
        capacity:     0,
        maxCapacity:  2,
    }
}

func (c *Cache) setEvictionAlgo(e EvictionAlgo) {
    c.evictionAlgo = e
}

func (c *Cache) add(key, value string) {
    if c.capacity == c.maxCapacity {
        c.evict()
    }
    c.capacity++
    c.storage[key] = value
}

func (c *Cache) get(key string) {
    delete(c.storage, key)
}

func (c *Cache) evict() {
    c.evictionAlgo.evict(c)
    c.capacity--
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** クライアント・コード

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    lfu := &amp;Lfu{}
    cache := initCache(lfu)

    cache.add(&quot;a&quot;, &quot;1&quot;)
    cache.add(&quot;b&quot;, &quot;2&quot;)

    cache.add(&quot;c&quot;, &quot;3&quot;)

    lru := &amp;Lru{}
    cache.setEvictionAlgo(lru)

    cache.add(&quot;d&quot;, &quot;4&quot;)

    fifo := &amp;Fifo{}
    cache.setEvictionAlgo(fifo)

    cache.add(&quot;e&quot;, &quot;5&quot;)

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Evicting by lfu strtegy
Evicting by lru strtegy
Evicting by fifo strtegy</code></pre>
</figure>

::: next
#### 次を読む

[Template Method を Go で []{.fa
.fa-arrow-right}](../../template-method/go/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} State を Go
で](../../state/go/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Strategy**

[![Strategy を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Strategy を C# で"){.prog-lang-link}
[![Strategy を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Strategy を C++ で"){.prog-lang-link}
[![Strategy を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Strategy を Java で"){.prog-lang-link}
[![Strategy を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Strategy を PHP で"){.prog-lang-link}
[![Strategy を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Strategy を Python で"){.prog-lang-link}
[![Strategy を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Strategy を Ruby で"){.prog-lang-link}
[![Strategy を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Strategy を Rust で"){.prog-lang-link}
[![Strategy を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Strategy を Swift で"){.prog-lang-link}
[![Strategy を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Strategy を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
