:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Iterator {#iterator .title}

::: {.section .intent}
##  Intent

**Iterator** is a behavioral design pattern that lets you traverse
elements of a collection without exposing its underlying representation
(list, stack, tree, etc.).

<figure class="image">
<img
src="../images/patterns/content/iterator/iterator-enfc4d.png?id=d19123d71d355d01b0ede4be173eb695"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-en.png?id=d19123d71d355d01b0ede4be173eb695 640w,/images/patterns/content/iterator/iterator-en-1.5x.png?id=54aa477cafffe8f9b1b6e63c2e88c21b 960w,/images/patterns/content/iterator/iterator-en-2x.png?id=2a85705e8e5fab257802b2ce36d6d236 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Iterator design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Collections are one of the most used data types in programming.
Nonetheless, a collection is just a container for a group of objects.

<figure class="image">
<img
src="../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Various types of collections" />
<figcaption><p>Various types of collections.</p></figcaption>
</figure>

Most collections store their elements in simple lists. However, some of
them are based on stacks, trees, graphs and other complex data
structures.

But no matter how a collection is structured, it must provide some way
of accessing its elements so that other code can use these elements.
There should be a way to go through each element of the collection
without accessing the same elements over and over.

This may sound like an easy job if you have a collection based on a
list. You just loop over all of the elements. But how do you
sequentially traverse elements of a complex data structure, such as a
tree? For example, one day you might be just fine with depth-first
traversal of a tree. Yet the next day you might require breadth-first
traversal. And the next week, you might need something else, like random
access to the tree elements.

<figure class="image">
<img
src="../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Various traversal algorithms" />
<figcaption><p>The same collection can be traversed in several
different ways.</p></figcaption>
</figure>

Adding more and more traversal algorithms to the collection gradually
blurs its primary responsibility, which is efficient data storage.
Additionally, some algorithms might be tailored for a specific
application, so including them into a generic collection class would be
weird.

On the other hand, the client code that's supposed to work with various
collections may not even care how they store their elements. However,
since collections all provide different ways of accessing their
elements, you have no option other than to couple your code to the
specific collection classes.
:::

::: {.section .solution}
##  Solution

The main idea of the Iterator pattern is to extract the traversal
behavior of a collection into a separate object called an *iterator*.

<figure class="image">
<img
src="../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Iterators implement various traversal algorithms" />
<figcaption><p>Iterators implement various traversal algorithms. Several
iterator objects can traverse the same collection at the
same time.</p></figcaption>
</figure>

In addition to implementing the algorithm itself, an iterator object
encapsulates all of the traversal details, such as the current position
and how many elements are left till the end. Because of this, several
iterators can go through the same collection at the same time,
independently of each other.

Usually, iterators provide one primary method for fetching elements of
the collection. The client can keep running this method until it doesn't
return anything, which means that the iterator has traversed all of the
elements.

All iterators must implement the same interface. This makes the client
code compatible with any collection type or any traversal algorithm as
long as there's a proper iterator. If you need a special way to traverse
a collection, you just create a new iterator class, without having to
change the collection or the client.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/iterator/iterator-comic-1-en9d72.png?id=fa30f5a944179e6fb12203fef5d5ed9d"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-en.png?id=fa30f5a944179e6fb12203fef5d5ed9d 640w,/images/patterns/content/iterator/iterator-comic-1-en-1.5x.png?id=857dcd2ebfa02428843767f526ec8813 960w,/images/patterns/content/iterator/iterator-comic-1-en-2x.png?id=b90df2dbda9280336db189924683e121 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Various ways to walk around Rome" />
<figcaption><p>Various ways to walk around Rome.</p></figcaption>
</figure>

You plan to visit Rome for a few days and visit all of its main sights
and attractions. But once there, you could waste a lot of time walking
in circles, unable to find even the Colosseum.

On the other hand, you could buy a virtual guide app for your smartphone
and use it for navigation. It's smart and inexpensive, and you could be
staying at some interesting places for as long as you want.

A third alternative is that you could spend some of the trip's budget
and hire a local guide who knows the city like the back of his hand. The
guide would be able to tailor the tour to your likings, show you every
attraction and tell a lot of exciting stories. That'll be even more fun;
but, alas, more expensive, too.

All of these options---the random directions born in your head, the
smartphone navigator or the human guide---act as iterators over the vast
collection of sights and attractions located in Rome.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Structure of the Iterator design pattern" /><img
src="../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Structure of the Iterator design pattern" />
</figure>
:::

1.  The **Iterator** interface declares the operations required for
    traversing a collection: fetching the next element, retrieving the
    current position, restarting iteration, etc.

2.  **Concrete Iterators** implement specific algorithms for traversing
    a collection. The iterator object should track the traversal
    progress on its own. This allows several iterators to traverse the
    same collection independently of each other.

3.  The **Collection** interface declares one or multiple methods for
    getting iterators compatible with the collection. Note that the
    return type of the methods must be declared as the iterator
    interface so that the concrete collections can return various kinds
    of iterators.

4.  **Concrete Collections** return new instances of a particular
    concrete iterator class each time the client requests one. You might
    be wondering, where's the rest of the collection's code? Don't
    worry, it should be in the same class. It's just that these details
    aren't crucial to the actual pattern, so we're omitting them.

5.  The **Client** works with both collections and iterators via their
    interfaces. This way the client isn't coupled to concrete classes,
    allowing you to use various collections and iterators with the same
    client code.

    Typically, clients don't create iterators on their own, but instead
    get them from collections. Yet, in certain cases, the client can
    create one directly; for example, when the client defines its own
    special iterator.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **Iterator** pattern is used to walk through a
special kind of collection which encapsulates access to Facebook's
social graph. The collection provides several iterators that can
traverse profiles in various ways.

<figure class="image">
<img
src="../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure of the Iterator pattern example" />
<figcaption><p>Example of iterating over
social profiles.</p></figcaption>
</figure>

The 'friends' iterator can be used to go over the friends of a given
profile. The 'colleagues' iterator does the same, except it omits
friends who don't work at the same company as a target person. Both
iterators implement a common interface which allows clients to fetch
profiles without diving into implementation details such as
authentication and sending REST requests.

The client code isn't coupled to concrete classes because it works with
collections and iterators only through interfaces. If you decide to
connect your app to a new social network, you simply need to provide new
collection and iterator classes without changing the existing code.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The collection interface must declare a factory method for
// producing iterators. You can declare several methods if there
// are different kinds of iteration available in your program.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Each concrete collection is coupled to a set of concrete
// iterator classes it returns. But the client isn&#39;t, since the
// signature of these methods returns iterator interfaces.
class Facebook implements SocialNetwork is
    // ... The bulk of the collection&#39;s code should go here ...

    // Iterator creation code.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// The common interface for all iterators.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// The concrete iterator class.
class FacebookIterator implements ProfileIterator is
    // The iterator needs a reference to the collection that it
    // traverses.
    private field facebook: Facebook
    private field profileId, type: string

    // An iterator object traverses the collection independently
    // from other iterators. Therefore it has to store the
    // iteration state.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Each concrete iterator class has its own implementation
    // of the common iterator interface.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Here is another useful trick: you can pass an iterator to a
// client class instead of giving it access to a whole
// collection. This way, you don&#39;t expose the collection to the
// client.
//
// And there&#39;s another benefit: you can change the way the
// client works with the collection at runtime by passing it a
// different iterator. This is possible because the client code
// isn&#39;t coupled to concrete iterator classes.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// The application class configures collections and iterators
// and then passes them to the client code.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the Iterator pattern when your collection has a complex data
structure under the hood, but you want to hide its complexity from
clients (either for convenience or security reasons).
:::

::: applicability-solution
The iterator encapsulates the details of working with a complex data
structure, providing the client with several simple methods of accessing
the collection elements. While this approach is very convenient for the
client, it also protects the collection from careless or malicious
actions which the client would be able to perform if working with the
collection directly.
:::

::: applicability-problem
Use the pattern to reduce duplication of the traversal code across your
app.
:::

::: applicability-solution
The code of non-trivial iteration algorithms tends to be very bulky.
When placed within the business logic of an app, it may blur the
responsibility of the original code and make it less maintainable.
Moving the traversal code to designated iterators can help you make the
code of the application more lean and clean.
:::

::: applicability-problem
Use the Iterator when you want your code to be able to traverse
different data structures or when types of these structures are unknown
beforehand.
:::

::: applicability-solution
The pattern provides a couple of generic interfaces for both collections
and iterators. Given that your code now uses these interfaces, it'll
still work if you pass it various kinds of collections and iterators
that implement these interfaces.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Declare the iterator interface. At the very least, it must have a
    method for fetching the next element from a collection. But for the
    sake of convenience you can add a couple of other methods, such as
    fetching the previous element, tracking the current position, and
    checking the end of the iteration.

2.  Declare the collection interface and describe a method for fetching
    iterators. The return type should be equal to that of the iterator
    interface. You may declare similar methods if you plan to have
    several distinct groups of iterators.

3.  Implement concrete iterator classes for the collections that you
    want to be traversable with iterators. An iterator object must be
    linked with a single collection instance. Usually, this link is
    established via the iterator's constructor.

4.  Implement the collection interface in your collection classes. The
    main idea is to provide the client with a shortcut for creating
    iterators, tailored for a particular collection class. The
    collection object must pass itself to the iterator's constructor to
    establish a link between them.

5.  Go over the client code to replace all of the collection traversal
    code with the use of iterators. The client fetches a new iterator
    object each time it needs to iterate over the collection elements.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    *Single Responsibility Principle*. You can clean up the client code
    and the collections by extracting bulky traversal algorithms into
    separate classes.
-    *Open/Closed Principle*. You can implement new types of collections
    and iterators and pass them to existing code without breaking
    anything.
-    You can iterate over the same collection in parallel because each
    iterator object contains its own iteration state.
-    For the same reason, you can delay an iteration and continue it
    when needed.
:::

::: col-sm-6
-    Applying the pattern can be an overkill if your app only works with
    simple collections.
-    Using an iterator may be less efficient than going through elements
    of some specialized collections directly.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   You can use [Iterators](iterator.html) to traverse
    [Composite](composite.html) trees.

-   You can use [Factory Method](factory-method.html) along with
    [Iterator](iterator.html) to let collection subclasses return
    different types of iterators that are compatible with the
    collections.

-   You can use [Memento](memento.html) along with
    [Iterator](iterator.html) to capture the current iteration state and
    roll it back if necessary.

-   You can use [Visitor](visitor.html) along with
    [Iterator](iterator.html) to traverse a complex data structure and
    execute some operation over its elements, even if they all have
    different classes.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Iterator in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Iterator in C#"){.prog-lang-link}
[![Iterator in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Iterator in C++"){.prog-lang-link}
[![Iterator in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Iterator in Go"){.prog-lang-link}
[![Iterator in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Iterator in Java"){.prog-lang-link}
[![Iterator in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Iterator in PHP"){.prog-lang-link}
[![Iterator in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Iterator in Python"){.prog-lang-link}
[![Iterator in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Iterator in Ruby"){.prog-lang-link}
[![Iterator in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Iterator in Rust"){.prog-lang-link}
[![Iterator in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Iterator in Swift"){.prog-lang-link}
[![Iterator in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Iterator in TypeScript"){.prog-lang-link}
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

[Mediator []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Command](command.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
