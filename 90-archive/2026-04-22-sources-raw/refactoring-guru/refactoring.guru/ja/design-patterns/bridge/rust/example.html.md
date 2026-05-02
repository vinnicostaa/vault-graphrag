::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} /
[デザインパターン](../../../design-patterns.html){.type} /
[Bridge](../../bridge.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Bridge](../../../../images/patterns/cards/bridge-mini9e60.png?id=b389101d8ee8e23ffa1b534c704d0774){srcset="/images/patterns/cards/bridge-mini-2x.png?id=2622384cf623ed150ee9c21a0812dd87 2x"}
:::

# **Bridge** を Rust で {#bridge-を-rust-で .pattern-example-title-block-title}
::::

:::::::::::::::::::::: pattern-example-body
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

::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
ナビゲーション
:::

::: en-intro
 [はじめに](#)
:::

::: en-intro
 [Devices and Remotes](#example-0)
:::

:::: en-inside
::: en-file
  [mod](#example-0--remotes-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [basic](#example-0--remotes-basic-rs)
:::
::::

:::: en-inside
::: en-file
  [advanced](#example-0--remotes-advanced-rs)
:::
::::

:::: en-inside
::: en-file
  [mod](#example-0--device-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [radio](#example-0--device-radio-rs)
:::
::::

:::: en-inside
::: en-file
  [tv](#example-0--device-tv-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::::::::::::
::::::::::::::::::::::

[]{#example-0}

## Devices and Remotes {#example-0-title}

This example illustrates how the Bridge pattern can help divide the
monolithic code of an app that manages devices and their remote
controls. The Device classes act as the implementation, whereas the
Remotes act as the abstraction.

#### []{#example-0--remotes-mod-rs .anchor} **remotes/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod advanced;
mod basic;

pub use advanced::AdvancedRemote;
pub use basic::BasicRemote;

use crate::device::Device;

pub trait HasMutableDevice&lt;D: Device&gt; {
    fn device(&amp;mut self) -&gt; &amp;mut D;
}

pub trait Remote&lt;D: Device&gt;: HasMutableDevice&lt;D&gt; {
    fn power(&amp;mut self) {
        println!(&quot;Remote: power toggle&quot;);
        if self.device().is_enabled() {
            self.device().disable();
        } else {
            self.device().enable();
        }
    }

    fn volume_down(&amp;mut self) {
        println!(&quot;Remote: volume down&quot;);
        let volume = self.device().volume();
        self.device().set_volume(volume - 10);
    }

    fn volume_up(&amp;mut self) {
        println!(&quot;Remote: volume up&quot;);
        let volume = self.device().volume();
        self.device().set_volume(volume + 10);
    }

    fn channel_down(&amp;mut self) {
        println!(&quot;Remote: channel down&quot;);
        let channel = self.device().channel();
        self.device().set_channel(channel - 1);
    }

    fn channel_up(&amp;mut self) {
        println!(&quot;Remote: channel up&quot;);
        let channel = self.device().channel();
        self.device().set_channel(channel + 1);
    }
}</code></pre>
</figure>

#### []{#example-0--remotes-basic-rs .anchor} **remotes/basic.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::device::Device;

use super::{HasMutableDevice, Remote};

pub struct BasicRemote&lt;D: Device&gt; {
    device: D,
}

impl&lt;D: Device&gt; BasicRemote&lt;D&gt; {
    pub fn new(device: D) -&gt; Self {
        Self { device }
    }
}

impl&lt;D: Device&gt; HasMutableDevice&lt;D&gt; for BasicRemote&lt;D&gt; {
    fn device(&amp;mut self) -&gt; &amp;mut D {
        &amp;mut self.device
    }
}

impl&lt;D: Device&gt; Remote&lt;D&gt; for BasicRemote&lt;D&gt; {}</code></pre>
</figure>

#### []{#example-0--remotes-advanced-rs .anchor} **remotes/advanced.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::device::Device;

use super::{HasMutableDevice, Remote};

pub struct AdvancedRemote&lt;D: Device&gt; {
    device: D,
}

impl&lt;D: Device&gt; AdvancedRemote&lt;D&gt; {
    pub fn new(device: D) -&gt; Self {
        Self { device }
    }

    pub fn mute(&amp;mut self) {
        println!(&quot;Remote: mute&quot;);
        self.device.set_volume(0);
    }
}

impl&lt;D: Device&gt; HasMutableDevice&lt;D&gt; for AdvancedRemote&lt;D&gt; {
    fn device(&amp;mut self) -&gt; &amp;mut D {
        &amp;mut self.device
    }
}

impl&lt;D: Device&gt; Remote&lt;D&gt; for AdvancedRemote&lt;D&gt; {}</code></pre>
</figure>

#### []{#example-0--device-mod-rs .anchor} **device/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod radio;
mod tv;

pub use radio::Radio;
pub use tv::Tv;

pub trait Device {
    fn is_enabled(&amp;self) -&gt; bool;
    fn enable(&amp;mut self);
    fn disable(&amp;mut self);
    fn volume(&amp;self) -&gt; u8;
    fn set_volume(&amp;mut self, percent: u8);
    fn channel(&amp;self) -&gt; u16;
    fn set_channel(&amp;mut self, channel: u16);
    fn print_status(&amp;self);
}</code></pre>
</figure>

#### []{#example-0--device-radio-rs .anchor} **device/radio.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Device;

#[derive(Clone)]
pub struct Radio {
    on: bool,
    volume: u8,
    channel: u16,
}

impl Default for Radio {
    fn default() -&gt; Self {
        Self {
            on: false,
            volume: 30,
            channel: 1,
        }
    }
}

impl Device for Radio {
    fn is_enabled(&amp;self) -&gt; bool {
        self.on
    }

    fn enable(&amp;mut self) {
        self.on = true;
    }

    fn disable(&amp;mut self) {
        self.on = false;
    }

    fn volume(&amp;self) -&gt; u8 {
        self.volume
    }

    fn set_volume(&amp;mut self, percent: u8) {
        self.volume = std::cmp::min(percent, 100);
    }

    fn channel(&amp;self) -&gt; u16 {
        self.channel
    }

    fn set_channel(&amp;mut self, channel: u16) {
        self.channel = channel;
    }

    fn print_status(&amp;self) {
        println!(&quot;------------------------------------&quot;);
        println!(&quot;| I&#39;m radio.&quot;);
        println!(&quot;| I&#39;m {}&quot;, if self.on { &quot;enabled&quot; } else { &quot;disabled&quot; });
        println!(&quot;| Current volume is {}%&quot;, self.volume);
        println!(&quot;| Current channel is {}&quot;, self.channel);
        println!(&quot;------------------------------------\n&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--device-tv-rs .anchor} **device/tv.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Device;

#[derive(Clone)]
pub struct Tv {
    on: bool,
    volume: u8,
    channel: u16,
}

impl Default for Tv {
    fn default() -&gt; Self {
        Self {
            on: false,
            volume: 30,
            channel: 1,
        }
    }
}

impl Device for Tv {
    fn is_enabled(&amp;self) -&gt; bool {
        self.on
    }

    fn enable(&amp;mut self) {
        self.on = true;
    }

    fn disable(&amp;mut self) {
        self.on = false;
    }

    fn volume(&amp;self) -&gt; u8 {
        self.volume
    }

    fn set_volume(&amp;mut self, percent: u8) {
        self.volume = std::cmp::min(percent, 100);
    }

    fn channel(&amp;self) -&gt; u16 {
        self.channel
    }

    fn set_channel(&amp;mut self, channel: u16) {
        self.channel = channel;
    }

    fn print_status(&amp;self) {
        println!(&quot;------------------------------------&quot;);
        println!(&quot;| I&#39;m TV set.&quot;);
        println!(&quot;| I&#39;m {}&quot;, if self.on { &quot;enabled&quot; } else { &quot;disabled&quot; });
        println!(&quot;| Current volume is {}%&quot;, self.volume);
        println!(&quot;| Current channel is {}&quot;, self.channel);
        println!(&quot;------------------------------------\n&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod device;
mod remotes;

use device::{Device, Radio, Tv};
use remotes::{AdvancedRemote, BasicRemote, HasMutableDevice, Remote};

fn main() {
    test_device(Tv::default());
    test_device(Radio::default());
}

fn test_device(device: impl Device + Clone) {
    println!(&quot;Tests with basic remote.&quot;);
    let mut basic_remote = BasicRemote::new(device.clone());
    basic_remote.power();
    basic_remote.device().print_status();

    println!(&quot;Tests with advanced remote.&quot;);
    let mut advanced_remote = AdvancedRemote::new(device);
    advanced_remote.power();
    advanced_remote.mute();
    advanced_remote.device().print_status();
}</code></pre>
</figure>

### Output

``` code
Tests with basic remote.
Remote: power toggle
------------------------------------
| I'm TV set.
| I'm enabled
| Current volume is 30%
| Current channel is 1
------------------------------------

Tests with advanced remote.
Remote: power toggle
Remote: mute
------------------------------------
| I'm TV set.
| I'm enabled
| Current volume is 0%
| Current channel is 1
------------------------------------

Tests with basic remote.
Remote: power toggle
------------------------------------
| I'm radio.
| I'm enabled
| Current volume is 30%
| Current channel is 1
------------------------------------

Tests with advanced remote.
Remote: power toggle
Remote: mute
------------------------------------
| I'm radio.
| I'm enabled
| Current volume is 0%
| Current channel is 1
------------------------------------```
```

::: next
#### 次を読む

[Composite を Rust で []{.fa
.fa-arrow-right}](../../composite/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Adapter を Rust
で](../../adapter/rust/example.html){.btn .btn-default rel="prev"}
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
[![Bridge を Java
で](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Bridge を Java で"){.prog-lang-link}
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
[![Bridge を Swift
で](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Bridge を Swift で"){.prog-lang-link}
[![Bridge を TypeScript
で](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Bridge を TypeScript で"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
