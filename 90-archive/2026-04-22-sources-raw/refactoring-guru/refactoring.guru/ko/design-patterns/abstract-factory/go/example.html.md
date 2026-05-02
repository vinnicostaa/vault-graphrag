::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [추상
팩토리](../../abstract-factory.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![추상
팩토리](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# Go로 작성된 **추상 팩토리** {#go로-작성된-추상-팩토리 .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**추상 팩토리**는 생성 디자인 패턴이며, 관련 객체들의 구상 클래스들을
지정하지 않고도 해당 객체들의 제품 패밀리들을 생성할 수 있도록 합니다.

추상 팩토리는 모든 고유한 제품들을 생성하기 위한 인터페이스를 정의하지만
실제 제품 생성은 구상 팩토리 클래스들에 맡깁니다. 또 각 팩토리 유형은
특정 제품군에 해당합니다.

클라이언트 코드는 생성자 호출​(`new` 연산자)​로 직접 제품들을 생성하는
대신 팩토리 객체의 생성 메서드들을 호출합니다. 팩토리는 단일 제품 변형에
해당하므로 해당 팩토리의 모든 제품이 호환될 것입니다.

클라이언트 코드는 추상 인터페이스를 통해서만 팩토리 및 제품과 함께
작동하며, 이렇게 하면 클라이언트 코드가 팩토리 객체에 의해 생성된 모든
제품 변형과 함께 작동할 수 있습니다. 새로운 구상 팩토리 클래스를 생성한
후 클라이언트 코드에 전달합니다.

> 다양한 팩토리 패턴들과 개념들의 차이점을 이해하지 못하셨다면 [팩토리
> 비교](../../factory-comparison.html)를 읽어보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 추상 팩토리에 대하여 더 자세히 알아보세요
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
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
 [i­Sports­Factory](#example-0--iSportsFactory-go)
:::

::: en-file
 [adidas](#example-0--adidas-go)
:::

::: en-file
 [nike](#example-0--nike-go)
:::

::: en-file
 [i­Shoe](#example-0--iShoe-go)
:::

::: en-file
 [adidas­Shoe](#example-0--adidasShoe-go)
:::

::: en-file
 [nike­Shoe](#example-0--nikeShoe-go)
:::

::: en-file
 [i­Shirt](#example-0--iShirt-go)
:::

::: en-file
 [adidas­Shirt](#example-0--adidasShirt-go)
:::

::: en-file
 [nike­Shirt](#example-0--nikeShirt-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

신발과 셔츠라는 두 가지 제품으로 구성된 스포츠 키트를 구매해야 한다고
가정해 봅시다. 모든 아이템이 일치하도록 같은 브랜드의 완전한 스포츠
키트를 구매하시고 싶으실 것입니다.

이것을 코드로 바꾸면 추상 팩토리는 제품들이 항상 서로 일치하는 제품
세트들을 만들 수 있습니다.

#### []{#example-0--iSportsFactory-go .anchor} **iSportsFactory.go:** 추상 팩토리 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type ISportsFactory interface {
    makeShoe() IShoe
    makeShirt() IShirt
}

func GetSportsFactory(brand string) (ISportsFactory, error) {
    if brand == &quot;adidas&quot; {
        return &amp;Adidas{}, nil
    }

    if brand == &quot;nike&quot; {
        return &amp;Nike{}, nil
    }

    return nil, fmt.Errorf(&quot;Wrong brand type passed&quot;)
}</code></pre>
</figure>

#### []{#example-0--adidas-go .anchor} **adidas.go:** 구상 팩토리

<figure class="code">
<pre class="code" lang="go"><code>package main

type Adidas struct {
}

func (a *Adidas) makeShoe() IShoe {
    return &amp;AdidasShoe{
        Shoe: Shoe{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}

func (a *Adidas) makeShirt() IShirt {
    return &amp;AdidasShirt{
        Shirt: Shirt{
            logo: &quot;adidas&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--nike-go .anchor} **nike.go:** 구상 팩토리

<figure class="code">
<pre class="code" lang="go"><code>package main

type Nike struct {
}

func (n *Nike) makeShoe() IShoe {
    return &amp;NikeShoe{
        Shoe: Shoe{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}

func (n *Nike) makeShirt() IShirt {
    return &amp;NikeShirt{
        Shirt: Shirt{
            logo: &quot;nike&quot;,
            size: 14,
        },
    }
}</code></pre>
</figure>

#### []{#example-0--iShoe-go .anchor} **iShoe.go:** 추상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShoe interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shoe struct {
    logo string
    size int
}

func (s *Shoe) setLogo(logo string) {
    s.logo = logo
}

func (s *Shoe) getLogo() string {
    return s.logo
}

func (s *Shoe) setSize(size int) {
    s.size = size
}

func (s *Shoe) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShoe-go .anchor} **adidasShoe.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--nikeShoe-go .anchor} **nikeShoe.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShoe struct {
    Shoe
}</code></pre>
</figure>

#### []{#example-0--iShirt-go .anchor} **iShirt.go:** 추상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type IShirt interface {
    setLogo(logo string)
    setSize(size int)
    getLogo() string
    getSize() int
}

type Shirt struct {
    logo string
    size int
}

func (s *Shirt) setLogo(logo string) {
    s.logo = logo
}

func (s *Shirt) getLogo() string {
    return s.logo
}

func (s *Shirt) setSize(size int) {
    s.size = size
}

func (s *Shirt) getSize() int {
    return s.size
}</code></pre>
</figure>

#### []{#example-0--adidasShirt-go .anchor} **adidasShirt.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type AdidasShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--nikeShirt-go .anchor} **nikeShirt.go:** 구상 제품

<figure class="code">
<pre class="code" lang="go"><code>package main

type NikeShirt struct {
    Shirt
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

func main() {
    adidasFactory, _ := GetSportsFactory(&quot;adidas&quot;)
    nikeFactory, _ := GetSportsFactory(&quot;nike&quot;)

    nikeShoe := nikeFactory.makeShoe()
    nikeShirt := nikeFactory.makeShirt()

    adidasShoe := adidasFactory.makeShoe()
    adidasShirt := adidasFactory.makeShirt()

    printShoeDetails(nikeShoe)
    printShirtDetails(nikeShirt)

    printShoeDetails(adidasShoe)
    printShirtDetails(adidasShirt)
}

func printShoeDetails(s IShoe) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}

func printShirtDetails(s IShirt) {
    fmt.Printf(&quot;Logo: %s&quot;, s.getLogo())
    fmt.Println()
    fmt.Printf(&quot;Size: %d&quot;, s.getSize())
    fmt.Println()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Logo: nike
Size: 14
Logo: nike
Size: 14
Logo: adidas
Size: 14
Logo: adidas
Size: 14</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 빌더 []{.fa
.fa-arrow-right}](../../builder/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된 디자인 패턴들](../../go.html){.btn
.btn-default rel="prev"}
:::

## 다른 언어로 작성된 **추상 팩토리**

[![C#으로 작성된 추상
팩토리](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 추상 팩토리"){.prog-lang-link}
[![C++로 작성된 추상
팩토리](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 추상 팩토리"){.prog-lang-link}
[![자바로 작성된 추상
팩토리](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 추상 팩토리"){.prog-lang-link}
[![PHP로 작성된 추상
팩토리](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 추상 팩토리"){.prog-lang-link}
[![파이썬으로 작성된 추상
팩토리](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 추상 팩토리"){.prog-lang-link}
[![루비로 작성된 추상
팩토리](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 추상 팩토리"){.prog-lang-link}
[![러스트로 작성된 추상
팩토리](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 추상 팩토리"){.prog-lang-link}
[![스위프트로 작성된 추상
팩토리](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 추상 팩토리"){.prog-lang-link}
[![타입스크립트로 작성된 추상
팩토리](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 추상 팩토리"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
