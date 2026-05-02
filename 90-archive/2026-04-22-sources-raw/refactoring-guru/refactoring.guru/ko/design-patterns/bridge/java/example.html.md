:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} /
[브리지](../../bridge.html){.type} / [자바](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![브리지](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# 자바로 작성된 **브리지** {#자바로-작성된-브리지 .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**브리지**는 구조 디자인 패턴입니다. 이 패턴은 비즈니스 로직 또는 거대한
클래스를 독립적으로 개발할 수 있는 별도의 클래스 계층구조들로 나눕니다.

종종 추상화라고 불리는 이러한 계층구조 중 하나는 두 번째 계층구조의
객체​(구현)​에 대한 참조를 얻습니다. 추상화의 호출들 일부​(때로는 대부분)​를
구현 객체에 위임할 수 있습니다. 모든 구현은 공통 인터페이스를 가지므로
추상화 내에서 상호 교환할 수 있습니다.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 브리지에 대하여 더 자세히 알아보세요 ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [장치와 리모컨 사이의 브리지](#example-0)
:::

::: en-file
 devices
:::

:::: en-inside
::: en-file
  [Device](#example-0--devices-Device-java)
:::
::::

:::: en-inside
::: en-file
  [Radio](#example-0--devices-Radio-java)
:::
::::

:::: en-inside
::: en-file
  [Tv](#example-0--devices-Tv-java)
:::
::::

::: en-file
 remotes
:::

:::: en-inside
::: en-file
  [Remote](#example-0--remotes-Remote-java)
:::
::::

:::: en-inside
::: en-file
  [Basic­Remote](#example-0--remotes-BasicRemote-java)
:::
::::

:::: en-inside
::: en-file
  [Advanced­Remote](#example-0--remotes-AdvancedRemote-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-txt)
:::
::::::::::::::::::::::

**복잡도:** []{.fa-stars}

**인기도:** []{.fa-stars}

**사용 예시들:** 브리지 패턴은 크로스 플랫폼 앱들을 처리할 때, 여러
유형의 데이터베이스 서버를 지원할 때 또는 특정 종류의 여러 API
제공자​(예: 클라우드 플랫폼, 소셜 네트워크 등)​와 작업할 때 특히
유용합니다.

**식별법:** 브리지는 일부 제어 개체가 해당 개체가 의존하는 여러 다른
플랫폼들과 명확하게 구분됩니다.
:::::::::::::::::::::::::

[]{#example-0}

## 장치와 리모컨 사이의 브리지 {#example-0-title}

이 예시는 리모컨 클래스들과 그들이 조절하는 장치들 사이의 분리를
보여줍니다.

리모컨들은 추상화로 작동하고, 장치들은 그들의 구현으로 작동합니다. 공통
인터페이스 덕분에 같은 리모컨들은 다른 장치들과 작동할 수 있고 그 반대의
경우도 가능합니다.

브리지 패턴은 반대 계층구조의 코드를 건드리지 않고도 클래스들을
변경하거나 새로운 클래스들을 생성할 수 있도록 합니다.

### []{#example-0--devices .anchor} **devices** {#devices .example-title}

#### []{#example-0--devices-Device-java .anchor} **devices/Device.java:** 모든 장치에 대한 공통 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.devices;

public interface Device {
    boolean isEnabled();

    void enable();

    void disable();

    int getVolume();

    void setVolume(int percent);

    int getChannel();

    void setChannel(int channel);

    void printStatus();
}</code></pre>
</figure>

#### []{#example-0--devices-Radio-java .anchor} **devices/Radio.java:** 라디오

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.devices;

public class Radio implements Device {
    private boolean on = false;
    private int volume = 30;
    private int channel = 1;

    @Override
    public boolean isEnabled() {
        return on;
    }

    @Override
    public void enable() {
        on = true;
    }

    @Override
    public void disable() {
        on = false;
    }

    @Override
    public int getVolume() {
        return volume;
    }

    @Override
    public void setVolume(int volume) {
        if (volume &gt; 100) {
            this.volume = 100;
        } else if (volume &lt; 0) {
            this.volume = 0;
        } else {
            this.volume = volume;
        }
    }

    @Override
    public int getChannel() {
        return channel;
    }

    @Override
    public void setChannel(int channel) {
        this.channel = channel;
    }

    @Override
    public void printStatus() {
        System.out.println(&quot;------------------------------------&quot;);
        System.out.println(&quot;| I&#39;m radio.&quot;);
        System.out.println(&quot;| I&#39;m &quot; + (on ? &quot;enabled&quot; : &quot;disabled&quot;));
        System.out.println(&quot;| Current volume is &quot; + volume + &quot;%&quot;);
        System.out.println(&quot;| Current channel is &quot; + channel);
        System.out.println(&quot;------------------------------------\n&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--devices-Tv-java .anchor} **devices/Tv.java:** 텔레비전

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.devices;

public class Tv implements Device {
    private boolean on = false;
    private int volume = 30;
    private int channel = 1;

    @Override
    public boolean isEnabled() {
        return on;
    }

    @Override
    public void enable() {
        on = true;
    }

    @Override
    public void disable() {
        on = false;
    }

    @Override
    public int getVolume() {
        return volume;
    }

    @Override
    public void setVolume(int volume) {
        if (volume &gt; 100) {
            this.volume = 100;
        } else if (volume &lt; 0) {
            this.volume = 0;
        } else {
            this.volume = volume;
        }
    }

    @Override
    public int getChannel() {
        return channel;
    }

    @Override
    public void setChannel(int channel) {
        this.channel = channel;
    }

    @Override
    public void printStatus() {
        System.out.println(&quot;------------------------------------&quot;);
        System.out.println(&quot;| I&#39;m TV set.&quot;);
        System.out.println(&quot;| I&#39;m &quot; + (on ? &quot;enabled&quot; : &quot;disabled&quot;));
        System.out.println(&quot;| Current volume is &quot; + volume + &quot;%&quot;);
        System.out.println(&quot;| Current channel is &quot; + channel);
        System.out.println(&quot;------------------------------------\n&quot;);
    }
}</code></pre>
</figure>

### []{#example-0--remotes .anchor} **remotes** {#remotes .example-title}

#### []{#example-0--remotes-Remote-java .anchor} **remotes/Remote.java:** 모든 리모컨에 대한 공통 인터페이스

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.remotes;

public interface Remote {
    void power();

    void volumeDown();

    void volumeUp();

    void channelDown();

    void channelUp();
}</code></pre>
</figure>

#### []{#example-0--remotes-BasicRemote-java .anchor} **remotes/BasicRemote.java:** 기초 리모콘

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.remotes;

import refactoring_guru.bridge.example.devices.Device;

public class BasicRemote implements Remote {
    protected Device device;

    public BasicRemote() {}

    public BasicRemote(Device device) {
        this.device = device;
    }

    @Override
    public void power() {
        System.out.println(&quot;Remote: power toggle&quot;);
        if (device.isEnabled()) {
            device.disable();
        } else {
            device.enable();
        }
    }

    @Override
    public void volumeDown() {
        System.out.println(&quot;Remote: volume down&quot;);
        device.setVolume(device.getVolume() - 10);
    }

    @Override
    public void volumeUp() {
        System.out.println(&quot;Remote: volume up&quot;);
        device.setVolume(device.getVolume() + 10);
    }

    @Override
    public void channelDown() {
        System.out.println(&quot;Remote: channel down&quot;);
        device.setChannel(device.getChannel() - 1);
    }

    @Override
    public void channelUp() {
        System.out.println(&quot;Remote: channel up&quot;);
        device.setChannel(device.getChannel() + 1);
    }
}</code></pre>
</figure>

#### []{#example-0--remotes-AdvancedRemote-java .anchor} **remotes/AdvancedRemote.java:** 고급 리모콘

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example.remotes;

import refactoring_guru.bridge.example.devices.Device;

public class AdvancedRemote extends BasicRemote {

    public AdvancedRemote(Device device) {
        super.device = device;
    }

    public void mute() {
        System.out.println(&quot;Remote: mute&quot;);
        device.setVolume(0);
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** 클라이언트 코드

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.bridge.example;

import refactoring_guru.bridge.example.devices.Device;
import refactoring_guru.bridge.example.devices.Radio;
import refactoring_guru.bridge.example.devices.Tv;
import refactoring_guru.bridge.example.remotes.AdvancedRemote;
import refactoring_guru.bridge.example.remotes.BasicRemote;

public class Demo {
    public static void main(String[] args) {
        testDevice(new Tv());
        testDevice(new Radio());
    }

    public static void testDevice(Device device) {
        System.out.println(&quot;Tests with basic remote.&quot;);
        BasicRemote basicRemote = new BasicRemote(device);
        basicRemote.power();
        device.printStatus();

        System.out.println(&quot;Tests with advanced remote.&quot;);
        AdvancedRemote advancedRemote = new AdvancedRemote(device);
        advancedRemote.power();
        advancedRemote.mute();
        device.printStatus();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 실행 결과

<figure class="code">
<pre class="code" lang="output"><code>Tests with basic remote.
Remote: power toggle
------------------------------------
| I&#39;m TV set.
| I&#39;m enabled
| Current volume is 30%
| Current channel is 1
------------------------------------

Tests with advanced remote.
Remote: power toggle
Remote: mute
------------------------------------
| I&#39;m TV set.
| I&#39;m disabled
| Current volume is 0%
| Current channel is 1
------------------------------------

Tests with basic remote.
Remote: power toggle
------------------------------------
| I&#39;m radio.
| I&#39;m enabled
| Current volume is 30%
| Current channel is 1
------------------------------------

Tests with advanced remote.
Remote: power toggle
Remote: mute
------------------------------------
| I&#39;m radio.
| I&#39;m disabled
| Current volume is 0%
| Current channel is 1
------------------------------------</code></pre>
</figure>

::: next
#### 다음을 보세요

[자바로 작성된 복합체 []{.fa
.fa-arrow-right}](../../composite/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 자바로 작성된
어댑터](../../adapter/java/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **브리지**

[![C#으로 작성된
브리지](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 브리지"){.prog-lang-link}
[![C++로 작성된
브리지](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 브리지"){.prog-lang-link}
[![Go로 작성된
브리지](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 브리지"){.prog-lang-link}
[![PHP로 작성된
브리지](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 브리지"){.prog-lang-link}
[![파이썬으로 작성된
브리지](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 브리지"){.prog-lang-link}
[![루비로 작성된
브리지](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 브리지"){.prog-lang-link}
[![러스트로 작성된
브리지](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "러스트로 작성된 브리지"){.prog-lang-link}
[![스위프트로 작성된
브리지](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 브리지"){.prog-lang-link}
[![타입스크립트로 작성된
브리지](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 브리지"){.prog-lang-link}
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
