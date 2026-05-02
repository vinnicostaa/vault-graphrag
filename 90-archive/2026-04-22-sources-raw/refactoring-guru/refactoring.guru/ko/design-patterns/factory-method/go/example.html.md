::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [팩토리
메서드](../../factory-method.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![팩토리
메서드](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# Go로 작성된 **팩토리 메서드** {#go로-작성된-팩토리-메서드 .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**팩토리 메서드**는 제품 객체들의 구상 클래스들을 지정하지 않고 해당
제품 객체들을 생성할 수 있도록 하는 생성 디자인 패턴입니다.

팩토리 메서드는 메서드를 정의하며, 이 메서드는 직접 생성자 호출​(`new`
연산자)​을 사용하여 객체를 생성하는 대신 객체 생성에 사용되여야 합니다.
자식 클래스들은 이 메서드를 오버라이드하여 생성될 객체들의 클래스를
변경할 수 있습니다.

> 다양한 팩토리 패턴들과 개념들의 차이점을 이해하지 못하셨다면 [팩토리
> 비교](../../factory-comparison.html)를 읽어보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 팩토리 메서드에 대하여 더 자세히 알아보세요
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Gun](#example-0--iGun-go)
:::

::: en-file
 [gun](#example-0--gun-go)
:::

::: en-file
 [ak47](#example-0--ak47-go)
:::

::: en-file
 [musket](#example-0--musket-go)
:::

::: en-file
 [gun­Factory](#example-0--gunFactory-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::
::::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

클래스 및 상속과 같은 객체 지향 프로그램 기능이 없으므로 Go에서 고전적인
팩토리 메서드 패턴을 구현하는 것은 불가능하나 여전히 Go에서 패턴의 기본
버전인 단순 팩토리​(Simple Factory)​를 구현할 수 있습니다.

이 예시에서는 팩토리 구조체​(struct)​를 사용하여 다양한 유형의 무기를 만들
것입니다.

먼저 총에 있어야 하는 모든 메서드들을 정의하는 `i­Gun` 인터페이스를
만듭니다. iGun 인터페이스를 구현하는 `gun` 구조체​(struct) 유형이
있습니다. 두 개의 구상 총들​(`ak47` 및 `musket`)​은 총 구조체​(struct)​를
포함하고 모든 `i­Gun` 메서드들을 간접적으로 구현합니다.

`gun­Factory` 구조체​(struct)​는 팩토리의 역할을 하며, 수신된 인수를
기반으로 요청된 유형의 총들을 생성합니다. *main.go*는 클라이언트의
역할을 하며 `ak47` 또는 `musket`과 직접 상호작용하는 대신 `gun­Factory`에
의존하여 다양한 총들의 인스턴스를 생성하고, 생성을 문자열 매개변수만
사용하여 제어합니다.

#### []{#example-0--iGun-go .anchor} **iGun.go:** 제품 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type IGun interface {
    setName(name string)
    setPower(power int)
    getName() string
    getPower() int
}</code></pre>
</figure>

#### []{#example-0--gun-go .anchor} **gun.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type Gun struct {
    name  string
    power int
}

func (g *Gun) setName(name string) {
    g.name = name
}

func (g *Gun) getName() string {
    return g.name
}

func (g *Gun) setPower(power int) {
    g.power = power
}

func (g *Gun) getPower() int {
    return g.power
}</code></pre>
</figure>

#### []{#example-0--ak47-go .anchor} **ak47.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type Ak47 struct {
    Gun
}

func newAk47() IGun {
    return &amp;Ak47{
        Gun: Gun{
            name:  &quot;AK47 gun&quot;,
            power: 4,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--musket-go .anchor} **musket.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type musket struct {
    Gun
}

func newMusket() IGun {
    return &amp;musket{
        Gun: Gun{
            name:  &quot;Musket gun&quot;,
            power: 1,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--gunFactory-go .anchor} **gunFactory.go:** 팩토리

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func getGun(gunType string) (IGun, error) {
    if gunType == &quot;ak47&quot; {
        return newAk47(), nil
    }
    if gunType == &quot;musket&quot; {
        return newMusket(), nil
    }
    return nil, fmt.Errorf(&quot;Wrong gun type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    ak47, _ := getGun(&quot;ak47&quot;)
    musket, _ := getGun(&quot;musket&quot;)

    printDetails(ak47)
    printDetails(musket)
}

func printDetails(g IGun) {
    fmt.Printf(&quot;Gun: %s&quot;, g.getName())
    fmt.Println()
    fmt.Printf(&quot;Power: %d&quot;, g.getPower())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Gun: AK47 gun
Power: 4
Gun: Musket gun
Power: 1</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 프로토타입 []{.fa
.fa-arrow-right}](../../prototype/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
빌더](../../builder/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **팩토리 메서드**

[![C#으로 작성된 팩토리
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 팩토리 메서드"){.prog-lang-link}
[![C++로 작성된 팩토리
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 팩토리 메서드"){.prog-lang-link}
[![자바로 작성된 팩토리
메서드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 팩토리 메서드"){.prog-lang-link}
[![PHP로 작성된 팩토리
메서드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 팩토리 메서드"){.prog-lang-link}
[![파이썬으로 작성된 팩토리
메서드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 팩토리 메서드"){.prog-lang-link}
[![루비로 작성된 팩토리
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 팩토리 메서드"){.prog-lang-link}
[![러스트로 작성된 팩토리
메서드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 팩토리 메서드"){.prog-lang-link}
[![스위프트로 작성된 팩토리
메서드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 팩토리 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 팩토리
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 팩토리 메서드"){.prog-lang-link}
::::::::::::::::::::::

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
::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::
