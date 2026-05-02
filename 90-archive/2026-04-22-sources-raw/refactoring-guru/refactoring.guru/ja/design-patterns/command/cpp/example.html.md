:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** を C++ で {#command-を-c-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Command パターンは[、]{.chpule2}[
]{.chpuri2}C++ コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}最もよく使われるのは[、]{.chpule2}[ ]{.chpuri2}UI
要素をアクションでパラメーター化する時のコールバックの代わりとしてです[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}タスクをキューに入れたり[、]{.chpule2}[
]{.chpuri2}操作履歴の管理などでも使われます[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Command
パターンは[、]{.chpule2}[ ]{.chpuri2}抽象またはインターフェース型[
]{.chpule1}[（]{.chpuri1}送り手[）]{.chpule2}[
]{.chpuri2}中の振る舞い系メソッド[
]{.chpule1}[（]{.chpuri1}複数[）]{.chpule2}[
]{.chpuri2}が違う抽象またはインターフェース型[
]{.chpule1}[（]{.chpuri1}受け手[）]{.chpule2}[
]{.chpuri2}中のある一つのメソッドを起動することから識別できます[。]{.chpule2}[
]{.chpuri2}受け手は[、]{.chpule2}[
]{.chpuri2}生成時にコマンドの実装によりカプセル化されています[。]{.chpule2}[
]{.chpuri2}コマンドのクラスは通常特定のアクションに限定されています[。]{.chpule2}[
]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Command**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-cc .anchor} **main.cc:** 概念的な例

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Command interface declares a method for executing a command.
 */
class Command {
 public:
  virtual ~Command() {
  }
  virtual void Execute() const = 0;
};
/**
 * Some commands can implement simple operations on their own.
 */
class SimpleCommand : public Command {
 private:
  std::string pay_load_;

 public:
  explicit SimpleCommand(std::string pay_load) : pay_load_(pay_load) {
  }
  void Execute() const override {
    std::cout &lt;&lt; &quot;SimpleCommand: See, I can do simple things like printing (&quot; &lt;&lt; this-&gt;pay_load_ &lt;&lt; &quot;)\n&quot;;
  }
};

/**
 * The Receiver classes contain some important business logic. They know how to
 * perform all kinds of operations, associated with carrying out a request. In
 * fact, any class may serve as a Receiver.
 */
class Receiver {
 public:
  void DoSomething(const std::string &amp;a) {
    std::cout &lt;&lt; &quot;Receiver: Working on (&quot; &lt;&lt; a &lt;&lt; &quot;.)\n&quot;;
  }
  void DoSomethingElse(const std::string &amp;b) {
    std::cout &lt;&lt; &quot;Receiver: Also working on (&quot; &lt;&lt; b &lt;&lt; &quot;.)\n&quot;;
  }
};

/**
 * However, some commands can delegate more complex operations to other objects,
 * called &quot;receivers.&quot;
 */
class ComplexCommand : public Command {
  /**
   * @var Receiver
   */
 private:
  Receiver *receiver_;
  /**
   * Context data, required for launching the receiver&#39;s methods.
   */
  std::string a_;
  std::string b_;
  /**
   * Complex commands can accept one or several receiver objects along with any
   * context data via the constructor.
   */
 public:
  ComplexCommand(Receiver *receiver, std::string a, std::string b) : receiver_(receiver), a_(a), b_(b) {
  }
  /**
   * Commands can delegate to any methods of a receiver.
   */
  void Execute() const override {
    std::cout &lt;&lt; &quot;ComplexCommand: Complex stuff should be done by a receiver object.\n&quot;;
    this-&gt;receiver_-&gt;DoSomething(this-&gt;a_);
    this-&gt;receiver_-&gt;DoSomethingElse(this-&gt;b_);
  }
};

/**
 * The Invoker is associated with one or several commands. It sends a request to
 * the command.
 */
class Invoker {
  /**
   * @var Command
   */
 private:
  Command *on_start_;
  /**
   * @var Command
   */
  Command *on_finish_;
  /**
   * Initialize commands.
   */
 public:
  ~Invoker() {
    delete on_start_;
    delete on_finish_;
  }

  void SetOnStart(Command *command) {
    this-&gt;on_start_ = command;
  }
  void SetOnFinish(Command *command) {
    this-&gt;on_finish_ = command;
  }
  /**
   * The Invoker does not depend on concrete command or receiver classes. The
   * Invoker passes a request to a receiver indirectly, by executing a command.
   */
  void DoSomethingImportant() {
    std::cout &lt;&lt; &quot;Invoker: Does anybody want something done before I begin?\n&quot;;
    if (this-&gt;on_start_) {
      this-&gt;on_start_-&gt;Execute();
    }
    std::cout &lt;&lt; &quot;Invoker: ...doing something really important...\n&quot;;
    std::cout &lt;&lt; &quot;Invoker: Does anybody want something done after I finish?\n&quot;;
    if (this-&gt;on_finish_) {
      this-&gt;on_finish_-&gt;Execute();
    }
  }
};
/**
 * The client code can parameterize an invoker with any commands.
 */

int main() {
  Invoker *invoker = new Invoker;
  invoker-&gt;SetOnStart(new SimpleCommand(&quot;Say Hi!&quot;));
  Receiver *receiver = new Receiver;
  invoker-&gt;SetOnFinish(new ComplexCommand(receiver, &quot;Send email&quot;, &quot;Save report&quot;));
  invoker-&gt;DoSomethingImportant();

  delete invoker;
  delete receiver;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object.
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

::: next
#### 次を読む

[Iterator を C++ で []{.fa
.fa-arrow-right}](../../iterator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Chain of Responsibility を C++
で](../../chain-of-responsibility/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## 他言語での **Command**

[![Command を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command を C# で"){.prog-lang-link}
[![Command を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command を Go で"){.prog-lang-link}
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
