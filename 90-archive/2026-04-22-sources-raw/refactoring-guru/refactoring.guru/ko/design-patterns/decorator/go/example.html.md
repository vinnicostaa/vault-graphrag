:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[데코레이터](../../decorator.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![데코레이터](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# Go로 작성된 **데코레이터** {#go로-작성된-데코레이터 .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**데코레이터**는 구조 패턴이며 새로운 행동들을 특수 래퍼 객체들 내에
넣어서 이러한 행동들을 객체들에 동적으로 추가할 수 있도록 합니다.

데코레이터를 사용하여 객체들을 제한 없이 래핑할 수 있습니다. 왜냐하면
대상 객체들과 데코레이터들은 같은 인터페이스를 따르기 때문입니다. 결과
객체는 모든 래퍼의 스태킹된 행동을 가질 것입니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 데코레이터에 대하여 더 자세히 알아보세요 ](../../decorator.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
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
 [pizza](#example-0--pizza-go)
:::

::: en-file
 [veggie­Mania](#example-0--veggieMania-go)
:::

::: en-file
 [tomato­Topping](#example-0--tomatoTopping-go)
:::

::: en-file
 [cheese­Topping](#example-0--cheeseTopping-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::
:::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

#### []{#example-0--pizza-go .anchor} **pizza.go:** 컴포넌트 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type IPizza interface {
    getPrice() int
}</code></pre>
</figure>

#### []{#example-0--veggieMania-go .anchor} **veggieMania.go:** 구상 컴포넌트

<figure class="code">
<pre class="code" lang="go"><code>package main

type VeggieMania struct {
}

func (p *VeggieMania) getPrice() int {
    return 15
}</code></pre>
</figure>

#### []{#example-0--tomatoTopping-go .anchor} **tomatoTopping.go:** 구상 데코레이터

<figure class="code">
<pre class="code" lang="go"><code>package main

type TomatoTopping struct {
    pizza IPizza
}

func (c *TomatoTopping) getPrice() int {
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 7
}</code></pre>
</figure>

#### []{#example-0--cheeseTopping-go .anchor} **cheeseTopping.go:** 구상 데코레이터

<figure class="code">
<pre class="code" lang="go"><code>package main

type CheeseTopping struct {
    pizza IPizza
}

func (c *CheeseTopping) getPrice() int {
    pizzaPrice := c.pizza.getPrice()
    return pizzaPrice + 10
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {

    pizza := &amp;VeggieMania{}

    //Add cheese topping
    pizzaWithCheese := &amp;CheeseTopping{
        pizza: pizza,
    }

    //Add tomato topping
    pizzaWithCheeseAndTomato := &amp;TomatoTopping{
        pizza: pizzaWithCheese,
    }

    fmt.Printf(&quot;Price of veggeMania with tomato and cheese topping is %d\n&quot;, pizzaWithCheeseAndTomato.getPrice())
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Price of veggeMania with tomato and cheese topping is 32</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 퍼사드 []{.fa
.fa-arrow-right}](../../facade/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
복합체](../../composite/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **데코레이터**

[![C#으로 작성된
데코레이터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 데코레이터"){.prog-lang-link}
[![C++로 작성된
데코레이터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 데코레이터"){.prog-lang-link}
[![자바로 작성된
데코레이터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 데코레이터"){.prog-lang-link}
[![PHP로 작성된
데코레이터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 데코레이터"){.prog-lang-link}
[![파이썬으로 작성된
데코레이터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 데코레이터"){.prog-lang-link}
[![루비로 작성된
데코레이터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 데코레이터"){.prog-lang-link}
[![러스트로 작성된
데코레이터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 데코레이터"){.prog-lang-link}
[![스위프트로 작성된
데코레이터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 데코레이터"){.prog-lang-link}
[![타입스크립트로 작성된
데코레이터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 데코레이터"){.prog-lang-link}
:::::::::::::::::::::

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
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
