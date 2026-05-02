:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[중재자](../../mediator.html){.type} / [C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![중재자](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# C#으로 작성된 **중재자** {#c으로-작성된-중재자 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**중재자**는 행동 디자인 패턴이며 프로그램의 컴포넌트들이 특수 중재자
객체를 통하여 간접적으로 소통하게 함으로써 해당 컴포넌트 간의 결합도를
줄입니다.

중재자는 개별 컴포넌트들을 편집, 확장 및 재사용하는 것을 쉽게 만드는데,
그 이유는 이들이 더 이상 수십 개의 다른 클래스들에 의존하지 않기
때문입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 중재자에 대하여 더 자세히 알아보세요 ](../../mediator.html){.btn
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
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** C 코드에서 중재자 패턴의 가장 인기 있는 사용 용도는
앱의 그래픽 사용자 인터페이스 컴포넌트 간의 통신을 쉽게 하는 것입니다.
MVC 패턴의 컨트롤러 부분의 동의어는 중재자입니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **중재자**의 구조를 보여주고 다음 질문에 중점을 둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--Program-cs .anchor} **Program.cs:** 개념적인 예시

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Mediator.Conceptual
{
    // The Mediator interface declares a method used by components to notify the
    // mediator about various events. The Mediator may react to these events and
    // pass the execution to other components.
    public interface IMediator
    {
        void Notify(object sender, string ev);
    }

    // Concrete Mediators implement cooperative behavior by coordinating several
    // components.
    class ConcreteMediator : IMediator
    {
        private Component1 _component1;

        private Component2 _component2;

        public ConcreteMediator(Component1 component1, Component2 component2)
        {
            this._component1 = component1;
            this._component1.SetMediator(this);
            this._component2 = component2;
            this._component2.SetMediator(this);
        } 

        public void Notify(object sender, string ev)
        {
            if (ev == &quot;A&quot;)
            {
                Console.WriteLine(&quot;Mediator reacts on A and triggers following operations:&quot;);
                this._component2.DoC();
            }
            if (ev == &quot;D&quot;)
            {
                Console.WriteLine(&quot;Mediator reacts on D and triggers following operations:&quot;);
                this._component1.DoB();
                this._component2.DoC();
            }
        }
    }

    // The Base Component provides the basic functionality of storing a
    // mediator&#39;s instance inside component objects.
    class BaseComponent
    {
        protected IMediator _mediator;

        public BaseComponent(IMediator mediator = null)
        {
            this._mediator = mediator;
        }

        public void SetMediator(IMediator mediator)
        {
            this._mediator = mediator;
        }
    }

    // Concrete Components implement various functionality. They don&#39;t depend on
    // other components. They also don&#39;t depend on any concrete mediator
    // classes.
    class Component1 : BaseComponent
    {
        public void DoA()
        {
            Console.WriteLine(&quot;Component 1 does A.&quot;);

            this._mediator.Notify(this, &quot;A&quot;);
        }

        public void DoB()
        {
            Console.WriteLine(&quot;Component 1 does B.&quot;);

            this._mediator.Notify(this, &quot;B&quot;);
        }
    }

    class Component2 : BaseComponent
    {
        public void DoC()
        {
            Console.WriteLine(&quot;Component 2 does C.&quot;);

            this._mediator.Notify(this, &quot;C&quot;);
        }

        public void DoD()
        {
            Console.WriteLine(&quot;Component 2 does D.&quot;);

            this._mediator.Notify(this, &quot;D&quot;);
        }
    }
    
    class Program
    {
        static void Main(string[] args)
        {
            // The client code.
            Component1 component1 = new Component1();
            Component2 component2 = new Component2();
            new ConcreteMediator(component1, component2);

            Console.WriteLine(&quot;Client triggers operation A.&quot;);
            component1.DoA();

            Console.WriteLine();

            Console.WriteLine(&quot;Client triggers operation D.&quot;);
            component2.DoD();
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

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

::: next
#### 다음을 보세요

[C#으로 작성된 메멘토 []{.fa
.fa-arrow-right}](../../memento/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} C#으로 작성된
반복자](../../iterator/csharp/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **중재자**

[![C++로 작성된
중재자](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 중재자"){.prog-lang-link}
[![Go로 작성된
중재자](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 중재자"){.prog-lang-link}
[![자바로 작성된
중재자](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 중재자"){.prog-lang-link}
[![PHP로 작성된
중재자](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 중재자"){.prog-lang-link}
[![파이썬으로 작성된
중재자](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 중재자"){.prog-lang-link}
[![루비로 작성된
중재자](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 중재자"){.prog-lang-link}
[![러스트로 작성된
중재자](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 중재자"){.prog-lang-link}
[![스위프트로 작성된
중재자](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 중재자"){.prog-lang-link}
[![타입스크립트로 작성된
중재자](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 중재자"){.prog-lang-link}
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
