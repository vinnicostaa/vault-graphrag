:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[상태](../../state.html){.type} /
[타입스크립트](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![상태](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# 타입스크립트로 작성된 **상태** {#타입스크립트로-작성된-상태 .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**상태**는 객체의 내부 상태가 변경될 때 해당 객체가 행동을 변경할 수
있도록 하는 행동 디자인 패턴입니다.

패턴은 상태 관련 행동들을 별도의 상태 클래스들로 추출하며 또 원래 객체가
자체적으로 작동하는 대신 위에 언급된 클래스들에 작업을 위임하도록
강제합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 상태에 대하여 더 자세히 알아보세요 ](../../state.html){.btn .btn-lg
.btn-primary}
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
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 사례들:** 상태 패턴은 일반적으로 타입스크립트에서 대규모 `switch`
기반 상태 머신들을 객체들로 변환하는 데 사용됩니다.

**식별:** 객체들의 상태에 따라 행동을 변경하는 메서드들이 있으면 패턴은
상태 패턴으로 초기 식별될 수 있으며 이 상태가 상태 객체들 자체를
포함하여 다른 객체들에 의해 제어되거나 대체될 수 있으면 해당 패턴은 상태
패턴입니다.
:::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

이 예시는 **상태** 디자인 패턴의 구조를 보여주고 다음 질문에 중점을
둡니다:

-   패턴은 어떤 클래스들로 구성되어 있나요?
-   이 클래스들은 어떤 역할을 하나요?
-   패턴의 요소들은 어떻게 서로 연관되어 있나요?

#### []{#example-0--index-ts .anchor} **index.ts:** 개념적인 예시

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Context defines the interface of interest to clients. It also maintains a
 * reference to an instance of a State subclass, which represents the current
 * state of the Context.
 */
class Context {
    /**
     * @type {State} A reference to the current state of the Context.
     */
    private state: State;

    constructor(state: State) {
        this.transitionTo(state);
    }

    /**
     * The Context allows changing the State object at runtime.
     */
    public transitionTo(state: State): void {
        console.log(`Context: Transition to ${(&lt;any&gt;state).constructor.name}.`);
        this.state = state;
        this.state.setContext(this);
    }

    /**
     * The Context delegates part of its behavior to the current State object.
     */
    public request1(): void {
        this.state.handle1();
    }

    public request2(): void {
        this.state.handle2();
    }
}

/**
 * The base State class declares methods that all Concrete State should
 * implement and also provides a backreference to the Context object, associated
 * with the State. This backreference can be used by States to transition the
 * Context to another State.
 */
abstract class State {
    protected context: Context;

    public setContext(context: Context) {
        this.context = context;
    }

    public abstract handle1(): void;

    public abstract handle2(): void;
}

/**
 * Concrete States implement various behaviors, associated with a state of the
 * Context.
 */
class ConcreteStateA extends State {
    public handle1(): void {
        console.log(&#39;ConcreteStateA handles request1.&#39;);
        console.log(&#39;ConcreteStateA wants to change the state of the context.&#39;);
        this.context.transitionTo(new ConcreteStateB());
    }

    public handle2(): void {
        console.log(&#39;ConcreteStateA handles request2.&#39;);
    }
}

class ConcreteStateB extends State {
    public handle1(): void {
        console.log(&#39;ConcreteStateB handles request1.&#39;);
    }

    public handle2(): void {
        console.log(&#39;ConcreteStateB handles request2.&#39;);
        console.log(&#39;ConcreteStateB wants to change the state of the context.&#39;);
        this.context.transitionTo(new ConcreteStateA());
    }
}

/**
 * The client code.
 */
const context = new Context(new ConcreteStateA());
context.request1();
context.request2();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA.
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB.
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA.</code></pre>
</figure>

::: next
#### 다음을 보세요

[타입스크립트로 작성된 전략 []{.fa
.fa-arrow-right}](../../strategy/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 타입스크립트로 작성된
옵서버](../../observer/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **상태**

[![C#으로 작성된
상태](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 상태"){.prog-lang-link}
[![C++로 작성된
상태](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 상태"){.prog-lang-link}
[![Go로 작성된
상태](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 상태"){.prog-lang-link}
[![자바로 작성된
상태](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 상태"){.prog-lang-link}
[![PHP로 작성된
상태](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 상태"){.prog-lang-link}
[![파이썬으로 작성된
상태](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 상태"){.prog-lang-link}
[![루비로 작성된
상태](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 상태"){.prog-lang-link}
[![러스트로 작성된
상태](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 상태"){.prog-lang-link}
[![스위프트로 작성된
상태](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 상태"){.prog-lang-link}
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
