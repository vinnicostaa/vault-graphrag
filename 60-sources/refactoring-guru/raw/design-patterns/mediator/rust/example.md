:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** in Rust {#mediator-in-rust .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** is a behavioral design pattern that reduces coupling
between components of a program by making them communicate indirectly,
through a special mediator object.

The Mediator makes it easy to modify, extend and reuse individual
components because they're no longer dependent on the dozens of other
classes.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Mediator ](../../mediator.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Top-Down Ownership](#example-0)
:::

::: en-file
 [train_station](#example-0--train_station-rs)
:::

:::: en-inside
::: en-file
  [mod](#example-0--trains-mod-rs)
:::
::::

:::: en-inside
::: en-file
  [freight_train](#example-0--trains-freight_train-rs)
:::
::::

:::: en-inside
::: en-file
  [passenger_train](#example-0--trains-passenger_train-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Top-Down Ownership {#example-0-title}

Top-Down Ownership approach allows to apply Mediator in Rust as it is a
suitable for Rust's ownership model with strict borrow checker rules.
It's not the only way to implement Mediator, but it's a fundamental one.

The key point is thinking in terms of OWNERSHIP.

1.  A mediator takes ownership of all components.

2.  A component doesn't preserve a reference to a mediator. Instead, it
    gets the reference via a method call.

    <figure class="code">
    <pre class="code" lang="rust"><code>// A train gets a mediator object by reference.
    pub trait Train {
        fn name(&amp;self) -&gt; &amp;String;
        fn arrive(&amp;mut self, mediator: &amp;mut dyn Mediator);
        fn depart(&amp;mut self, mediator: &amp;mut dyn Mediator);
    }

    // Mediator has notification methods.
    pub trait Mediator {
        fn notify_about_arrival(&amp;mut self, train_name: &amp;str) -&gt; bool;
        fn notify_about_departure(&amp;mut self, train_name: &amp;str);
    }</code></pre>
    </figure>

3.  Control flow starts from `fn main()` where the mediator receives
    external events/commands.

4.  `Mediator` trait for the interaction between components
    (`notify_about_arrival`, `notify_about_departure`) is not the same
    as its external API for receiving external events (`accept`,
    `depart` commands from the main loop).

    <figure class="code">
    <pre class="code" lang="rust"><code>let train1 = PassengerTrain::new(&quot;Train 1&quot;);
    let train2 = FreightTrain::new(&quot;Train 2&quot;);

    // Station has `accept` and `depart` methods,
    // but it also implements `Mediator`.
    let mut station = TrainStation::default();

    // Station is taking ownership of the trains.
    station.accept(train1);
    station.accept(train2);

    // `train1` and `train2` have been moved inside,
    // but we can use train names to depart them.
    station.depart(&quot;Train 1&quot;);
    station.depart(&quot;Train 2&quot;);
    station.depart(&quot;Train 3&quot;);</code></pre>
    </figure>

<figure class="image">
<img
src="https://github.com/fadeevab/mediator-pattern-rust/raw/main/images/mediator-rust-approach.jpg"
alt="Top-Down Ownership" />
</figure>

#### Extra info

There is a research and discussion of the Mediator pattern in Rust:
<https://github.com/fadeevab/mediator-pattern-rust>

#### []{#example-0--train_station-rs .anchor} **train_station.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use std::collections::{HashMap, VecDeque};

use crate::trains::Train;

// Mediator has notification methods.
pub trait Mediator {
    fn notify_about_arrival(&amp;mut self, train_name: &amp;str) -&gt; bool;
    fn notify_about_departure(&amp;mut self, train_name: &amp;str);
}

#[derive(Default)]
pub struct TrainStation {
    trains: HashMap&lt;String, Box&lt;dyn Train&gt;&gt;,
    train_queue: VecDeque&lt;String&gt;,
    train_on_platform: Option&lt;String&gt;,
}

impl Mediator for TrainStation {
    fn notify_about_arrival(&amp;mut self, train_name: &amp;str) -&gt; bool {
        if self.train_on_platform.is_some() {
            self.train_queue.push_back(train_name.into());
            false
        } else {
            self.train_on_platform.replace(train_name.into());
            true
        }
    }

    fn notify_about_departure(&amp;mut self, train_name: &amp;str) {
        if Some(train_name.into()) == self.train_on_platform {
            self.train_on_platform = None;

            if let Some(next_train_name) = self.train_queue.pop_front() {
                let mut next_train = self.trains.remove(&amp;next_train_name).unwrap();
                next_train.arrive(self);
                self.trains.insert(next_train_name.clone(), next_train);

                self.train_on_platform = Some(next_train_name);
            }
        }
    }
}

impl TrainStation {
    pub fn accept(&amp;mut self, mut train: impl Train + &#39;static) {
        if self.trains.contains_key(train.name()) {
            println!(&quot;{} has already arrived&quot;, train.name());
            return;
        }

        train.arrive(self);
        self.trains.insert(train.name().clone(), Box::new(train));
    }

    pub fn depart(&amp;mut self, name: &amp;&#39;static str) {
        let train = self.trains.remove(name);
        if let Some(mut train) = train {
            train.depart(self);
        } else {
            println!(&quot;&#39;{}&#39; is not on the station!&quot;, name);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--trains-mod-rs .anchor} **trains/mod.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod freight_train;
mod passenger_train;

pub use freight_train::FreightTrain;
pub use passenger_train::PassengerTrain;

use crate::train_station::Mediator;

// A train gets a mediator object by reference.
pub trait Train {
    fn name(&amp;self) -&gt; &amp;String;
    fn arrive(&amp;mut self, mediator: &amp;mut dyn Mediator);
    fn depart(&amp;mut self, mediator: &amp;mut dyn Mediator);
}</code></pre>
</figure>

#### []{#example-0--trains-freight_train-rs .anchor} **trains/freight_train.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Train;
use crate::train_station::Mediator;

pub struct FreightTrain {
    name: String,
}

impl FreightTrain {
    pub fn new(name: &amp;&#39;static str) -&gt; Self {
        Self { name: name.into() }
    }
}

impl Train for FreightTrain {
    fn name(&amp;self) -&gt; &amp;String {
        &amp;self.name
    }

    fn arrive(&amp;mut self, mediator: &amp;mut dyn Mediator) {
        if !mediator.notify_about_arrival(&amp;self.name) {
            println!(&quot;Freight train {}: Arrival blocked, waiting&quot;, self.name);
            return;
        }

        println!(&quot;Freight train {}: Arrived&quot;, self.name);
    }

    fn depart(&amp;mut self, mediator: &amp;mut dyn Mediator) {
        println!(&quot;Freight train {}: Leaving&quot;, self.name);
        mediator.notify_about_departure(&amp;self.name);
    }
}</code></pre>
</figure>

#### []{#example-0--trains-passenger_train-rs .anchor} **trains/passenger_train.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Train;
use crate::train_station::Mediator;

pub struct PassengerTrain {
    name: String,
}

impl PassengerTrain {
    pub fn new(name: &amp;&#39;static str) -&gt; Self {
        Self { name: name.into() }
    }
}

impl Train for PassengerTrain {
    fn name(&amp;self) -&gt; &amp;String {
        &amp;self.name
    }

    fn arrive(&amp;mut self, mediator: &amp;mut dyn Mediator) {
        if !mediator.notify_about_arrival(&amp;self.name) {
            println!(&quot;Passenger train {}: Arrival blocked, waiting&quot;, self.name);
            return;
        }

        println!(&quot;Passenger train {}: Arrived&quot;, self.name);
    }

    fn depart(&amp;mut self, mediator: &amp;mut dyn Mediator) {
        println!(&quot;Passenger train {}: Leaving&quot;, self.name);
        mediator.notify_about_departure(&amp;self.name);
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>mod train_station;
mod trains;

use train_station::TrainStation;
use trains::{FreightTrain, PassengerTrain};

fn main() {
    let train1 = PassengerTrain::new(&quot;Train 1&quot;);
    let train2 = FreightTrain::new(&quot;Train 2&quot;);

    // Station has `accept` and `depart` methods,
    // but it also implements `Mediator`.
    let mut station = TrainStation::default();

    // Station is taking ownership of the trains.
    station.accept(train1);
    station.accept(train2);

    // `train1` and `train2` have been moved inside,
    // but we can use train names to depart them.
    station.depart(&quot;Train 1&quot;);
    station.depart(&quot;Train 2&quot;);
    station.depart(&quot;Train 3&quot;);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Passenger train Train 1: Arrived
Freight train Train 2: Arrival blocked, waiting
Passenger train Train 1: Leaving
Freight train Train 2: Arrived
Freight train Train 2: Leaving
&#39;Train 3&#39; is not on the station!</code></pre>
</figure>

::: next
#### Read next

[Memento in Rust []{.fa
.fa-arrow-right}](../../memento/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Iterator in
Rust](../../iterator/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** in Other Languages

[![Mediator in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator in C#"){.prog-lang-link}
[![Mediator in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Mediator in C++"){.prog-lang-link}
[![Mediator in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator in Go"){.prog-lang-link}
[![Mediator in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator in Java"){.prog-lang-link}
[![Mediator in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator in PHP"){.prog-lang-link}
[![Mediator in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator in Python"){.prog-lang-link}
[![Mediator in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator in Ruby"){.prog-lang-link}
[![Mediator in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator in Swift"){.prog-lang-link}
[![Mediator in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
