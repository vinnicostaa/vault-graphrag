:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[커맨드](../../command.html){.type} / [루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![커맨드](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# 루비로 작성된 **커맨드** {#루비로-작성된-커맨드 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**커맨드**는 요청 또는 간단한 작업을 객체로 변환하는 행동 디자인
패턴입니다.

이러한 변환은 명령의 지연 또는 원격 실행, 명령 기록 저장 등을
허용합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 커맨드에 대하여 더 자세히 알아보세요 ](../../command.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [개념적인 예시](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 커맨드 패턴은 루비 코드에서 매우 일반적입니다. 대부분의
경우 작업으로 UI 요소를 매개 변수화하기 위한 콜백의 대안으로 사용되며
작업 대기, 작업 기록 추적 등에도 사용됩니다.

**식별:** 커맨드 패턴은 다음과 같은 특징이 있습니다. 추상/인터페이스
유형​(발신자)​의 행동 메서드들이 있으며 이러한 메서드들은 다른
추상/인터페이스 유형​(수신자)​의 구현에서 메서드를 호출하며 이 메서드는
생성되는 동안 커맨드 구현으로 캡슐화되었습니다. 또 커맨드 클래스는
일반적으로 특정 작업만 수행할 수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **커맨드** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

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

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

## 다른 언어로 작성된 **커맨드**

[![C#으로 작성된
커맨드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 커맨드"){.prog-lang-link}
[![C++로 작성된
커맨드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 커맨드"){.prog-lang-link}
[![Go로 작성된
커맨드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 커맨드"){.prog-lang-link}
[![자바로 작성된
커맨드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 커맨드"){.prog-lang-link}
[![PHP로 작성된
커맨드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 커맨드"){.prog-lang-link}
[![파이썬으로 작성된
커맨드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 커맨드"){.prog-lang-link}
[![러스트로 작성된
커맨드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 커맨드"){.prog-lang-link}
[![스위프트로 작성된
커맨드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 커맨드"){.prog-lang-link}
[![타입스크립트로 작성된
커맨드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 커맨드"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
