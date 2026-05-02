:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Creational
Patterns](creational-patterns.html){.type}
:::

# Abstract Factory {#abstract-factory .title}

::: {.section .intent}
##  Intent

**Abstract Factory** is a creational design pattern that lets you
produce families of related objects without specifying their
concrete classes.

<figure class="image">
<img
src="../images/patterns/content/abstract-factory/abstract-factory-ene22f.png?id=d0210ee255712a245fead94a3fafabe0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-en.png?id=d0210ee255712a245fead94a3fafabe0 640w,/images/patterns/content/abstract-factory/abstract-factory-en-1.5x.png?id=c0cb95bcdeda5fc9df4840e8b6316ac3 960w,/images/patterns/content/abstract-factory/abstract-factory-en-2x.png?id=a488ca862db731876fa0513bb2105640 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Abstract Factory pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you're creating a furniture shop simulator. Your code
consists of classes that represent:

1.  A family of related products, say: `Chair` + `Sofa` + `CoffeeTable`.

2.  Several variants of this family. For example, products `Chair` +
    `Sofa` + `CoffeeTable` are available in these variants: `Modern`,
    `Victorian`, `ArtDeco`.

<figure class="image">
<img
src="../images/patterns/diagrams/abstract-factory/problem-en62f5.png?id=e38c307511e684828be898de02d6c268"
style="aspect-ratio: 1.4;"
srcset="/images/patterns/diagrams/abstract-factory/problem-en.png?id=e38c307511e684828be898de02d6c268 420w,/images/patterns/diagrams/abstract-factory/problem-en-1.5x.png?id=6055a66f7a4f424792a7a07cf116f55d 630w,/images/patterns/diagrams/abstract-factory/problem-en-2x.png?id=7de667bc24583c3ac03ccb80f3613cfe 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="Product families and their variants." />
<figcaption><p>Product families and their variants.</p></figcaption>
</figure>

You need a way to create individual furniture objects so that they match
other objects of the same family. Customers get quite mad when they
receive non-matching furniture.

<figure class="image">
<img
src="../images/patterns/content/abstract-factory/abstract-factory-comic-1-en9ab4.png?id=f4012920c5034122eedbb0c9fec0cdb3"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-1-en.png?id=f4012920c5034122eedbb0c9fec0cdb3 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-en-1.5x.png?id=00f48d1c1854a6973a10d5488a82122a 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-1-en-2x.png?id=e2d4e7bbdd41899a3a85ebefa88bca3e 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>A Modern-style sofa doesn’t match
Victorian-style chairs.</p></figcaption>
</figure>

Also, you don't want to change existing code when adding new products or
families of products to the program. Furniture vendors update their
catalogs very often, and you wouldn't want to change the core code each
time it happens.
:::

::: {.section .solution}
##  Solution

The first thing the Abstract Factory pattern suggests is to explicitly
declare interfaces for each distinct product of the product family
(e.g., chair, sofa or coffee table). Then you can make all variants of
products follow those interfaces. For example, all chair variants can
implement the `Chair` interface; all coffee table variants can implement
the `CoffeeTable` interface, and so on.

<figure class="image">
<img
src="../images/patterns/diagrams/abstract-factory/solution1a1ac.png?id=71f2018d8bb443b9cce90d57110675e3"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/abstract-factory/solution1.png?id=71f2018d8bb443b9cce90d57110675e3 420w,/images/patterns/diagrams/abstract-factory/solution1-1.5x.png?id=4366e971c850514cde5d33cb7956de8b 630w,/images/patterns/diagrams/abstract-factory/solution1-2x.png?id=eacec286153e058db9255d26541c0529 840w"
sizes="(max-width: 720px) 100vw, 420px" loading="lazy" width="420"
alt="The Chairs class hierarchy" />
<figcaption><p>All variants of the same object must be moved to a single
class hierarchy.</p></figcaption>
</figure>

The next move is to declare the *Abstract Factory*---an interface with a
list of creation methods for all products that are part of the product
family (for example, `createChair`, `createSofa` and
`createCoffeeTable`). These methods must return **abstract** product
types represented by the interfaces we extracted previously: `Chair`,
`Sofa`, `CoffeeTable` and so on.

<figure class="image">
<img
src="../images/patterns/diagrams/abstract-factory/solution24d8e.png?id=53975d6e4714c6f942633a879f7ac571"
style="aspect-ratio: 2;"
srcset="/images/patterns/diagrams/abstract-factory/solution2.png?id=53975d6e4714c6f942633a879f7ac571 640w,/images/patterns/diagrams/abstract-factory/solution2-1.5x.png?id=581a6cdc0dcd5511f1c30e509b1d4a7f 960w,/images/patterns/diagrams/abstract-factory/solution2-2x.png?id=b21557081f75ac7b4110427e89a10378 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="The _Factories_ class hierarchy" />
<figcaption><p>Each concrete factory corresponds to a specific
product variant.</p></figcaption>
</figure>

Now, how about the product variants? For each variant of a product
family, we create a separate factory class based on the
`AbstractFactory` interface. A factory is a class that returns products
of a particular kind. For example, the `ModernFurnitureFactory` can only
create `ModernChair`, `ModernSofa` and `ModernCoffeeTable` objects.

The client code has to work with both factories and products via their
respective abstract interfaces. This lets you change the type of a
factory that you pass to the client code, as well as the product variant
that the client code receives, without breaking the actual client code.

<figure class="image">
<img
src="../images/patterns/content/abstract-factory/abstract-factory-comic-2-en1e19.png?id=fbce1a263acfefc76074fd20fae7b8c3"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/abstract-factory/abstract-factory-comic-2-en.png?id=fbce1a263acfefc76074fd20fae7b8c3 600w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-en-1.5x.png?id=455338a4d7c25d4a46819260e79ffa0b 900w,/images/patterns/content/abstract-factory/abstract-factory-comic-2-en-2x.png?id=824023e4cfdc6f4d2457e6dc3e51ccfb 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600" />
<figcaption><p>The client shouldn’t care about the concrete class of the
factory it works with.</p></figcaption>
</figure>

Say the client wants a factory to produce a chair. The client doesn't
have to be aware of the factory's class, nor does it matter what kind of
chair it gets. Whether it's a Modern model or a Victorian-style chair,
the client must treat all chairs in the same manner, using the abstract
`Chair` interface. With this approach, the only thing that the client
knows about the chair is that it implements the `sitOn` method in some
way. Also, whichever variant of the chair is returned, it'll always
match the type of sofa or coffee table produced by the same factory
object.

There's one more thing left to clarify: if the client is only exposed to
the abstract interfaces, what creates the actual factory objects?
Usually, the application creates a concrete factory object at the
initialization stage. Just before that, the app must select the factory
type depending on the configuration or the environment settings.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/abstract-factory/structureb809.png?id=a3112cdd98765406af94595a3c5e7762"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/abstract-factory/structure.png?id=a3112cdd98765406af94595a3c5e7762 720w,/images/patterns/diagrams/abstract-factory/structure-1.5x.png?id=d2964e541620d9c087e693bd0e0a0836 1080w,/images/patterns/diagrams/abstract-factory/structure-2x.png?id=c4d3634ec2e74e02a0fe1a83ce9b50f6 1440w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="720"
alt="Abstract Factory design pattern" /><img
src="../images/patterns/diagrams/abstract-factory/structure-indexed028d.png?id=6ae1c99cbd90cf58753c633624fb1a04"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.56;"
srcset="/images/patterns/diagrams/abstract-factory/structure-indexed.png?id=6ae1c99cbd90cf58753c633624fb1a04 700w,/images/patterns/diagrams/abstract-factory/structure-indexed-1.5x.png?id=473be2975c5716162e6969e6532210ac 1050w,/images/patterns/diagrams/abstract-factory/structure-indexed-2x.png?id=cb6d4e1e89826c42966dc7097374f889 1400w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="700"
alt="Abstract Factory design pattern" />
</figure>
:::

1.  **Abstract Products** declare interfaces for a set of distinct but
    related products which make up a product family.

2.  **Concrete Products** are various implementations of abstract
    products, grouped by variants. Each abstract product (chair/sofa)
    must be implemented in all given variants (Victorian/Modern).

3.  The **Abstract Factory** interface declares a set of methods for
    creating each of the abstract products.

4.  **Concrete Factories** implement creation methods of the abstract
    factory. Each concrete factory corresponds to a specific variant of
    products and creates only those product variants.

5.  Although concrete factories instantiate concrete products,
    signatures of their creation methods must return corresponding
    *abstract* products. This way the client code that uses a factory
    doesn't get coupled to the specific variant of the product it gets
    from a factory. The **Client** can work with any concrete
    factory/product variant, as long as it communicates with their
    objects via abstract interfaces.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

This example illustrates how the **Abstract Factory** pattern can be
used for creating cross-platform UI elements without coupling the client
code to concrete UI classes, while keeping all created elements
consistent with a selected operating system.

<figure class="image">
<img
src="../images/patterns/diagrams/abstract-factory/exampled62f.png?id=5928a61d18bf00b047463471c599100a"
style="aspect-ratio: 1.42;"
srcset="/images/patterns/diagrams/abstract-factory/example.png?id=5928a61d18bf00b047463471c599100a 640w,/images/patterns/diagrams/abstract-factory/example-1.5x.png?id=baee3bac793ec97e0ec91c49d9382e7d 960w,/images/patterns/diagrams/abstract-factory/example-2x.png?id=eb5127b1d6486f6fad73be2d5e444449 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="The class diagram for the Abstract Factory pattern example" />
<figcaption><p>The cross-platform UI classes example.</p></figcaption>
</figure>

The same UI elements in a cross-platform application are expected to
behave similarly, but look a little bit different under different
operating systems. Moreover, it's your job to make sure that the UI
elements match the style of the current operating system. You wouldn't
want your program to render macOS controls when it's executed in
Windows.

The Abstract Factory interface declares a set of creation methods that
the client code can use to produce different types of UI elements.
Concrete factories correspond to specific operating systems and create
the UI elements that match that particular OS.

It works like this: when an application launches, it checks the type of
the current operating system. The app uses this information to create a
factory object from a class that matches the operating system. The rest
of the code uses this factory to create UI elements. This prevents the
wrong elements from being created.

With this approach, the client code doesn't depend on concrete classes
of factories and UI elements as long as it works with these objects via
their abstract interfaces. This also lets the client code support other
factories or UI elements that you might add in the future.

As a result, you don't need to modify the client code each time you add
a new variation of UI elements to your app. You just have to create a
new factory class that produces these elements and slightly modify the
app's initialization code so it selects that class when appropriate.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The abstract factory interface declares a set of methods that
// return different abstract products. These products are called
// a family and are related by a high-level theme or concept.
// Products of one family are usually able to collaborate among
// themselves. A family of products may have several variants,
// but the products of one variant are incompatible with the
// products of another variant.
interface GUIFactory is
    method createButton():Button
    method createCheckbox():Checkbox


// Concrete factories produce a family of products that belong
// to a single variant. The factory guarantees that the
// resulting products are compatible. Signatures of the concrete
// factory&#39;s methods return an abstract product, while inside
// the method a concrete product is instantiated.
class WinFactory implements GUIFactory is
    method createButton():Button is
        return new WinButton()
    method createCheckbox():Checkbox is
        return new WinCheckbox()

// Each concrete factory has a corresponding product variant.
class MacFactory implements GUIFactory is
    method createButton():Button is
        return new MacButton()
    method createCheckbox():Checkbox is
        return new MacCheckbox()


// Each distinct product of a product family should have a base
// interface. All variants of the product must implement this
// interface.
interface Button is
    method paint()

// Concrete products are created by corresponding concrete
// factories.
class WinButton implements Button is
    method paint() is
        // Render a button in Windows style.

class MacButton implements Button is
    method paint() is
        // Render a button in macOS style.

// Here&#39;s the base interface of another product. All products
// can interact with each other, but proper interaction is
// possible only between products of the same concrete variant.
interface Checkbox is
    method paint()

class WinCheckbox implements Checkbox is
    method paint() is
        // Render a checkbox in Windows style.

class MacCheckbox implements Checkbox is
    method paint() is
        // Render a checkbox in macOS style.


// The client code works with factories and products only
// through abstract types: GUIFactory, Button and Checkbox. This
// lets you pass any factory or product subclass to the client
// code without breaking it.
class Application is
    private field factory: GUIFactory
    private field button: Button
    constructor Application(factory: GUIFactory) is
        this.factory = factory
    method createUI() is
        this.button = factory.createButton()
    method paint() is
        button.paint()


// The application picks the factory type depending on the
// current configuration or environment settings and creates it
// at runtime (usually at the initialization stage).
class ApplicationConfigurator is
    method main() is
        config = readApplicationConfigFile()

        if (config.OS == &quot;Windows&quot;) then
            factory = new WinFactory()
        else if (config.OS == &quot;Mac&quot;) then
            factory = new MacFactory()
        else
            throw new Exception(&quot;Error! Unknown operating system.&quot;)

        Application app = new Application(factory)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Abstract Factory when your code needs to work with various
families of related products, but you don't want it to depend on the
concrete classes of those products---they might be unknown beforehand or
you simply want to allow for future extensibility.
:::

::: applicability-solution
The Abstract Factory provides you with an interface for creating objects
from each class of the product family. As long as your code creates
objects via this interface, you don't have to worry about creating the
wrong variant of a product which doesn't match the products already
created by your app.
:::

::: applicability-problem
Consider implementing the Abstract Factory when you have a class with a
set of [Factory Methods](factory-method.html) that blur its primary
responsibility.
:::

::: applicability-solution
In a well-designed program *each class is responsible only for one
thing*. When a class deals with multiple product types, it may be worth
extracting its factory methods into a stand-alone factory class or a
full-blown Abstract Factory implementation.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Map out a matrix of distinct product types versus variants of these
    products.

2.  Declare abstract product interfaces for all product types. Then make
    all concrete product classes implement these interfaces.

3.  Declare the abstract factory interface with a set of creation
    methods for all abstract products.

4.  Implement a set of concrete factory classes, one for each product
    variant.

5.  Create factory initialization code somewhere in the app. It should
    instantiate one of the concrete factory classes, depending on the
    application configuration or the current environment. Pass this
    factory object to all classes that construct products.

6.  Scan through the code and find all direct calls to product
    constructors. Replace them with calls to the appropriate creation
    method on the factory object.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can be sure that the products you're getting from a factory are
    compatible with each other.
-    You avoid tight coupling between concrete products and client code.
-    *Single Responsibility Principle*. You can extract the product
    creation code into one place, making the code easier to support.
-    *Open/Closed Principle*. You can introduce new variants of products
    without breaking existing client code.
:::

::: col-sm-6
-    The code may become more complicated than it should be, since a lot
    of new interfaces and classes are introduced along with the pattern.
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

-   [Abstract Factory](abstract-factory.html) classes are often based on
    a set of [Factory Methods](factory-method.html), but you can also
    use [Prototype](prototype.html) to compose the methods on these
    classes.

-   [Abstract Factory](abstract-factory.html) can serve as an
    alternative to [Facade](facade.html) when you only want to hide the
    way the subsystem objects are created from the client code.

-   You can use [Abstract Factory](abstract-factory.html) along with
    [Bridge](bridge.html). This pairing is useful when some abstractions
    defined by *Bridge* can only work with specific implementations. In
    this case, *Abstract Factory* can encapsulate these relations and
    hide the complexity from the client code.

-   [Abstract Factories](abstract-factory.html),
    [Builders](builder.html) and [Prototypes](prototype.html) can all be
    implemented as [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Abstract Factory in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](abstract-factory/csharp/example.html "Abstract Factory in C#"){.prog-lang-link}
[![Abstract Factory in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](abstract-factory/cpp/example.html "Abstract Factory in C++"){.prog-lang-link}
[![Abstract Factory in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](abstract-factory/go/example.html "Abstract Factory in Go"){.prog-lang-link}
[![Abstract Factory in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](abstract-factory/java/example.html "Abstract Factory in Java"){.prog-lang-link}
[![Abstract Factory in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](abstract-factory/php/example.html "Abstract Factory in PHP"){.prog-lang-link}
[![Abstract Factory in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](abstract-factory/python/example.html "Abstract Factory in Python"){.prog-lang-link}
[![Abstract Factory in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](abstract-factory/ruby/example.html "Abstract Factory in Ruby"){.prog-lang-link}
[![Abstract Factory in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](abstract-factory/rust/example.html "Abstract Factory in Rust"){.prog-lang-link}
[![Abstract Factory in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](abstract-factory/swift/example.html "Abstract Factory in Swift"){.prog-lang-link}
[![Abstract Factory in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](abstract-factory/typescript/example.html "Abstract Factory in TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Extra Content {#extras}

-   Read our [Factory Comparison](factory-comparison.html) to learn more
    about the differences between various factory patterns and concepts.
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

[Factory Comparison []{.fa
.fa-arrow-right}](factory-comparison.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Factory Method](factory-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
