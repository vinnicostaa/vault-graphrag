:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[커맨드](../../command.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![커맨드](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# C++로 작성된 **커맨드** {#c로-작성된-커맨드 .pattern-example-title-block-title}
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 커맨드 패턴은 C++ 코드에서 매우 일반적입니다. 대부분의
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

#### []{#example-0--main-cc .anchor} **main.cc:** 개념적인 예시

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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
#### 다음을 보세요

[C++로 작성된 반복자 []{.fa
.fa-arrow-right}](../../iterator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C++로 작성된 책임
연쇄](../../chain-of-responsibility/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **커맨드**

[![C#으로 작성된
커맨드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 커맨드"){.prog-lang-link}
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
[![루비로 작성된
커맨드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 커맨드"){.prog-lang-link}
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
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
