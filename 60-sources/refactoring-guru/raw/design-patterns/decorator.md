::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Structural
Patterns](structural-patterns.html){.type}
:::

# Decorator {#decorator .title}

::: aka
Also known as: [Wrapper]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Decorator** is a structural design pattern that lets you attach new
behaviors to objects by placing these objects inside special wrapper
objects that contain the behaviors.

<figure class="image">
<img
src="../images/patterns/content/decorator/decoratore8f5.png?id=710c66670c7123e0928d3b3758aea79e"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/decorator/decorator.png?id=710c66670c7123e0928d3b3758aea79e 640w,/images/patterns/content/decorator/decorator-1.5x.png?id=72aafc603a289fc64e028e83e8aa843b 960w,/images/patterns/content/decorator/decorator-2x.png?id=736ab07b1d8920ab2c7a70c9cb1305cc 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Decorator design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you're working on a notification library which lets other
programs notify their users about important events.

The initial version of the library was based on the `Notifier` class
that had only a few fields, a constructor and a single `send` method.
The method could accept a message argument from a client and send the
message to a list of emails that were passed to the notifier via its
constructor. A third-party app which acted as a client was supposed to
create and configure the notifier object once, and then use it each time
something important happened.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/problem1-en1fd8.png?id=7658efddaaf43acb64ac63a92025cc1e"
style="aspect-ratio: 2.45;"
srcset="/images/patterns/diagrams/decorator/problem1-en.png?id=7658efddaaf43acb64ac63a92025cc1e 540w,/images/patterns/diagrams/decorator/problem1-en-1.5x.png?id=484df9e28899cb97fdd0816e087815d3 810w,/images/patterns/diagrams/decorator/problem1-en-2x.png?id=0bf0496ca959de8698bee735e4e62aac 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure of the library before applying the Decorator pattern" />
<figcaption><p>A program could use the notifier class to send
notifications about important events to a predefined set
of emails.</p></figcaption>
</figure>

At some point, you realize that users of the library expect more than
just email notifications. Many of them would like to receive an SMS
about critical issues. Others would like to be notified on Facebook and,
of course, the corporate users would love to get Slack notifications.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/problem2a2f8.png?id=ba5d5e106ea8c4848d60e230feca9135"
style="aspect-ratio: 2.59;"
srcset="/images/patterns/diagrams/decorator/problem2.png?id=ba5d5e106ea8c4848d60e230feca9135 440w,/images/patterns/diagrams/decorator/problem2-1.5x.png?id=4f15b6208533188dd09e7da7dd6b509a 660w,/images/patterns/diagrams/decorator/problem2-2x.png?id=28b2c8509b4e78c031d728424b876ebc 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Structure of the library after implementing other notification types" />
<figcaption><p>Each notification type is implemented as a
notifier’s subclass.</p></figcaption>
</figure>

How hard can that be? You extended the `Notifier` class and put the
additional notification methods into new subclasses. Now the client was
supposed to instantiate the desired notification class and use it for
all further notifications.

But then someone reasonably asked you, "Why can't you use several
notification types at once? If your house is on fire, you'd probably
want to be informed through every channel."

You tried to address that problem by creating special subclasses which
combined several notification methods within one class. However, it
quickly became apparent that this approach would bloat the code
immensely, not only the library code but the client code as well.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/problem34033.png?id=f3b3e7a107d870871f2c3167adcb7ccb"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/decorator/problem3.png?id=f3b3e7a107d870871f2c3167adcb7ccb 630w,/images/patterns/diagrams/decorator/problem3-1.5x.png?id=f4ef9d367df838c77121e9f84260b09c 945w,/images/patterns/diagrams/decorator/problem3-2x.png?id=abb7a87b521ce97d7661dd9c0b988cc3 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Structure of the library after creating class combinations" />
<figcaption><p>Combinatorial explosion of subclasses.</p></figcaption>
</figure>

You have to find some other way to structure notifications classes so
that their number won't accidentally break some Guinness record.
:::

::: {.section .solution}
##  Solution

Extending a class is the first thing that comes to mind when you need to
alter an object's behavior. However, inheritance has several serious
caveats that you need to be aware of.

-   Inheritance is static. You can't alter the behavior of an existing
    object at runtime. You can only replace the whole object with
    another one that's created from a different subclass.
-   Subclasses can have just one parent class. In most languages,
    inheritance doesn't let a class inherit behaviors of multiple
    classes at the same time.

One of the ways to overcome these caveats is by using *Aggregation* or
*Composition* []{.pop-i}[*Aggregation*: object A contains objects B; B
can live without A.\
*Composition*: object A consists of objects B; A manages life cycle of
B; B can't live without A.]{.pop-content} instead of *Inheritance*. Both
of the alternatives work almost the same way: one object *has a*
reference to another and delegates it some work, whereas with
inheritance, the object itself *is* able to do that work, inheriting the
behavior from its superclass.

With this new approach you can easily substitute the linked "helper"
object with another, changing the behavior of the container at runtime.
An object can use the behavior of various classes, having references to
multiple objects and delegating them all kinds of work.
Aggregation/composition is the key principle behind many design
patterns, including Decorator. On that note, let's return to the pattern
discussion.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/solution1-enb68c.png?id=468e68f1e9ae21649d63dd454500741d"
style="aspect-ratio: 3.44;"
srcset="/images/patterns/diagrams/decorator/solution1-en.png?id=468e68f1e9ae21649d63dd454500741d 550w,/images/patterns/diagrams/decorator/solution1-en-1.5x.png?id=7c9b30ff8e2af1f6354cb144a67f38d5 825w,/images/patterns/diagrams/decorator/solution1-en-2x.png?id=0acaa7d75290a1647f5402bc5d1c03e7 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="Inheritance vs. Aggregation" />
<figcaption><p>Inheritance vs. Aggregation</p></figcaption>
</figure>

"Wrapper" is the alternative nickname for the Decorator pattern that
clearly expresses the main idea of the pattern. A *wrapper* is an object
that can be linked with some *target* object. The wrapper contains the
same set of methods as the target and delegates to it all requests it
receives. However, the wrapper may alter the result by doing something
either before or after it passes the request to the target.

When does a simple wrapper become the real decorator? As I mentioned,
the wrapper implements the same interface as the wrapped object. That's
why from the client's perspective these objects are identical. Make the
wrapper's reference field accept any object that follows that interface.
This will let you cover an object in multiple wrappers, adding the
combined behavior of all the wrappers to it.

In our notifications example, let's leave the simple email notification
behavior inside the base `Notifier` class, but turn all other
notification methods into decorators.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/solution2e4ab.png?id=cbee4a27080ce3a0bf773482613e1347"
style="aspect-ratio: 1.39;"
srcset="/images/patterns/diagrams/decorator/solution2.png?id=cbee4a27080ce3a0bf773482613e1347 640w,/images/patterns/diagrams/decorator/solution2-1.5x.png?id=d2fa0c0b9bf5cba0e38be7aff7e7b7a8 960w,/images/patterns/diagrams/decorator/solution2-2x.png?id=7775f76b94dbd5cd25f711ce81f59262 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="The solution with the Decorator pattern" />
<figcaption><p>Various notification methods
become decorators.</p></figcaption>
</figure>

The client code would need to wrap a basic notifier object into a set of
decorators that match the client's preferences. The resulting objects
will be structured as a stack.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/solution3-enf2ff.png?id=b7e2e2036435265350ba0c6796162ab5"
style="aspect-ratio: 1.03;"
srcset="/images/patterns/diagrams/decorator/solution3-en.png?id=b7e2e2036435265350ba0c6796162ab5 300w,/images/patterns/diagrams/decorator/solution3-en-1.5x.png?id=3c9b7a4aec3a8d2f68d64bffb79fa4f7 450w,/images/patterns/diagrams/decorator/solution3-en-2x.png?id=9a4ef2b4267685a83d0233d78775497b 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="Apps might configure complex stacks of notification decorators" />
<figcaption><p>Apps might configure complex stacks of
notification decorators.</p></figcaption>
</figure>

The last decorator in the stack would be the object that the client
actually works with. Since all decorators implement the same interface
as the base notifier, the rest of the client code won't care whether it
works with the "pure" notifier object or the decorated one.

We could apply the same approach to other behaviors such as formatting
messages or composing the recipient list. The client can decorate the
object with any custom decorators, as long as they follow the same
interface as the others.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/decorator/decorator-comic-1cbff.png?id=80d95baacbfb91f5bcdbdc7814b0c64d"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/decorator/decorator-comic-1.png?id=80d95baacbfb91f5bcdbdc7814b0c64d 600w,/images/patterns/content/decorator/decorator-comic-1-1.5x.png?id=490ef9754d7554c2046b69f6f213c6da 900w,/images/patterns/content/decorator/decorator-comic-1-2x.png?id=ba869f621b6e0ea173fdc2b535fc7eed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Example of the Decorator pattern" />
<figcaption><p>You get a combined effect from wearing multiple pieces
of clothing.</p></figcaption>
</figure>

Wearing clothes is an example of using decorators. When you're cold, you
wrap yourself in a sweater. If you're still cold with a sweater, you can
wear a jacket on top. If it's raining, you can put on a raincoat. All of
these garments "extend" your basic behavior but aren't part of you, and
you can easily take off any piece of clothing whenever you don't need
it.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/decorator/structure128f.png?id=8c95d894aecce5315cc1b12093a7ea0c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/decorator/structure.png?id=8c95d894aecce5315cc1b12093a7ea0c 480w,/images/patterns/diagrams/decorator/structure-1.5x.png?id=4b2cd91d4483d9e3bba725f0e45d281d 720w,/images/patterns/diagrams/decorator/structure-2x.png?id=3cfa1f10417a4ef0c12580bc4a63b80d 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Structure of the Decorator design pattern" /><img
src="../images/patterns/diagrams/decorator/structure-indexeda93c.png?id=09401b230a58f2249e4c9a1195d485a0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/structure-indexed.png?id=09401b230a58f2249e4c9a1195d485a0 500w,/images/patterns/diagrams/decorator/structure-indexed-1.5x.png?id=dccc54182965078ccb4cfdeee41acbe5 750w,/images/patterns/diagrams/decorator/structure-indexed-2x.png?id=2733e7d0e322bfb2f150ccf8a878dbf6 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Structure of the Decorator design pattern" />
</figure>
:::

1.  The **Component** declares the common interface for both wrappers
    and wrapped objects.

2.  **Concrete Component** is a class of objects being wrapped. It
    defines the basic behavior, which can be altered by decorators.

3.  The **Base Decorator** class has a field for referencing a wrapped
    object. The field's type should be declared as the component
    interface so it can contain both concrete components and decorators.
    The base decorator delegates all operations to the wrapped object.

4.  **Concrete Decorators** define extra behaviors that can be added to
    components dynamically. Concrete decorators override methods of the
    base decorator and execute their behavior either before or after
    calling the parent method.

5.  The **Client** can wrap components in multiple layers of decorators,
    as long as it works with all objects via the component interface.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Decorator** pattern lets you compress and encrypt
sensitive data independently from the code that actually uses this data.

<figure class="image">
<img
src="../images/patterns/diagrams/decorator/example42cb.png?id=eec9dc488f00c85f50e764323baa723e"
style="aspect-ratio: 0.98;"
srcset="/images/patterns/diagrams/decorator/example.png?id=eec9dc488f00c85f50e764323baa723e 540w,/images/patterns/diagrams/decorator/example-1.5x.png?id=40ccaba4f5a8e6a2ebac697f04b9f10a 810w,/images/patterns/diagrams/decorator/example-2x.png?id=4891323a27d5601a174eec366187d833 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure of the Decorator pattern example" />
<figcaption><p>The encryption and compression
decorators example.</p></figcaption>
</figure>

The application wraps the data source object with a pair of decorators.
Both wrappers change the way the data is written to and read from the
disk:

-   Just before the data is **written to disk**, the decorators encrypt
    and compress it. The original class writes the encrypted and
    protected data to the file without knowing about the change.

-   Right after the data is **read from disk**, it goes through the same
    decorators, which decompress and decode it.

The decorators and the data source class implement the same interface,
which makes them all interchangeable in the client code.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The component interface defines operations that can be
// altered by decorators.
interface DataSource is
    method writeData(data)
    method readData():data

// Concrete components provide default implementations for the
// operations. There might be several variations of these
// classes in a program.
class FileDataSource implements DataSource is
    constructor FileDataSource(filename) { ... }

    method writeData(data) is
        // Write data to file.

    method readData():data is
        // Read data from file.

// The base decorator class follows the same interface as the
// other components. The primary purpose of this class is to
// define the wrapping interface for all concrete decorators.
// The default implementation of the wrapping code might include
// a field for storing a wrapped component and the means to
// initialize it.
class DataSourceDecorator implements DataSource is
    protected field wrappee: DataSource

    constructor DataSourceDecorator(source: DataSource) is
        wrappee = source

    // The base decorator simply delegates all work to the
    // wrapped component. Extra behaviors can be added in
    // concrete decorators.
    method writeData(data) is
        wrappee.writeData(data)

    // Concrete decorators may call the parent implementation of
    // the operation instead of calling the wrapped object
    // directly. This approach simplifies extension of decorator
    // classes.
    method readData():data is
        return wrappee.readData()

// Concrete decorators must call methods on the wrapped object,
// but may add something of their own to the result. Decorators
// can execute the added behavior either before or after the
// call to a wrapped object.
class EncryptionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Encrypt passed data.
        // 2. Pass encrypted data to the wrappee&#39;s writeData
        // method.

    method readData():data is
        // 1. Get data from the wrappee&#39;s readData method.
        // 2. Try to decrypt it if it&#39;s encrypted.
        // 3. Return the result.

// You can wrap objects in several layers of decorators.
class CompressionDecorator extends DataSourceDecorator is
    method writeData(data) is
        // 1. Compress passed data.
        // 2. Pass compressed data to the wrappee&#39;s writeData
        // method.

    method readData():data is
        // 1. Get data from the wrappee&#39;s readData method.
        // 2. Try to decompress it if it&#39;s compressed.
        // 3. Return the result.


// Option 1. A simple example of a decorator assembly.
class Application is
    method dumbUsageExample() is
        source = new FileDataSource(&quot;somefile.dat&quot;)
        source.writeData(salaryRecords)
        // The target file has been written with plain data.

        source = new CompressionDecorator(source)
        source.writeData(salaryRecords)
        // The target file has been written with compressed
        // data.

        source = new EncryptionDecorator(source)
        // The source variable now contains this:
        // Encryption &gt; Compression &gt; FileDataSource
        source.writeData(salaryRecords)
        // The file has been written with compressed and
        // encrypted data.


// Option 2. Client code that uses an external data source.
// SalaryManager objects neither know nor care about data
// storage specifics. They work with a pre-configured data
// source received from the app configurator.
class SalaryManager is
    field source: DataSource

    constructor SalaryManager(source: DataSource) { ... }

    method load() is
        return source.readData()

    method save() is
        source.writeData(salaryRecords)
    // ...Other useful methods...


// The app can assemble different stacks of decorators at
// runtime, depending on the configuration or environment.
class ApplicationConfigurator is
    method configurationExample() is
        source = new FileDataSource(&quot;salary.dat&quot;)
        if (enabledEncryption)
            source = new EncryptionDecorator(source)
        if (enabledCompression)
            source = new CompressionDecorator(source)

        logger = new SalaryManager(source)
        salary = logger.load()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Decorator pattern when you need to be able to assign extra
behaviors to objects at runtime without breaking the code that uses
these objects.
:::

::: applicability-solution
The Decorator lets you structure your business logic into layers, create
a decorator for each layer and compose objects with various combinations
of this logic at runtime. The client code can treat all these objects in
the same way, since they all follow a common interface.
:::

::: applicability-problem
Use the pattern when it's awkward or not possible to extend an object's
behavior using inheritance.
:::

::: applicability-solution
Many programming languages have the `final` keyword that can be used to
prevent further extension of a class. For a final class, the only way to
reuse the existing behavior would be to wrap the class with your own
wrapper, using the Decorator pattern.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Make sure your business domain can be represented as a primary
    component with multiple optional layers over it.

2.  Figure out what methods are common to both the primary component and
    the optional layers. Create a component interface and declare those
    methods there.

3.  Create a concrete component class and define the base behavior in
    it.

4.  Create a base decorator class. It should have a field for storing a
    reference to a wrapped object. The field should be declared with the
    component interface type to allow linking to concrete components as
    well as decorators. The base decorator must delegate all work to the
    wrapped object.

5.  Make sure all classes implement the component interface.

6.  Create concrete decorators by extending them from the base
    decorator. A concrete decorator must execute its behavior before or
    after the call to the parent method (which always delegates to the
    wrapped object).

7.  The client code must be responsible for creating decorators and
    composing them in the way the client needs.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can extend an object's behavior without making a new subclass.
-    You can add or remove responsibilities from an object at runtime.
-    You can combine several behaviors by wrapping an object into
    multiple decorators.
-    *Single Responsibility Principle*. You can divide a monolithic
    class that implements many possible variants of behavior into
    several smaller classes.
:::

::: col-sm-6
-    It's hard to remove a specific wrapper from the wrappers stack.
-    It's hard to implement a decorator in such a way that its behavior
    doesn't depend on the order in the decorators stack.
-    The initial configuration code of layers might look pretty ugly.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   [Adapter](adapter.html) provides a completely different interface
    for accessing an existing object. On the other hand, with the
    [Decorator](decorator.html) pattern the interface either stays the
    same or gets extended. In addition, *Decorator* supports recursive
    composition, which isn't possible when you use *Adapter*.

-   With [Adapter](adapter.html) you access an existing object via
    different interface. With [Proxy](proxy.html), the interface stays
    the same. With [Decorator](decorator.html) you access the object via
    an enhanced interface.

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

-   [Composite](composite.html) and [Decorator](decorator.html) have
    similar structure diagrams since both rely on recursive composition
    to organize an open-ended number of objects.

    A *Decorator* is like a *Composite* but only has one child
    component. There's another significant difference: *Decorator* adds
    additional responsibilities to the wrapped object, while *Composite*
    just "sums up" its children's results.

    However, the patterns can also cooperate: you can use *Decorator* to
    extend the behavior of a specific object in the *Composite* tree.

-   Designs that make heavy use of [Composite](composite.html) and
    [Decorator](decorator.html) can often benefit from using
    [Prototype](prototype.html). Applying the pattern lets you clone
    complex structures instead of re-constructing them from scratch.

-   [Decorator](decorator.html) lets you change the skin of an object,
    while [Strategy](strategy.html) lets you change the guts.

-   [Decorator](decorator.html) and [Proxy](proxy.html) have similar
    structures, but very different intents. Both patterns are built on
    the composition principle, where one object is supposed to delegate
    some of the work to another. The difference is that a *Proxy*
    usually manages the life cycle of its service object on its own,
    whereas the composition of *Decorators* is always controlled by the
    client.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Decorator in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](decorator/csharp/example.html "Decorator in C#"){.prog-lang-link}
[![Decorator in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](decorator/cpp/example.html "Decorator in C++"){.prog-lang-link}
[![Decorator in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](decorator/go/example.html "Decorator in Go"){.prog-lang-link}
[![Decorator in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](decorator/java/example.html "Decorator in Java"){.prog-lang-link}
[![Decorator in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](decorator/php/example.html "Decorator in PHP"){.prog-lang-link}
[![Decorator in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](decorator/python/example.html "Decorator in Python"){.prog-lang-link}
[![Decorator in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](decorator/ruby/example.html "Decorator in Ruby"){.prog-lang-link}
[![Decorator in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](decorator/rust/example.html "Decorator in Rust"){.prog-lang-link}
[![Decorator in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](decorator/swift/example.html "Decorator in Swift"){.prog-lang-link}
[![Decorator in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](decorator/typescript/example.html "Decorator in TypeScript"){.prog-lang-link}
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

[Facade []{.fa .fa-arrow-right}](facade.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Composite](composite.html){.btn .btn-default
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
