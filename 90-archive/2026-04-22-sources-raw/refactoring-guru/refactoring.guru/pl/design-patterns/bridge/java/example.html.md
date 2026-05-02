:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Most](../../bridge.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Most](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Most** w języku Java {#most-w-języku-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Most** jest strukturalnym wzorcem projektowym zakładającym podział
logiki biznesowej lub dużej klasy na osobne hierarchie klas które
następnie można rozwijać niezależnie od siebie.

Jedna z takich hierarchii (zwana często Abstrakcją) posiada
referencję do obiektu drugiej hierarchii (zwanej Implementacją) i
deleguje mu część (czasem większość) wywołań. Ponieważ wszystkie
implementacje mają wspólny interfejs, z punktu widzenia abstrakcji są
wymienialne.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Most ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Most łączący urządzenia z pilotami zdalnego sterowania](#example-0)
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

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Most jest szczególnie użyteczny gdy ma
się do czynienia z aplikacjami wieloplatformowymi, korzystającymi z
wielu serwerów baz danych lub z różnych interfejsów programowania
aplikacji danego typu (na przykład chmura, platforma społecznościowa,
itp.)

**Identyfikacja:** Most można rozpoznać po wyraźnym rozdzieleniu na
część kontrolującą i wiele różnych platform od których ta część zależy.
:::::::::::::::::::::::::

[]{#example-0}

## Most łączący urządzenia z pilotami zdalnego sterowania {#example-0-title}

Poniższy przykład ukazuje separację pomiędzy klasami pilotami zdalnego
sterowania i urządzeniami jakie można za ich pomocą kontrolować.

Piloty służą za abstrakcje, a urządzenia są ich implementacjami. Dzięki
wspólnemu interfejsowi te same piloty mogą sterować wieloma różnymi
urządzeniami i odwrotnie.

Wzorzec Most pozwala zmieniać, a nawet tworzyć nowe klasy bez
konieczności modyfikacji kodu drugiej hierarchii.

### []{#example-0--devices .anchor} **devices** {#devices .example-title}

#### []{#example-0--devices-Device-java .anchor} **devices/Device.java:** Wspólny interfejs wszystkich urządzeń

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

#### []{#example-0--devices-Radio-java .anchor} **devices/Radio.java:** Radio

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

#### []{#example-0--devices-Tv-java .anchor} **devices/Tv.java:** TV

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

#### []{#example-0--remotes-Remote-java .anchor} **remotes/Remote.java:** Wspólny interfejs wszystkich pilotów

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

#### []{#example-0--remotes-BasicRemote-java .anchor} **remotes/BasicRemote.java:** Podstawowy pilot zdalnego sterowania

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

#### []{#example-0--remotes-AdvancedRemote-java .anchor} **remotes/AdvancedRemote.java:** Zaawansowany pilot zdalnego sterowania

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Kod klienta

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Wynik działania

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
#### Czytaj dalej

[Kompozyt w języku Java []{.fa
.fa-arrow-right}](../../composite/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Adapter w języku
Java](../../adapter/java/example.html){.btn .btn-default rel="prev"}
:::

## **Most** w innych językach

[![Most w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Most w języku C#"){.prog-lang-link}
[![Most w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Most w języku C++"){.prog-lang-link}
[![Most w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Most w języku Go"){.prog-lang-link}
[![Most w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Most w języku PHP"){.prog-lang-link}
[![Most w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Most w języku Python"){.prog-lang-link}
[![Most w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Most w języku Ruby"){.prog-lang-link}
[![Most w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Most w języku Rust"){.prog-lang-link}
[![Most w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Most w języku Swift"){.prog-lang-link}
[![Most w języku
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Most w języku TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
