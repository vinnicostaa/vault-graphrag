:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Creational
Patterns](creational-patterns.html){.type}
:::

# Singleton {#singleton .title}

::: {.section .intent}
##  Intent

**Singleton** is a creational design pattern that lets you ensure that a
class has only one instance, while providing a global access point to
this instance.

<figure class="image">
<img
src="../images/patterns/content/singleton/singleton4263.png?id=108a0b9b5ea5c4426e0afa4504491d6f"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/singleton/singleton.png?id=108a0b9b5ea5c4426e0afa4504491d6f 640w,/images/patterns/content/singleton/singleton-1.5x.png?id=29490ad6cd1426c63cf5f88243a1701c 960w,/images/patterns/content/singleton/singleton-2x.png?id=accb2cc7594f7a491ce01dddf0d2f876 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Singleton pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

The Singleton pattern solves two problems at the same time, violating
the *Single Responsibility Principle*:

1.  **Ensure that a class has just a single instance**. Why would anyone
    want to control how many instances a class has? The most common
    reason for this is to control access to some shared resource---for
    example, a database or a file.

    Here's how it works: imagine that you created an object, but after a
    while decided to create a new one. Instead of receiving a fresh
    object, you'll get the one you already created.

    Note that this behavior is impossible to implement with a regular
    constructor since a constructor call **must** always return a new
    object by design.

<figure class="image">
<img
src="../images/patterns/content/singleton/singleton-comic-1-encddd.png?id=157509c5693a657ba465c7a9d58a7c25"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/singleton/singleton-comic-1-en.png?id=157509c5693a657ba465c7a9d58a7c25 600w,/images/patterns/content/singleton/singleton-comic-1-en-1.5x.png?id=a7b61d84c09d0d8d23bc2df0ec1af0bb 900w,/images/patterns/content/singleton/singleton-comic-1-en-2x.png?id=05678e879d13f7f6a377bab7fba18acc 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="The global access to an object" />
<figcaption><p>Clients may not even realize that they’re working with
the same object all the time.</p></figcaption>
</figure>

2.  **Provide a global access point to that instance**. Remember those
    global variables that you (all right, me) used to store some
    essential objects? While they're very handy, they're also very
    unsafe since any code can potentially overwrite the contents of
    those variables and crash the app.

    Just like a global variable, the Singleton pattern lets you access
    some object from anywhere in the program. However, it also protects
    that instance from being overwritten by other code.

    There's another side to this problem: you don't want the code that
    solves problem #1 to be scattered all over your program. It's much
    better to have it within one class, especially if the rest of your
    code already depends on it.

Nowadays, the Singleton pattern has become so popular that people may
call something a *singleton* even if it solves just one of the listed
problems.
:::

::: {.section .solution}
##  Solution

All implementations of the Singleton have these two steps in common:

-   Make the default constructor private, to prevent other objects from
    using the `new` operator with the Singleton class.
-   Create a static creation method that acts as a constructor. Under
    the hood, this method calls the private constructor to create an
    object and saves it in a static field. All following calls to this
    method return the cached object.

If your code has access to the Singleton class, then it's able to call
the Singleton's static method. So whenever that method is called, the
same object is always returned.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

The government is an excellent example of the Singleton pattern. A
country can have only one official government. Regardless of the
personal identities of the individuals who form governments, the title,
"The Government of X", is a global point of access that identifies the
group of people in charge.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/singleton/structure-en36ef.png?id=4e4306d3a90f40d74c7a4d2d2506b8ec"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-en.png?id=4e4306d3a90f40d74c7a4d2d2506b8ec 430w,/images/patterns/diagrams/singleton/structure-en-1.5x.png?id=91bf9c0046139f46d5f484fefabf67fc 645w,/images/patterns/diagrams/singleton/structure-en-2x.png?id=cae4853e43f06db09f249668c35d61a1 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="The structure of the Singleton pattern" /><img
src="../images/patterns/diagrams/singleton/structure-en-indexed7327.png?id=b0217ae066cd3b757677d119551f9a8f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.48;"
srcset="/images/patterns/diagrams/singleton/structure-en-indexed.png?id=b0217ae066cd3b757677d119551f9a8f 430w,/images/patterns/diagrams/singleton/structure-en-indexed-1.5x.png?id=2ff28af0464f779f92c0d6056ebeb078 645w,/images/patterns/diagrams/singleton/structure-en-indexed-2x.png?id=d552220be21d0eda3a8cbe5ccec6dad1 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="The structure of the Singleton pattern" />
</figure>
:::

1.  The **Singleton** class declares the static method `getInstance`
    that returns the same instance of its own class.

    The Singleton's constructor should be hidden from the client code.
    Calling the `getInstance` method should be the only way of getting
    the Singleton object.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the database connection class acts as a **Singleton**.
This class doesn't have a public constructor, so the only way to get its
object is to call the `getInstance` method. This method caches the first
created object and returns it in all subsequent calls.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The Database class defines the `getInstance` method that lets
// clients access the same instance of a database connection
// throughout the program.
class Database is
    // The field for storing the singleton instance should be
    // declared static.
    private static field instance: Database

    // The singleton&#39;s constructor should always be private to
    // prevent direct construction calls with the `new`
    // operator.
    private constructor Database() is
        // Some initialization code, such as the actual
        // connection to a database server.
        // ...

    // The static method that controls access to the singleton
    // instance.
    public static method getInstance() is
        if (Database.instance == null) then
            acquireThreadLock() and then
                // Ensure that the instance hasn&#39;t yet been
                // initialized by another thread while this one
                // has been waiting for the lock&#39;s release.
                if (Database.instance == null) then
                    Database.instance = new Database()
        return Database.instance

    // Finally, any singleton should define some business logic
    // which can be executed on its instance.
    public method query(sql) is
        // For instance, all database queries of an app go
        // through this method. Therefore, you can place
        // throttling or caching logic here.
        // ...

class Application is
    method main() is
        Database foo = Database.getInstance()
        foo.query(&quot;SELECT ...&quot;)
        // ...
        Database bar = Database.getInstance()
        bar.query(&quot;SELECT ...&quot;)
        // The variable `bar` will contain the same object as
        // the variable `foo`.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Singleton pattern when a class in your program should have just
a single instance available to all clients; for example, a single
database object shared by different parts of the program.
:::

::: applicability-solution
The Singleton pattern disables all other means of creating objects of a
class except for the special creation method. This method either creates
a new object or returns an existing one if it has already been created.
:::

::: applicability-problem
Use the Singleton pattern when you need stricter control over global
variables.
:::

::: applicability-solution
Unlike global variables, the Singleton pattern guarantees that there's
just one instance of a class. Nothing, except for the Singleton class
itself, can replace the cached instance.

Note that you can always adjust this limitation and allow creating any
number of Singleton instances. The only piece of code that needs
changing is the body of the `getInstance` method.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Add a private static field to the class for storing the singleton
    instance.

2.  Declare a public static creation method for getting the singleton
    instance.

3.  Implement "lazy initialization" inside the static method. It should
    create a new object on its first call and put it into the static
    field. The method should always return that instance on all
    subsequent calls.

4.  Make the constructor of the class private. The static method of the
    class will still be able to call the constructor, but not the other
    objects.

5.  Go over the client code and replace all direct calls to the
    singleton's constructor with calls to its static creation method.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can be sure that a class has only a single instance.
-    You gain a global access point to that instance.
-    The singleton object is initialized only when it's requested for
    the first time.
:::

::: col-sm-6
-    Violates the *Single Responsibility Principle*. The pattern solves
    two problems at the time.
-    The Singleton pattern can mask bad design, for instance, when the
    components of the program know too much about each other.
-    The pattern requires special treatment in a multithreaded
    environment so that multiple threads won't create a singleton object
    several times.
-    It may be difficult to unit test the client code of the Singleton
    because many test frameworks rely on inheritance when producing mock
    objects. Since the constructor of the singleton class is private and
    overriding static methods is impossible in most languages, you will
    need to think of a creative way to mock the singleton. Or just don't
    write the tests. Or don't use the Singleton pattern.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   A [Facade](facade.html) class can often be transformed into a
    [Singleton](singleton.html) since a single facade object is
    sufficient in most cases.

-   [Flyweight](flyweight.html) would resemble
    [Singleton](singleton.html) if you somehow managed to reduce all
    shared states of the objects to just one flyweight object. But there
    are two fundamental differences between these patterns:

    1.  There should be only one Singleton instance, whereas a
        *Flyweight* class can have multiple instances with different
        intrinsic states.
    2.  The *Singleton* object can be mutable. Flyweight objects are
        immutable.

-   [Abstract Factories](abstract-factory.html),
    [Builders](builder.html) and [Prototypes](prototype.html) can all be
    implemented as [Singletons](singleton.html).
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Singleton in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](singleton/csharp/example.html "Singleton in C#"){.prog-lang-link}
[![Singleton in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](singleton/cpp/example.html "Singleton in C++"){.prog-lang-link}
[![Singleton in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](singleton/go/example.html "Singleton in Go"){.prog-lang-link}
[![Singleton in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](singleton/java/example.html "Singleton in Java"){.prog-lang-link}
[![Singleton in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](singleton/php/example.html "Singleton in PHP"){.prog-lang-link}
[![Singleton in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](singleton/python/example.html "Singleton in Python"){.prog-lang-link}
[![Singleton in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](singleton/ruby/example.html "Singleton in Ruby"){.prog-lang-link}
[![Singleton in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](singleton/rust/example.html "Singleton in Rust"){.prog-lang-link}
[![Singleton in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](singleton/swift/example.html "Singleton in Swift"){.prog-lang-link}
[![Singleton in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](singleton/typescript/example.html "Singleton in TypeScript"){.prog-lang-link}
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

[Structural Patterns []{.fa
.fa-arrow-right}](structural-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Prototype](prototype.html){.btn .btn-default
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
