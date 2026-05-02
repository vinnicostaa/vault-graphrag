:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** を Ruby で {#mediator-を-ruby-で .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** は[、]{.chpule2}[
]{.chpuri2}振る舞いに関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}プログラムのコンポーネント間の通信を特別なメディエーター・オブジェクトを通して行うことで[、]{.chpule2}[
]{.chpuri2}結合を疎にします[。]{.chpule2}[ ]{.chpuri2}

Mediator により[、]{.chpule2}[
]{.chpuri2}個々のコンポーネントは[、]{.chpule2}[
]{.chpuri2}何十ものクラスへの依存がなくなるため[、]{.chpule2}[
]{.chpuri2}変更[、]{.chpule2}[ ]{.chpuri2}拡張[、]{.chpule2}[
]{.chpuri2}再利用が容易になります[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Mediator の詳細 ](../../mediator.html){.btn .btn-lg .btn-primary}
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

**使用例[：]{.chpule2}[ ]{.chpuri2}** Ruby コードで Mediator
パターンがよく使われるのは[、]{.chpule2}[ ]{.chpuri2}アプリの GUI
コンポーネント間の通信のやりとりです[。]{.chpule2}[ ]{.chpuri2}Mediator
は[、]{.chpule2}[ ]{.chpuri2}MVC パターンの C の部分[、]{.chpule2}[
]{.chpuri2}Controller と同義語です[。]{.chpule2}[ ]{.chpuri2}
:::::::::::

[]{#example-0}

## 概念的な例 {#example-0-title}

この例は[、]{.chpule2}[ ]{.chpuri2}**Mediator**
デザインパターンの構造を説明するためのものです[。]{.chpule2}[
]{.chpuri2}以下の質問に答えることを目的としています[：]{.chpule2}[
]{.chpuri2}

-   どういうクラスからできているか[？]{.chpule2}[ ]{.chpuri2}
-   それぞれのクラスの役割は[？]{.chpule2}[ ]{.chpuri2}
-   パターンの要素同士はどう関係しているのか[？]{.chpule2}[ ]{.chpuri2}

#### []{#example-0--main-rb .anchor} **main.rb:** 概念的な例

<figure class="code">
<pre class="code" lang="ruby"><code># The Mediator interface declares a method used by components to notify the
# mediator about various events. The Mediator may react to these events and pass
# the execution to other components.
class Mediator
  # @abstract
  #
  # @param [Object] sender
  # @param [String] event
  def notify(_sender, _event)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

class ConcreteMediator &lt; Mediator
  # @param [Component1] component1
  # @param [Component2] component2
  def initialize(component1, component2)
    @component1 = component1
    @component1.mediator = self
    @component2 = component2
    @component2.mediator = self
  end

  # @param [Object] sender
  # @param [String] event
  def notify(_sender, event)
    if event == &#39;A&#39;
      puts &#39;Mediator reacts on A and triggers following operations:&#39;
      @component2.do_c
    elsif event == &#39;D&#39;
      puts &#39;Mediator reacts on D and triggers following operations:&#39;
      @component1.do_b
      @component2.do_c
    end
  end
end

# The Base Component provides the basic functionality of storing a mediator&#39;s
# instance inside component objects.
class BaseComponent
  # @return [Mediator]
  attr_accessor :mediator

  # @param [Mediator] mediator
  def initialize(mediator = nil)
    @mediator = mediator
  end
end

# Concrete Components implement various functionality. They don&#39;t depend on
# other components. They also don&#39;t depend on any concrete mediator classes.
class Component1 &lt; BaseComponent
  def do_a
    puts &#39;Component 1 does A.&#39;
    @mediator.notify(self, &#39;A&#39;)
  end

  def do_b
    puts &#39;Component 1 does B.&#39;
    @mediator.notify(self, &#39;B&#39;)
  end
end

class Component2 &lt; BaseComponent
  def do_c
    puts &#39;Component 2 does C.&#39;
    @mediator.notify(self, &#39;C&#39;)
  end

  def do_d
    puts &#39;Component 2 does D.&#39;
    @mediator.notify(self, &#39;D&#39;)
  end
end

# The client code.
c1 = Component1.new
c2 = Component2.new
ConcreteMediator.new(c1, c2)

puts &#39;Client triggers operation A.&#39;
c1.do_a

puts &quot;\n&quot;

puts &#39;Client triggers operation D.&#39;
c2.do_d</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 実行結果

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

## 他言語での **Mediator**

[![Mediator を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator を C# で"){.prog-lang-link}
[![Mediator を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator を C++ で"){.prog-lang-link}
[![Mediator を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator を Go で"){.prog-lang-link}
[![Mediator を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator を Java で"){.prog-lang-link}
[![Mediator を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator を PHP で"){.prog-lang-link}
[![Mediator を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator を Python で"){.prog-lang-link}
[![Mediator を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator を Rust で"){.prog-lang-link}
[![Mediator を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator を Swift で"){.prog-lang-link}
[![Mediator を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator を TypeScript で"){.prog-lang-link}
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
