:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[커맨드](../../command.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![커맨드](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# Go로 작성된 **커맨드** {#go로-작성된-커맨드 .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**커맨드**는 요청 또는 간단한 작업을 객체로 변환하는 행동 디자인
패턴입니다.

이러한 변환은 명령의 지연 또는 원격 실행, 명령 기록 저장 등을
허용합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 커맨드에 대하여 더 자세히 알아보세요 ](../../command.html){.btn
.btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [button](#example-0--button-go)
:::

::: en-file
 [command](#example-0--command-go)
:::

::: en-file
 [on­Command](#example-0--onCommand-go)
:::

::: en-file
 [off­Command](#example-0--offCommand-go)
:::

::: en-file
 [device](#example-0--device-go)
:::

::: en-file
 [tv](#example-0--tv-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## 개념적인 예시 {#example-0-title}

TV를 예시로 든 커맨드 패턴을 살펴봅시다. 다음 중 하나를 통해 TV를 켤 수
있습니다:

-   리모컨의 ON버튼
-   실제 TV의 ON 버튼.

일단 TV를 수신기로 사용하여 ON 커맨드 객체를 구현할 수 있습니다. 이
커맨드에 실행 메서드가 호출되면 해당 커맨드는 `TV.on` 함수를 호출합니다.
마지막으로 호출자를 정의해야 합니다. 우리는 실제로 두 호출자가 있을
것입니다: 리모컨과 실제 TV. 두 커맨드 다 ON 커맨드 객체를 포함할
것입니다.

같은 요청을 여러 호출자로 래핑한 점을 주목해 주세요. 다른 커맨드들을
같은 방법으로 래핑할 수 있습니다. 별도의 커맨드 객체를 만드는 이점은
기본 비즈니스 논리에서 사용자 인터페이스 논리를 분리한다는 점이며 또 각
호출자에 대해 다른 핸들러를 개발할 필요가 없습니다. 커맨드 객체에는
실행에 필요한 모든 정보가 포함되어 있습니다. 따라서 해당 객체는 지연된
실행에도 사용될 수 있습니다.

#### []{#example-0--button-go .anchor} **button.go:** 호출자

<figure class="code">
<pre class="code" lang="go"><code>package main

type Button struct {
    command Command
}

func (b *Button) press() {
    b.command.execute()
}</code></pre>
</figure>

#### []{#example-0--command-go .anchor} **command.go:** 커맨드 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Command interface {
    execute()
}</code></pre>
</figure>

#### []{#example-0--onCommand-go .anchor} **onCommand.go:** 구상 커맨드

<figure class="code">
<pre class="code" lang="go"><code>package main

type OnCommand struct {
    device Device
}

func (c *OnCommand) execute() {
    c.device.on()
}</code></pre>
</figure>

#### []{#example-0--offCommand-go .anchor} **offCommand.go:** 구상 커맨드

<figure class="code">
<pre class="code" lang="go"><code>package main

type OffCommand struct {
    device Device
}

func (c *OffCommand) execute() {
    c.device.off()
}</code></pre>
</figure>

#### []{#example-0--device-go .anchor} **device.go:** 수신자 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Device interface {
    on()
    off()
}</code></pre>
</figure>

#### []{#example-0--tv-go .anchor} **tv.go:** 구상 수신자

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Tv struct {
    isRunning bool
}

func (t *Tv) on() {
    t.isRunning = true
    fmt.Println(&quot;Turning tv on&quot;)
}

func (t *Tv) off() {
    t.isRunning = false
    fmt.Println(&quot;Turning tv off&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    tv := &amp;Tv{}

    onCommand := &amp;OnCommand{
        device: tv,
    }

    offCommand := &amp;OffCommand{
        device: tv,
    }

    onButton := &amp;Button{
        command: onCommand,
    }
    onButton.press()

    offButton := &amp;Button{
        command: offCommand,
    }
    offButton.press()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Turning tv on
Turning tv off</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 반복자 []{.fa
.fa-arrow-right}](../../iterator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된 책임
연쇄](../../chain-of-responsibility/go/example.html){.btn .btn-default
rel="prev"}
:::

## 다른 언어로 작성된 **커맨드**

[![C#으로 작성된
커맨드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 커맨드"){.prog-lang-link}
[![C++로 작성된
커맨드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 커맨드"){.prog-lang-link}
[![자바로 작성된
커맨드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 커맨드"){.prog-lang-link}
[![PHP로 작성된
커맨드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 커맨드"){.prog-lang-link}
[![파이썬으로 작성된
커맨드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 커맨드"){.prog-lang-link}
[![루비로 작성된
커맨드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 커맨드"){.prog-lang-link}
[![러스트로 작성된
커맨드](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 커맨드"){.prog-lang-link}
[![스위프트로 작성된
커맨드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 커맨드"){.prog-lang-link}
[![타입스크립트로 작성된
커맨드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 커맨드"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
