::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[비지터](../../visitor.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![비지터](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# Go로 작성된 **비지터** {#go로-작성된-비지터 .pattern-example-title-block-title}
::::

:::::::::::::::::: pattern-example-body
::: pattern-example-brief
**비지터**는 기존 코드를 변경하지 않고 기존 클래스 계층구조에 새로운
행동들을 추가할 수 있도록 하는 행동 디자인 패턴입니다.

> 제 설명글 \[비지터와 이중 디스패치\]{비지터와 이중 디스패치}에서 왜
> 단순히 비지터들을 메서드 오버로딩으로 대체할 수 없는지 알아보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 비지터에 대하여 더 자세히 알아보세요 ](../../visitor.html){.btn
.btn-lg .btn-primary}
:::

::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [shape](#example-0--shape-go)
:::

::: en-file
 [square](#example-0--square-go)
:::

::: en-file
 [circle](#example-0--circle-go)
:::

::: en-file
 [rectangle](#example-0--rectangle-go)
:::

::: en-file
 [visitor](#example-0--visitor-go)
:::

::: en-file
 [area­Calculator](#example-0--areaCalculator-go)
:::

::: en-file
 [middle­Coordinates](#example-0--middleCoordinates-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::
::::::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

비지터 패턴은 실제로 구조체​(struct)​를 수정하지 않고도 구조체에 행동을
추가할 수 있도록 합니다. 당신이 다음과 같은 다른 모양들의
구조체​(structs)​들을 가진 lib의 관리자라고 가정해 봅시다:

-   직사각형
-   원
-   삼각형

위의 각 모양의 구조체​(struct)​는 공통 모양 인터페이스를 구현합니다.

회사 사원들이 당신의 멋진 lib을 사용하기 시작하면 그들은 여러 기능들을
요청할 것입니다. 이 중 가장 간단한 요청을 검토해 봅시다: 회사 부서가
모양 구조체들에 `get­Area`​(면적 가져오기) 행동을 추가해달라고
요청했습니다.

이 문제를 해결하기 위한 여러 옵션이 있습니다.

가장 먼저 떠오르는 옵션은 `get­Area` 메서드를 모양 인터페이스에 직접
추가한 다음 각 모양 구조체​(struct)​에서 구현하는 것입니다. 좋은 옵션인
듯싶으나 단점이 있습니다. 당신은 라이브러리의 관리자로서 누군가가 다른
행동을 요청할 때마다 당신의 귀중한 코드가 손상되는 위험을 감수하고 싶지
않을 것입니다. 그럼에도 불구하고 당신은 다른 팀들이 당신의 라이브러리를
어떻게든 확장하기를 원합니다.

두 번째 옵션은 기능을 요청하는 부서가 직접 행동을 구현하도록 하는 것이나
요청된 행동은 비공개 코드에 의존할 수 있으므로 항상 가능한 옵션이
아닙니다.

세 번째 옵션은 비지터 패턴을 사용하여 위의 문제를 해결하는 것이며,
다음과 같이 비지터 인터페이스를 정의하는 것으로 시작합니다:

<figure class="code">
<pre class="code"><code>type visitor interface {
   visitForSquare(square)
   visitForCircle(circle)
   visitForTriangle(triangle)
}</code></pre>
</figure>

`visit­For­Square­(square)`, `visit­For­Circle­(circle)`,
`visit­For­Triangle­(triangle)` 함수들은 사각형, 원 및 삼각형에 각각
기능들을 추가할 수 있도록 합니다.

비지터 인터페이스에 단일 메서드 `visit­(shape)`가 있을 수 없는 이유가
궁금하나요? 그 이유는 Go 언어가 메서드 오버로딩을 지원하지 않기 때문에
이름은 같지만 매개변수들이 다른 메서드들을 가질 수 없도록 하기
때문입니다.

이제 두 번째 중요한 부분은 모양 인터페이스에 `accept` 메서드를 추가하는
것입니다.

<figure class="code">
<pre class="code"><code>func accept(v visitor)</code></pre>
</figure>

모든 모양 구조체들​(structs)​은 이 메서드를 정의해야 하며, 코드는 다음과
비슷하게 작성됩니다:

<figure class="code">
<pre class="code"><code>func (obj *square) accept(v visitor){
    v.visitForSquare(obj)
}</code></pre>
</figure>

잠시만요, 방금 기존 모양 구조체들​(structs)​을 수정하고 싶지 않다고
언급하지 않았나요? 불행히도 비지터 패턴을 사용할 때 모양 구조체를
수정해야 하긴 합니다. 그러나 이 수정은 단 한 번만 수행됩니다.

`get­Num­Sides`와 `get­Middle­Coordinates`같은 다른 행동들을 추가하는 경우
우리는 모양 구조체들​(structs)​에 추가 변경 없이 같은 `accept­(visitor)`
함수를 사용합니다.

최종적으로 모양 구조체들​(structs)​은 한 번만 수정하면 되며, 다른 행동들을
추가해 달라는 모든 향후 요청은 같은 accept 함수를 사용하여 처리할 수
있습니다. 팀에서 `get­Area` 행동을 요청하면 단순히 비지터 인터페이스의
구상 구현을 정의한 후 해당 구현에 면적 계산 로직을 작성하여 요청된
행동을 추가할 수 있습니다.

#### []{#example-0--shape-go .anchor} **shape.go:** 요소

<figure class="code">
<pre class="code" lang="go"><code>package main

type Shape interface {
    getType() string
    accept(Visitor)
}</code></pre>
</figure>

#### []{#example-0--square-go .anchor} **square.go:** 구상 요소

<figure class="code">
<pre class="code" lang="go"><code>package main

type Square struct {
    side int
}

func (s *Square) accept(v Visitor) {
    v.visitForSquare(s)
}

func (s *Square) getType() string {
    return &quot;Square&quot;
}</code></pre>
</figure>

#### []{#example-0--circle-go .anchor} **circle.go:** 구상 요소

<figure class="code">
<pre class="code" lang="go"><code>package main

type Circle struct {
    radius int
}

func (c *Circle) accept(v Visitor) {
    v.visitForCircle(c)
}

func (c *Circle) getType() string {
    return &quot;Circle&quot;
}</code></pre>
</figure>

#### []{#example-0--rectangle-go .anchor} **rectangle.go:** 구상 요소

<figure class="code">
<pre class="code" lang="go"><code>package main

type Rectangle struct {
    l int
    b int
}

func (t *Rectangle) accept(v Visitor) {
    v.visitForrectangle(t)
}

func (t *Rectangle) getType() string {
    return &quot;rectangle&quot;
}</code></pre>
</figure>

#### []{#example-0--visitor-go .anchor} **visitor.go:** 비지터

<figure class="code">
<pre class="code" lang="go"><code>package main

type Visitor interface {
    visitForSquare(*Square)
    visitForCircle(*Circle)
    visitForrectangle(*Rectangle)
}</code></pre>
</figure>

#### []{#example-0--areaCalculator-go .anchor} **areaCalculator.go:** 구상 비지터

<figure class="code">
<pre class="code" lang="go"><code>package main

import (
    &quot;fmt&quot;
)

type AreaCalculator struct {
    area int
}

func (a *AreaCalculator) visitForSquare(s *Square) {
    // Calculate area for square.
    // Then assign in to the area instance variable.
    fmt.Println(&quot;Calculating area for square&quot;)
}

func (a *AreaCalculator) visitForCircle(s *Circle) {
    fmt.Println(&quot;Calculating area for circle&quot;)
}
func (a *AreaCalculator) visitForrectangle(s *Rectangle) {
    fmt.Println(&quot;Calculating area for rectangle&quot;)
}</code></pre>
</figure>

#### []{#example-0--middleCoordinates-go .anchor} **middleCoordinates.go:** 구상 비지터

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type MiddleCoordinates struct {
    x int
    y int
}

func (a *MiddleCoordinates) visitForSquare(s *Square) {
    // Calculate middle point coordinates for square.
    // Then assign in to the x and y instance variable.
    fmt.Println(&quot;Calculating middle point coordinates for square&quot;)
}

func (a *MiddleCoordinates) visitForCircle(c *Circle) {
    fmt.Println(&quot;Calculating middle point coordinates for circle&quot;)
}
func (a *MiddleCoordinates) visitForrectangle(t *Rectangle) {
    fmt.Println(&quot;Calculating middle point coordinates for rectangle&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    square := &amp;Square{side: 2}
    circle := &amp;Circle{radius: 3}
    rectangle := &amp;Rectangle{l: 2, b: 3}

    areaCalculator := &amp;AreaCalculator{}

    square.accept(areaCalculator)
    circle.accept(areaCalculator)
    rectangle.accept(areaCalculator)

    fmt.Println()
    middleCoordinates := &amp;MiddleCoordinates{}
    square.accept(middleCoordinates)
    circle.accept(middleCoordinates)
    rectangle.accept(middleCoordinates)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Calculating area for square
Calculating area for circle
Calculating area for rectangle

Calculating middle point coordinates for square
Calculating middle point coordinates for circle
Calculating middle point coordinates for rectangle</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 디자인 패턴들 []{.fa
.fa-arrow-right}](../../java.html){.btn .btn-primary rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된 템플릿
메서드](../../template-method/go/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **비지터**

[![C#으로 작성된
비지터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 비지터"){.prog-lang-link}
[![C++로 작성된
비지터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 비지터"){.prog-lang-link}
[![자바로 작성된
비지터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 비지터"){.prog-lang-link}
[![PHP로 작성된
비지터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 비지터"){.prog-lang-link}
[![파이썬으로 작성된
비지터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 비지터"){.prog-lang-link}
[![루비로 작성된
비지터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 비지터"){.prog-lang-link}
[![러스트로 작성된
비지터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 비지터"){.prog-lang-link}
[![스위프트로 작성된
비지터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 비지터"){.prog-lang-link}
[![타입스크립트로 작성된
비지터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 비지터"){.prog-lang-link}
::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
