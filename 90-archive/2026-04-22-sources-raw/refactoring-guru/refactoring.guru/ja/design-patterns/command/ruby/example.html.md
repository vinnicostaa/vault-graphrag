:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** を Ruby で {#command-を-ruby-で .pattern-example-title-block-title}
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Command パターンは[、]{.chpule2}[
]{.chpuri2}Ruby コードではよく見かけます[。]{.chpule2}[
]{.chpuri2}最もよく使われるのは[、]{.chpule2}[ ]{.chpuri2}UI
要素をアクションでパラメーター化する時のコールバックの代わりとしてです[。]{.chpule2}[
]{.chpuri2}また[、]{.chpule2}[
]{.chpuri2}タスクをキューに入れたり[、]{.chpule2}[
]{.chpuri2}操作履歴の管理などでも使われます[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Command
パターンは[、]{.chpule2}[ ]{.chpuri2}抽象またはインターフェース型[
]{.chpule1}[（]{.chpuri1}送り手[）]{.chpule2}[
]{.chpuri2}中の行動的メソッド[
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

#### []{#example-0--main-rb .anchor} **main.rb:** 概念的な例

<figure class="code">
<pre class="code" lang="ruby"><code># The Command interface declares a method for executing a command.
class Command
  # @abstract
  def execute
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Some commands can implement simple operations on their own.
class SimpleCommand &lt; Command
  # @param [String] payload
  def initialize(payload)
    @payload = payload
  end

  def execute
    puts &quot;SimpleCommand: See, I can do simple things like printing (#{@payload})&quot;
  end
end

# However, some commands can delegate more complex operations to other objects,
# called &quot;receivers&quot;.
class ComplexCommand &lt; Command
  # Complex commands can accept one or several receiver objects along with any
  # context data via the constructor.
  def initialize(receiver, a, b)
    @receiver = receiver
    @a = a
    @b = b
  end

  # Commands can delegate to any methods of a receiver.
  def execute
    print &#39;ComplexCommand: Complex stuff should be done by a receiver object&#39;
    @receiver.do_something(@a)
    @receiver.do_something_else(@b)
  end
end

# The Receiver classes contain some important business logic. They know how to
# perform all kinds of operations, associated with carrying out a request. In
# fact, any class may serve as a Receiver.
class Receiver
  # @param [String] a
  def do_something(a)
    print &quot;\nReceiver: Working on (#{a}.)&quot;
  end

  # @param [String] b
  def do_something_else(b)
    print &quot;\nReceiver: Also working on (#{b}.)&quot;
  end
end

# The Invoker is associated with one or several commands. It sends a request to
# the command.
class Invoker
  # Initialize commands.

  # @param [Command] command
  def on_start=(command)
    @on_start = command
  end

  # @param [Command] command
  def on_finish=(command)
    @on_finish = command
  end

  # The Invoker does not depend on concrete command or receiver classes. The
  # Invoker passes a request to a receiver indirectly, by executing a command.
  def do_something_important
    puts &#39;Invoker: Does anybody want something done before I begin?&#39;
    @on_start.execute if @on_start.is_a? Command

    puts &#39;Invoker: ...doing something really important...&#39;

    puts &#39;Invoker: Does anybody want something done after I finish?&#39;
    @on_finish.execute if @on_finish.is_a? Command
  end
end

# The client code can parameterize an invoker with any commands.
invoker = Invoker.new
invoker.on_start = SimpleCommand.new(&#39;Say Hi!&#39;)
receiver = Receiver.new
invoker.on_finish = ComplexCommand.new(receiver, &#39;Send email&#39;, &#39;Save report&#39;)

invoker.do_something_important</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

## 他言語での **Command**

[![Command を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command を C# で"){.prog-lang-link}
[![Command を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command を C++ で"){.prog-lang-link}
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
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
