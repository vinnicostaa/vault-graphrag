::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Structural
Patterns](structural-patterns.html){.type}
:::

# Composite {#composite .title}

::: aka
Also known as: [Object Tree]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Composite** is a structural design pattern that lets you compose
objects into tree structures and then work with these structures as if
they were individual objects.

<figure class="image">
<img
src="../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Composite design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Using the Composite pattern makes sense only when the core model of your
app can be represented as a tree.

For example, imagine that you have two types of objects: `Products` and
`Boxes`. A `Box` can contain several `Products` as well as a number of
smaller `Boxes`. These little `Boxes` can also hold some `Products` or
even smaller `Boxes`, and so on.

Say you decide to create an ordering system that uses these classes.
Orders could contain simple products without any wrapping, as well as
boxes stuffed with products\...and other boxes. How would you determine
the total price of such an order?

<figure class="image">
<img
src="../images/patterns/diagrams/composite/problem-enec7f.png?id=3320d7ddc5bdc3e43752bb4393710794"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-en.png?id=3320d7ddc5bdc3e43752bb4393710794 370w,/images/patterns/diagrams/composite/problem-en-1.5x.png?id=4e1e6d2b8d6c7aefeead44809bb1aa6a 555w,/images/patterns/diagrams/composite/problem-en-2x.png?id=5c7d443ccce3e46c4308d43fd1e51cca 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure of a complex order" />
<figcaption><p>An order might comprise various products, packaged in
boxes, which are packaged in bigger boxes and so on. The whole structure
looks like an upside down tree.</p></figcaption>
</figure>

You could try the direct approach: unwrap all the boxes, go over all the
products and then calculate the total. That would be doable in the real
world; but in a program, it's not as simple as running a loop. You have
to know the classes of `Products` and `Boxes` you're going through, the
nesting level of the boxes and other nasty details beforehand. All of
this makes the direct approach either too awkward or even impossible.
:::

::: {.section .solution}
##  Solution

The Composite pattern suggests that you work with `Products` and `Boxes`
through a common interface which declares a method for calculating the
total price.

How would this method work? For a product, it'd simply return the
product's price. For a box, it'd go over each item the box contains, ask
its price and then return a total for this box. If one of these items
were a smaller box, that box would also start going over its contents
and so on, until the prices of all inner components were calculated. A
box could even add some extra cost to the final price, such as packaging
cost.

<figure class="image">
<img
src="../images/patterns/content/composite/composite-comic-1-ene5e7.png?id=5510969e5584e7c0cb65d533901bb8f6"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-en.png?id=5510969e5584e7c0cb65d533901bb8f6 600w,/images/patterns/content/composite/composite-comic-1-en-1.5x.png?id=ef596364925e02ba34081737c2c58332 900w,/images/patterns/content/composite/composite-comic-1-en-2x.png?id=e2f3fb69d636c211520c2528be94f251 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Solution suggested by the Composite pattern" />
<figcaption><p>The Composite pattern lets you run a behavior recursively
over all components of an object tree.</p></figcaption>
</figure>

The greatest benefit of this approach is that you don't need to care
about the concrete classes of objects that compose the tree. You don't
need to know whether an object is a simple product or a sophisticated
box. You can treat them all the same via the common interface. When you
call a method, the objects themselves pass the request down the tree.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="An example of a military structure" />
<figcaption><p>An example of a military structure.</p></figcaption>
</figure>

Armies of most countries are structured as hierarchies. An army consists
of several divisions; a division is a set of brigades, and a brigade
consists of platoons, which can be broken down into squads. Finally, a
squad is a small group of real soldiers. Orders are given at the top of
the hierarchy and passed down onto each level until every soldier knows
what needs to be done.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/composite/structure-en176a.png?id=b7f114558b594dfb220d225398b2b744"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.82;"
srcset="/images/patterns/diagrams/composite/structure-en.png?id=b7f114558b594dfb220d225398b2b744 360w,/images/patterns/diagrams/composite/structure-en-1.5x.png?id=8e06210d59cadf817621d810accfa5f6 540w,/images/patterns/diagrams/composite/structure-en-2x.png?id=fc41be8ae17c7250ea6d29632a239ba4 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Structure of the Composite design pattern" /><img
src="../images/patterns/diagrams/composite/structure-en-indexed0633.png?id=9fbce571969f4bc9bb57ee4a7d786852"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-en-indexed.png?id=9fbce571969f4bc9bb57ee4a7d786852 400w,/images/patterns/diagrams/composite/structure-en-indexed-1.5x.png?id=34c97ed1cf5615a6b680ee23264dbf29 600w,/images/patterns/diagrams/composite/structure-en-indexed-2x.png?id=a5bbb62b1bc218bc52615bacf3fb3b73 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Structure of the Composite design pattern" />
</figure>
:::

1.  The **Component** interface describes operations that are common to
    both simple and complex elements of the tree.

2.  The **Leaf** is a basic element of a tree that doesn't have
    sub-elements.

    Usually, leaf components end up doing most of the real work, since
    they don't have anyone to delegate the work to.

3.  The **Container** (aka *composite*) is an element that has
    sub-elements: leaves or other containers. A container doesn't know
    the concrete classes of its children. It works with all sub-elements
    only via the component interface.

    Upon receiving a request, a container delegates the work to its
    sub-elements, processes intermediate results and then returns the
    final result to the client.

4.  The **Client** works with all elements through the component
    interface. As a result, the client can work in the same way with
    both simple or complex elements of the tree.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Composite** pattern lets you implement stacking
of geometric shapes in a graphical editor.

<figure class="image">
<img
src="../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure of the Composite example" />
<figcaption><p>The geometric shapes editor example.</p></figcaption>
</figure>

The `CompoundGraphic` class is a container that can comprise any number
of sub-shapes, including other compound shapes. A compound shape has the
same methods as a simple shape. However, instead of doing something on
its own, a compound shape passes the request recursively to all its
children and "sums up" the result.

The client code works with all shapes through the single interface
common to all shape classes. Thus, the client doesn't know whether it's
working with a simple shape or a compound one. The client can work with
very complex object structures without being coupled to concrete classes
that form that structure.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The component interface declares common operations for both
// simple and complex objects of a composition.
interface Graphic is
    method move(x, y)
    method draw()

// The leaf class represents end objects of a composition. A
// leaf object can&#39;t have any sub-objects. Usually, it&#39;s leaf
// objects that do the actual work, while composite objects only
// delegate to their sub-components.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Draw a dot at X and Y.

// All component classes can extend other components.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Draw a circle at X and Y with radius R.

// The composite class represents complex components that may
// have children. Composite objects usually delegate the actual
// work to their children and then &quot;sum up&quot; the result.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    // A composite object can add or remove other components
    // (both simple or complex) to or from its child list.
    method add(child: Graphic) is
        // Add a child to the array of children.

    method remove(child: Graphic) is
        // Remove a child from the array of children.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    // A composite executes its primary logic in a particular
    // way. It traverses recursively through all its children,
    // collecting and summing up their results. Since the
    // composite&#39;s children pass these calls to their own
    // children and so forth, the whole object tree is traversed
    // as a result.
    method draw() is
        // 1. For each child component:
        //     - Draw the component.
        //     - Update the bounding rectangle.
        // 2. Draw a dashed rectangle using the bounding
        // coordinates.


// The client code works with all the components via their base
// interface. This way the client code can support simple leaf
// components as well as complex composites.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Combine selected components into one complex composite
    // component.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // All components will be drawn.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Composite pattern when you have to implement a tree-like object
structure.
:::

::: applicability-solution
The Composite pattern provides you with two basic element types that
share a common interface: simple leaves and complex containers. A
container can be composed of both leaves and other containers. This lets
you construct a nested recursive object structure that resembles a tree.
:::

::: applicability-problem
Use the pattern when you want the client code to treat both simple and
complex elements uniformly.
:::

::: applicability-solution
All elements defined by the Composite pattern share a common interface.
Using this interface, the client doesn't have to worry about the
concrete class of the objects it works with.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Make sure that the core model of your app can be represented as a
    tree structure. Try to break it down into simple elements and
    containers. Remember that containers must be able to contain both
    simple elements and other containers.

2.  Declare the component interface with a list of methods that make
    sense for both simple and complex components.

3.  Create a leaf class to represent simple elements. A program may have
    multiple different leaf classes.

4.  Create a container class to represent complex elements. In this
    class, provide an array field for storing references to
    sub-elements. The array must be able to store both leaves and
    containers, so make sure it's declared with the component interface
    type.

    While implementing the methods of the component interface, remember
    that a container is supposed to be delegating most of the work to
    sub-elements.

5.  Finally, define the methods for adding and removal of child elements
    in the container.

    Keep in mind that these operations can be declared in the component
    interface. This would violate the *Interface Segregation Principle*
    because the methods will be empty in the leaf class. However, the
    client will be able to treat all the elements equally, even when
    composing the tree.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can work with complex tree structures more conveniently: use
    polymorphism and recursion to your advantage.
-    *Open/Closed Principle*. You can introduce new element types into
    the app without breaking the existing code, which now works with the
    object tree.
:::

::: col-sm-6
-    It might be difficult to provide a common interface for classes
    whose functionality differs too much. In certain scenarios, you'd
    need to overgeneralize the component interface, making it harder to
    comprehend.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   You can use [Builder](builder.html) when creating complex
    [Composite](composite.html) trees because you can program its
    construction steps to work recursively.

-   [Chain of Responsibility](chain-of-responsibility.html) is often
    used in conjunction with [Composite](composite.html). In this case,
    when a leaf component gets a request, it may pass it through the
    chain of all of the parent components down to the root of the object
    tree.

-   You can use [Iterators](iterator.html) to traverse
    [Composite](composite.html) trees.

-   You can use [Visitor](visitor.html) to execute an operation over an
    entire [Composite](composite.html) tree.

-   You can implement shared leaf nodes of the
    [Composite](composite.html) tree as [Flyweights](flyweight.html) to
    save some RAM.

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
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Composite in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Composite in C#"){.prog-lang-link}
[![Composite in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Composite in C++"){.prog-lang-link}
[![Composite in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Composite in Go"){.prog-lang-link}
[![Composite in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Composite in Java"){.prog-lang-link}
[![Composite in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Composite in PHP"){.prog-lang-link}
[![Composite in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Composite in Python"){.prog-lang-link}
[![Composite in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Composite in Ruby"){.prog-lang-link}
[![Composite in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Composite in Rust"){.prog-lang-link}
[![Composite in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Composite in Swift"){.prog-lang-link}
[![Composite in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Composite in TypeScript"){.prog-lang-link}
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

[Decorator []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Bridge](bridge.html){.btn .btn-default
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
