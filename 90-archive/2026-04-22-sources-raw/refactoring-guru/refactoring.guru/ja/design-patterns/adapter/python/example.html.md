:::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** を Python で {#adapter-を-python-で .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}非互換なオブジェクト同士の協働を可能とします[。]{.chpule2}[
]{.chpuri2}

アダプターは二つのオブジェクト間のラッパーとして機能します[。]{.chpule2}[
]{.chpuri2}一方のオブジェクトへの呼び出しを捕まえ[、]{.chpule2}[
]{.chpuri2}二つ目のオブジェクトが認識可能なデータ形式とインターフェースに変換します[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Adapter の詳細 ](../../adapter.html){.btn .btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [概念的な例 (継承による)](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [概念的な例（オブジェクト合成による）](#example-1)
:::

::: en-file
 [main](#example-1--main-py)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Adapter パターンは Python
コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}旧来のコードに基づいたシステムでよく使用されます[。]{.chpule2}[
]{.chpuri2}そのような場合[、]{.chpule2}[
]{.chpuri2}アダプターは旧来のコードを現代のクラスから使用可能にします[。]{.chpule2}[
]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**アダプターは[、]{.chpule2}[
]{.chpuri2}異なる抽象クラスまたはインターフェース型のインスタンスを取るコンストラクターの存在により認識できます[。]{.chpule2}[
]{.chpuri2}アダプターは[、]{.chpule2}[
]{.chpuri2}いずれかのメソッドへの呼び出しを受け取ると[、]{.chpule2}[
]{.chpuri2}パラメーターを適切な形式に変換し[、]{.chpule2}[
]{.chpuri2}ラップされたオブジェクトの一つまたは複数のメソッドを呼び出します[。]{.chpule2}[
]{.chpuri2}
::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[概念的な例 (継承による)](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"}
[概念的な例（オブジェクト合成による）](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## 概念的な例 (継承による) {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Adapter**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-py .anchor} **main.py:** 概念的な例

<figure class="code">
<pre class="code" lang="python"><code>class Target:
    &quot;&quot;&quot;
    The Target defines the domain-specific interface used by the client code.
    &quot;&quot;&quot;

    def request(self) -&gt; str:
        return &quot;Target: The default target&#39;s behavior.&quot;


class Adaptee:
    &quot;&quot;&quot;
    The Adaptee contains some useful behavior, but its interface is incompatible
    with the existing client code. The Adaptee needs some adaptation before the
    client code can use it.
    &quot;&quot;&quot;

    def specific_request(self) -&gt; str:
        return &quot;.eetpadA eht fo roivaheb laicepS&quot;


class Adapter(Target, Adaptee):
    &quot;&quot;&quot;
    The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
    interface via multiple inheritance.
    &quot;&quot;&quot;

    def request(self) -&gt; str:
        return f&quot;Adapter: (TRANSLATED) {self.specific_request()[::-1]}&quot;


def client_code(target: &quot;Target&quot;) -&gt; None:
    &quot;&quot;&quot;
    The client code supports all classes that follow the Target interface.
    &quot;&quot;&quot;

    print(target.request(), end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    print(&quot;Client: I can work just fine with the Target objects:&quot;)
    target = Target()
    client_code(target)
    print(&quot;\n&quot;)

    adaptee = Adaptee()
    print(&quot;Client: The Adaptee class has a weird interface. &quot;
          &quot;See, I don&#39;t understand it:&quot;)
    print(f&quot;Adaptee: {adaptee.specific_request()}&quot;, end=&quot;\n\n&quot;)

    print(&quot;Client: But I can work with it via the Adapter:&quot;)
    adapter = Adapter()
    client_code(adapter)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## 概念的な例（オブジェクト合成による） {#example-1-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Adapter**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-1--main-py .anchor} **main.py:** 概念的な例

<figure class="code">
<pre class="code" lang="python"><code>class Target:
    &quot;&quot;&quot;
    The Target defines the domain-specific interface used by the client code.
    &quot;&quot;&quot;

    def request(self) -&gt; str:
        return &quot;Target: The default target&#39;s behavior.&quot;


class Adaptee:
    &quot;&quot;&quot;
    The Adaptee contains some useful behavior, but its interface is incompatible
    with the existing client code. The Adaptee needs some adaptation before the
    client code can use it.
    &quot;&quot;&quot;

    def specific_request(self) -&gt; str:
        return &quot;.eetpadA eht fo roivaheb laicepS&quot;


class Adapter(Target):
    &quot;&quot;&quot;
    The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
    interface via composition.
    &quot;&quot;&quot;

    def __init__(self, adaptee: Adaptee) -&gt; None:
        self.adaptee = adaptee

    def request(self) -&gt; str:
        return f&quot;Adapter: (TRANSLATED) {self.adaptee.specific_request()[::-1]}&quot;


def client_code(target: Target) -&gt; None:
    &quot;&quot;&quot;
    The client code supports all classes that follow the Target interface.
    &quot;&quot;&quot;

    print(target.request(), end=&quot;&quot;)


if __name__ == &quot;__main__&quot;:
    print(&quot;Client: I can work just fine with the Target objects:&quot;)
    target = Target()
    client_code(target)
    print(&quot;\n&quot;)

    adaptee = Adaptee()
    print(&quot;Client: The Adaptee class has a weird interface. &quot;
          &quot;See, I don&#39;t understand it:&quot;)
    print(f&quot;Adaptee: {adaptee.specific_request()}&quot;, end=&quot;\n\n&quot;)

    print(&quot;Client: But I can work with it via the Adapter:&quot;)
    adapter = Adapter(adaptee)
    client_code(adapter)</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.

Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS

Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[概念的な例 (継承による)](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"}
[概念的な例（オブジェクト合成による）](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 次を読む

[Bridge を Python で []{.fa
.fa-arrow-right}](../../bridge/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Singleton を Python
で](../../singleton/python/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Adapter**

[![Adapter を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter を C# で"){.prog-lang-link}
[![Adapter を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter を C++ で"){.prog-lang-link}
[![Adapter を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adapter を Go で"){.prog-lang-link}
[![Adapter を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adapter を Java で"){.prog-lang-link}
[![Adapter を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adapter を PHP で"){.prog-lang-link}
[![Adapter を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter を Ruby で"){.prog-lang-link}
[![Adapter を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter を Rust で"){.prog-lang-link}
[![Adapter を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Adapter を Swift で"){.prog-lang-link}
[![Adapter を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
