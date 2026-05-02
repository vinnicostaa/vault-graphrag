:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** を C++ で {#chain-of-responsibility-を-c-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}潜在的なハンドラーの連鎖の上を[、]{.chpule2}[
]{.chpuri2}ハンドラーのどれかが処理するまで[、]{.chpule2}[
]{.chpuri2}リクエストを回していきます[。]{.chpule2}[ ]{.chpuri2}

このパターンを利用すると[、]{.chpule2}[
]{.chpuri2}送り手のクラスと受け手の具象クラスとを結合することなく[、]{.chpule2}[
]{.chpuri2}複数のオブジェクトにリクエストを処理する機会を与えることができます[。]{.chpule2}[
]{.chpuri2}連鎖は実行時に[、]{.chpule2}[
]{.chpuri2}標準のハンドラー・インターフェースに従うハンドラーから動的に構成されます[。]{.chpule2}[
]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Chain of Responsibility の詳細
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Chain of Responsibility
パターンは[、]{.chpule2}[ ]{.chpuri2}C++
ではよく見かけます[。]{.chpule2}[
]{.chpuri2}フィルターやイベント・チェーンのようなオブジェクトの連鎖を対象に動作するコードを書く時に[、]{.chpule2}[
]{.chpuri2}最も役に立ちます[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[
]{.chpuri2}**共通のインターフェースに従うオブジェクトのグループで[、]{.chpule2}[
]{.chpuri2}実作業を行うメソッドが[、]{.chpule2}[
]{.chpuri2}別のオブジェクトの同一メソッドを呼ぶことから[、]{.chpule2}[
]{.chpuri2}このパターンを識別できます[。]{.chpule2}[ ]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Chain of Responsibility**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-cc .anchor} **main.cc:** 概念的な例

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Handler interface declares a method for building the chain of handlers.
 * It also declares a method for executing a request.
 */
class Handler {
 public:
  virtual Handler *SetNext(Handler *handler) = 0;
  virtual std::string Handle(std::string request) = 0;
};
/**
 * The default chaining behavior can be implemented inside a base handler class.
 */
class AbstractHandler : public Handler {
  /**
   * @var Handler
   */
 private:
  Handler *next_handler_;

 public:
  AbstractHandler() : next_handler_(nullptr) {
  }
  Handler *SetNext(Handler *handler) override {
    this-&gt;next_handler_ = handler;
    // Returning a handler from here will let us link handlers in a convenient
    // way like this:
    // $monkey-&gt;setNext($squirrel)-&gt;setNext($dog);
    return handler;
  }
  std::string Handle(std::string request) override {
    if (this-&gt;next_handler_) {
      return this-&gt;next_handler_-&gt;Handle(request);
    }

    return {};
  }
};
/**
 * All Concrete Handlers either handle a request or pass it to the next handler
 * in the chain.
 */
class MonkeyHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;Banana&quot;) {
      return &quot;Monkey: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
class SquirrelHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;Nut&quot;) {
      return &quot;Squirrel: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
class DogHandler : public AbstractHandler {
 public:
  std::string Handle(std::string request) override {
    if (request == &quot;MeatBall&quot;) {
      return &quot;Dog: I&#39;ll eat the &quot; + request + &quot;.\n&quot;;
    } else {
      return AbstractHandler::Handle(request);
    }
  }
};
/**
 * The client code is usually suited to work with a single handler. In most
 * cases, it is not even aware that the handler is part of a chain.
 */
void ClientCode(Handler &amp;handler) {
  std::vector&lt;std::string&gt; food = {&quot;Nut&quot;, &quot;Banana&quot;, &quot;Cup of coffee&quot;};
  for (const std::string &amp;f : food) {
    std::cout &lt;&lt; &quot;Client: Who wants a &quot; &lt;&lt; f &lt;&lt; &quot;?\n&quot;;
    const std::string result = handler.Handle(f);
    if (!result.empty()) {
      std::cout &lt;&lt; &quot;  &quot; &lt;&lt; result;
    } else {
      std::cout &lt;&lt; &quot;  &quot; &lt;&lt; f &lt;&lt; &quot; was left untouched.\n&quot;;
    }
  }
}
/**
 * The other part of the client code constructs the actual chain.
 */
int main() {
  MonkeyHandler *monkey = new MonkeyHandler;
  SquirrelHandler *squirrel = new SquirrelHandler;
  DogHandler *dog = new DogHandler;
  monkey-&gt;SetNext(squirrel)-&gt;SetNext(dog);

  /**
   * The client should be able to send a request to any handler, not just the
   * first one in the chain.
   */
  std::cout &lt;&lt; &quot;Chain: Monkey &gt; Squirrel &gt; Dog\n\n&quot;;
  ClientCode(*monkey);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;Subchain: Squirrel &gt; Dog\n\n&quot;;
  ClientCode(*squirrel);

  delete monkey;
  delete squirrel;
  delete dog;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Chain: Monkey &gt; Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Monkey: I&#39;ll eat the Banana.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.

Subchain: Squirrel &gt; Dog

Client: Who wants a Nut?
  Squirrel: I&#39;ll eat the Nut.
Client: Who wants a Banana?
  Banana was left untouched.
Client: Who wants a Cup of coffee?
  Cup of coffee was left untouched.</code></pre>
</figure>

::: next
#### 次を読む

[Command を C++ で []{.fa
.fa-arrow-right}](../../command/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Proxy を C++
で](../../proxy/cpp/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Chain of Responsibility**

[![Chain of Responsibility を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility を C# で"){.prog-lang-link}
[![Chain of Responsibility を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chain of Responsibility を Go で"){.prog-lang-link}
[![Chain of Responsibility を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility を Java で"){.prog-lang-link}
[![Chain of Responsibility を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility を PHP で"){.prog-lang-link}
[![Chain of Responsibility を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility を Python で"){.prog-lang-link}
[![Chain of Responsibility を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility を Ruby で"){.prog-lang-link}
[![Chain of Responsibility を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility を Rust で"){.prog-lang-link}
[![Chain of Responsibility を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility を Swift で"){.prog-lang-link}
[![Chain of Responsibility を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility を TypeScript で"){.prog-lang-link}
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
