:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[옵서버](../../observer.html){.type} /
[파이썬](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![옵서버](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# 파이썬으로 작성된 **옵서버** {#파이썬으로-작성된-옵서버 .pattern-example-title-block-title}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 옵서버 패턴은 파이썬 코드, 특히 그래픽 사용자
인터페이스 컴포넌트들에서 매우 일반적입니다. 다른 객체들과의 클래스들과
결합하지 않고 해당 객체들에서 발생하는 이벤트들에 반응하는 방법을
제공합니다.

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

#### []{#example-0--main-py .anchor} **main.py:** 개념적인 예시

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from random import randrange
from typing import List


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares a set of methods for managing subscribers.
    &quot;&quot;&quot;

    @abstractmethod
    def attach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Attach an observer to the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def detach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Detach an observer from the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Notify all observers about an event.
        &quot;&quot;&quot;
        pass


class ConcreteSubject(Subject):
    &quot;&quot;&quot;
    The Subject owns some important state and notifies observers when the state
    changes.
    &quot;&quot;&quot;

    _state: int = None
    &quot;&quot;&quot;
    For the sake of simplicity, the Subject&#39;s state, essential to all
    subscribers, is stored in this variable.
    &quot;&quot;&quot;

    _observers: List[Observer] = []
    &quot;&quot;&quot;
    List of subscribers. In real life, the list of subscribers can be stored
    more comprehensively (categorized by event type, etc.).
    &quot;&quot;&quot;

    def attach(self, observer: Observer) -&gt; None:
        print(&quot;Subject: Attached an observer.&quot;)
        self._observers.append(observer)

    def detach(self, observer: Observer) -&gt; None:
        self._observers.remove(observer)

    &quot;&quot;&quot;
    The subscription management methods.
    &quot;&quot;&quot;

    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Trigger an update in each subscriber.
        &quot;&quot;&quot;

        print(&quot;Subject: Notifying observers...&quot;)
        for observer in self._observers:
            observer.update(self)

    def some_business_logic(self) -&gt; None:
        &quot;&quot;&quot;
        Usually, the subscription logic is only a fraction of what a Subject can
        really do. Subjects commonly hold some important business logic, that
        triggers a notification method whenever something important is about to
        happen (or after it).
        &quot;&quot;&quot;

        print(&quot;\nSubject: I&#39;m doing something important.&quot;)
        self._state = randrange(0, 10)

        print(f&quot;Subject: My state has just changed to: {self._state}&quot;)
        self.notify()


class Observer(ABC):
    &quot;&quot;&quot;
    The Observer interface declares the update method, used by subjects.
    &quot;&quot;&quot;

    @abstractmethod
    def update(self, subject: Subject) -&gt; None:
        &quot;&quot;&quot;
        Receive update from subject.
        &quot;&quot;&quot;
        pass


&quot;&quot;&quot;
Concrete Observers react to the updates issued by the Subject they had been
attached to.
&quot;&quot;&quot;


class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state &lt; 3:
            print(&quot;ConcreteObserverA: Reacted to the event&quot;)


class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state == 0 or subject._state &gt;= 2:
            print(&quot;ConcreteObserverB: Reacted to the event&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code.

    subject = ConcreteSubject()

    observer_a = ConcreteObserverA()
    subject.attach(observer_a)

    observer_b = ConcreteObserverB()
    subject.attach(observer_b)

    subject.some_business_logic()
    subject.some_business_logic()

    subject.detach(observer_a)

    subject.some_business_logic()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 5
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event</code></pre>
</figure>

::: next
#### 다음을 보세요

[파이썬으로 작성된 상태 []{.fa
.fa-arrow-right}](../../state/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 파이썬으로 작성된
메멘토](../../memento/python/example.html){.btn .btn-default rel="prev"}
:::

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
