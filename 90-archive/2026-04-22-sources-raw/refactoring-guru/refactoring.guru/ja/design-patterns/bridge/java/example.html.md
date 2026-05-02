:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** を Java で {#bridge-を-java-で .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Bridge** は[、]{.chpule2}[
]{.chpuri2}構造に関するデザインパターンの一つで[、]{.chpule2}[
]{.chpuri2}ビジネス・ロジックや巨大なクラスを独立して開発可能なクラス階層に分割します[。]{.chpule2}[
]{.chpuri2}

階層の一つ[ ]{.chpule1}[（]{.chpuri1}抽象化と呼ばれる[）]{.chpule2}[
]{.chpuri2}は[、]{.chpule2}[ ]{.chpuri2}二つ目の階層[
]{.chpule1}[（]{.chpuri1}実装[）]{.chpule2}[
]{.chpuri2}のオブジェクトへの参照を持ちます[。]{.chpule2}[
]{.chpuri2}抽象化階層は[、]{.chpule2}[
]{.chpuri2}その呼び出しのいくつか[
]{.chpule1}[（]{.chpuri1}場合によっては大多数[）]{.chpule2}[
]{.chpuri2}を実装階層のオブジェクトに委任します[。]{.chpule2}[
]{.chpuri2}すべての実装は[、]{.chpule2}[
]{.chpuri2}共通のインターフェースを持っているので[、]{.chpule2}[
]{.chpuri2}抽象化の中で入れ替え可能です[。]{.chpule2}[ ]{.chpuri2}
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Bridge の詳細 ](../../bridge.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [デバイスとリモート・コントロールの間のブリッジ](#example-0)
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

**複雑度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**人気度[：]{.chpule2}[ ]{.chpuri2}**[]{.fa-stars}

**使用例[：]{.chpule2}[ ]{.chpuri2}** Bridge パターンは[、]{.chpule2}[
]{.chpuri2}クロス・プラットフォーム・アプリを扱う時[、]{.chpule2}[
]{.chpuri2}複数の種類のデータベース・サーバーをサポートする時[、]{.chpule2}[
]{.chpuri2}あるいはある種の API プロバイダー[
]{.chpule1}[（]{.chpuri1}クラウド・プラットフォーム[、]{.chpule2}[
]{.chpuri2}ソーシャル・ネットワークなど[）]{.chpule2}[
]{.chpuri2}を複数利用したい場合に特に便利です[。]{.chpule2}[ ]{.chpuri2}

**見つけ方[：]{.chpule2}[ ]{.chpuri2}** Bridge は[、]{.chpule2}[
]{.chpuri2}制御するものと[、]{.chpule2}[
]{.chpuri2}それが依存するいくつかの異なるプラットフォームとが明らかに分かれていることから識別できます[。]{.chpule2}[
]{.chpuri2}
:::::::::::::::::::::::::

[]{#example-0}

## デバイスとリモート・コントロールの間のブリッジ {#example-0-title}

この例では[、]{.chpule2}[
]{.chpuri2}複数のリモコンのクラスとそれらが制御するデバイスの間の分離を示します[。]{.chpule2}[
]{.chpuri2}

リモコン[ ]{.chpule1}[（]{.chpuri1}コード例では[、]{.chpule2}[
]{.chpuri2}Remote [）]{.chpule2}[ ]{.chpuri2}は[、]{.chpule2}[
]{.chpuri2}抽象化階層で[、]{.chpule2}[
]{.chpuri2}デバイスは実装階層です[。]{.chpule2}[
]{.chpuri2}共通インターフェースのおかげで[、]{.chpule2}[
]{.chpuri2}同じリモコンは異なるデバイスを操作でき[、]{.chpule2}[
]{.chpuri2}逆も同様です[。]{.chpule2}[ ]{.chpuri2}

Bridge パターンに従うと[、]{.chpule2}[
]{.chpuri2}他方の層のコードを変更することなく[、]{.chpule2}[
]{.chpuri2}クラスの変更や新規作成ができます[。]{.chpule2}[ ]{.chpuri2}

### []{#example-0--devices .anchor} **devices** {#devices .example-title}

#### []{#example-0--devices-Device-java .anchor} **devices/Device.java:** すべてのデバイスに共通なインターフェース

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

#### []{#example-0--devices-Radio-java .anchor} **devices/Radio.java:** ラジオ

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

#### []{#example-0--devices-Tv-java .anchor} **devices/Tv.java:** テレビ

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

#### []{#example-0--remotes-Remote-java .anchor} **remotes/Remote.java:** すべてのリモコンに共通なインターフェース

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

#### []{#example-0--remotes-BasicRemote-java .anchor} **remotes/BasicRemote.java:** 基本的なリモコン

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

#### []{#example-0--remotes-AdvancedRemote-java .anchor} **remotes/AdvancedRemote.java:** 高級リモコン

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** クライアント・コード

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

#### []{#example-0--OutputDemo-txt .anchor} **OutputDemo.txt:** 実行結果

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
#### 次を読む

[Composite を Java で []{.fa
.fa-arrow-right}](../../composite/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Adapter を Java
で](../../adapter/java/example.html){.btn .btn-default rel="prev"}
:::

## 他言語での **Bridge**

[![Bridge を C#
で](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Bridge を C# で"){.prog-lang-link}
[![Bridge を C++
で](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Bridge を C++ で"){.prog-lang-link}
[![Bridge を Go
で](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Bridge を Go で"){.prog-lang-link}
[![Bridge を PHP
で](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Bridge を PHP で"){.prog-lang-link}
[![Bridge を Python
で](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Bridge を Python で"){.prog-lang-link}
[![Bridge を Ruby
で](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Bridge を Ruby で"){.prog-lang-link}
[![Bridge を Rust
で](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Bridge を Rust で"){.prog-lang-link}
[![Bridge を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge を Swift で"){.prog-lang-link}
[![Bridge を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge を TypeScript で"){.prog-lang-link}
:::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ja" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ コード例の書庫](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
**直撃！デザインパターン**を電子書籍で購入し、IDE
で開くことができる数多くの詳細な例を含む書庫にアクセス。

[ 詳細](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
