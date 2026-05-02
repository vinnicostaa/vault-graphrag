:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** em Java {#bridge-em-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
O **Bridge** é um padrão de projeto estrutural que divide a lógica de
negócio ou uma enorme classe em hierarquias de classe separadas que
podem ser desenvolvidas independentemente.

Uma dessas hierarquias (geralmente chamada de Abstração) obterá uma
referência a um objeto da segunda hierarquia (Implementação). A
abstração poderá delegar algumas (às vezes, a maioria) de suas chamadas
para o objeto de implementações. Como todas as implementações terão uma
interface comum, elas seriam intercambiáveis dentro da abstração.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Bridge ](../../bridge.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Bridge entre dispositivos e controles remotos](#example-0)
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

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Bridge é especialmente útil ao lidar com
aplicações de multi plataforma, oferecer suporte a vários tipos de
servidores de banco de dados, ou ao trabalhar com vários provedores de
API de um determinado tipo (por exemplo, plataformas em nuvem, redes
sociais etc.)

**Identificação:** O Bridge pode ser reconhecido por uma distinção clara
entre alguma entidade controladora e várias plataformas diferentes nas
quais ela se baseia.
:::::::::::::::::::::::::

[]{#example-0}

## Bridge entre dispositivos e controles remotos {#example-0-title}

Este exemplo mostra a separação entre as classes de controles remotos e
dispositivos que eles controlam.

Os controles remotos atuam como abstrações e os dispositivos são suas
implementações. Graças às interfaces comuns, os mesmos controles remotos
podem funcionar com dispositivos diferentes e vice-versa.

O padrão Bridge permite alterar ou mesmo criar novas classes sem tocar
no código da hierarquia oposta.

### []{#example-0--devices .anchor} **devices** {#devices .example-title}

#### []{#example-0--devices-Device-java .anchor} **devices/Device.java:** Interface comum para todos os dispositivos

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

#### []{#example-0--devices-Radio-java .anchor} **devices/Radio.java:** Rádio

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

#### []{#example-0--remotes-Remote-java .anchor} **remotes/Remote.java:** Interface comum para todos os controles remotos

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

#### []{#example-0--remotes-BasicRemote-java .anchor} **remotes/BasicRemote.java:** Controle remoto básico

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

#### []{#example-0--remotes-AdvancedRemote-java .anchor} **remotes/AdvancedRemote.java:** Controle remoto avançado

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código cliente

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** Resultados da execução

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
#### Leia a seguir

[Composite em Java []{.fa
.fa-arrow-right}](../../composite/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Adapter em
Java](../../adapter/java/example.html){.btn .btn-default rel="prev"}
:::

## **Bridge** em outras linguagens

[![Bridge em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge em C#"){.prog-lang-link}
[![Bridge em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge em C++"){.prog-lang-link}
[![Bridge em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge em Go"){.prog-lang-link}
[![Bridge em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge em PHP"){.prog-lang-link}
[![Bridge em
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Bridge em Python"){.prog-lang-link}
[![Bridge em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge em Ruby"){.prog-lang-link}
[![Bridge em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge em Rust"){.prog-lang-link}
[![Bridge em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge em Swift"){.prog-lang-link}
[![Bridge em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge em TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
