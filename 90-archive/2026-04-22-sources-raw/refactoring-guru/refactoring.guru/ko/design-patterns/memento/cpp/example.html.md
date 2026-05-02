:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[메멘토](../../memento.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![메멘토](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# C++로 작성된 **메멘토** {#c로-작성된-메멘토 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**메멘토** 패턴은 행동 디자인 패턴입니다. 이 패턴은 객체 상태의 스냅숏을
만든 후 나중에 복원할 수 있도록 합니다.

메멘토는 함께 작동하는 객체의 내부 구조와 스냅숏들 내부에 보관된
데이터를 손상하지 않습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 메멘토에 대하여 더 자세히 알아보세요 ](../../memento.html){.btn
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

**사용 사례들:** 메멘토의 원칙은 직렬화를 사용하여 달성할 수 있으며,
이는 C++ 코드에서 매우 일반적입니다. 직렬화는 객체 상태의 스냅숏을
만드는 유일한 또는 가장 효율적인 방법은 아니나 다른 객체로부터
오리지네이터의 구조를 보호하면서 상태 백업을 저장할 수 있도록 합니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **메멘토** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-cc .anchor} **main.cc:** 개념적인 예시

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Memento interface provides a way to retrieve the memento&#39;s metadata, such
 * as creation date or name. However, it doesn&#39;t expose the Originator&#39;s state.
 */
class Memento {
 public:
  virtual ~Memento() {}
  virtual std::string GetName() const = 0;
  virtual std::string date() const = 0;
  virtual std::string state() const = 0;
};

/**
 * The Concrete Memento contains the infrastructure for storing the Originator&#39;s
 * state.
 */
class ConcreteMemento : public Memento {
 private:
  std::string state_;
  std::string date_;

 public:
  ConcreteMemento(std::string state) : state_(state) {
    this-&gt;state_ = state;
    std::time_t now = std::time(0);
    this-&gt;date_ = std::ctime(&amp;now);
  }
  /**
   * The Originator uses this method when restoring its state.
   */
  std::string state() const override {
    return this-&gt;state_;
  }
  /**
   * The rest of the methods are used by the Caretaker to display metadata.
   */
  std::string GetName() const override {
    return this-&gt;date_ + &quot; / (&quot; + this-&gt;state_.substr(0, 9) + &quot;...)&quot;;
  }
  std::string date() const override {
    return this-&gt;date_;
  }
};

/**
 * The Originator holds some important state that may change over time. It also
 * defines a method for saving the state inside a memento and another method for
 * restoring the state from it.
 */
class Originator {
  /**
   * @var string For the sake of simplicity, the originator&#39;s state is stored
   * inside a single variable.
   */
 private:
  std::string state_;

  std::string GenerateRandomString(int length = 10) {
    const char alphanum[] =
        &quot;0123456789&quot;
        &quot;ABCDEFGHIJKLMNOPQRSTUVWXYZ&quot;
        &quot;abcdefghijklmnopqrstuvwxyz&quot;;
    int stringLength = sizeof(alphanum) - 1;

    std::string random_string;
    for (int i = 0; i &lt; length; i++) {
      random_string += alphanum[std::rand() % stringLength];
    }
    return random_string;
  }

 public:
  Originator(std::string state) : state_(state) {
    std::cout &lt;&lt; &quot;Originator: My initial state is: &quot; &lt;&lt; this-&gt;state_ &lt;&lt; &quot;\n&quot;;
  }
  /**
   * The Originator&#39;s business logic may affect its internal state. Therefore,
   * the client should backup the state before launching methods of the business
   * logic via the save() method.
   */
  void DoSomething() {
    std::cout &lt;&lt; &quot;Originator: I&#39;m doing something important.\n&quot;;
    this-&gt;state_ = this-&gt;GenerateRandomString(30);
    std::cout &lt;&lt; &quot;Originator: and my state has changed to: &quot; &lt;&lt; this-&gt;state_ &lt;&lt; &quot;\n&quot;;
  }

  /**
   * Saves the current state inside a memento.
   */
  Memento *Save() {
    return new ConcreteMemento(this-&gt;state_);
  }
  /**
   * Restores the Originator&#39;s state from a memento object.
   */
  void Restore(Memento *memento) {
    this-&gt;state_ = memento-&gt;state();
    std::cout &lt;&lt; &quot;Originator: My state has changed to: &quot; &lt;&lt; this-&gt;state_ &lt;&lt; &quot;\n&quot;;
    delete memento;
  }
};

/**
 * The Caretaker doesn&#39;t depend on the Concrete Memento class. Therefore, it
 * doesn&#39;t have access to the originator&#39;s state, stored inside the memento. It
 * works with all mementos via the base Memento interface.
 */
class Caretaker {
  /**
   * @var Memento[]
   */
 private:
  std::vector&lt;Memento *&gt; mementos_;

  /**
   * @var Originator
   */
  Originator *originator_;

 public:
     Caretaker(Originator* originator) : originator_(originator) {
     }

     ~Caretaker() {
         for (auto m : mementos_) delete m;
     }

  void Backup() {
    std::cout &lt;&lt; &quot;\nCaretaker: Saving Originator&#39;s state...\n&quot;;
    this-&gt;mementos_.push_back(this-&gt;originator_-&gt;Save());
  }
  void Undo() {
    if (!this-&gt;mementos_.size()) {
      return;
    }
    Memento *memento = this-&gt;mementos_.back();
    this-&gt;mementos_.pop_back();
    std::cout &lt;&lt; &quot;Caretaker: Restoring state to: &quot; &lt;&lt; memento-&gt;GetName() &lt;&lt; &quot;\n&quot;;
    try {
      this-&gt;originator_-&gt;Restore(memento);
    } catch (...) {
      this-&gt;Undo();
    }
  }
  void ShowHistory() const {
    std::cout &lt;&lt; &quot;Caretaker: Here&#39;s the list of mementos:\n&quot;;
    for (Memento *memento : this-&gt;mementos_) {
      std::cout &lt;&lt; memento-&gt;GetName() &lt;&lt; &quot;\n&quot;;
    }
  }
};
/**
 * Client code.
 */

void ClientCode() {
  Originator *originator = new Originator(&quot;Super-duper-super-puper-super.&quot;);
  Caretaker *caretaker = new Caretaker(originator);
  caretaker-&gt;Backup();
  originator-&gt;DoSomething();
  caretaker-&gt;Backup();
  originator-&gt;DoSomething();
  caretaker-&gt;Backup();
  originator-&gt;DoSomething();
  std::cout &lt;&lt; &quot;\n&quot;;
  caretaker-&gt;ShowHistory();
  std::cout &lt;&lt; &quot;\nClient: Now, let&#39;s rollback!\n\n&quot;;
  caretaker-&gt;Undo();
  std::cout &lt;&lt; &quot;\nClient: Once more!\n\n&quot;;
  caretaker-&gt;Undo();

  delete originator;
  delete caretaker;
}

int main() {
  std::srand(static_cast&lt;unsigned int&gt;(std::time(NULL)));
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: uOInE8wmckHYPwZS7PtUTwuwZfCIbz

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: te6RGmykRpbqaWo5MEwjji1fpM1t5D

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: hX5xWDVljcQ9ydD7StUfbBt5Z7pcSN

Caretaker: Here&#39;s the list of mementos:
Sat Oct 19 18:09:37 2019
 / (Super-dup...)
Sat Oct 19 18:09:37 2019
 / (uOInE8wmc...)
Sat Oct 19 18:09:37 2019
 / (te6RGmykR...)

Client: Now, let&#39;s rollback!

Caretaker: Restoring state to: Sat Oct 19 18:09:37 2019
 / (te6RGmykR...)
Originator: My state has changed to: te6RGmykRpbqaWo5MEwjji1fpM1t5D

Client: Once more!

Caretaker: Restoring state to: Sat Oct 19 18:09:37 2019
 / (uOInE8wmc...)
Originator: My state has changed to: uOInE8wmckHYPwZS7PtUTwuwZfCIbz</code></pre>
</figure>

::: next
#### 다음을 보세요

[C++로 작성된 옵서버 []{.fa
.fa-arrow-right}](../../observer/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C++로 작성된
중재자](../../mediator/cpp/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **메멘토**

[![C#으로 작성된
메멘토](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 메멘토"){.prog-lang-link}
[![Go로 작성된
메멘토](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 메멘토"){.prog-lang-link}
[![자바로 작성된
메멘토](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 메멘토"){.prog-lang-link}
[![PHP로 작성된
메멘토](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 메멘토"){.prog-lang-link}
[![파이썬으로 작성된
메멘토](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 메멘토"){.prog-lang-link}
[![루비로 작성된
메멘토](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 메멘토"){.prog-lang-link}
[![러스트로 작성된
메멘토](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 메멘토"){.prog-lang-link}
[![스위프트로 작성된
메멘토](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 메멘토"){.prog-lang-link}
[![타입스크립트로 작성된
메멘토](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 메멘토"){.prog-lang-link}
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
