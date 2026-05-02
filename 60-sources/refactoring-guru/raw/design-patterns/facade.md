:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Structural
Patterns](structural-patterns.html){.type}
:::

# Facade {#facade .title}

::: {.section .intent}
##  Intent

**Facade** is a structural design pattern that provides a simplified
interface to a library, a framework, or any other complex set
of classes.

<figure class="image">
<img
src="../images/patterns/content/facade/facade721d.png?id=1f4be17305b6316fbd548edf1937ac3b"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/facade/facade.png?id=1f4be17305b6316fbd548edf1937ac3b 640w,/images/patterns/content/facade/facade-1.5x.png?id=a6af5003b243b59990842d24bdaae2df 960w,/images/patterns/content/facade/facade-2x.png?id=b69fce5943703f5f07c0ba38e3baaed0 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Facade design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you must make your code work with a broad set of objects
that belong to a sophisticated library or framework. Ordinarily, you'd
need to initialize all of those objects, keep track of dependencies,
execute methods in the correct order, and so on.

As a result, the business logic of your classes would become tightly
coupled to the implementation details of 3rd-party classes, making it
hard to comprehend and maintain.
:::

::: {.section .solution}
##  Solution

A facade is a class that provides a simple interface to a complex
subsystem which contains lots of moving parts. A facade might provide
limited functionality in comparison to working with the subsystem
directly. However, it includes only those features that clients really
care about.

Having a facade is handy when you need to integrate your app with a
sophisticated library that has dozens of features, but you just need a
tiny bit of its functionality.

For instance, an app that uploads short funny videos with cats to social
media could potentially use a professional video conversion library.
However, all that it really needs is a class with the single method
`encode(filename, format)`. After creating such a class and connecting
it with the video conversion library, you'll have your first facade.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/diagrams/facade/live-example-en22e6.png?id=461900f9fbacdd0ce981dcd24e121078"
style="aspect-ratio: 2.58;"
srcset="/images/patterns/diagrams/facade/live-example-en.png?id=461900f9fbacdd0ce981dcd24e121078 490w,/images/patterns/diagrams/facade/live-example-en-1.5x.png?id=ab3b02c5e627f13e03de353604d5a2dd 735w,/images/patterns/diagrams/facade/live-example-en-2x.png?id=db1110e957a690955425d8cb6c0a0f8b 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="An example of taking a phone order" />
<figcaption><p>Placing orders by phone.</p></figcaption>
</figure>

When you call a shop to place a phone order, an operator is your facade
to all services and departments of the shop. The operator provides you
with a simple voice interface to the ordering system, payment gateways,
and various delivery services.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/facade/structure2883.png?id=258401362234ac77a2aaf1cde62339e7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/facade/structure.png?id=258401362234ac77a2aaf1cde62339e7 560w,/images/patterns/diagrams/facade/structure-1.5x.png?id=98d134de0c368d382909ba162ec21767 840w,/images/patterns/diagrams/facade/structure-2x.png?id=528ca429555bce293b7c3bd90954e097 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Structure of the Facade design pattern" /><img
src="../images/patterns/diagrams/facade/structure-indexeddb1c.png?id=2da06d6b850701ea15cf72f9d2642fb8"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.58;"
srcset="/images/patterns/diagrams/facade/structure-indexed.png?id=2da06d6b850701ea15cf72f9d2642fb8 600w,/images/patterns/diagrams/facade/structure-indexed-1.5x.png?id=fad7d667f4d8d9a7d522748bcd37dfde 900w,/images/patterns/diagrams/facade/structure-indexed-2x.png?id=4d181bcf1df5d58c533e6c92461e9571 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure of the Facade design pattern" />
</figure>
:::

1.  The **Facade** provides convenient access to a particular part of
    the subsystem's functionality. It knows where to direct the client's
    request and how to operate all the moving parts.

2.  An **Additional Facade** class can be created to prevent polluting a
    single facade with unrelated features that might make it yet another
    complex structure. Additional facades can be used by both clients
    and other facades.

3.  The **Complex Subsystem** consists of dozens of various objects. To
    make them all do something meaningful, you have to dive deep into
    the subsystem's implementation details, such as initializing objects
    in the correct order and supplying them with data in the proper
    format.

    Subsystem classes aren't aware of the facade's existence. They
    operate within the system and work with each other directly.

4.  The **Client** uses the facade instead of calling the subsystem
    objects directly.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Facade** pattern simplifies interaction with a
complex video conversion framework.

<figure class="image">
<img
src="../images/patterns/diagrams/facade/example16bc.png?id=2249d134e3ff83819dfc19032f02eced"
style="aspect-ratio: 1.5;"
srcset="/images/patterns/diagrams/facade/example.png?id=2249d134e3ff83819dfc19032f02eced 570w,/images/patterns/diagrams/facade/example-1.5x.png?id=034ecbe0f3cc108f4ae49d05d1c77dbe 855w,/images/patterns/diagrams/facade/example-2x.png?id=f2c846d74041626c923ff3e8919b68a9 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="The structure of the Facade pattern example" />
<figcaption><p>An example of isolating multiple dependencies within a
single facade class.</p></figcaption>
</figure>

Instead of making your code work with dozens of the framework classes
directly, you create a facade class which encapsulates that
functionality and hides it from the rest of the code. This structure
also helps you to minimize the effort of upgrading to future versions of
the framework or replacing it with another one. The only thing you'd
need to change in your app would be the implementation of the facade's
methods.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// These are some of the classes of a complex 3rd-party video
// conversion framework. We don&#39;t control that code, therefore
// can&#39;t simplify it.

class VideoFile
// ...

class OggCompressionCodec
// ...

class MPEG4CompressionCodec
// ...

class CodecFactory
// ...

class BitrateReader
// ...

class AudioMixer
// ...


// We create a facade class to hide the framework&#39;s complexity
// behind a simple interface. It&#39;s a trade-off between
// functionality and simplicity.
class VideoConverter is
    method convert(filename, format):File is
        file = new VideoFile(filename)
        sourceCodec = (new CodecFactory).extract(file)
        if (format == &quot;mp4&quot;)
            destinationCodec = new MPEG4CompressionCodec()
        else
            destinationCodec = new OggCompressionCodec()
        buffer = BitrateReader.read(filename, sourceCodec)
        result = BitrateReader.convert(buffer, destinationCodec)
        result = (new AudioMixer()).fix(result)
        return new File(result)

// Application classes don&#39;t depend on a billion classes
// provided by the complex framework. Also, if you decide to
// switch frameworks, you only need to rewrite the facade class.
class Application is
    method main() is
        convertor = new VideoConverter()
        mp4 = convertor.convert(&quot;funny-cats-video.ogg&quot;, &quot;mp4&quot;)
        mp4.save()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Facade pattern when you need to have a limited but
straightforward interface to a complex subsystem.
:::

::: applicability-solution
Often, subsystems get more complex over time. Even applying design
patterns typically leads to creating more classes. A subsystem may
become more flexible and easier to reuse in various contexts, but the
amount of configuration and boilerplate code it demands from a client
grows ever larger. The Facade attempts to fix this problem by providing
a shortcut to the most-used features of the subsystem which fit most
client requirements.
:::

::: applicability-problem
Use the Facade when you want to structure a subsystem into layers.
:::

::: applicability-solution
Create facades to define entry points to each level of a subsystem. You
can reduce coupling between multiple subsystems by requiring them to
communicate only through facades.

For example, let's return to our video conversion framework. It can be
broken down into two layers: video- and audio-related. For each layer,
you can create a facade and then make the classes of each layer
communicate with each other via those facades. This approach looks very
similar to the [Mediator](mediator.html) pattern.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Check whether it's possible to provide a simpler interface than what
    an existing subsystem already provides. You're on the right track if
    this interface makes the client code independent from many of the
    subsystem's classes.

2.  Declare and implement this interface in a new facade class. The
    facade should redirect the calls from the client code to appropriate
    objects of the subsystem. The facade should be responsible for
    initializing the subsystem and managing its further life cycle
    unless the client code already does this.

3.  To get the full benefit from the pattern, make all the client code
    communicate with the subsystem only via the facade. Now the client
    code is protected from any changes in the subsystem code. For
    example, when a subsystem gets upgraded to a new version, you will
    only need to modify the code in the facade.

4.  If the facade becomes [too big](../smells/large-class.html),
    consider extracting part of its behavior to a new, refined facade
    class.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can isolate your code from the complexity of a subsystem.
:::

::: col-sm-6
-    A facade can become [a god
    object](https://en.wikipedia.org/wiki/God_object) coupled to all
    classes of an app.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   [Facade](facade.html) defines a new interface for existing objects,
    whereas [Adapter](adapter.html) tries to make the existing interface
    usable. *Adapter* usually wraps just one object, while *Facade*
    works with an entire subsystem of objects.

-   [Abstract Factory](abstract-factory.html) can serve as an
    alternative to [Facade](facade.html) when you only want to hide the
    way the subsystem objects are created from the client code.

-   [Flyweight](flyweight.html) shows how to make lots of little
    objects, whereas [Facade](facade.html) shows how to make a single
    object that represents an entire subsystem.

-   [Facade](facade.html) and [Mediator](mediator.html) have similar
    jobs: they try to organize collaboration between lots of tightly
    coupled classes.

    -   *Facade* defines a simplified interface to a subsystem of
        objects, but it doesn't introduce any new functionality. The
        subsystem itself is unaware of the facade. Objects within the
        subsystem can communicate directly.
    -   *Mediator* centralizes communication between components of the
        system. The components only know about the mediator object and
        don't communicate directly.

-   A [Facade](facade.html) class can often be transformed into a
    [Singleton](singleton.html) since a single facade object is
    sufficient in most cases.

-   [Facade](facade.html) is similar to [Proxy](proxy.html) in that both
    buffer a complex entity and initialize it on its own. Unlike
    *Facade*, *Proxy* has the same interface as its service object,
    which makes them interchangeable.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Facade in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](facade/csharp/example.html "Facade in C#"){.prog-lang-link}
[![Facade in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](facade/cpp/example.html "Facade in C++"){.prog-lang-link}
[![Facade in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](facade/go/example.html "Facade in Go"){.prog-lang-link}
[![Facade in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](facade/java/example.html "Facade in Java"){.prog-lang-link}
[![Facade in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](facade/php/example.html "Facade in PHP"){.prog-lang-link}
[![Facade in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](facade/python/example.html "Facade in Python"){.prog-lang-link}
[![Facade in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](facade/ruby/example.html "Facade in Ruby"){.prog-lang-link}
[![Facade in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](facade/rust/example.html "Facade in Rust"){.prog-lang-link}
[![Facade in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](facade/swift/example.html "Facade in Swift"){.prog-lang-link}
[![Facade in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](facade/typescript/example.html "Facade in TypeScript"){.prog-lang-link}
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

[Flyweight []{.fa .fa-arrow-right}](flyweight.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Decorator](decorator.html){.btn .btn-default
rel="prev"}
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
