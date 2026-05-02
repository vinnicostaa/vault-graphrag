::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Observer {#observer .title}

::: aka
Also known as:
[Event-Subscriber, ]{style="display:inline-block"}[Listener]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Observer** is a behavioral design pattern that lets you define a
subscription mechanism to notify multiple objects about any events that
happen to the object they're observing.

<figure class="image">
<img
src="../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Observer Design Pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you have two types of objects: a `Customer` and a `Store`.
The customer is very interested in a particular brand of product (say,
it's a new model of the iPhone) which should become available in the
store very soon.

The customer could visit the store every day and check product
availability. But while the product is still en route, most of these
trips would be pointless.

<figure class="image">
<img
src="../images/patterns/content/observer/observer-comic-1-enaf15.png?id=1ec8571b22ea8fd2ed537f06cc763152"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-en.png?id=1ec8571b22ea8fd2ed537f06cc763152 600w,/images/patterns/content/observer/observer-comic-1-en-1.5x.png?id=4be5e56d6fa2a3ebbed9a16c109c3a4a 900w,/images/patterns/content/observer/observer-comic-1-en-2x.png?id=8e89674eb83b01e82203987e600ba59e 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Visiting store vs. sending spam" />
<figcaption><p>Visiting the store vs. sending spam</p></figcaption>
</figure>

On the other hand, the store could send tons of emails (which might be
considered spam) to all customers each time a new product becomes
available. This would save some customers from endless trips to the
store. At the same time, it'd upset other customers who aren't
interested in new products.

It looks like we've got a conflict. Either the customer wastes time
checking product availability or the store wastes resources notifying
the wrong customers.
:::

::: {.section .solution}
##  Solution

The object that has some interesting state is often called *subject*,
but since it's also going to notify other objects about the changes to
its state, we'll call it *publisher*. All other objects that want to
track changes to the publisher's state are called *subscribers*.

The Observer pattern suggests that you add a subscription mechanism to
the publisher class so individual objects can subscribe to or
unsubscribe from a stream of events coming from that publisher. Fear
not! Everything isn't as complicated as it sounds. In reality, this
mechanism consists of 1) an array field for storing a list of references
to subscriber objects and 2) several public methods which allow adding
subscribers to and removing them from that list.

<figure class="image">
<img
src="../images/patterns/diagrams/observer/solution1-en847d.png?id=60fb9a2822649dec1c68b78733479c57"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-en.png?id=60fb9a2822649dec1c68b78733479c57 470w,/images/patterns/diagrams/observer/solution1-en-1.5x.png?id=dee761b815f7ca11febafb78cb7ab157 705w,/images/patterns/diagrams/observer/solution1-en-2x.png?id=a6bc643488b8fbc8bbb309539139c316 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Subscription mechanism" />
<figcaption><p>A subscription mechanism lets individual objects
subscribe to event notifications.</p></figcaption>
</figure>

Now, whenever an important event happens to the publisher, it goes over
its subscribers and calls the specific notification method on their
objects.

Real apps might have dozens of different subscriber classes that are
interested in tracking events of the same publisher class. You wouldn't
want to couple the publisher to all of those classes. Besides, you might
not even know about some of them beforehand if your publisher class is
supposed to be used by other people.

That's why it's crucial that all subscribers implement the same
interface and that the publisher communicates with them only via that
interface. This interface should declare the notification method along
with a set of parameters that the publisher can use to pass some
contextual data along with the notification.

<figure class="image">
<img
src="../images/patterns/diagrams/observer/solution2-en0a63.png?id=fcea7791ac77b6ecb6fea2c2b4128d4a"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-en.png?id=fcea7791ac77b6ecb6fea2c2b4128d4a 460w,/images/patterns/diagrams/observer/solution2-en-1.5x.png?id=829bd5830aa52fbbd6ab31d64193b667 690w,/images/patterns/diagrams/observer/solution2-en-2x.png?id=630cfb84753c258aa4e8500e189c0b65 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Notification methods" />
<figcaption><p>Publisher notifies subscribers by calling the specific
notification method on their objects.</p></figcaption>
</figure>

If your app has several different types of publishers and you want to
make your subscribers compatible with all of them, you can go even
further and make all publishers follow the same interface. This
interface would only need to describe a few subscription methods. The
interface would allow subscribers to observe publishers' states without
coupling to their concrete classes.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/observer/observer-comic-2-en6c7d.png?id=a9be31ab5f90e47b0f250fe9821c34c5"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-en.png?id=a9be31ab5f90e47b0f250fe9821c34c5 600w,/images/patterns/content/observer/observer-comic-2-en-1.5x.png?id=62681e0ee15ef2c1a55dc900e3b3f844 900w,/images/patterns/content/observer/observer-comic-2-en-2x.png?id=2147046fb16c427533db8ed85e8cce4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Magazine and newspaper subscriptions" />
<figcaption><p>Magazine and newspaper subscriptions.</p></figcaption>
</figure>

If you subscribe to a newspaper or magazine, you no longer need to go to
the store to check if the next issue is available. Instead, the
publisher sends new issues directly to your mailbox right after
publication or even in advance.

The publisher maintains a list of subscribers and knows which magazines
they're interested in. Subscribers can leave the list at any time when
they wish to stop the publisher sending new magazine issues to them.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Structure of the Observer design pattern" /><img
src="../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Structure of the Observer design pattern" />
</figure>
:::

1.  The **Publisher** issues events of interest to other objects. These
    events occur when the publisher changes its state or executes some
    behaviors. Publishers contain a subscription infrastructure that
    lets new subscribers join and current subscribers leave the list.

2.  When a new event happens, the publisher goes over the subscription
    list and calls the notification method declared in the subscriber
    interface on each subscriber object.

3.  The **Subscriber** interface declares the notification interface. In
    most cases, it consists of a single `update` method. The method may
    have several parameters that let the publisher pass some event
    details along with the update.

4.  **Concrete Subscribers** perform some actions in response to
    notifications issued by the publisher. All of these classes must
    implement the same interface so the publisher isn't coupled to
    concrete classes.

5.  Usually, subscribers need some contextual information to handle the
    update correctly. For this reason, publishers often pass some
    context data as arguments of the notification method. The publisher
    can pass itself as an argument, letting subscriber fetch any
    required data directly.

6.  The **Client** creates publisher and subscriber objects separately
    and then registers subscribers for publisher updates.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Observer** pattern lets the text editor object
notify other service objects about changes in its state.

<figure class="image">
<img
src="../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Structure of the Observer pattern example" />
<figcaption><p>Notifying objects about events that happen to
other objects.</p></figcaption>
</figure>

The list of subscribers is compiled dynamically: objects can start or
stop listening to notifications at runtime, depending on the desired
behavior of your app.

In this implementation, the editor class doesn't maintain the
subscription list by itself. It delegates this job to the special helper
object devoted to just that. You could upgrade that object to serve as a
centralized event dispatcher, letting any object act as a publisher.

Adding new subscribers to the program doesn't require changes to
existing publisher classes, as long as they work with all subscribers
through the same interface.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The base publisher class includes subscription management
// code and notification methods.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// The concrete publisher contains real business logic that&#39;s
// interesting for some subscribers. We could derive this class
// from the base publisher, but that isn&#39;t always possible in
// real life because the concrete publisher might already be a
// subclass. In this case, you can patch the subscription logic
// in with composition, as we did here.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Methods of business logic can notify subscribers about
    // changes.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)

    // ...


// Here&#39;s the subscriber interface. If your programming language
// supports functional types, you can replace the whole
// subscriber hierarchy with a set of functions.
interface EventListener is
    method update(filename)

// Concrete subscribers react to updates issued by the publisher
// they are attached to.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// An application can configure publishers and subscribers at
// runtime.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened the file: %s&quot;)
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Observer pattern when changes to the state of one object may
require changing other objects, and the actual set of objects is unknown
beforehand or changes dynamically.
:::

::: applicability-solution
You can often experience this problem when working with classes of the
graphical user interface. For example, you created custom button
classes, and you want to let the clients hook some custom code to your
buttons so that it fires whenever a user presses a button.

The Observer pattern lets any object that implements the subscriber
interface subscribe for event notifications in publisher objects. You
can add the subscription mechanism to your buttons, letting the clients
hook up their custom code via custom subscriber classes.
:::

::: applicability-problem
Use the pattern when some objects in your app must observe others, but
only for a limited time or in specific cases.
:::

::: applicability-solution
The subscription list is dynamic, so subscribers can join or leave the
list whenever they need to.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Look over your business logic and try to break it down into two
    parts: the core functionality, independent from other code, will act
    as the publisher; the rest will turn into a set of subscriber
    classes.

2.  Declare the subscriber interface. At a bare minimum, it should
    declare a single `update` method.

3.  Declare the publisher interface and describe a pair of methods for
    adding a subscriber object to and removing it from the list.
    Remember that publishers must work with subscribers only via the
    subscriber interface.

4.  Decide where to put the actual subscription list and the
    implementation of subscription methods. Usually, this code looks the
    same for all types of publishers, so the obvious place to put it is
    in an abstract class derived directly from the publisher interface.
    Concrete publishers extend that class, inheriting the subscription
    behavior.

    However, if you're applying the pattern to an existing class
    hierarchy, consider an approach based on composition: put the
    subscription logic into a separate object, and make all real
    publishers use it.

5.  Create concrete publisher classes. Each time something important
    happens inside a publisher, it must notify all its subscribers.

6.  Implement the update notification methods in concrete subscriber
    classes. Most subscribers would need some context data about the
    event. It can be passed as an argument of the notification method.

    But there's another option. Upon receiving a notification, the
    subscriber can fetch any data directly from the notification. In
    this case, the publisher must pass itself via the update method. The
    less flexible option is to link a publisher to the subscriber
    permanently via the constructor.

7.  The client must create all necessary subscribers and register them
    with proper publishers.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    *Open/Closed Principle*. You can introduce new subscriber classes
    without having to change the publisher's code (and vice versa if
    there's a publisher interface).
-    You can establish relations between objects at runtime.
:::

::: col-sm-6
-    Subscribers are notified in random order.
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

-   The difference between [Mediator](mediator.html) and
    [Observer](observer.html) is often elusive. In most cases, you can
    implement either of these patterns; but sometimes you can apply both
    simultaneously. Let's see how we can do that.

    The primary goal of *Mediator* is to eliminate mutual dependencies
    among a set of system components. Instead, these components become
    dependent on a single mediator object. The goal of *Observer* is to
    establish dynamic one-way connections between objects, where some
    objects act as subordinates of others.

    There's a popular implementation of the *Mediator* pattern that
    relies on *Observer*. The mediator object plays the role of
    publisher, and the components act as subscribers which subscribe to
    and unsubscribe from the mediator's events. When *Mediator* is
    implemented this way, it may look very similar to *Observer*.

    When you're confused, remember that you can implement the Mediator
    pattern in other ways. For example, you can permanently link all the
    components to the same mediator object. This implementation won't
    resemble *Observer* but will still be an instance of the Mediator
    pattern.

    Now imagine a program where all components have become publishers,
    allowing dynamic connections between each other. There won't be a
    centralized mediator object, only a distributed set of observers.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Observer in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Observer in C#"){.prog-lang-link}
[![Observer in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Observer in C++"){.prog-lang-link}
[![Observer in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Observer in Go"){.prog-lang-link}
[![Observer in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Observer in Java"){.prog-lang-link}
[![Observer in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Observer in PHP"){.prog-lang-link}
[![Observer in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Observer in Python"){.prog-lang-link}
[![Observer in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Observer in Ruby"){.prog-lang-link}
[![Observer in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Observer in Rust"){.prog-lang-link}
[![Observer in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Observer in Swift"){.prog-lang-link}
[![Observer in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Observer in TypeScript"){.prog-lang-link}
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

[State []{.fa .fa-arrow-right}](state.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Memento](memento.html){.btn .btn-default
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
