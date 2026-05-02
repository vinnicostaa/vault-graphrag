:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** を Python で {#proxy-を-python-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Proxy** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}クライアントが使う本物のサービス・オブジェクトの代理として機能するオブジェクト[
]{.chpule1}[（]{.chpuri1}プロキシー[）]{.chpule2}[
]{.chpuri2}を提供します[。]{.chpule2}[
]{.chpuri2}プロキシーは[、]{.chpule2}[
]{.chpuri2}アクセス制御[、]{.chpule2}[
]{.chpuri2}キャッシングなど[、]{.chpule2}[
]{.chpuri2}何らかの作業を行なった後[、]{.chpule2}[
]{.chpuri2}リクエストをサービス・オブジェクトに渡します[。]{.chpule2}[
]{.chpuri2}

プロキシー・オブジェクトはサービスと同じインターフェースを持ち[、]{.chpule2}[
]{.chpuri2}クライアントにとっては[、]{.chpule2}[
]{.chpuri2}本物のオブジェクトと交換可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Proxy の詳細 ](../../proxy.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Proxy パターンは[、]{.chpule2}[
]{.chpuri2}ほとんどの Python アプリケーションにおいては[、]{.chpule2}[
]{.chpuri2}あまり見かけませんが[、]{.chpule2}[
]{.chpuri2}いくつかの特殊なケースでは便利です[。]{.chpule2}[
]{.chpuri2}何らかの既存クラスのオブジェクトに何らかの振る舞いを追加したいがクライアント・コードには手を加えたくない時[、]{.chpule2}[
]{.chpuri2}かけがえのないものです[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}**プロキシーは[、]{.chpule2}[
]{.chpuri2}実際の作業はすべて他のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}プロキシーがサービスのサブクラスである場合を除き[、]{.chpule2}[
]{.chpuri2}プロキシーのメソッドのそれぞれは[、]{.chpule2}[
]{.chpuri2}最終的にはサービス・オブジェクトを参照するはずです[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Proxy**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-py .anchor} **main.py:** 概念的な例

<figure class="code">
<pre class="code" lang="python"><code>from abc import ABC, abstractmethod


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares common operations for both RealSubject and
    the Proxy. As long as the client works with RealSubject using this
    interface, you&#39;ll be able to pass it a proxy instead of a real subject.
    &quot;&quot;&quot;

    @abstractmethod
    def request(self) -&gt; None:
        pass


class RealSubject(Subject):
    &quot;&quot;&quot;
    The RealSubject contains some core business logic. Usually, RealSubjects are
    capable of doing some useful work which may also be very slow or sensitive -
    e.g. correcting input data. A Proxy can solve these issues without any
    changes to the RealSubject&#39;s code.
    &quot;&quot;&quot;

    def request(self) -&gt; None:
        print(&quot;RealSubject: Handling request.&quot;)


class Proxy(Subject):
    &quot;&quot;&quot;
    The Proxy has an interface identical to the RealSubject.
    &quot;&quot;&quot;

    def __init__(self, real_subject: RealSubject) -&gt; None:
        self._real_subject = real_subject

    def request(self) -&gt; None:
        &quot;&quot;&quot;
        The most common applications of the Proxy pattern are lazy loading,
        caching, controlling the access, logging, etc. A Proxy can perform one
        of these things and then, depending on the result, pass the execution to
        the same method in a linked RealSubject object.
        &quot;&quot;&quot;

        if self.check_access():
            self._real_subject.request()
            self.log_access()

    def check_access(self) -&gt; bool:
        print(&quot;Proxy: Checking access prior to firing a real request.&quot;)
        return True

    def log_access(self) -&gt; None:
        print(&quot;Proxy: Logging the time of request.&quot;, end=&quot;&quot;)


def client_code(subject: Subject) -&gt; None:
    &quot;&quot;&quot;
    The client code is supposed to work with all objects (both subjects and
    proxies) via the Subject interface in order to support both real subjects
    and proxies. In real life, however, clients mostly work with their real
    subjects directly. In this case, to implement the pattern more easily, you
    can extend your proxy from the real subject&#39;s class.
    &quot;&quot;&quot;

    # ...

    subject.request()

    # ...


if __name__ == &quot;__main__&quot;:
    print(&quot;Client: Executing the client code with a real subject:&quot;)
    real_subject = RealSubject()
    client_code(real_subject)

    print(&quot;&quot;)

    print(&quot;Client: Executing the same client code with a proxy:&quot;)
    proxy = Proxy(real_subject)
    client_code(proxy)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>

::: next
#### 次を読む

[Chain of Responsibility を Python で []{.fa
.fa-arrow-right}](../../chain-of-responsibility/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Flyweight を Python
で](../../flyweight/python/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Proxy**

[![Proxy を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy を C# で"){.prog-lang-link}
[![Proxy を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy を C++ で"){.prog-lang-link}
[![Proxy を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy を Go で"){.prog-lang-link}
[![Proxy を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy を Java で"){.prog-lang-link}
[![Proxy を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy を PHP で"){.prog-lang-link}
[![Proxy を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy を Ruby で"){.prog-lang-link}
[![Proxy を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy を Rust で"){.prog-lang-link}
[![Proxy を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy を Swift で"){.prog-lang-link}
[![Proxy を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy を TypeScript で"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
