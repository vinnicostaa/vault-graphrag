::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Chain of Responsibility {#chain-of-responsibility .title}

::: aka
Also known as: [CoR, ]{style="display:inline-block"}[Chain of
Command]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Chain of Responsibility** is a behavioral design pattern that lets you
pass requests along a chain of handlers. Upon receiving a request, each
handler decides either to process the request or to pass it to the next
handler in the chain.

<figure class="image">
<img
src="../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Chain of Responsibility design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you're working on an online ordering system. You want to
restrict access to the system so only authenticated users can create
orders. Also, users who have administrative permissions must have full
access to all orders.

After a bit of planning, you realized that these checks must be
performed sequentially. The application can attempt to authenticate a
user to the system whenever it receives a request that contains the
user's credentials. However, if those credentials aren't correct and
authentication fails, there's no reason to proceed with any other
checks.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/problem1-ene244.png?id=dde084d408d2b14d632ba82583d16612"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-en.png?id=dde084d408d2b14d632ba82583d16612 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-en-1.5x.png?id=645017a237c198fc7b35fda7238bf2b7 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-en-2x.png?id=17786228efa05f83435fd39316cb940c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Problem, solved by Chain of Responsibility" />
<figcaption><p>The request must pass a series of checks before the
ordering system itself can handle it.</p></figcaption>
</figure>

During the next few months, you implemented several more of those
sequential checks.

-   One of your colleagues suggested that it's unsafe to pass raw data
    straight to the ordering system. So you added an extra validation
    step to sanitize the data in a request.

-   Later, somebody noticed that the system is vulnerable to brute force
    password cracking. To negate this, you promptly added a check that
    filters repeated failed requests coming from the same IP address.

-   Someone else suggested that you could speed up the system by
    returning cached results on repeated requests containing the same
    data. Hence, you added another check which lets the request pass
    through to the system only if there's no suitable cached response.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/problem2-en7337.png?id=88c684d3eab7707bee7b1550a2d8ae04"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-en.png?id=88c684d3eab7707bee7b1550a2d8ae04 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-en-1.5x.png?id=7f2bce2135d4b0e860277fcc190ce6d4 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-en-2x.png?id=bea31844e04cdd110755cef1571ca088 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="With each new check the code became bigger, messier, and uglier" />
<figcaption><p>The bigger the code grew, the messier
it became.</p></figcaption>
</figure>

The code of the checks, which had already looked like a mess, became
more and more bloated as you added each new feature. Changing one check
sometimes affected the others. Worst of all, when you tried to reuse the
checks to protect other components of the system, you had to duplicate
some of the code since those components required some of the checks, but
not all of them.

The system became very hard to comprehend and expensive to maintain. You
struggled with the code for a while, until one day you decided to
refactor the whole thing.
:::

::: {.section .solution}
##  Solution

Like many other behavioral design patterns, the **Chain of
Responsibility** relies on transforming particular behaviors into
stand-alone objects called *handlers*. In our case, each check should be
extracted to its own class with a single method that performs the check.
The request, along with its data, is passed to this method as an
argument.

The pattern suggests that you link these handlers into a chain. Each
linked handler has a field for storing a reference to the next handler
in the chain. In addition to processing a request, handlers pass the
request further along the chain. The request travels along the chain
until all handlers have had a chance to process it.

Here's the best part: a handler can decide not to pass the request
further down the chain and effectively stop any further processing.

In our example with ordering systems, a handler performs the processing
and then decides whether to pass the request further down the chain.
Assuming the request contains the right data, all the handlers can
execute their primary behavior, whether it's authentication checks or
caching.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/solution1-enc1b3.png?id=dccad3e628bd2b8f1856c99369ca6e5b"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-en.png?id=dccad3e628bd2b8f1856c99369ca6e5b 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-en-1.5x.png?id=c50a4fd7f58fb902cd23e5f52d15e9ba 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-en-2x.png?id=5942fb6064f55d3894ca71c0c0df3fd8 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Handlers are lined-up one by one, forming a chain" />
<figcaption><p>Handlers are lined up one by one, forming
a chain.</p></figcaption>
</figure>

However, there's a slightly different approach (and it's a bit more
canonical) in which, upon receiving a request, a handler decides whether
it can process it. If it can, it doesn't pass the request any further.
So it's either only one handler that processes the request or none at
all. This approach is very common when dealing with events in stacks of
elements within a graphical user interface.

For instance, when a user clicks a button, the event propagates through
the chain of GUI elements that starts with the button, goes along its
containers (like forms or panels), and ends up with the main application
window. The event is processed by the first element in the chain that's
capable of handling it. This example is also noteworthy because it shows
that a chain can always be extracted from an object tree.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/solution2-en71d9.png?id=cc5bab096e1b37105e1027c43a92cc8a"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-en.png?id=cc5bab096e1b37105e1027c43a92cc8a 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-en-1.5x.png?id=e310e9af8040174984077f0b3282155e 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-en-2x.png?id=3ba9dfc081064c3ecd3882f931431a0e 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="A chain can be formed from a branch of an object tree" />
<figcaption><p>A chain can be formed from a branch of an
object tree.</p></figcaption>
</figure>

It's crucial that all handler classes implement the same interface. Each
concrete handler should only care about the following one having the
`execute` method. This way you can compose chains at runtime, using
various handlers without coupling your code to their concrete classes.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-en1c1c.png?id=bcd771fd1a61c754911bd580cd80463e"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-en.png?id=bcd771fd1a61c754911bd580cd80463e 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-en-1.5x.png?id=2e13d156642005bdb2ef9559a06946c3 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-en-2x.png?id=169e558d5a5b869b4465f88b697a10ec 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Talking with tech support can be hard" />
<figcaption><p>A call to tech support can go through
multiple operators.</p></figcaption>
</figure>

You've just bought and installed a new piece of hardware on your
computer. Since you're a geek, the computer has several operating
systems installed. You try to boot all of them to see whether the
hardware is supported. Windows detects and enables the hardware
automatically. However, your beloved Linux refuses to work with the new
hardware. With a small flicker of hope, you decide to call the
tech-support phone number written on the box.

The first thing you hear is the robotic voice of the autoresponder. It
suggests nine popular solutions to various problems, none of which are
relevant to your case. After a while, the robot connects you to a live
operator.

Alas, the operator isn't able to suggest anything specific either. He
keeps quoting lengthy excerpts from the manual, refusing to listen to
your comments. After hearing the phrase "have you tried turning the
computer off and on again?" for the 10th time, you demand to be
connected to a proper engineer.

Eventually, the operator passes your call to one of the engineers, who
had probably longed for a live human chat for hours as he sat in his
lonely server room in the dark basement of some office building. The
engineer tells you where to download proper drivers for your new
hardware and how to install them on Linux. Finally, the solution! You
end the call, bursting with joy.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Structure of the Chain Of Responsibility design pattern" /><img
src="../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Structure of the Chain Of Responsibility design pattern" />
</figure>
:::

1.  The **Handler** declares the interface, common for all concrete
    handlers. It usually contains just a single method for handling
    requests, but sometimes it may also have another method for setting
    the next handler on the chain.

2.  The **Base Handler** is an optional class where you can put the
    boilerplate code that's common to all handler classes.

    Usually, this class defines a field for storing a reference to the
    next handler. The clients can build a chain by passing a handler to
    the constructor or setter of the previous handler. The class may
    also implement the default handling behavior: it can pass execution
    to the next handler after checking for its existence.

3.  **Concrete Handlers** contain the actual code for processing
    requests. Upon receiving a request, each handler must decide whether
    to process it and, additionally, whether to pass it along the chain.

    Handlers are usually self-contained and immutable, accepting all
    necessary data just once via the constructor.

4.  The **Client** may compose chains just once or compose them
    dynamically, depending on the application's logic. Note that a
    request can be sent to any handler in the chain---it doesn't have to
    be the first one.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Chain of Responsibility** pattern is responsible
for displaying contextual help information for active GUI elements.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/example-en4118.png?id=4b890b18dbff5193b3b538a438b6c5a4"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-en.png?id=4b890b18dbff5193b3b538a438b6c5a4 610w,/images/patterns/diagrams/chain-of-responsibility/example-en-1.5x.png?id=9b08bd8fbc187c1251cb2cc70a57e3f6 915w,/images/patterns/diagrams/chain-of-responsibility/example-en-2x.png?id=0f25d3d948f33c87482e832a55c3c680 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Structure of the Chain of Responsibility example" />
<figcaption><p>The GUI classes are built with the Composite pattern.
Each element is linked to its container element. At any point, you can
build a chain of elements that starts with the element itself and goes
through all of its container elements.</p></figcaption>
</figure>

The application's GUI is usually structured as an object tree. For
example, the `Dialog` class, which renders the main window of the app,
would be the root of the object tree. The dialog contains `Panels`,
which might contain other panels or simple low-level elements like
`Buttons` and `TextFields`.

A simple component can show brief contextual tooltips, as long as the
component has some help text assigned. But more complex components
define their own way of showing contextual help, such as showing an
excerpt from the manual or opening a page in a browser.

<figure class="image">
<img
src="../images/patterns/diagrams/chain-of-responsibility/example2-en1f29.png?id=ea5e6ea07b5cab132e51bac80467ca5a"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-en.png?id=ea5e6ea07b5cab132e51bac80467ca5a 250w,/images/patterns/diagrams/chain-of-responsibility/example2-en-1.5x.png?id=8f829167ba656e6bd57d61c9f76d5b17 375w,/images/patterns/diagrams/chain-of-responsibility/example2-en-2x.png?id=f6d72166631d9a4a80b013a4fa3d886b 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Structure of the Chain of Responsibility example" />
<figcaption><p>That’s how a help request traverses
GUI objects.</p></figcaption>
</figure>

When a user points the mouse cursor at an element and presses the `F1`
key, the application detects the component under the pointer and sends
it a help request. The request bubbles up through all the element's
containers until it reaches the element that's capable of displaying the
help information.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The handler interface declares a method for executing a
// request.
interface ComponentWithContextualHelp is
    method showHelp()


// The base class for simple components.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // The component&#39;s container acts as the next link in the
    // chain of handlers.
    protected field container: Container

    // The component shows a tooltip if there&#39;s help text
    // assigned to it. Otherwise it forwards the call to the
    // container, if it exists.
    method showHelp() is
        if (tooltipText != null)
            // Show tooltip.
        else
            container.showHelp()


// Containers can contain both simple components and other
// containers as children. The chain relationships are
// established here. The class inherits showHelp behavior from
// its parent.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Primitive components may be fine with default help
// implementation...
class Button extends Component is
    // ...

// But complex components may override the default
// implementation. If the help text can&#39;t be provided in a new
// way, the component can always call the base implementation
// (see Component class).
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Show a modal window with the help text.
        else
            super.showHelp()

// ...same as above...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Open the wiki help page.
        else
            super.showHelp()


// Client code.
class Application is
    // Every application configures the chain differently.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Imagine what happens here.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Chain of Responsibility pattern when your program is expected to
process different kinds of requests in various ways, but the exact types
of requests and their sequences are unknown beforehand.
:::

::: applicability-solution
The pattern lets you link several handlers into one chain and, upon
receiving a request, "ask" each handler whether it can process it. This
way all handlers get a chance to process the request.
:::

::: applicability-problem
Use the pattern when it's essential to execute several handlers in a
particular order.
:::

::: applicability-solution
Since you can link the handlers in the chain in any order, all requests
will get through the chain exactly as you planned.
:::

::: applicability-problem
Use the CoR pattern when the set of handlers and their order are
supposed to change at runtime.
:::

::: applicability-solution
If you provide setters for a reference field inside the handler classes,
you'll be able to insert, remove or reorder handlers dynamically.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Declare the handler interface and describe the signature of a method
    for handling requests.

    Decide how the client will pass the request data into the method.
    The most flexible way is to convert the request into an object and
    pass it to the handling method as an argument.

2.  To eliminate duplicate boilerplate code in concrete handlers, it
    might be worth creating an abstract base handler class, derived from
    the handler interface.

    This class should have a field for storing a reference to the next
    handler in the chain. Consider making the class immutable. However,
    if you plan to modify chains at runtime, you need to define a setter
    for altering the value of the reference field.

    You can also implement the convenient default behavior for the
    handling method, which is to forward the request to the next object
    unless there's none left. Concrete handlers will be able to use this
    behavior by calling the parent method.

3.  One by one create concrete handler subclasses and implement their
    handling methods. Each handler should make two decisions when
    receiving a request:

    -   Whether it'll process the request.
    -   Whether it'll pass the request along the chain.

4.  The client may either assemble chains on its own or receive
    pre-built chains from other objects. In the latter case, you must
    implement some factory classes to build chains according to the
    configuration or environment settings.

5.  The client may trigger any handler in the chain, not just the first
    one. The request will be passed along the chain until some handler
    refuses to pass it further or until it reaches the end of the chain.

6.  Due to the dynamic nature of the chain, the client should be ready
    to handle the following scenarios:

    -   The chain may consist of a single link.
    -   Some requests may not reach the end of the chain.
    -   Others may reach the end of the chain unhandled.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can control the order of request handling.
-    *Single Responsibility Principle*. You can decouple classes that
    invoke operations from classes that perform operations.
-    *Open/Closed Principle*. You can introduce new handlers into the
    app without breaking the existing client code.
:::

::: col-sm-6
-    Some requests may end up unhandled.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   [Chain of Responsibility](chain-of-responsibility.html),
    [Command](command.html), [Mediator](mediator.html) and
    [Observer](observer.html) address various ways of connecting senders
    and receivers of requests:

    -   *Chain of Responsibility* passes a request sequentially along a
        dynamic chain of potential receivers until one of them handles
        it.
    -   *Command* establishes unidirectional connections between senders
        and receivers.
    -   *Mediator* eliminates direct connections between senders and
        receivers, forcing them to communicate indirectly via a mediator
        object.
    -   *Observer* lets receivers dynamically subscribe to and
        unsubscribe from receiving requests.

-   [Chain of Responsibility](chain-of-responsibility.html) is often
    used in conjunction with [Composite](composite.html). In this case,
    when a leaf component gets a request, it may pass it through the
    chain of all of the parent components down to the root of the object
    tree.

-   Handlers in [Chain of Responsibility](chain-of-responsibility.html)
    can be implemented as [Commands](command.html). In this case, you
    can execute a lot of different operations over the same context
    object, represented by a request.

    However, there's another approach, where the request itself is a
    *Command* object. In this case, you can execute the same operation
    in a series of different contexts linked into a chain.

-   [Chain of Responsibility](chain-of-responsibility.html) and
    [Decorator](decorator.html) have very similar class structures. Both
    patterns rely on recursive composition to pass the execution through
    a series of objects. However, there are several crucial differences.

    The *CoR* handlers can execute arbitrary operations independently of
    each other. They can also stop passing the request further at any
    point. On the other hand, various *Decorators* can extend the
    object's behavior while keeping it consistent with the base
    interface. In addition, decorators aren't allowed to break the flow
    of the request.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Chain of Responsibility in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Chain of Responsibility in C#"){.prog-lang-link}
[![Chain of Responsibility in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Chain of Responsibility in C++"){.prog-lang-link}
[![Chain of Responsibility in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Chain of Responsibility in Go"){.prog-lang-link}
[![Chain of Responsibility in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Chain of Responsibility in Java"){.prog-lang-link}
[![Chain of Responsibility in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Chain of Responsibility in PHP"){.prog-lang-link}
[![Chain of Responsibility in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Chain of Responsibility in Python"){.prog-lang-link}
[![Chain of Responsibility in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Chain of Responsibility in Ruby"){.prog-lang-link}
[![Chain of Responsibility in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Chain of Responsibility in Rust"){.prog-lang-link}
[![Chain of Responsibility in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Chain of Responsibility in Swift"){.prog-lang-link}
[![Chain of Responsibility in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Chain of Responsibility in TypeScript"){.prog-lang-link}
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

[Command []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Behavioral
Patterns](behavioral-patterns.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
