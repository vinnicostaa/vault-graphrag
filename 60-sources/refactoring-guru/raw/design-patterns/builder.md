::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Creational
Patterns](creational-patterns.html){.type}
:::

# Builder {#builder .title}

::: {.section .intent}
##  Intent

**Builder** is a creational design pattern that lets you construct
complex objects step by step. The pattern allows you to produce
different types and representations of an object using the same
construction code.

<figure class="image">
<img
src="../images/patterns/content/builder/builder-en7fc8.png?id=617612423ea3752477dc90929115b3ee"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/builder/builder-en.png?id=617612423ea3752477dc90929115b3ee 640w,/images/patterns/content/builder/builder-en-1.5x.png?id=cfd4148e3bdc07d41a84b1a7a1d278f8 960w,/images/patterns/content/builder/builder-en-2x.png?id=8da2aa97abfdabf265e622579fc448a1 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Builder design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine a complex object that requires laborious, step-by-step
initialization of many fields and nested objects. Such initialization
code is usually buried inside a monstrous constructor with lots of
parameters. Or even worse: scattered all over the client code.

<figure class="image">
<img
src="../images/patterns/diagrams/builder/problem16274.png?id=11e715c5c97811f848c48e0f399bb05e"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem1.png?id=11e715c5c97811f848c48e0f399bb05e 600w,/images/patterns/diagrams/builder/problem1-1.5x.png?id=a46778018c4f0f4bbf2357940c1f5f40 900w,/images/patterns/diagrams/builder/problem1-2x.png?id=02ffbd7ad294600e42aa78989d441b4d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Lots of subclasses create another problem" />
<figcaption><p>You might make the program too complex by creating a
subclass for every possible configuration of an object.</p></figcaption>
</figure>

For example, let's think about how to create a `House` object. To build
a simple house, you need to construct four walls and a floor, install a
door, fit a pair of windows, and build a roof. But what if you want a
bigger, brighter house, with a backyard and other goodies (like a
heating system, plumbing, and electrical wiring)?

The simplest solution is to extend the base `House` class and create a
set of subclasses to cover all combinations of the parameters. But
eventually you'll end up with a considerable number of subclasses. Any
new parameter, such as the porch style, will require growing this
hierarchy even more.

There's another approach that doesn't involve breeding subclasses. You
can create a giant constructor right in the base `House` class with all
possible parameters that control the house object. While this approach
indeed eliminates the need for subclasses, it creates another problem.

<figure class="image">
<img
src="../images/patterns/diagrams/builder/problem27ab5.png?id=2e91039b6c7d2d2df6ee519983a3b036"
style="aspect-ratio: 1.71;"
srcset="/images/patterns/diagrams/builder/problem2.png?id=2e91039b6c7d2d2df6ee519983a3b036 600w,/images/patterns/diagrams/builder/problem2-1.5x.png?id=3d302cf2762fd04d70cbb91cb54e923c 900w,/images/patterns/diagrams/builder/problem2-2x.png?id=5e7975a91c0e4f4ba960f908cc9c2ea2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="The telescoping constructor" />
<figcaption><p>The constructor with lots of parameters has its downside:
not all the parameters are needed at all times.</p></figcaption>
</figure>

In most cases most of the parameters will be unused, making [the
constructor calls pretty ugly](../smells/long-parameter-list.html). For
instance, only a fraction of houses have swimming pools, so the
parameters related to swimming pools will be useless nine times out of
ten.
:::

::: {.section .solution}
##  Solution

The Builder pattern suggests that you extract the object construction
code out of its own class and move it to separate objects called
*builders*.

<figure class="image">
<img
src="../images/patterns/diagrams/builder/solution13e5f.png?id=8ce82137f8935998de802cae59e00e11"
style="aspect-ratio: 1.46;"
srcset="/images/patterns/diagrams/builder/solution1.png?id=8ce82137f8935998de802cae59e00e11 410w,/images/patterns/diagrams/builder/solution1-1.5x.png?id=8ab77eb73760a75c35940bae243d43f2 615w,/images/patterns/diagrams/builder/solution1-2x.png?id=a9c2ab02f0b2aca1a7512022194dd113 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Applying the Builder pattern" />
<figcaption><p>The Builder pattern lets you construct complex objects
step by step. The Builder doesn’t allow other objects to access the
product while it’s being built.</p></figcaption>
</figure>

The pattern organizes object construction into a set of steps
(`buildWalls`, `buildDoor`, etc.). To create an object, you execute a
series of these steps on a builder object. The important part is that
you don't need to call all of the steps. You can call only those steps
that are necessary for producing a particular configuration of an
object.

Some of the construction steps might require different implementation
when you need to build various representations of the product. For
example, walls of a cabin may be built of wood, but the castle walls
must be built with stone.

In this case, you can create several different builder classes that
implement the same set of building steps, but in a different manner.
Then you can use these builders in the construction process (i.e., an
ordered set of calls to the building steps) to produce different kinds
of objects.

<figure class="image">
<img
src="../images/patterns/content/builder/builder-comic-1-enb07c.png?id=605a699e1cb1241162db0530c7c1af4c"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/builder/builder-comic-1-en.png?id=605a699e1cb1241162db0530c7c1af4c 600w,/images/patterns/content/builder/builder-comic-1-en-1.5x.png?id=76cdbb286c9fa310e5b67c6ed25ad4e8 900w,/images/patterns/content/builder/builder-comic-1-en-2x.png?id=99728c9881fbf45fd3b6e0e3373935f1 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>Different builders execute the same task in
various ways.</p></figcaption>
</figure>

For example, imagine a builder that builds everything from wood and
glass, a second one that builds everything with stone and iron and a
third one that uses gold and diamonds. By calling the same set of steps,
you get a regular house from the first builder, a small castle from the
second and a palace from the third. However, this would only work if the
client code that calls the building steps is able to interact with
builders using a common interface.

#### Director

You can go further and extract a series of calls to the builder steps
you use to construct a product into a separate class called *director*.
The director class defines the order in which to execute the building
steps, while the builder provides the implementation for those steps.

<figure class="image">
<img
src="../images/patterns/content/builder/builder-comic-2-en1871.png?id=e19ff53e1494c06178476e2b8c068ec8"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/builder/builder-comic-2-en.png?id=e19ff53e1494c06178476e2b8c068ec8 343w,/images/patterns/content/builder/builder-comic-2-en-1.5x.png?id=dd6b9f1c3b74272f0eb68173eb653964 515w,/images/patterns/content/builder/builder-comic-2-en-2x.png?id=15035f2ea0317a93eca0177fc7ce2f22 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343" />
<figcaption><p>The director knows which building steps to execute to get
a working product.</p></figcaption>
</figure>

Having a director class in your program isn't strictly necessary. You
can always call the building steps in a specific order directly from the
client code. However, the director class might be a good place to put
various construction routines so you can reuse them across your program.

In addition, the director class completely hides the details of product
construction from the client code. The client only needs to associate a
builder with a director, launch the construction with the director, and
get the result from the builder.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/builder/structure05e4.png?id=fe9e23559923ea0657aa5fe75efef333"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.79;"
srcset="/images/patterns/diagrams/builder/structure.png?id=fe9e23559923ea0657aa5fe75efef333 460w,/images/patterns/diagrams/builder/structure-1.5x.png?id=91ea8cd3137b403542c23abf63938f00 690w,/images/patterns/diagrams/builder/structure-2x.png?id=dca1b1508e23c266cbedc80ffb84311a 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Structure of the Builder design pattern" /><img
src="../images/patterns/diagrams/builder/structure-indexedadac.png?id=44b3d763ce91dbada5d8394ef777437f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/builder/structure-indexed.png?id=44b3d763ce91dbada5d8394ef777437f 470w,/images/patterns/diagrams/builder/structure-indexed-1.5x.png?id=d57a6ff9342ea31736ea98e5066e0748 705w,/images/patterns/diagrams/builder/structure-indexed-2x.png?id=153eed9a51784cbe00d0ca8b3d6b268d 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Structure of the Builder design pattern" />
</figure>
:::

1.  The **Builder** interface declares product construction steps that
    are common to all types of builders.

2.  **Concrete Builders** provide different implementations of the
    construction steps. Concrete builders may produce products that
    don't follow the common interface.

3.  **Products** are resulting objects. Products constructed by
    different builders don't have to belong to the same class hierarchy
    or interface.

4.  The **Director** class defines the order in which to call
    construction steps, so you can create and reuse specific
    configurations of products.

5.  The **Client** must associate one of the builder objects with the
    director. Usually, it's done just once, via parameters of the
    director's constructor. Then the director uses that builder object
    for all further construction. However, there's an alternative
    approach for when the client passes the builder object to the
    production method of the director. In this case, you can use a
    different builder each time you produce something with the director.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

This example of the **Builder** pattern illustrates how you can reuse
the same object construction code when building different types of
products, such as cars, and create the corresponding manuals for them.

<figure class="image">
<img
src="../images/patterns/diagrams/builder/example-en2add.png?id=8386bdde073f993b5dfc4727697cf2fe"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/builder/example-en.png?id=8386bdde073f993b5dfc4727697cf2fe 500w,/images/patterns/diagrams/builder/example-en-1.5x.png?id=0f141ecb8991b743245f46a14b0cbffc 750w,/images/patterns/diagrams/builder/example-en-2x.png?id=d961b9d70bf16679f472119bbbe214da 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="The structure of the Builder pattern example" />
<figcaption><p>The example of step-by-step construction of cars and the
user guides that fit those car models.</p></figcaption>
</figure>

A car is a complex object that can be constructed in a hundred different
ways. Instead of bloating the `Car` class with a huge constructor, we
extracted the car assembly code into a separate car builder class. This
class has a set of methods for configuring various parts of a car.

If the client code needs to assemble a special, fine-tuned model of a
car, it can work with the builder directly. On the other hand, the
client can delegate the assembly to the director class, which knows how
to use a builder to construct several of the most popular models of
cars.

You might be shocked, but every car needs a manual (seriously, who reads
them?). The manual describes every feature of the car, so the details in
the manuals vary across the different models. That's why it makes sense
to reuse an existing construction process for both real cars and their
respective manuals. Of course, building a manual isn't the same as
building a car, and that's why we must provide another builder class
that specializes in composing manuals. This class implements the same
building methods as its car-building sibling, but instead of crafting
car parts, it describes them. By passing these builders to the same
director object, we can construct either a car or a manual.

The final part is fetching the resulting object. A metal car and a paper
manual, although related, are still very different things. We can't
place a method for fetching results in the director without coupling the
director to concrete product classes. Hence, we obtain the result of the
construction from the builder which performed the job.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Using the Builder pattern makes sense only when your products
// are quite complex and require extensive configuration. The
// following two products are related, although they don&#39;t have
// a common interface.
class Car is
    // A car can have a GPS, trip computer and some number of
    // seats. Different models of cars (sports car, SUV,
    // cabriolet) might have different features installed or
    // enabled.

class Manual is
    // Each car should have a user manual that corresponds to
    // the car&#39;s configuration and describes all its features.


// The builder interface specifies methods for creating the
// different parts of the product objects.
interface Builder is
    method reset()
    method setSeats(...)
    method setEngine(...)
    method setTripComputer(...)
    method setGPS(...)

// The concrete builder classes follow the builder interface and
// provide specific implementations of the building steps. Your
// program may have several variations of builders, each
// implemented differently.
class CarBuilder implements Builder is
    private field car:Car

    // A fresh builder instance should contain a blank product
    // object which it uses in further assembly.
    constructor CarBuilder() is
        this.reset()

    // The reset method clears the object being built.
    method reset() is
        this.car = new Car()

    // All production steps work with the same product instance.
    method setSeats(...) is
        // Set the number of seats in the car.

    method setEngine(...) is
        // Install a given engine.

    method setTripComputer(...) is
        // Install a trip computer.

    method setGPS(...) is
        // Install a global positioning system.

    // Concrete builders are supposed to provide their own
    // methods for retrieving results. That&#39;s because various
    // types of builders may create entirely different products
    // that don&#39;t all follow the same interface. Therefore such
    // methods can&#39;t be declared in the builder interface (at
    // least not in a statically-typed programming language).
    //
    // Usually, after returning the end result to the client, a
    // builder instance is expected to be ready to start
    // producing another product. That&#39;s why it&#39;s a usual
    // practice to call the reset method at the end of the
    // `getProduct` method body. However, this behavior isn&#39;t
    // mandatory, and you can make your builder wait for an
    // explicit reset call from the client code before disposing
    // of the previous result.
    method getProduct():Car is
        product = this.car
        this.reset()
        return product

// Unlike other creational patterns, builder lets you construct
// products that don&#39;t follow the common interface.
class CarManualBuilder implements Builder is
    private field manual:Manual

    constructor CarManualBuilder() is
        this.reset()

    method reset() is
        this.manual = new Manual()

    method setSeats(...) is
        // Document car seat features.

    method setEngine(...) is
        // Add engine instructions.

    method setTripComputer(...) is
        // Add trip computer instructions.

    method setGPS(...) is
        // Add GPS instructions.

    method getProduct():Manual is
        // Return the manual and reset the builder.


// The director is only responsible for executing the building
// steps in a particular sequence. It&#39;s helpful when producing
// products according to a specific order or configuration.
// Strictly speaking, the director class is optional, since the
// client can control builders directly.
class Director is
    // The director works with any builder instance that the
    // client code passes to it. This way, the client code may
    // alter the final type of the newly assembled product.
    // The director can construct several product variations
    // using the same building steps.
    method constructSportsCar(builder: Builder) is
        builder.reset()
        builder.setSeats(2)
        builder.setEngine(new SportEngine())
        builder.setTripComputer(true)
        builder.setGPS(true)

    method constructSUV(builder: Builder) is
        // ...


// The client code creates a builder object, passes it to the
// director and then initiates the construction process. The end
// result is retrieved from the builder object.
class Application is

    method makeCar() is
        director = new Director()

        CarBuilder builder = new CarBuilder()
        director.constructSportsCar(builder)
        Car car = builder.getProduct()

        CarManualBuilder builder = new CarManualBuilder()
        director.constructSportsCar(builder)

        // The final product is often retrieved from a builder
        // object since the director isn&#39;t aware of and not
        // dependent on concrete builders and products.
        Manual manual = builder.getProduct()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Builder pattern to get rid of a "telescoping constructor".
:::

::: applicability-solution
Say you have a constructor with ten optional parameters. Calling such a
beast is very inconvenient; therefore, you overload the constructor and
create several shorter versions with fewer parameters. These
constructors still refer to the main one, passing some default values
into any omitted parameters.

<figure class="code">
<pre class="code" lang="java"><code>class Pizza {
    Pizza(int size) { ... }
    Pizza(int size, boolean cheese) { ... }
    Pizza(int size, boolean cheese, boolean pepperoni) { ... }
    // ...</code></pre>
<figcaption><p>Creating such a monster is only possible in languages
that support method overloading, such as C# or Java.</p></figcaption>
</figure>

The Builder pattern lets you build objects step by step, using only
those steps that you really need. After implementing the pattern, you
don't have to cram dozens of parameters into your constructors anymore.
:::

::: applicability-problem
Use the Builder pattern when you want your code to be able to create
different representations of some product (for example, stone and wooden
houses).
:::

::: applicability-solution
The Builder pattern can be applied when construction of various
representations of the product involves similar steps that differ only
in the details.

The base builder interface defines all possible construction steps, and
concrete builders implement these steps to construct particular
representations of the product. Meanwhile, the director class guides the
order of construction.
:::

::: applicability-problem
Use the Builder to construct [Composite](composite.html) trees or other
complex objects.
:::

::: applicability-solution
The Builder pattern lets you construct products step-by-step. You could
defer execution of some steps without breaking the final product. You
can even call steps recursively, which comes in handy when you need to
build an object tree.

A builder doesn't expose the unfinished product while running
construction steps. This prevents the client code from fetching an
incomplete result.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Make sure that you can clearly define the common construction steps
    for building all available product representations. Otherwise, you
    won't be able to proceed with implementing the pattern.

2.  Declare these steps in the base builder interface.

3.  Create a concrete builder class for each of the product
    representations and implement their construction steps.

    Don't forget about implementing a method for fetching the result of
    the construction. The reason why this method can't be declared
    inside the builder interface is that various builders may construct
    products that don't have a common interface. Therefore, you don't
    know what would be the return type for such a method. However, if
    you're dealing with products from a single hierarchy, the fetching
    method can be safely added to the base interface.

4.  Think about creating a director class. It may encapsulate various
    ways to construct a product using the same builder object.

5.  The client code creates both the builder and the director objects.
    Before construction starts, the client must pass a builder object to
    the director. Usually, the client does this only once, via
    parameters of the director's class constructor. The director uses
    the builder object in all further construction. There's an
    alternative approach, where the builder is passed to a specific
    product construction method of the director.

6.  The construction result can be obtained directly from the director
    only if all products follow the same interface. Otherwise, the
    client should fetch the result from the builder.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can construct objects step-by-step, defer construction steps or
    run steps recursively.
-    You can reuse the same construction code when building various
    representations of products.
-    *Single Responsibility Principle*. You can isolate complex
    construction code from the business logic of the product.
:::

::: col-sm-6
-    The overall complexity of the code increases since the pattern
    requires creating multiple new classes.
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

-   [Builder](builder.html) focuses on constructing complex objects step
    by step. [Abstract Factory](abstract-factory.html) specializes in
    creating families of related objects. *Abstract Factory* returns the
    product immediately, whereas *Builder* lets you run some additional
    construction steps before fetching the product.

-   You can use [Builder](builder.html) when creating complex
    [Composite](composite.html) trees because you can program its
    construction steps to work recursively.

-   You can combine [Builder](builder.html) with [Bridge](bridge.html):
    the director class plays the role of the abstraction, while
    different builders act as implementations.

-   [Abstract Factories](abstract-factory.html),
    [Builders](builder.html) and [Prototypes](prototype.html) can all be
    implemented as [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Builder in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](builder/csharp/example.html "Builder in C#"){.prog-lang-link}
[![Builder in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](builder/cpp/example.html "Builder in C++"){.prog-lang-link}
[![Builder in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](builder/go/example.html "Builder in Go"){.prog-lang-link}
[![Builder in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](builder/java/example.html "Builder in Java"){.prog-lang-link}
[![Builder in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](builder/php/example.html "Builder in PHP"){.prog-lang-link}
[![Builder in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](builder/python/example.html "Builder in Python"){.prog-lang-link}
[![Builder in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](builder/ruby/example.html "Builder in Ruby"){.prog-lang-link}
[![Builder in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](builder/rust/example.html "Builder in Rust"){.prog-lang-link}
[![Builder in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](builder/swift/example.html "Builder in Swift"){.prog-lang-link}
[![Builder in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](builder/typescript/example.html "Builder in TypeScript"){.prog-lang-link}
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

[Prototype []{.fa .fa-arrow-right}](prototype.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Factory
Comparison](factory-comparison.html){.btn .btn-default rel="prev"}
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
