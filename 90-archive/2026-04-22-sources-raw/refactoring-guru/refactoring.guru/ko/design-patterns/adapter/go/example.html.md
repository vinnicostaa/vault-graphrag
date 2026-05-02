::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[어댑터](../../adapter.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![어댑터](../../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# Go로 작성된 **어댑터** {#go로-작성된-어댑터 .pattern-example-title-block-title}
::::

:::::::::::::::: pattern-example-body
::: pattern-example-brief
**어댑터**는 구조 디자인 패턴이며, 호환되지 않는 객체들이 협업할 수
있도록 합니다.

어댑터는 두 객체 사이의 래퍼 역할을 합니다. 하나의 객체에 대한 호출을
캐치하고 두 번째 객체가 인식할 수 있는 형식과 인터페이스로 변환합니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 어댑터에 대하여 더 자세히 알아보세요 ](../../adapter.html){.btn
.btn-lg .btn-primary}
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
 [client](#example-0--client-go)
:::

::: en-file
 [computer](#example-0--computer-go)
:::

::: en-file
 [mac](#example-0--mac-go)
:::

::: en-file
 [windows](#example-0--windows-go)
:::

::: en-file
 [windows­Adapter](#example-0--windowsAdapter-go)
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

어떤 객체의 일부 기능들​(예: 라이트닝 포트)​을 기대하는 클라이언트가
있습니다. 그러나 또 *adaptee*(윈도우 랩톱)​라는 객체가 있습니다. 이
객체는 같은 기능성을 다른 인터페이스​(USB 포트)​를 통해 제공합니다.

어댑터 패턴은 이러한 상황에 사용할 수 있습니다. 이제 다음을 수행하는
*[어]{.cjk}[댑]{.cjk}[터]{.cjk}*라는 구조체 유형을 만듭니다:

-   클라이언트가 기대하는 것과 같은 인터페이스​(라이트닝 포트)​를 준수.

-   클라이언트의 요청을 adaptee가 예상하는 형식으로 변환합니다. 어댑터는
    라이트닝 커넥터를 받아들인 다음 신호를 USB 형식으로 변환하고 윈도우
    랩톱의 USB 포트로 전달합니다.

#### []{#example-0--client-go .anchor} **client.go:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Client struct {
}

func (c *Client) InsertLightningConnectorIntoComputer(com Computer) {
    fmt.Println(&quot;Client inserts Lightning connector into computer.&quot;)
    com.InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--computer-go .anchor} **computer.go:** 클라이언트 인터페이스

<figure class="code">
<pre class="code" lang="go"><code>package main

type Computer interface {
    InsertIntoLightningPort()
}</code></pre>
</figure>

#### []{#example-0--mac-go .anchor} **mac.go:** 서비스

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Mac struct {
}

func (m *Mac) InsertIntoLightningPort() {
    fmt.Println(&quot;Lightning connector is plugged into mac machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windows-go .anchor} **windows.go:** 알려지지 않은 서비스

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Windows struct{}

func (w *Windows) insertIntoUSBPort() {
    fmt.Println(&quot;USB connector is plugged into windows machine.&quot;)
}</code></pre>
</figure>

#### []{#example-0--windowsAdapter-go .anchor} **windowsAdapter.go:** 어댑터

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type WindowsAdapter struct {
    windowMachine *Windows
}

func (w *WindowsAdapter) InsertIntoLightningPort() {
    fmt.Println(&quot;Adapter converts Lightning signal to USB.&quot;)
    w.windowMachine.insertIntoUSBPort()
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go**

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    client := &amp;Client{}
    mac := &amp;Mac{}

    client.InsertLightningConnectorIntoComputer(mac)

    windowsMachine := &amp;Windows{}
    windowsMachineAdapter := &amp;WindowsAdapter{
        windowMachine: windowsMachine,
    }

    client.InsertLightningConnectorIntoComputer(windowsMachineAdapter)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Client inserts Lightning connector into computer.
Lightning connector is plugged into mac machine.
Client inserts Lightning connector into computer.
Adapter converts Lightning signal to USB.
USB connector is plugged into windows machine.</code></pre>
</figure>

::: next
#### 다음을 보세요

[Go로 작성된 브리지 []{.fa
.fa-arrow-right}](../../bridge/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Go로 작성된
싱글턴](../../singleton/go/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **어댑터**

[![C#으로 작성된
어댑터](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 어댑터"){.prog-lang-link}
[![C++로 작성된
어댑터](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 어댑터"){.prog-lang-link}
[![자바로 작성된
어댑터](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 어댑터"){.prog-lang-link}
[![PHP로 작성된
어댑터](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 어댑터"){.prog-lang-link}
[![파이썬으로 작성된
어댑터](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 어댑터"){.prog-lang-link}
[![루비로 작성된
어댑터](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 어댑터"){.prog-lang-link}
[![러스트로 작성된
어댑터](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 어댑터"){.prog-lang-link}
[![스위프트로 작성된
어댑터](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 어댑터"){.prog-lang-link}
[![타입스크립트로 작성된
어댑터](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 어댑터"){.prog-lang-link}
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
