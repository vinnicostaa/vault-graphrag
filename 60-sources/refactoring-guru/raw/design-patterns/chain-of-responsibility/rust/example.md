::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** in Rust {#chain-of-responsibility-in-rust .pattern-example-title-block-title}
::::

:::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** is behavioral design pattern that allows
passing request along the chain of potential handlers until one of them
handles request.

The pattern allows multiple objects to handle the request without
coupling sender class to the concrete classes of the receivers. The
chain can be composed dynamically at runtime with any handler that
follows a standard handler interface.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Chain of Responsibility
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [patient](#example-0--patient-rs)
:::

::: en-file
 [department](#example-0--department-rs)
:::

:::: en-inside
::: en-file
  [cashier](#example-0--department-cashier-rs)
:::
::::

:::: en-inside
::: en-file
  [doctor](#example-0--department-doctor-rs)
:::
::::

:::: en-inside
::: en-file
  [medical](#example-0--department-medical-rs)
:::
::::

:::: en-inside
::: en-file
  [reception](#example-0--department-reception-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::::::::::
::::::::::::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

The example demonstrates processing a patient through a chain of
departments. The chain of responsibility is constructed as follows:

<figure class="code">
<pre class="code"><code>Patient -&gt; Reception -&gt; Doctor -&gt; Medical -&gt; Cashier</code></pre>
</figure>

The chain is constructed using `Box` pointers, which means dynamic
dispatch in runtime. Why? It seems quite difficult to narrow down
implementation to a strict compile-time typing using generics: in order
to construct a type of a full chain Rust needs full knowledge of the
"next of the next" link in the chain. Thus, it would look like this:

<figure class="code">
<pre class="code" lang="rust"><code>let mut reception = Reception::&lt;Doctor::&lt;Medical::&lt;Cashier&gt;&gt;&gt;::new(doctor); // 😱</code></pre>
</figure>

Instead, `Box` allows chaining in any combination:

<figure class="code">
<pre class="code" lang="rust"><code>let mut reception = Reception::new(doctor); // 👍

let mut reception = Reception::new(cashier); // 🕵️‍♀️</code></pre>
</figure>

#### []{#example-0--patient-rs .anchor} **patient.rs:** Request

<figure class="code">
<pre class="code" lang="rust"><code>#[derive(Default)]
pub struct Patient {
    pub name: String,
    pub registration_done: bool,
    pub doctor_check_up_done: bool,
    pub medicine_done: bool,
    pub payment_done: bool,
}</code></pre>
</figure>

#### []{#example-0--department-rs .anchor} **department.rs:** Handlers

<figure class="code">
<pre class="code" lang="rust"><code>mod cashier;
mod doctor;
mod medical;
mod reception;

pub use cashier::Cashier;
pub use doctor::Doctor;
pub use medical::Medical;
pub use reception::Reception;

use crate::patient::Patient;

/// A single role of objects that make up a chain.
/// A typical trait implementation must have `handle` and `next` methods,
/// while `execute` is implemented by default and contains a proper chaining
/// logic.
pub trait Department {
    fn execute(&amp;mut self, patient: &amp;mut Patient) {
        self.handle(patient);

        if let Some(next) = &amp;mut self.next() {
            next.execute(patient);
        }
    }

    fn handle(&amp;mut self, patient: &amp;mut Patient);
    fn next(&amp;mut self) -&gt; &amp;mut Option&lt;Box&lt;dyn Department&gt;&gt;;
}

/// Helps to wrap an object into a boxed type.
pub fn into_next(department: impl Department + Sized + &#39;static) -&gt; Option&lt;Box&lt;dyn Department&gt;&gt; {
    Some(Box::new(department))
}</code></pre>
</figure>

#### []{#example-0--department-cashier-rs .anchor} **department/cashier.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::{Department, Patient};

#[derive(Default)]
pub struct Cashier {
    next: Option&lt;Box&lt;dyn Department&gt;&gt;,
}

impl Department for Cashier {
    fn handle(&amp;mut self, patient: &amp;mut Patient) {
        if patient.payment_done {
            println!(&quot;Payment done&quot;);
        } else {
            println!(&quot;Cashier getting money from a patient {}&quot;, patient.name);
            patient.payment_done = true;
        }
    }

    fn next(&amp;mut self) -&gt; &amp;mut Option&lt;Box&lt;dyn Department&gt;&gt; {
        &amp;mut self.next
    }
}</code></pre>
</figure>

#### []{#example-0--department-doctor-rs .anchor} **department/doctor.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::{into_next, Department, Patient};

pub struct Doctor {
    next: Option&lt;Box&lt;dyn Department&gt;&gt;,
}

impl Doctor {
    pub fn new(next: impl Department + &#39;static) -&gt; Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Doctor {
    fn handle(&amp;mut self, patient: &amp;mut Patient) {
        if patient.doctor_check_up_done {
            println!(&quot;A doctor checkup is already done&quot;);
        } else {
            println!(&quot;Doctor checking a patient {}&quot;, patient.name);
            patient.doctor_check_up_done = true;
        }
    }

    fn next(&amp;mut self) -&gt; &amp;mut Option&lt;Box&lt;dyn Department&gt;&gt; {
        &amp;mut self.next
    }
}</code></pre>
</figure>

#### []{#example-0--department-medical-rs .anchor} **department/medical.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::{into_next, Department, Patient};

pub struct Medical {
    next: Option&lt;Box&lt;dyn Department&gt;&gt;,
}

impl Medical {
    pub fn new(next: impl Department + &#39;static) -&gt; Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Medical {
    fn handle(&amp;mut self, patient: &amp;mut Patient) {
        if patient.medicine_done {
            println!(&quot;Medicine is already given to a patient&quot;);
        } else {
            println!(&quot;Medical giving medicine to a patient {}&quot;, patient.name);
            patient.medicine_done = true;
        }
    }

    fn next(&amp;mut self) -&gt; &amp;mut Option&lt;Box&lt;dyn Department&gt;&gt; {
        &amp;mut self.next
    }
}</code></pre>
</figure>

#### []{#example-0--department-reception-rs .anchor} **department/reception.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::{into_next, Department, Patient};

#[derive(Default)]
pub struct Reception {
    next: Option&lt;Box&lt;dyn Department&gt;&gt;,
}

impl Reception {
    pub fn new(next: impl Department + &#39;static) -&gt; Self {
        Self {
            next: into_next(next),
        }
    }
}

impl Department for Reception {
    fn handle(&amp;mut self, patient: &amp;mut Patient) {
        if patient.registration_done {
            println!(&quot;Patient registration is already done&quot;);
        } else {
            println!(&quot;Reception registering a patient {}&quot;, patient.name);
            patient.registration_done = true;
        }
    }

    fn next(&amp;mut self) -&gt; &amp;mut Option&lt;Box&lt;dyn Department&gt;&gt; {
        &amp;mut self.next
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>mod department;
mod patient;

use department::{Cashier, Department, Doctor, Medical, Reception};
use patient::Patient;

fn main() {
    let cashier = Cashier::default();
    let medical = Medical::new(cashier);
    let doctor = Doctor::new(medical);
    let mut reception = Reception::new(doctor);

    let mut patient = Patient {
        name: &quot;John&quot;.into(),
        ..Patient::default()
    };

    // Reception handles a patient passing him to the next link in the chain.
    // Reception -&gt; Doctor -&gt; Medical -&gt; Cashier.
    reception.execute(&amp;mut patient);

    println!(&quot;\nThe patient has been already handled:\n&quot;);

    reception.execute(&amp;mut patient);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Reception registering a patient John
Doctor checking a patient John
Medical giving medicine to a patient John
Cashier getting money from a patient John

The patient has been already handled:

Patient registration is already done
A doctor checkup is already done
Medicine is already given to a patient
Payment done</code></pre>
</figure>

::: next
#### Read next

[Command in Rust []{.fa
.fa-arrow-right}](../../command/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Proxy in
Rust](../../proxy/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Chain of Responsibility** in Other Languages

[![Chain of Responsibility in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility in C#"){.prog-lang-link}
[![Chain of Responsibility in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility in C++"){.prog-lang-link}
[![Chain of Responsibility in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Chain of Responsibility in Go"){.prog-lang-link}
[![Chain of Responsibility in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility in Java"){.prog-lang-link}
[![Chain of Responsibility in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility in PHP"){.prog-lang-link}
[![Chain of Responsibility in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility in Python"){.prog-lang-link}
[![Chain of Responsibility in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility in Ruby"){.prog-lang-link}
[![Chain of Responsibility in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility in Swift"){.prog-lang-link}
[![Chain of Responsibility in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility in TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
