:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Creational
Patterns](creational-patterns.html){.type}
:::

# Prototype {#prototype .title}

::: aka
Also known as: [Clone]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Prototype** is a creational design pattern that lets you copy existing
objects without making your code dependent on their classes.

<figure class="image">
<img
src="../images/patterns/content/prototype/prototype3cb9.png?id=e912b1ada20bbf7b2ffc09e93b9fab20"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/prototype/prototype.png?id=e912b1ada20bbf7b2ffc09e93b9fab20 640w,/images/patterns/content/prototype/prototype-1.5x.png?id=a0f76894fb657783b65b9ed899857468 960w,/images/patterns/content/prototype/prototype-2x.png?id=670789c80c8a114e25838ede2da4a881 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Prototype Design Pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Say you have an object, and you want to create an exact copy of it. How
would you do it? First, you have to create a new object of the same
class. Then you have to go through all the fields of the original object
and copy their values over to the new object.

Nice! But there's a catch. Not all objects can be copied that way
because some of the object's fields may be private and not visible from
outside of the object itself.

<figure class="image">
<img
src="../images/patterns/content/prototype/prototype-comic-1-ene940.png?id=4cc45ae42e26cc9533a6ac540713d1fa"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-1-en.png?id=4cc45ae42e26cc9533a6ac540713d1fa 600w,/images/patterns/content/prototype/prototype-comic-1-en-1.5x.png?id=6c44fe035234a3432e7181bb73db1c80 900w,/images/patterns/content/prototype/prototype-comic-1-en-2x.png?id=1f5b9eeb00df663f4630ca6d38c4804d 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="What can go wrong when copying things &quot;from the outside&quot;?" />
<figcaption><p>Copying an object “from the outside” <a
href="https://vimeo.com/120816425">isn’t</a>
always possible.</p></figcaption>
</figure>

There's one more problem with the direct approach. Since you have to
know the object's class to create a duplicate, your code becomes
dependent on that class. If the extra dependency doesn't scare you,
there's another catch. Sometimes you only know the interface that the
object follows, but not its concrete class, when, for example, a
parameter in a method accepts any objects that follow some interface.
:::

::: {.section .solution}
##  Solution

The Prototype pattern delegates the cloning process to the actual
objects that are being cloned. The pattern declares a common interface
for all objects that support cloning. This interface lets you clone an
object without coupling your code to the class of that object. Usually,
such an interface contains just a single `clone` method.

The implementation of the `clone` method is very similar in all classes.
The method creates an object of the current class and carries over all
of the field values of the old object into the new one. You can even
copy private fields because most programming languages let objects
access private fields of other objects that belong to the same class.

An object that supports cloning is called a *prototype*. When your
objects have dozens of fields and hundreds of possible configurations,
cloning them might serve as an alternative to subclassing.

<figure class="image">
<img
src="../images/patterns/content/prototype/prototype-comic-2-en87c7.png?id=e1df2dc39404c5eb2d485b7ae7c9914f"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/content/prototype/prototype-comic-2-en.png?id=e1df2dc39404c5eb2d485b7ae7c9914f 343w,/images/patterns/content/prototype/prototype-comic-2-en-1.5x.png?id=11f0e701b7e78777025bedaaf036f6a3 515w,/images/patterns/content/prototype/prototype-comic-2-en-2x.png?id=dda1589983832b69aee763037293beab 687w"
sizes="(max-width: 720px) 100vw, 343px" loading="lazy" width="343"
alt="Pre-built prototypes" />
<figcaption><p>Pre-built prototypes can be an alternative
to subclassing.</p></figcaption>
</figure>

Here's how it works: you create a set of objects, configured in various
ways. When you need an object like the one you've configured, you just
clone a prototype instead of constructing a new object from scratch.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

In real life, prototypes are used for performing various tests before
starting mass production of a product. However, in this case, prototypes
don't participate in any actual production, playing a passive role
instead.

<figure class="image">
<img
src="../images/patterns/content/prototype/prototype-comic-3-en47fb.png?id=ff16dedbd0c10944d27bf87d2adcf8a6"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/prototype/prototype-comic-3-en.png?id=ff16dedbd0c10944d27bf87d2adcf8a6 600w,/images/patterns/content/prototype/prototype-comic-3-en-1.5x.png?id=308d1e79f37742ce29555d4b4c18b4c2 900w,/images/patterns/content/prototype/prototype-comic-3-en-2x.png?id=63dd16812865486d174b646882effd9a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="The cell division" />
<figcaption><p>The division of a cell.</p></figcaption>
</figure>

Since industrial prototypes don't really copy themselves, a much closer
analogy to the pattern is the process of mitotic cell division (biology,
remember?). After mitotic division, a pair of identical cells is formed.
The original cell acts as a prototype and takes an active role in
creating the copy.
:::

:::::::: {.section .structure-container}
##  Structure

::::::: structure
#### Basic implementation

:::: prototype-normal
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/prototype/structure87f6.png?id=088102c5e9785ff45debbbce86f4df81"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.25;"
srcset="/images/patterns/diagrams/prototype/structure.png?id=088102c5e9785ff45debbbce86f4df81 500w,/images/patterns/diagrams/prototype/structure-1.5x.png?id=beec6a1a5242268e10e39f9fdc0b894b 750w,/images/patterns/diagrams/prototype/structure-2x.png?id=ba75079f42f08028ae4cdbda0cfecc26 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="The structure of the Prototype design pattern" /><img
src="../images/patterns/diagrams/prototype/structure-indexed6499.png?id=0e1c809842f5c43aca0541a2eba1f844"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/prototype/structure-indexed.png?id=0e1c809842f5c43aca0541a2eba1f844 520w,/images/patterns/diagrams/prototype/structure-indexed-1.5x.png?id=05b072b5b83f5ff1024a7ba448ea9d59 780w,/images/patterns/diagrams/prototype/structure-indexed-2x.png?id=74584ac729c0f6b49d2a97a53cc266ff 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="The structure of the Prototype design pattern" />
</figure>
:::
::::

1.  The **Prototype** interface declares the cloning methods. In most
    cases, it's a single `clone` method.

2.  The **Concrete Prototype** class implements the cloning method. In
    addition to copying the original object's data to the clone, this
    method may also handle some edge cases of the cloning process
    related to cloning linked objects, untangling recursive
    dependencies, etc.

3.  The **Client** can produce a copy of any object that follows the
    prototype interface.

#### Prototype registry implementation

:::: prototype-pool
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/prototype/structure-prototype-cache9d19.png?id=609c2af5d14ed55dcbb218a00f98e7d5"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache.png?id=609c2af5d14ed55dcbb218a00f98e7d5 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-1.5x.png?id=8ca0b829185d49c5acbe19966a0659d4 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-2x.png?id=a1e4514bbcc9b10968b856f19b407105 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="The prototype registry" /><img
src="../images/patterns/diagrams/prototype/structure-prototype-cache-indexed1801.png?id=10a4a84a1a318f59dbc2b806fc936d04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/prototype/structure-prototype-cache-indexed.png?id=10a4a84a1a318f59dbc2b806fc936d04 550w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-1.5x.png?id=cb56c95533a4020368c48db9f9e2a37d 825w,/images/patterns/diagrams/prototype/structure-prototype-cache-indexed-2x.png?id=47b99eb7ae51158bdbb31deea4f5e98f 1100w"
sizes="(max-width: 720px) 100vw, 550px" loading="lazy" width="550"
alt="The prototype registry" />
</figure>
:::
::::

1.  The **Prototype Registry** provides an easy way to access
    frequently-used prototypes. It stores a set of pre-built objects
    that are ready to be copied. The simplest prototype registry is a
    `name → prototype` hash map. However, if you need better search
    criteria than a simple name, you can build a much more robust
    version of the registry.
:::::::
::::::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Prototype** pattern lets you produce exact copies
of geometric objects, without coupling the code to their classes.

<figure class="image">
<img
src="../images/patterns/diagrams/prototype/examplee491.png?id=47bc6c1058cb100b81e675b5ca6bda6c"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/prototype/example.png?id=47bc6c1058cb100b81e675b5ca6bda6c 470w,/images/patterns/diagrams/prototype/example-1.5x.png?id=067f3a2415370fadf5f92aadaa12b5e2 705w,/images/patterns/diagrams/prototype/example-2x.png?id=80393e0df17ae8130e5ada832d494949 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="The structure of the Prototype pattern example" />
<figcaption><p>Cloning a set of objects that belong to a
class hierarchy.</p></figcaption>
</figure>

All shape classes follow the same interface, which provides a cloning
method. A subclass may call the parent's cloning method before copying
its own field values to the resulting object.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Base prototype.
abstract class Shape is
    field X: int
    field Y: int
    field color: string

    // A regular constructor.
    constructor Shape() is
        // ...

    // The prototype constructor. A fresh object is initialized
    // with values from the existing object.
    constructor Shape(source: Shape) is
        this()
        this.X = source.X
        this.Y = source.Y
        this.color = source.color

    // The clone operation returns one of the Shape subclasses.
    abstract method clone():Shape


// Concrete prototype. The cloning method creates a new object
// in one go by calling the constructor of the current class and
// passing the current object as the constructor&#39;s argument.
// Performing all the actual copying in the constructor helps to
// keep the result consistent: the constructor will not return a
// result until the new object is fully built; thus, no object
// can have a reference to a partially-built clone.
class Rectangle extends Shape is
    field width: int
    field height: int

    constructor Rectangle(source: Rectangle) is
        // A parent constructor call is needed to copy private
        // fields defined in the parent class.
        super(source)
        this.width = source.width
        this.height = source.height

    method clone():Shape is
        return new Rectangle(this)


class Circle extends Shape is
    field radius: int

    constructor Circle(source: Circle) is
        super(source)
        this.radius = source.radius

    method clone():Shape is
        return new Circle(this)


// Somewhere in the client code.
class Application is
    field shapes: array of Shape

    constructor Application() is
        Circle circle = new Circle()
        circle.X = 10
        circle.Y = 10
        circle.radius = 20
        shapes.add(circle)

        Circle anotherCircle = circle.clone()
        shapes.add(anotherCircle)
        // The `anotherCircle` variable contains an exact copy
        // of the `circle` object.

        Rectangle rectangle = new Rectangle()
        rectangle.width = 10
        rectangle.height = 20
        shapes.add(rectangle)

    method businessLogic() is
        // Prototype rocks because it lets you produce a copy of
        // an object without knowing anything about its type.
        Array shapesCopy = new Array of Shapes.

        // For instance, we don&#39;t know the exact elements in the
        // shapes array. All we know is that they are all
        // shapes. But thanks to polymorphism, when we call the
        // `clone` method on a shape the program checks its real
        // class and runs the appropriate clone method defined
        // in that class. That&#39;s why we get proper clones
        // instead of a set of simple Shape objects.
        foreach (s in shapes) do
            shapesCopy.add(s.clone())

        // The `shapesCopy` array contains exact copies of the
        // `shape` array&#39;s children.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Prototype pattern when your code shouldn't depend on the
concrete classes of objects that you need to copy.
:::

::: applicability-solution
This happens a lot when your code works with objects passed to you from
3rd-party code via some interface. The concrete classes of these objects
are unknown, and you couldn't depend on them even if you wanted to.

The Prototype pattern provides the client code with a general interface
for working with all objects that support cloning. This interface makes
the client code independent from the concrete classes of objects that it
clones.
:::

::: applicability-problem
Use the pattern when you want to reduce the number of subclasses that
only differ in the way they initialize their respective objects.
:::

::: applicability-solution
Suppose you have a complex class that requires a laborious configuration
before it can be used. There are several common ways to configure this
class, and this code is scattered through your app. To reduce the
duplication, you create several subclasses and put every common
configuration code into their constructors. You solved the duplication
problem, but now you have lots of dummy subclasses.

The Prototype pattern lets you use a set of pre-built objects configured
in various ways as prototypes. Instead of instantiating a subclass that
matches some configuration, the client can simply look for an
appropriate prototype and clone it.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Create the prototype interface and declare the `clone` method in it.
    Or just add the method to all classes of an existing class
    hierarchy, if you have one.

2.  A prototype class must define the alternative constructor that
    accepts an object of that class as an argument. The constructor must
    copy the values of all fields defined in the class from the passed
    object into the newly created instance. If you're changing a
    subclass, you must call the parent constructor to let the superclass
    handle the cloning of its private fields.

    If your programming language doesn't support method overloading, you
    won't be able to create a separate "prototype" constructor. Thus,
    copying the object's data into the newly created clone will have to
    be performed within the `clone` method. Still, having this code in a
    regular constructor is safer because the resulting object is
    returned fully configured right after you call the `new` operator.

3.  The cloning method usually consists of just one line: running a
    `new` operator with the prototypical version of the constructor.
    Note, that every class must explicitly override the cloning method
    and use its own class name along with the `new` operator. Otherwise,
    the cloning method may produce an object of a parent class.

4.  Optionally, create a centralized prototype registry to store a
    catalog of frequently used prototypes.

    You can implement the registry as a new factory class or put it in
    the base prototype class with a static method for fetching the
    prototype. This method should search for a prototype based on search
    criteria that the client code passes to the method. The criteria
    might either be a simple string tag or a complex set of search
    parameters. After the appropriate prototype is found, the registry
    should clone it and return the copy to the client.

    Finally, replace the direct calls to the subclasses' constructors
    with calls to the factory method of the prototype registry.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can clone objects without coupling to their concrete classes.
-    You can get rid of repeated initialization code in favor of cloning
    pre-built prototypes.
-    You can produce complex objects more conveniently.
-    You get an alternative to inheritance when dealing with
    configuration presets for complex objects.
:::

::: col-sm-6
-    Cloning complex objects that have circular references might be very
    tricky.
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

-   [Prototype](prototype.html) can help when you need to save copies of
    [Commands](command.html) into history.

-   Designs that make heavy use of [Composite](composite.html) and
    [Decorator](decorator.html) can often benefit from using
    [Prototype](prototype.html). Applying the pattern lets you clone
    complex structures instead of re-constructing them from scratch.

-   [Prototype](prototype.html) isn't based on inheritance, so it
    doesn't have its drawbacks. On the other hand, *Prototype* requires
    a complicated initialization of the cloned object. [Factory
    Method](factory-method.html) is based on inheritance but doesn't
    require an initialization step.

-   Sometimes [Prototype](prototype.html) can be a simpler alternative
    to [Memento](memento.html). This works if the object, the state of
    which you want to store in the history, is fairly straightforward
    and doesn't have links to external resources, or the links are easy
    to re-establish.

-   [Abstract Factories](abstract-factory.html),
    [Builders](builder.html) and [Prototypes](prototype.html) can all be
    implemented as [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Prototype in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](prototype/csharp/example.html "Prototype in C#"){.prog-lang-link}
[![Prototype in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](prototype/cpp/example.html "Prototype in C++"){.prog-lang-link}
[![Prototype in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](prototype/go/example.html "Prototype in Go"){.prog-lang-link}
[![Prototype in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](prototype/java/example.html "Prototype in Java"){.prog-lang-link}
[![Prototype in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](prototype/php/example.html "Prototype in PHP"){.prog-lang-link}
[![Prototype in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](prototype/python/example.html "Prototype in Python"){.prog-lang-link}
[![Prototype in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](prototype/ruby/example.html "Prototype in Ruby"){.prog-lang-link}
[![Prototype in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](prototype/rust/example.html "Prototype in Rust"){.prog-lang-link}
[![Prototype in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](prototype/swift/example.html "Prototype in Swift"){.prog-lang-link}
[![Prototype in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](prototype/typescript/example.html "Prototype in TypeScript"){.prog-lang-link}
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

[Singleton []{.fa .fa-arrow-right}](singleton.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Builder](builder.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
