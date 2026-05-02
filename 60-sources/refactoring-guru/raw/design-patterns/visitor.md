::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Visitor {#visitor .title}

::: {.section .intent}
##  Intent

**Visitor** is a behavioral design pattern that lets you separate
algorithms from the objects on which they operate.

<figure class="image">
<img
src="../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Visitor Design Pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that your team develops an app which works with geographic
information structured as one colossal graph. Each node of the graph may
represent a complex entity such as a city, but also more granular things
like industries, sightseeing areas, etc. The nodes are connected with
others if there's a road between the real objects that they represent.
Under the hood, each node type is represented by its own class, while
each specific node is an object.

<figure class="image">
<img
src="../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Exporting the graph into XML" />
<figcaption><p>Exporting the graph into XML.</p></figcaption>
</figure>

At some point, you got a task to implement exporting the graph into XML
format. At first, the job seemed pretty straightforward. You planned to
add an export method to each node class and then leverage recursion to
go over each node of the graph, executing the export method. The
solution was simple and elegant: thanks to polymorphism, you weren't
coupling the code which called the export method to concrete classes of
nodes.

Unfortunately, the system architect refused to allow you to alter
existing node classes. He said that the code was already in production
and he didn't want to risk breaking it because of a potential bug in
your changes.

<figure class="image">
<img
src="../images/patterns/diagrams/visitor/problem2-end5a4.png?id=f53c592d755890f5d027d7950b9967fc"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-en.png?id=f53c592d755890f5d027d7950b9967fc 500w,/images/patterns/diagrams/visitor/problem2-en-1.5x.png?id=3db8e786967e33cd0f26c785af9c918b 750w,/images/patterns/diagrams/visitor/problem2-en-2x.png?id=8a241a88057253b879e7b756023b52a1 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="The XML export method had to be added into all node classes" />
<figcaption><p>The XML export method had to be added into all node
classes, which bore the risk of breaking the whole application if any
bugs slipped through along with the change.</p></figcaption>
</figure>

Besides, he questioned whether it makes sense to have the XML export
code within the node classes. The primary job of these classes was to
work with geodata. The XML export behavior would look alien there.

There was another reason for the refusal. It was highly likely that
after this feature was implemented, someone from the marketing
department would ask you to provide the ability to export into a
different format, or request some other weird stuff. This would force
you to change those precious and fragile classes again.
:::

::: {.section .solution}
##  Solution

The Visitor pattern suggests that you place the new behavior into a
separate class called *visitor*, instead of trying to integrate it into
existing classes. The original object that had to perform the behavior
is now passed to one of the visitor's methods as an argument, providing
the method access to all necessary data contained within the object.

Now, what if that behavior can be executed over objects of different
classes? For example, in our case with XML export, the actual
implementation will probably be a little bit different across various
node classes. Thus, the visitor class may define not one, but a set of
methods, each of which could take arguments of different types, like
this:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

But how exactly would we call these methods, especially when dealing
with the whole graph? These methods have different signatures, so we
can't use polymorphism. To pick a proper visitor method that's able to
process a given object, we'd need to check its class. Doesn't this sound
like a nightmare?

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...
}</code></pre>
</figure>

You might ask, why don't we use method overloading? That's when you give
all methods the same name, even if they support different sets of
parameters. Unfortunately, even assuming that our programming language
supports it at all (as Java and C# do), it won't help us. Since the
exact class of a node object is unknown in advance, the overloading
mechanism won't be able to determine the correct method to execute.
It'll default to the method that takes an object of the base `Node`
class.

However, the Visitor pattern addresses this problem. It uses a technique
called [Double Dispatch](visitor-double-dispatch.html), which helps to
execute the proper method on an object without cumbersome conditionals.
Instead of letting the client select a proper version of the method to
call, how about we delegate this choice to objects we're passing to the
visitor as an argument? Since the objects know their own classes,
they'll be able to pick a proper method on the visitor less awkwardly.
They "accept" a visitor and tell it what visiting method should be
executed.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Client code
foreach (Node node in graph)
    node.accept(exportVisitor)

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

I confess. We had to change the node classes after all. But at least the
change is trivial and it lets us add further behaviors without altering
the code once again.

Now, if we extract a common interface for all visitors, all existing
nodes can work with any visitor you introduce into the app. If you find
yourself introducing a new behavior related to nodes, all you have to do
is implement a new visitor class.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Insurance agent" />
<figcaption><p>A good insurance agent is always ready to offer different
policies to various types of organizations.</p></figcaption>
</figure>

Imagine a seasoned insurance agent who's eager to get new customers. He
can visit every building in a neighborhood, trying to sell insurance to
everyone he meets. Depending on the type of organization that occupies
the building, he can offer specialized insurance policies:

-   If it's a residential building, he sells medical insurance.
-   If it's a bank, he sells theft insurance.
-   If it's a coffee shop, he sells fire and flood insurance.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/visitor/structure-en1ee4.png?id=34126311c4e0d5c9fbb970595d2f1777"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-en.png?id=34126311c4e0d5c9fbb970595d2f1777 520w,/images/patterns/diagrams/visitor/structure-en-1.5x.png?id=6b25f7171bc76113633dc377b80e6811 780w,/images/patterns/diagrams/visitor/structure-en-2x.png?id=8f0367e7fdc92dbe05df3a86f2d0db45 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure of the Visitor design pattern" /><img
src="../images/patterns/diagrams/visitor/structure-en-indexedf9aa.png?id=5896a26c1b13364872b585022f29f29c"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-en-indexed.png?id=5896a26c1b13364872b585022f29f29c 520w,/images/patterns/diagrams/visitor/structure-en-indexed-1.5x.png?id=9f0d143e757c15cd6e16d6396721fe8e 780w,/images/patterns/diagrams/visitor/structure-en-indexed-2x.png?id=16068894aa794946000a539b5f950086 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure of the Visitor design pattern" />
</figure>
:::

1.  The **Visitor** interface declares a set of visiting methods that
    can take concrete elements of an object structure as arguments.
    These methods may have the same names if the program is written in a
    language that supports overloading, but the type of their parameters
    must be different.

2.  Each **Concrete Visitor** implements several versions of the same
    behaviors, tailored for different concrete element classes.

3.  The **Element** interface declares a method for "accepting"
    visitors. This method should have one parameter declared with the
    type of the visitor interface.

4.  Each **Concrete Element** must implement the acceptance method. The
    purpose of this method is to redirect the call to the proper
    visitor's method corresponding to the current element class. Be
    aware that even if a base element class implements this method, all
    subclasses must still override this method in their own classes and
    call the appropriate method on the visitor object.

5.  The **Client** usually represents a collection or some other complex
    object (for example, a [Composite](composite.html) tree). Usually,
    clients aren't aware of all the concrete element classes because
    they work with objects from that collection via some abstract
    interface.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Visitor** pattern adds XML export support to the
class hierarchy of geometric shapes.

<figure class="image">
<img
src="../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Structure of the Visitor pattern example" />
<figcaption><p>Exporting various types of objects into XML format via a
visitor object.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The element interface declares an `accept` method that takes
// the base visitor interface as an argument.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Each concrete element class must implement the `accept`
// method in such a way that it calls the visitor&#39;s method that
// corresponds to the element&#39;s class.
class Dot implements Shape is
    // ...

    // Note that we&#39;re calling `visitDot`, which matches the
    // current class name. This way we let the visitor know the
    // class of the element it works with.
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// The Visitor interface declares a set of visiting methods that
// correspond to element classes. The signature of a visiting
// method lets the visitor identify the exact class of the
// element that it&#39;s dealing with.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Concrete visitors implement several versions of the same
// algorithm, which can work with all concrete element classes.
//
// You can experience the biggest benefit of the Visitor pattern
// when using it with a complex object structure such as a
// Composite tree. In this case, it might be helpful to store
// some intermediate state of the algorithm while executing the
// visitor&#39;s methods over various objects of the structure.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Export the dot&#39;s ID and center coordinates.

    method visitCircle(c: Circle) is
        // Export the circle&#39;s ID, center coordinates and
        // radius.

    method visitRectangle(r: Rectangle) is
        // Export the rectangle&#39;s ID, left-top coordinates,
        // width and height.

    method visitCompoundShape(cs: CompoundShape) is
        // Export the shape&#39;s ID as well as the list of its
        // children&#39;s IDs.


// The client code can run visitor operations over any set of
// elements without figuring out their concrete classes. The
// accept operation directs a call to the appropriate operation
// in the visitor object.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

If you wonder why we need the `accept` method in this example, my
article [Visitor and Double Dispatch](visitor-double-dispatch.html)
addresses this question in detail.
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Visitor when you need to perform an operation on all elements of
a complex object structure (for example, an object tree).
:::

::: applicability-solution
The Visitor pattern lets you execute an operation over a set of objects
with different classes by having a visitor object implement several
variants of the same operation, which correspond to all target classes.
:::

::: applicability-problem
Use the Visitor to clean up the business logic of auxiliary behaviors.
:::

::: applicability-solution
The pattern lets you make the primary classes of your app more focused
on their main jobs by extracting all other behaviors into a set of
visitor classes.
:::

::: applicability-problem
Use the pattern when a behavior makes sense only in some classes of a
class hierarchy, but not in others.
:::

::: applicability-solution
You can extract this behavior into a separate visitor class and
implement only those visiting methods that accept objects of relevant
classes, leaving the rest empty.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Declare the visitor interface with a set of "visiting" methods, one
    per each concrete element class that exists in the program.

2.  Declare the element interface. If you're working with an existing
    element class hierarchy, add the abstract "acceptance" method to the
    base class of the hierarchy. This method should accept a visitor
    object as an argument.

3.  Implement the acceptance methods in all concrete element classes.
    These methods must simply redirect the call to a visiting method on
    the incoming visitor object which matches the class of the current
    element.

4.  The element classes should only work with visitors via the visitor
    interface. Visitors, however, must be aware of all concrete element
    classes, referenced as parameter types of the visiting methods.

5.  For each behavior that can't be implemented inside the element
    hierarchy, create a new concrete visitor class and implement all of
    the visiting methods.

    You might encounter a situation where the visitor will need access
    to some private members of the element class. In this case, you can
    either make these fields or methods public, violating the element's
    encapsulation, or nest the visitor class in the element class. The
    latter is only possible if you're lucky to work with a programming
    language that supports nested classes.

6.  The client must create visitor objects and pass them into elements
    via "acceptance" methods.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    *Open/Closed Principle*. You can introduce a new behavior that can
    work with objects of different classes without changing these
    classes.
-    *Single Responsibility Principle*. You can move multiple versions
    of the same behavior into the same class.
-    A visitor object can accumulate some useful information while
    working with various objects. This might be handy when you want to
    traverse some complex object structure, such as an object tree, and
    apply the visitor to each object of this structure.
:::

::: col-sm-6
-    You need to update all visitors each time a class gets added to or
    removed from the element hierarchy.
-    Visitors might lack the necessary access to the private fields and
    methods of the elements that they're supposed to work with.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   You can treat [Visitor](visitor.html) as a powerful version of the
    [Command](command.html) pattern. Its objects can execute operations
    over various objects of different classes.

-   You can use [Visitor](visitor.html) to execute an operation over an
    entire [Composite](composite.html) tree.

-   You can use [Visitor](visitor.html) along with
    [Iterator](iterator.html) to traverse a complex data structure and
    execute some operation over its elements, even if they all have
    different classes.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Visitor in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Visitor in C#"){.prog-lang-link}
[![Visitor in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Visitor in C++"){.prog-lang-link}
[![Visitor in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Visitor in Go"){.prog-lang-link}
[![Visitor in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Visitor in Java"){.prog-lang-link}
[![Visitor in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Visitor in PHP"){.prog-lang-link}
[![Visitor in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Visitor in Python"){.prog-lang-link}
[![Visitor in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Visitor in Ruby"){.prog-lang-link}
[![Visitor in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Visitor in Rust"){.prog-lang-link}
[![Visitor in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Visitor in Swift"){.prog-lang-link}
[![Visitor in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Visitor in TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Extra Content {#extras}

-   Puzzled why we can't simply replace the Visitor pattern with method
    overloading? Read my article [Visitor and Double
    Dispatch](visitor-double-dispatch.html) to learn about the nasty
    details.
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

[Visitor and Double Dispatch []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Template Method](template-method.html){.btn
.btn-default rel="prev"}
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
