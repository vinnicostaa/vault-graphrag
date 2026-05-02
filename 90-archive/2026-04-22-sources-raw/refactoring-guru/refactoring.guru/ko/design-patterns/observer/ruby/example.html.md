:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[옵서버](../../observer.html){.type} / [루비](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![옵서버](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# 루비로 작성된 **옵서버** {#루비로-작성된-옵서버 .pattern-example-title-block-title}
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 옵서버 패턴은 루비 코드, 특히 그래픽 사용자 인터페이스
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

#### []{#example-0--main-rb .anchor} **main.rb:** 개념적인 예시

<figure class="code">
<pre class="code" lang="ruby"><code># The Subject interface declares a set of methods for managing subscribers.
class Subject
  # Attach an observer to the subject.
  def attach(observer)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Detach an observer from the subject.
  def detach(observer)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Notify all observers about an event.
  def notify
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The Subject owns some important state and notifies observers when the state
# changes.
class ConcreteSubject &lt; Subject
  # For the sake of simplicity, the Subject&#39;s state, essential to all
  # subscribers, is stored in this variable.
  attr_accessor :state

  # @!attribute observers
  # @return [Array&lt;Observer&gt;] attr_accessor :observers private :observers

  def initialize
    @observers = []
  end

  # List of subscribers. In real life, the list of subscribers can be stored
  # more comprehensively (categorized by event type, etc.).

  # @param [Observer] observer
  def attach(observer)
    puts &#39;Subject: Attached an observer.&#39;
    @observers &lt;&lt; observer
  end

  # @param [Observer] observer
  def detach(observer)
    @observers.delete(observer)
  end

  # The subscription management methods.

  # Trigger an update in each subscriber.
  def notify
    puts &#39;Subject: Notifying observers...&#39;
    @observers.each { |observer| observer.update(self) }
  end

  # Usually, the subscription logic is only a fraction of what a Subject can
  # really do. Subjects commonly hold some important business logic, that
  # triggers a notification method whenever something important is about to
  # happen (or after it).
  def some_business_logic
    puts &quot;\nSubject: I&#39;m doing something important.&quot;
    @state = rand(0..10)

    puts &quot;Subject: My state has just changed to: #{@state}&quot;
    notify
  end
end

# The Observer interface declares the update method, used by subjects.
class Observer
  # Receive update from subject.
  def update(_subject)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Observers react to the updates issued by the Subject they had been
# attached to.

class ConcreteObserverA &lt; Observer
  # @param [Subject] subject
  def update(subject)
    puts &#39;ConcreteObserverA: Reacted to the event&#39; if subject.state &lt; 3
  end
end

class ConcreteObserverB &lt; Observer
  # @param [Subject] subject
  def update(subject)
    return unless subject.state.zero? || subject.state &gt;= 2

    puts &#39;ConcreteObserverB: Reacted to the event&#39;
  end
end

# The client code.

subject = ConcreteSubject.new

observer_a = ConcreteObserverA.new
subject.attach(observer_a)

observer_b = ConcreteObserverB.new
subject.attach(observer_b)

subject.some_business_logic
subject.some_business_logic

subject.detach(observer_a)

subject.some_business_logic</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 1
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 10
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 2
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event</code></pre>
</figure>

## 다른 언어로 작성된 **옵서버**

[![C#으로 작성된
옵서버](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 옵서버"){.prog-lang-link}
[![C++로 작성된
옵서버](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 옵서버"){.prog-lang-link}
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
