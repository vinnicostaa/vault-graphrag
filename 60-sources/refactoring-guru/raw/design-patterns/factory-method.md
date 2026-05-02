::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Creational
Patterns](creational-patterns.html){.type}
:::

# Factory Method {#factory-method .title}

::: aka
Also known as: [Virtual Constructor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Factory Method** is a creational design pattern that provides an
interface for creating objects in a superclass, but allows subclasses to
alter the type of objects that will be created.

<figure class="image">
<img
src="../images/patterns/content/factory-method/factory-method-enfb9e.png?id=cfa26f33dc8473e803fadae0d262100a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/factory-method/factory-method-en.png?id=cfa26f33dc8473e803fadae0d262100a 640w,/images/patterns/content/factory-method/factory-method-en-1.5x.png?id=d035a9f21f1ee88860cc44bf63d0e1c0 960w,/images/patterns/content/factory-method/factory-method-en-2x.png?id=b3961995a4449fb90820a693013511df 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Factory Method pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you're creating a logistics management application. The
first version of your app can only handle transportation by trucks, so
the bulk of your code lives inside the `Truck` class.

After a while, your app becomes pretty popular. Each day you receive
dozens of requests from sea transportation companies to incorporate sea
logistics into the app.

<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/problem1-en51bc.png?id=de638561be0bbb1025ada6bfcb815def"
style="aspect-ratio: 2.4;"
srcset="/images/patterns/diagrams/factory-method/problem1-en.png?id=de638561be0bbb1025ada6bfcb815def 600w,/images/patterns/diagrams/factory-method/problem1-en-1.5x.png?id=a46ffaf7114d367522cabda6012404bf 900w,/images/patterns/diagrams/factory-method/problem1-en-2x.png?id=9a4959d9dde4edadf809be33d3da0c74 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Adding a new transportation class to the program causes an issue" />
<figcaption><p>Adding a new class to the program isn’t that simple if
the rest of the code is already coupled to
existing classes.</p></figcaption>
</figure>

Great news, right? But how about the code? At present, most of your code
is coupled to the `Truck` class. Adding `Ships` into the app would
require making changes to the entire codebase. Moreover, if later you
decide to add another type of transportation to the app, you will
probably need to make all of these changes again.

As a result, you will end up with pretty nasty code, riddled with
conditionals that switch the app's behavior depending on the class of
transportation objects.
:::

::: {.section .solution}
##  Solution

The Factory Method pattern suggests that you replace direct object
construction calls (using the `new` operator) with calls to a special
*factory* method. Don't worry: the objects are still created via the
`new` operator, but it's being called from within the factory method.
Objects returned by a factory method are often referred to as
*products.*

<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/solution16ec2.png?id=fc756d2af296b5b4d482e548214d08ef"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/factory-method/solution1.png?id=fc756d2af296b5b4d482e548214d08ef 620w,/images/patterns/diagrams/factory-method/solution1-1.5x.png?id=22d3b6bb74e966d1cb3a4d8f380cefa3 930w,/images/patterns/diagrams/factory-method/solution1-2x.png?id=c482b3cd7a4d8dd73b4c8c12dfcaa03c 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="The structure of creator classes" />
<figcaption><p>Subclasses can alter the class of objects being returned
by the factory method.</p></figcaption>
</figure>

At first glance, this change may look pointless: we just moved the
constructor call from one part of the program to another. However,
consider this: now you can override the factory method in a subclass and
change the class of products being created by the method.

There's a slight limitation though: subclasses may return different
types of products only if these products have a common base class or
interface. Also, the factory method in the base class should have its
return type declared as this interface.

<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/solution2-en23b8.png?id=db5de848c1d490b835666ef54d131d46"
style="aspect-ratio: 1.96;"
srcset="/images/patterns/diagrams/factory-method/solution2-en.png?id=db5de848c1d490b835666ef54d131d46 490w,/images/patterns/diagrams/factory-method/solution2-en-1.5x.png?id=eac87ef445435f4ed86bf271679c009d 735w,/images/patterns/diagrams/factory-method/solution2-en-2x.png?id=1209a3156e450b9d7c437ca6bb98b219 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="The structure of the products hierarchy" />
<figcaption><p>All products must follow the
same interface.</p></figcaption>
</figure>

For example, both `Truck` and `Ship` classes should implement the
`Transport` interface, which declares a method called `deliver`. Each
class implements this method differently: trucks deliver cargo by land,
ships deliver cargo by sea. The factory method in the `RoadLogistics`
class returns truck objects, whereas the factory method in the
`SeaLogistics` class returns ships.

<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/solution3-en8f03.png?id=b6f53911fc0d56f9ef99501fc4aec059"
style="aspect-ratio: 1.83;"
srcset="/images/patterns/diagrams/factory-method/solution3-en.png?id=b6f53911fc0d56f9ef99501fc4aec059 640w,/images/patterns/diagrams/factory-method/solution3-en-1.5x.png?id=f66d5c45f61b619d513a12b7e0a755d5 960w,/images/patterns/diagrams/factory-method/solution3-en-2x.png?id=542c0ba89e91ac11ea79e94bc0229f70 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="The structure of the code after applying the factory method pattern" />
<figcaption><p>As long as all product classes implement a common
interface, you can pass their objects to the client code without
breaking it.</p></figcaption>
</figure>

The code that uses the factory method (often called the *client* code)
doesn't see a difference between the actual products returned by various
subclasses. The client treats all the products as abstract `Transport`.
The client knows that all transport objects are supposed to have the
`deliver` method, but exactly how it works isn't important to the
client.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/structure78c3.png?id=4cba0803f42517cfe8548c9bc7dc4c9b"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure.png?id=4cba0803f42517cfe8548c9bc7dc4c9b 660w,/images/patterns/diagrams/factory-method/structure-1.5x.png?id=ece8300890afc14494770a6b6d370428 990w,/images/patterns/diagrams/factory-method/structure-2x.png?id=9ea3aa8a47f8be22e12e523c15b448fd 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="The structure of the Factory Method pattern" /><img
src="../images/patterns/diagrams/factory-method/structure-indexed6136.png?id=4c603207859ca1f939b17b60a3a2e9e0"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/factory-method/structure-indexed.png?id=4c603207859ca1f939b17b60a3a2e9e0 660w,/images/patterns/diagrams/factory-method/structure-indexed-1.5x.png?id=626b6d7f577e1c265cce244678b2cf7f 990w,/images/patterns/diagrams/factory-method/structure-indexed-2x.png?id=c794e4f2d05013fb176464a1d1a5d7ab 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="The structure of the Factory Method pattern" />
</figure>
:::

1.  The **Product** declares the interface, which is common to all
    objects that can be produced by the creator and its subclasses.

2.  **Concrete Products** are different implementations of the product
    interface.

3.  The **Creator** class declares the factory method that returns new
    product objects. It's important that the return type of this method
    matches the product interface.

    You can declare the factory method as `abstract` to force all
    subclasses to implement their own versions of the method. As an
    alternative, the base factory method can return some default product
    type.

    Note, despite its name, product creation is **not** the primary
    responsibility of the creator. Usually, the creator class already
    has some core business logic related to products. The factory method
    helps to decouple this logic from the concrete product classes. Here
    is an analogy: a large software development company can have a
    training department for programmers. However, the primary function
    of the company as a whole is still writing code, not producing
    programmers.

4.  **Concrete Creators** override the base factory method so it returns
    a different type of product.

    Note that the factory method doesn't have to **create** new
    instances all the time. It can also return existing objects from a
    cache, an object pool, or another source.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

This example illustrates how the **Factory Method** can be used for
creating cross-platform UI elements without coupling the client code to
concrete UI classes.

<figure class="image">
<img
src="../images/patterns/diagrams/factory-method/examplec59f.png?id=67db9a5cb817913444efcb1c067c9835"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/factory-method/example.png?id=67db9a5cb817913444efcb1c067c9835 640w,/images/patterns/diagrams/factory-method/example-1.5x.png?id=921d97276e5e2fd8e64609c9cfa021e7 960w,/images/patterns/diagrams/factory-method/example-2x.png?id=a2470830778e318263155000dbdc5870 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="The structure of the Factory Method pattern example" />
<figcaption><p>The cross-platform dialog example.</p></figcaption>
</figure>

The base `Dialog` class uses different UI elements to render its window.
Under various operating systems, these elements may look a little bit
different, but they should still behave consistently. A button in
Windows is still a button in Linux.

When the factory method comes into play, you don't need to rewrite the
logic of the `Dialog` class for each operating system. If we declare a
factory method that produces buttons inside the base `Dialog` class, we
can later create a subclass that returns Windows-styled buttons from the
factory method. The subclass then inherits most of the code from the
base class, but, thanks to the factory method, can render
Windows-looking buttons on the screen.

For this pattern to work, the base `Dialog` class must work with
abstract buttons: a base class or an interface that all concrete buttons
follow. This way the code within `Dialog` remains functional, whichever
type of buttons it works with.

Of course, you can apply this approach to other UI elements as well.
However, with each new factory method you add to the `Dialog`, you get
closer to the [Abstract Factory](abstract-factory.html) pattern. Fear
not, we'll talk about this pattern later.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The creator class declares the factory method that must
// return an object of a product class. The creator&#39;s subclasses
// usually provide the implementation of this method.
class Dialog is
    // The creator may also provide some default implementation
    // of the factory method.
    abstract method createButton():Button

    // Note that, despite its name, the creator&#39;s primary
    // responsibility isn&#39;t creating products. It usually
    // contains some core business logic that relies on product
    // objects returned by the factory method. Subclasses can
    // indirectly change that business logic by overriding the
    // factory method and returning a different type of product
    // from it.
    method render() is
        // Call the factory method to create a product object.
        Button okButton = createButton()
        // Now use the product.
        okButton.onClick(closeDialog)
        okButton.render()


// Concrete creators override the factory method to change the
// resulting product&#39;s type.
class WindowsDialog extends Dialog is
    method createButton():Button is
        return new WindowsButton()

class WebDialog extends Dialog is
    method createButton():Button is
        return new HTMLButton()


// The product interface declares the operations that all
// concrete products must implement.
interface Button is
    method render()
    method onClick(f)

// Concrete products provide various implementations of the
// product interface.
class WindowsButton implements Button is
    method render(a, b) is
        // Render a button in Windows style.
    method onClick(f) is
        // Bind a native OS click event.

class HTMLButton implements Button is
    method render(a, b) is
        // Return an HTML representation of a button.
    method onClick(f) is
        // Bind a web browser click event.


class Application is
    field dialog: Dialog

    // The application picks a creator&#39;s type depending on the
    // current configuration or environment settings.
    method initialize() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            dialog = new WindowsDialog()
        else if (config.OS == &quot;Web&quot;) then
            dialog = new WebDialog()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

    // The client code works with an instance of a concrete
    // creator, albeit through its base interface. As long as
    // the client keeps working with the creator via the base
    // interface, you can pass it any creator&#39;s subclass.
    method main() is
        this.initialize()
        dialog.render()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Factory Method when you don't know beforehand the exact types
and dependencies of the objects your code should work with.
:::

::: applicability-solution
The Factory Method separates product construction code from the code
that actually uses the product. Therefore it's easier to extend the
product construction code independently from the rest of the code.

For example, to add a new product type to the app, you'll only need to
create a new creator subclass and override the factory method in it.
:::

::: applicability-problem
Use the Factory Method when you want to provide users of your library or
framework with a way to extend its internal components.
:::

::: applicability-solution
Inheritance is probably the easiest way to extend the default behavior
of a library or framework. But how would the framework recognize that
your subclass should be used instead of a standard component?

The solution is to reduce the code that constructs components across the
framework into a single factory method and let anyone override this
method in addition to extending the component itself.

Let's see how that would work. Imagine that you write an app using an
open source UI framework. Your app should have round buttons, but the
framework only provides square ones. You extend the standard `Button`
class with a glorious `RoundButton` subclass. But now you need to tell
the main `UIFramework` class to use the new button subclass instead of a
default one.

To achieve this, you create a subclass `UIWithRoundButtons` from a base
framework class and override its `createButton` method. While this
method returns `Button` objects in the base class, you make your
subclass return `RoundButton` objects. Now use the `UIWithRoundButtons`
class instead of `UIFramework`. And that's about it!
:::

::: applicability-problem
Use the Factory Method when you want to save system resources by reusing
existing objects instead of rebuilding them each time.
:::

::: applicability-solution
You often experience this need when dealing with large,
resource-intensive objects such as database connections, file systems,
and network resources.

Let's think about what has to be done to reuse an existing object:

1.  First, you need to create some storage to keep track of all of the
    created objects.
2.  When someone requests an object, the program should look for a free
    object inside that pool.
3.  ... and then return it to the client code.
4.  If there are no free objects, the program should create a new one
    (and add it to the pool).

That's a lot of code! And it must all be put into a single place so that
you don't pollute the program with duplicate code.

Probably the most obvious and convenient place where this code could be
placed is the constructor of the class whose objects we're trying to
reuse. However, a constructor must always return **new objects** by
definition. It can't return existing instances.

Therefore, you need to have a regular method capable of creating new
objects as well as reusing existing ones. That sounds very much like a
factory method.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Make all products follow the same interface. This interface should
    declare methods that make sense in every product.

2.  Add an empty factory method inside the creator class. The return
    type of the method should match the common product interface.

3.  In the creator's code find all references to product constructors.
    One by one, replace them with calls to the factory method, while
    extracting the product creation code into the factory method.

    You might need to add a temporary parameter to the factory method to
    control the type of returned product.

    At this point, the code of the factory method may look pretty ugly.
    It may have a large `switch` statement that picks which product
    class to instantiate. But don't worry, we'll fix it soon enough.

4.  Now, create a set of creator subclasses for each type of product
    listed in the factory method. Override the factory method in the
    subclasses and extract the appropriate bits of construction code
    from the base method.

5.  If there are too many product types and it doesn't make sense to
    create subclasses for all of them, you can reuse the control
    parameter from the base class in subclasses.

    For instance, imagine that you have the following hierarchy of
    classes: the base `Mail` class with a couple of subclasses:
    `AirMail` and `GroundMail`; the `Transport` classes are `Plane`,
    `Truck` and `Train`. While the `AirMail` class only uses `Plane`
    objects, `GroundMail` may work with both `Truck` and `Train`
    objects. You can create a new subclass (say `TrainMail`) to handle
    both cases, but there's another option. The client code can pass an
    argument to the factory method of the `GroundMail` class to control
    which product it wants to receive.

6.  If, after all of the extractions, the base factory method has become
    empty, you can make it abstract. If there's something left, you can
    make it a default behavior of the method.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You avoid tight coupling between the creator and the concrete
    products.
-    *Single Responsibility Principle*. You can move the product
    creation code into one place in the program, making the code easier
    to support.
-    *Open/Closed Principle*. You can introduce new types of products
    into the program without breaking existing client code.
:::

::: col-sm-6
-    The code may become more complicated since you need to introduce a
    lot of new subclasses to implement the pattern. The best case
    scenario is when you're introducing the pattern into an existing
    hierarchy of creator classes.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   Many designs start by using [Factory Method](factory-method.html)
    (less complicated and more customizable via subclasses) and evolve
    toward [Abstract Factory](abstract-factory.html),
    [Prototype](prototype.html), or [Builder](builder.html) (more
    flexible, but more complicated).

-   [Abstract Factory](abstract-factory.html) classes are often based on
    a set of [Factory Methods](factory-method.html), but you can also
    use [Prototype](prototype.html) to compose the methods on these
    classes.

-   You can use [Factory Method](factory-method.html) along with
    [Iterator](iterator.html) to let collection subclasses return
    different types of iterators that are compatible with the
    collections.

-   [Prototype](prototype.html) isn't based on inheritance, so it
    doesn't have its drawbacks. On the other hand, *Prototype* requires
    a complicated initialization of the cloned object. [Factory
    Method](factory-method.html) is based on inheritance but doesn't
    require an initialization step.

-   [Factory Method](factory-method.html) is a specialization of
    [Template Method](template-method.html). At the same time, a
    *Factory Method* may serve as a step in a large *Template Method*.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Factory Method in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](factory-method/csharp/example.html "Factory Method in C#"){.prog-lang-link}
[![Factory Method in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](factory-method/cpp/example.html "Factory Method in C++"){.prog-lang-link}
[![Factory Method in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](factory-method/go/example.html "Factory Method in Go"){.prog-lang-link}
[![Factory Method in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](factory-method/java/example.html "Factory Method in Java"){.prog-lang-link}
[![Factory Method in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](factory-method/php/example.html "Factory Method in PHP"){.prog-lang-link}
[![Factory Method in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](factory-method/python/example.html "Factory Method in Python"){.prog-lang-link}
[![Factory Method in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](factory-method/ruby/example.html "Factory Method in Ruby"){.prog-lang-link}
[![Factory Method in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](factory-method/rust/example.html "Factory Method in Rust"){.prog-lang-link}
[![Factory Method in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](factory-method/swift/example.html "Factory Method in Swift"){.prog-lang-link}
[![Factory Method in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](factory-method/typescript/example.html "Factory Method in TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Extra Content {#extras}

-   Read our [Factory Comparison](factory-comparison.html) if you can't
    figure out the difference between various factory patterns and
    concepts.
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

[Abstract Factory []{.fa .fa-arrow-right}](abstract-factory.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Creational
Patterns](creational-patterns.html){.btn .btn-default rel="prev"}
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
