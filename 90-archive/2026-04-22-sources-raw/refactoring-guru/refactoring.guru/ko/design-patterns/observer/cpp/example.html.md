:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[옵서버](../../observer.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![옵서버](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# C++로 작성된 **옵서버** {#c로-작성된-옵서버 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**옵서버** 패턴은 일부 객체들이 다른 객체들에 자신의 상태 변경에 대해
알릴 수 있는 행동 디자인 패턴입니다.

옵서버 패턴은 구독자 인터페이스를 구현하는 모든 객체에 대한 이러한
이벤트들을 구독 및 구독 취소하는 방법을 제공합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 옵서버에 대하여 더 자세히 알아보세요 ](../../observer.html){.btn
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

**사용 사례들:** 옵서버 패턴은 C++ 코드, 특히 그래픽 사용자 인터페이스
컴포넌트들에서 매우 일반적입니다. 다른 객체들과의 클래스들과 결합하지
않고 해당 객체들에서 발생하는 이벤트들에 반응하는 방법을 제공합니다.

**식별:** 옵서버 패턴은 들어오는 메서드들을 목록에 저장하는 구독
메서드로 초기 식별할 수 있으며, 만약 위 구독 메서드가 목록의 객체들을
순회하고 그들의 \'업데이트\' 메서드를 호출하면 해당 패턴은 옵서버
패턴으로 확정지을 수 있습니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **옵서버** 패턴의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--main-cc .anchor} **main.cc:** 개념적인 예시

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Observer Design Pattern
 *
 * Intent: Lets you define a subscription mechanism to notify multiple objects
 * about any events that happen to the object they&#39;re observing.
 *
 * Note that there&#39;s a lot of different terms with similar meaning associated
 * with this pattern. Just remember that the Subject is also called the
 * Publisher and the Observer is often called the Subscriber and vice versa.
 * Also the verbs &quot;observe&quot;, &quot;listen&quot; or &quot;track&quot; usually mean the same thing.
 */

#include &lt;iostream&gt;
#include &lt;list&gt;
#include &lt;string&gt;

class IObserver {
 public:
  virtual ~IObserver(){};
  virtual void Update(const std::string &amp;message_from_subject) = 0;
};

class ISubject {
 public:
  virtual ~ISubject(){};
  virtual void Attach(IObserver *observer) = 0;
  virtual void Detach(IObserver *observer) = 0;
  virtual void Notify() = 0;
};

/**
 * The Subject owns some important state and notifies observers when the state
 * changes.
 */

class Subject : public ISubject {
 public:
  virtual ~Subject() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Subject.\n&quot;;
  }

  /**
   * The subscription management methods.
   */
  void Attach(IObserver *observer) override {
    list_observer_.push_back(observer);
  }
  void Detach(IObserver *observer) override {
    list_observer_.remove(observer);
  }
  void Notify() override {
    std::list&lt;IObserver *&gt;::iterator iterator = list_observer_.begin();
    HowManyObserver();
    while (iterator != list_observer_.end()) {
      (*iterator)-&gt;Update(message_);
      ++iterator;
    }
  }

  void CreateMessage(std::string message = &quot;Empty&quot;) {
    this-&gt;message_ = message;
    Notify();
  }
  void HowManyObserver() {
    std::cout &lt;&lt; &quot;There are &quot; &lt;&lt; list_observer_.size() &lt;&lt; &quot; observers in the list.\n&quot;;
  }

  /**
   * Usually, the subscription logic is only a fraction of what a Subject can
   * really do. Subjects commonly hold some important business logic, that
   * triggers a notification method whenever something important is about to
   * happen (or after it).
   */
  void SomeBusinessLogic() {
    this-&gt;message_ = &quot;change message message&quot;;
    Notify();
    std::cout &lt;&lt; &quot;I&#39;m about to do some thing important\n&quot;;
  }

 private:
  std::list&lt;IObserver *&gt; list_observer_;
  std::string message_;
};

class Observer : public IObserver {
 public:
  Observer(Subject &amp;subject) : subject_(subject) {
    this-&gt;subject_.Attach(this);
    std::cout &lt;&lt; &quot;Hi, I&#39;m the Observer \&quot;&quot; &lt;&lt; ++Observer::static_number_ &lt;&lt; &quot;\&quot;.\n&quot;;
    this-&gt;number_ = Observer::static_number_;
  }
  virtual ~Observer() {
    std::cout &lt;&lt; &quot;Goodbye, I was the Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;.\n&quot;;
  }

  void Update(const std::string &amp;message_from_subject) override {
    message_from_subject_ = message_from_subject;
    PrintInfo();
  }
  void RemoveMeFromTheList() {
    subject_.Detach(this);
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; number_ &lt;&lt; &quot;\&quot; removed from the list.\n&quot;;
  }
  void PrintInfo() {
    std::cout &lt;&lt; &quot;Observer \&quot;&quot; &lt;&lt; this-&gt;number_ &lt;&lt; &quot;\&quot;: a new message is available --&gt; &quot; &lt;&lt; this-&gt;message_from_subject_ &lt;&lt; &quot;\n&quot;;
  }

 private:
  std::string message_from_subject_;
  Subject &amp;subject_;
  static int static_number_;
  int number_;
};

int Observer::static_number_ = 0;

void ClientCode() {
  Subject *subject = new Subject;
  Observer *observer1 = new Observer(*subject);
  Observer *observer2 = new Observer(*subject);
  Observer *observer3 = new Observer(*subject);
  Observer *observer4;
  Observer *observer5;

  subject-&gt;CreateMessage(&quot;Hello World! :D&quot;);
  observer3-&gt;RemoveMeFromTheList();

  subject-&gt;CreateMessage(&quot;The weather is hot today! :p&quot;);
  observer4 = new Observer(*subject);

  observer2-&gt;RemoveMeFromTheList();
  observer5 = new Observer(*subject);

  subject-&gt;CreateMessage(&quot;My new car is great! ;)&quot;);
  observer5-&gt;RemoveMeFromTheList();

  observer4-&gt;RemoveMeFromTheList();
  observer1-&gt;RemoveMeFromTheList();

  delete observer5;
  delete observer4;
  delete observer3;
  delete observer2;
  delete observer1;
  delete subject;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Hi, I&#39;m the Observer &quot;1&quot;.
Hi, I&#39;m the Observer &quot;2&quot;.
Hi, I&#39;m the Observer &quot;3&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;2&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot;: a new message is available --&gt; Hello World! :D
Observer &quot;3&quot; removed from the list.
There are 2 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; The weather is hot today! :p
Observer &quot;2&quot;: a new message is available --&gt; The weather is hot today! :p
Hi, I&#39;m the Observer &quot;4&quot;.
Observer &quot;2&quot; removed from the list.
Hi, I&#39;m the Observer &quot;5&quot;.
There are 3 observers in the list.
Observer &quot;1&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;4&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot;: a new message is available --&gt; My new car is great! ;)
Observer &quot;5&quot; removed from the list.
Observer &quot;4&quot; removed from the list.
Observer &quot;1&quot; removed from the list.
Goodbye, I was the Observer &quot;5&quot;.
Goodbye, I was the Observer &quot;4&quot;.
Goodbye, I was the Observer &quot;3&quot;.
Goodbye, I was the Observer &quot;2&quot;.
Goodbye, I was the Observer &quot;1&quot;.
Goodbye, I was the Subject.</code></pre>
</figure>

::: next
#### 다음을 보세요

[C++로 작성된 상태 []{.fa
.fa-arrow-right}](../../state/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C++로 작성된
메멘토](../../memento/cpp/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **옵서버**

[![C#으로 작성된
옵서버](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 옵서버"){.prog-lang-link}
[![Go로 작성된
옵서버](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 옵서버"){.prog-lang-link}
[![자바로 작성된
옵서버](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 옵서버"){.prog-lang-link}
[![PHP로 작성된
옵서버](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 옵서버"){.prog-lang-link}
[![파이썬으로 작성된
옵서버](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 옵서버"){.prog-lang-link}
[![루비로 작성된
옵서버](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 옵서버"){.prog-lang-link}
[![러스트로 작성된
옵서버](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 옵서버"){.prog-lang-link}
[![스위프트로 작성된
옵서버](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 옵서버"){.prog-lang-link}
[![타입스크립트로 작성된
옵서버](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 옵서버"){.prog-lang-link}
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
