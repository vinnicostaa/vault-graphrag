::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Structural
Patterns](structural-patterns.html){.type}
:::

# Bridge {#bridge .title}

::: {.section .intent}
##  Intent

**Bridge** is a structural design pattern that lets you split a large
class or a set of closely related classes into two separate
hierarchies---abstraction and implementation---which can be developed
independently of each other.

<figure class="image">
<img
src="../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Bridge design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

*Abstraction?* *Implementation?* Sound scary? Stay calm and let's
consider a simple example.

Say you have a geometric `Shape` class with a pair of subclasses:
`Circle` and `Square`. You want to extend this class hierarchy to
incorporate colors, so you plan to create `Red` and `Blue` shape
subclasses. However, since you already have two subclasses, you'll need
to create four class combinations such as `BlueCircle` and `RedSquare`.

<figure class="image">
<img
src="../images/patterns/diagrams/bridge/problem-enaaf9.png?id=81f8ed6e6f5d673e15203b22a7a3c502"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-en.png?id=81f8ed6e6f5d673e15203b22a7a3c502 480w,/images/patterns/diagrams/bridge/problem-en-1.5x.png?id=a24f6ae98a0ecced40e8f3feeab058d5 720w,/images/patterns/diagrams/bridge/problem-en-2x.png?id=c67b62720e0465821bbcb84debbbaab0 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Bridge pattern problem" />
<figcaption><p>Number of class combinations grows in
geometric progression.</p></figcaption>
</figure>

Adding new shape types and colors to the hierarchy will grow it
exponentially. For example, to add a triangle shape you'd need to
introduce two subclasses, one for each color. And after that, adding a
new color would require creating three subclasses, one for each shape
type. The further we go, the worse it becomes.
:::

::: {.section .solution}
##  Solution

This problem occurs because we're trying to extend the shape classes in
two independent dimensions: by form and by color. That's a very common
issue with class inheritance.

The Bridge pattern attempts to solve this problem by switching from
inheritance to the object composition. What this means is that you
extract one of the dimensions into a separate class hierarchy, so that
the original classes will reference an object of the new hierarchy,
instead of having all of its state and behaviors within one class.

<figure class="image">
<img
src="../images/patterns/diagrams/bridge/solution-endee9.png?id=b72caae18c400d6088072f2f3adda7cd"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-en.png?id=b72caae18c400d6088072f2f3adda7cd 460w,/images/patterns/diagrams/bridge/solution-en-1.5x.png?id=18771f23c35b4614eac048975a5f3167 690w,/images/patterns/diagrams/bridge/solution-en-2x.png?id=9439c9a85ac3db0399844b45fbbaecf6 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Solution suggested by the Bridge pattern" />
<figcaption><p>You can prevent the explosion of a class hierarchy by
transforming it into several related hierarchies.</p></figcaption>
</figure>

Following this approach, we can extract the color-related code into its
own class with two subclasses: `Red` and `Blue`. The `Shape` class then
gets a reference field pointing to one of the color objects. Now the
shape can delegate any color-related work to the linked color object.
That reference will act as a bridge between the `Shape` and `Color`
classes. From now on, adding new colors won't require changing the shape
hierarchy, and vice versa.

#### Abstraction and Implementation

The GoF book []{.pop-i}["Gang of Four" is a nickname given to the four
authors of the original book about design patterns: *Design Patterns:
Elements of Reusable Object-Oriented Software*
[https://refactoring.guru/gof-book](https://www.amazon.com/gp/product/0201633612/).]{.pop-content}
introduces the terms *Abstraction* and *Implementation* as part of the
Bridge definition. In my opinion, the terms sound too academic and make
the pattern seem more complicated than it really is. Having read the
simple example with shapes and colors, let's decipher the meaning behind
the GoF book's scary words.

*Abstraction* (also called *interface*) is a high-level control layer
for some entity. This layer isn't supposed to do any real work on its
own. It should delegate the work to the *implementation* layer (also
called *platform*).

Note that we're not talking about *interfaces* or *abstract classes*
from your programming language. These aren't the same things.

When talking about real applications, the abstraction can be represented
by a graphical user interface (GUI), and the implementation could be the
underlying operating system code (API) which the GUI layer calls in
response to user interactions.

Generally speaking, you can extend such an app in two independent
directions:

-   Have several different GUIs (for instance, tailored for regular
    customers or admins).
-   Support several different APIs (for example, to be able to launch
    the app under Windows, Linux, and macOS).

In a worst-case scenario, this app might look like a giant spaghetti
bowl, where hundreds of conditionals connect different types of GUI with
various APIs all over the code.

<figure class="image">
<img
src="../images/patterns/content/bridge/bridge-3-enc925.png?id=15b8262114938f7bef6602af33f0a62e"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-en.png?id=15b8262114938f7bef6602af33f0a62e 600w,/images/patterns/content/bridge/bridge-3-en-1.5x.png?id=1c4de58902d1030a13b786480def264a 900w,/images/patterns/content/bridge/bridge-3-en-2x.png?id=65f98465cec6c4c7e38a635689943822 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Managing changes is much easier in modular code" />
<figcaption><p>Making even a simple change to a monolithic codebase is
pretty hard because you must understand the <em>entire thing</em> very
well. Making changes to smaller, well-defined modules is
much easier.</p></figcaption>
</figure>

You can bring order to this chaos by extracting the code related to
specific interface-platform combinations into separate classes. However,
soon you'll discover that there are *lots* of these classes. The class
hierarchy will grow exponentially because adding a new GUI or supporting
a different API would require creating more and more classes.

Let's try to solve this issue with the Bridge pattern. It suggests that
we divide the classes into two hierarchies:

-   Abstraction: the GUI layer of the app.
-   Implementation: the operating systems' APIs.

<figure class="image">
<img
src="../images/patterns/content/bridge/bridge-2-enca76.png?id=5c5aef57ca6aa8c3c97fd8922fc8bb58"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-en.png?id=5c5aef57ca6aa8c3c97fd8922fc8bb58 640w,/images/patterns/content/bridge/bridge-2-en-1.5x.png?id=72ed90c886ed6ed2bb3b9dd45f7724bb 960w,/images/patterns/content/bridge/bridge-2-en-2x.png?id=bbd64c96e6711636356944b3564ad67e 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Cross-platform architecture" />
<figcaption><p>One of the ways to structure a
cross-platform application.</p></figcaption>
</figure>

The abstraction object controls the appearance of the app, delegating
the actual work to the linked implementation object. Different
implementations are interchangeable as long as they follow a common
interface, enabling the same GUI to work under Windows and Linux.

As a result, you can change the GUI classes without touching the
API-related classes. Moreover, adding support for another operating
system only requires creating a subclass in the implementation
hierarchy.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/bridge/structure-en068f.png?id=827afa4b40008dc29d26fe0f4d41b9cc"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-en.png?id=827afa4b40008dc29d26fe0f4d41b9cc 560w,/images/patterns/diagrams/bridge/structure-en-1.5x.png?id=b1968bfca1fece32e9cb65cd151dacdd 840w,/images/patterns/diagrams/bridge/structure-en-2x.png?id=77e749744fb5375839b1cf1aa1061648 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Bridge design pattern" /><img
src="../images/patterns/diagrams/bridge/structure-en-indexedf231.png?id=0461ee029a15b02e03e9735f2ca576d4"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-en-indexed.png?id=0461ee029a15b02e03e9735f2ca576d4 560w,/images/patterns/diagrams/bridge/structure-en-indexed-1.5x.png?id=648521378ae7eac370e7e25ccd52c607 840w,/images/patterns/diagrams/bridge/structure-en-indexed-2x.png?id=99713473c8ba3c08ce6a3540f1453ebc 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Bridge design pattern" />
</figure>
:::

1.  The **Abstraction** provides high-level control logic. It relies on
    the implementation object to do the actual low-level work.

2.  The **Implementation** declares the interface that's common for all
    concrete implementations. An abstraction can only communicate with
    an implementation object via methods that are declared here.

    The abstraction may list the same methods as the implementation, but
    usually the abstraction declares some complex behaviors that rely on
    a wide variety of primitive operations declared by the
    implementation.

3.  **Concrete Implementations** contain platform-specific code.

4.  **Refined Abstractions** provide variants of control logic. Like
    their parent, they work with different implementations via the
    general implementation interface.

5.  Usually, the **Client** is only interested in working with the
    abstraction. However, it's the client's job to link the abstraction
    object with one of the implementation objects.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

This example illustrates how the **Bridge** pattern can help divide the
monolithic code of an app that manages devices and their remote
controls. The `Device` classes act as the implementation, whereas the
`Remote`s act as the abstraction.

<figure class="image">
<img
src="../images/patterns/diagrams/bridge/example-enab39.png?id=89c406a189c45885004d7fa094f616b1"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-en.png?id=89c406a189c45885004d7fa094f616b1 640w,/images/patterns/diagrams/bridge/example-en-1.5x.png?id=bb870a109edc52920a51b3fc9110e7f4 960w,/images/patterns/diagrams/bridge/example-en-2x.png?id=19bb8272b4082c5f47c4d047e6cb9967 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Structure of the Bridge pattern example" />
<figcaption><p>The original class hierarchy is divided into two parts:
devices and remote controls.</p></figcaption>
</figure>

The base remote control class declares a reference field that links it
with a device object. All remotes work with the devices via the general
device interface, which lets the same remote support multiple device
types.

You can develop the remote control classes independently from the device
classes. All that's needed is to create a new remote subclass. For
example, a basic remote control might only have two buttons, but you
could extend it with additional features, such as an extra battery or a
touchscreen.

The client code links the desired type of remote control with a specific
device object via the remote's constructor.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The &quot;abstraction&quot; defines the interface for the &quot;control&quot;
// part of the two class hierarchies. It maintains a reference
// to an object of the &quot;implementation&quot; hierarchy and delegates
// all of the real work to this object.
class RemoteControl is
    protected field device: Device
    constructor RemoteControl(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// You can extend classes from the abstraction hierarchy
// independently from device classes.
class AdvancedRemoteControl extends RemoteControl is
    method mute() is
        device.setVolume(0)


// The &quot;implementation&quot; interface declares methods common to all
// concrete implementation classes. It doesn&#39;t have to match the
// abstraction&#39;s interface. In fact, the two interfaces can be
// entirely different. Typically the implementation interface
// provides only primitive operations, while the abstraction
// defines higher-level operations based on those primitives.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// All devices follow the same interface.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Somewhere in client code.
tv = new Tv()
remote = new RemoteControl(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemoteControl(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Bridge pattern when you want to divide and organize a monolithic
class that has several variants of some functionality (for example, if
the class can work with various database servers).
:::

::: applicability-solution
The bigger a class becomes, the harder it is to figure out how it works,
and the longer it takes to make a change. The changes made to one of the
variations of functionality may require making changes across the whole
class, which often results in making errors or not addressing some
critical side effects.

The Bridge pattern lets you split the monolithic class into several
class hierarchies. After this, you can change the classes in each
hierarchy independently of the classes in the others. This approach
simplifies code maintenance and minimizes the risk of breaking existing
code.
:::

::: applicability-problem
Use the pattern when you need to extend a class in several orthogonal
(independent) dimensions.
:::

::: applicability-solution
The Bridge suggests that you extract a separate class hierarchy for each
of the dimensions. The original class delegates the related work to the
objects belonging to those hierarchies instead of doing everything on
its own.
:::

::: applicability-problem
Use the Bridge if you need to be able to switch implementations at
runtime.
:::

::: applicability-solution
Although it's optional, the Bridge pattern lets you replace the
implementation object inside the abstraction. It's as easy as assigning
a new value to a field.

By the way, this last item is the main reason why so many people confuse
the Bridge with the [Strategy](strategy.html) pattern. Remember that a
pattern is more than just a certain way to structure your classes. It
may also communicate intent and a problem being addressed.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Identify the orthogonal dimensions in your classes. These
    independent concepts could be: abstraction/platform,
    domain/infrastructure, front-end/back-end, or
    interface/implementation.

2.  See what operations the client needs and define them in the base
    abstraction class.

3.  Determine the operations available on all platforms. Declare the
    ones that the abstraction needs in the general implementation
    interface.

4.  For all platforms in your domain create concrete implementation
    classes, but make sure they all follow the implementation interface.

5.  Inside the abstraction class, add a reference field for the
    implementation type. The abstraction delegates most of the work to
    the implementation object that's referenced in that field.

6.  If you have several variants of high-level logic, create refined
    abstractions for each variant by extending the base abstraction
    class.

7.  The client code should pass an implementation object to the
    abstraction's constructor to associate one with the other. After
    that, the client can forget about the implementation and work only
    with the abstraction object.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can create platform-independent classes and apps.
-    The client code works with high-level abstractions. It isn't
    exposed to the platform details.
-    *Open/Closed Principle*. You can introduce new abstractions and
    implementations independently from each other.
-    *Single Responsibility Principle*. You can focus on high-level
    logic in the abstraction and on platform details in the
    implementation.
:::

::: col-sm-6
-    You might make the code more complicated by applying the pattern to
    a highly cohesive class.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   [Bridge](bridge.html) is usually designed up-front, letting you
    develop parts of an application independently of each other. On the
    other hand, [Adapter](adapter.html) is commonly used with an
    existing app to make some otherwise-incompatible classes work
    together nicely.

-   [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (and to some degree
    [Adapter](adapter.html)) have very similar structures. Indeed, all
    of these patterns are based on composition, which is delegating work
    to other objects. However, they all solve different problems. A
    pattern isn't just a recipe for structuring your code in a specific
    way. It can also communicate to other developers the problem the
    pattern solves.

-   You can use [Abstract Factory](abstract-factory.html) along with
    [Bridge](bridge.html). This pairing is useful when some abstractions
    defined by *Bridge* can only work with specific implementations. In
    this case, *Abstract Factory* can encapsulate these relations and
    hide the complexity from the client code.

-   You can combine [Builder](builder.html) with [Bridge](bridge.html):
    the director class plays the role of the abstraction, while
    different builders act as implementations.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Bridge in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Bridge in C#"){.prog-lang-link}
[![Bridge in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Bridge in C++"){.prog-lang-link}
[![Bridge in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Bridge in Go"){.prog-lang-link}
[![Bridge in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Bridge in Java"){.prog-lang-link}
[![Bridge in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Bridge in PHP"){.prog-lang-link}
[![Bridge in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Bridge in Python"){.prog-lang-link}
[![Bridge in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Bridge in Ruby"){.prog-lang-link}
[![Bridge in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Bridge in Rust"){.prog-lang-link}
[![Bridge in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Bridge in Swift"){.prog-lang-link}
[![Bridge in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Bridge in TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set2}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Support our free website and own the eBook!" creative-id="standard-en" position="content_bottom"}
::: banner-image
[![](../images/patterns/banners/patterns-book-banner-3a29e.png?id=7d445df13c80287beaab234b4f3b698c){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-3-2x.png?id=0cc3f77ab421d1a5c02ee46488231c3a 2x"}](book.html)
:::

::: banner-text
### Support our free website and own the eBook! {#support-our-free-website-and-own-the-ebook .title}

-   22 design patterns and 8 principles explained in depth.
-   409 well-structured, easy to read, jargon-free pages.
-   225 clear and helpful illustrations and diagrams.
-   An archive with code examples in 11 languages.
-   All devices supported: PDF/EPUB/MOBI/KFX formats.

[ Learn more...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Read next

[Composite []{.fa .fa-arrow-right}](composite.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Adapter](adapter.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../images/patterns/book/web-cover-en00eb.png?id=328861769fd11617674e3b8a7e2dd9e7){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-en-2x.png?id=02940141c5652ed5a426d87390b16366 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
This article is a part of our eBook\
**Dive Into Design Patterns**.

[ Learn more...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
