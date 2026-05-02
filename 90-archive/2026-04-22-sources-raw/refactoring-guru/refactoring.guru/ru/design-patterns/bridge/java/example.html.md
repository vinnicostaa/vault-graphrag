:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Мост](../../bridge.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Мост](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Мост** на Java {#мост-на-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Мост** --- это структурный паттерн, который разделяет бизнес-логику
или большой класс на несколько отдельных иерархий, которые потом можно
развивать отдельно друг от друга.

Одна из этих иерархий (абстракция) получит ссылку на объекты другой
иерархии (реализация) и будет делегировать им основную работу. Благодаря
тому, что все реализации будут следовать общему интерфейсу, их можно
будет взаимозаменять внутри абстракции.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Мост ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Мост между приборами и пультами дистанционного управления](#example-0)
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

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн Мост особенно полезен когда вам приходится
делать кросс-платформенные приложения, поддерживать несколько типов баз
данных или работать с разными поставщиками похожего API (например,
cloud-сервисы, социальные сети и т. д.)

**Признаки применения паттерна:** Если в программе чётко выделены классы
«управления» и несколько видов классов «платформ», причём управляющие
объекты делегируют выполнение платформам, то можно сказать, что у вас
используется Мост.
:::::::::::::::::::::::::

[]{#example-0}

## Мост между приборами и пультами дистанционного управления {#example-0-title}

Этот пример показывает разделение кода пультов дистанционного управления
и бытовых приборов.

Пульты выступают «абстракцией», а приборы --- «реализацией». Одни и те
же приборы могут работать с разными пультами, а пульты управлять разными
устройствами.

Применив паттерн Мост, мы можем изменять классы пультов и приборов
независимо друг от друга.

### []{#example-0--devices .anchor} **devices** {#devices .example-title}

#### []{#example-0--devices-Device-java .anchor} **devices/Device.java:** Общий интерфейс всех устройств

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

#### []{#example-0--devices-Radio-java .anchor} **devices/Radio.java:** Радио

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

#### []{#example-0--devices-Tv-java .anchor} **devices/Tv.java:** ТВ

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

#### []{#example-0--remotes-Remote-java .anchor} **remotes/Remote.java:** Общий интерфейс всех пультов ДУ

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

#### []{#example-0--remotes-BasicRemote-java .anchor} **remotes/BasicRemote.java:** Стандартный пульт

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

#### []{#example-0--remotes-AdvancedRemote-java .anchor} **remotes/AdvancedRemote.java:** Улучшенный пульт

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Клиентский код

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Результат выполнения

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
#### Читаем дальше

[Компоновщик на Java []{.fa
.fa-arrow-right}](../../composite/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Адаптер на
Java](../../adapter/java/example.html){.btn .btn-default rel="prev"}
:::

## **Мост** на других языках программирования

[![Мост на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Мост на C#"){.prog-lang-link}
[![Мост на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Мост на C++"){.prog-lang-link}
[![Мост на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Мост на Go"){.prog-lang-link}
[![Мост на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Мост на PHP"){.prog-lang-link}
[![Мост на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Мост на Python"){.prog-lang-link}
[![Мост на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Мост на Ruby"){.prog-lang-link}
[![Мост на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Мост на Rust"){.prog-lang-link}
[![Мост на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Мост на Swift"){.prog-lang-link}
[![Мост на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Мост на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
