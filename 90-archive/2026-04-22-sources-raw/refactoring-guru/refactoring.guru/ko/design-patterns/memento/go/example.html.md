::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[메멘토](../../memento.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![메멘토](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# Go로 작성된 **메멘토** {#go로-작성된-메멘토 .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
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

::::::::::: {.sidebar-navigation .with-scroll}
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
 [originator](#example-0--originator-go)
:::

::: en-file
 [memento](#example-0--memento-go)
:::

::: en-file
 [caretaker](#example-0--caretaker-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

메멘토 패턴은 객체의 상태에 대한 스냅숏들을 저장할 수 있도록 하며 당신은
이러한 스냅숏들을 사용하여 객체를 이전 상태로 되돌릴 수 있습니다. 이
패턴은 객체에 대한 실행 취소-다시 실행 작업을 구현해야 할 때 편리합니다.

#### []{#example-0--originator-go .anchor} **originator.go:** 오리지네이터

<figure class="code">
<pre class="code" lang="go"><code>package main

type Originator struct {
    state string
}

func (e *Originator) createMemento() *Memento {
    return &amp;Memento{state: e.state}
}

func (e *Originator) restoreMemento(m *Memento) {
    e.state = m.getSavedState()
}

func (e *Originator) setState(state string) {
    e.state = state
}

func (e *Originator) getState() string {
    return e.state
}</code></pre>
</figure>

#### []{#example-0--memento-go .anchor} **memento.go:** 메멘토

<figure class="code">
<pre class="code" lang="go"><code>package main

type Memento struct {
    state string
}

func (m *Memento) getSavedState() string {
    return m.state
}</code></pre>
</figure>

#### []{#example-0--caretaker-go .anchor} **caretaker.go:** 케어테이커

<figure class="code">
<pre class="code" lang="go"><code>package main

type Caretaker struct {
    mementoArray []*Memento
}

func (c *Caretaker) addMemento(m *Memento) {
    c.mementoArray = append(c.mementoArray, m)
}

func (c *Caretaker) getMemento(index int) *Memento {
    return c.mementoArray[index]
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    caretaker := &amp;Caretaker{
        mementoArray: make([]*Memento, 0),
    }

    originator := &amp;Originator{
        state: &quot;A&quot;,
    }

    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;B&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.setState(&quot;C&quot;)
    fmt.Printf(&quot;Originator Current State: %s\n&quot;, originator.getState())
    caretaker.addMemento(originator.createMemento())

    originator.restoreMemento(caretaker.getMemento(1))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

    originator.restoreMemento(caretaker.getMemento(0))
    fmt.Printf(&quot;Restored to State: %s\n&quot;, originator.getState())

}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>originator Current State: A
originator Current State: B
originator Current State: C
Restored to State: B
Restored to State: A</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 옵서버 []{.fa
.fa-arrow-right}](../../observer/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
중재자](../../mediator/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **메멘토**

[![C#으로 작성된
메멘토](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 메멘토"){.prog-lang-link}
[![C++로 작성된
메멘토](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 메멘토"){.prog-lang-link}
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
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
