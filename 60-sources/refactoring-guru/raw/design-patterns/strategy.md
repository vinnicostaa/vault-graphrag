:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Strategy {#strategy .title}

::: {.section .intent}
##  Intent

**Strategy** is a behavioral design pattern that lets you define a
family of algorithms, put each of them into a separate class, and make
their objects interchangeable.

<figure class="image">
<img
src="../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Strategy design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

One day you decided to create a navigation app for casual travelers. The
app was centered around a beautiful map which helped users quickly
orient themselves in any city.

One of the most requested features for the app was automatic route
planning. A user should be able to enter an address and see the fastest
route to that destination displayed on the map.

The first version of the app could only build the routes over roads.
People who traveled by car were bursting with joy. But apparently, not
everybody likes to drive on their vacation. So with the next update, you
added an option to build walking routes. Right after that, you added
another option to let people use public transport in their routes.

However, that was only the beginning. Later you planned to add route
building for cyclists. And even later, another option for building
routes through all of a city's tourist attractions.

<figure class="image">
<img
src="../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="The code of the navigator became very bloated" />
<figcaption><p>The code of the navigator
became bloated.</p></figcaption>
</figure>

While from a business perspective the app was a success, the technical
part caused you many headaches. Each time you added a new routing
algorithm, the main class of the navigator doubled in size. At some
point, the beast became too hard to maintain.

Any change to one of the algorithms, whether it was a simple bug fix or
a slight adjustment of the street score, affected the whole class,
increasing the chance of creating an error in already-working code.

In addition, teamwork became inefficient. Your teammates, who had been
hired right after the successful release, complain that they spend too
much time resolving merge conflicts. Implementing a new feature requires
you to change the same huge class, conflicting with the code produced by
other people.
:::

::: {.section .solution}
##  Solution

The Strategy pattern suggests that you take a class that does something
specific in a lot of different ways and extract all of these algorithms
into separate classes called *strategies*.

The original class, called *context*, must have a field for storing a
reference to one of the strategies. The context delegates the work to a
linked strategy object instead of executing it on its own.

The context isn't responsible for selecting an appropriate algorithm for
the job. Instead, the client passes the desired strategy to the context.
In fact, the context doesn't know much about strategies. It works with
all strategies through the same generic interface, which only exposes a
single method for triggering the algorithm encapsulated within the
selected strategy.

This way the context becomes independent of concrete strategies, so you
can add new algorithms or modify existing ones without changing the code
of the context or other strategies.

<figure class="image">
<img
src="../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Route planning strategies" />
<figcaption><p>Route planning strategies.</p></figcaption>
</figure>

In our navigation app, each routing algorithm can be extracted to its
own class with a single `buildRoute` method. The method accepts an
origin and destination and returns a collection of the route's
checkpoints.

Even though given the same arguments, each routing class might build a
different route, the main navigator class doesn't really care which
algorithm is selected since its primary job is to render a set of
checkpoints on the map. The class has a method for switching the active
routing strategy, so its clients, such as the buttons in the user
interface, can replace the currently selected routing behavior with
another one.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/content/strategy/strategy-comic-1-en590c.png?id=f64d7d8e6f83a7792a769039a66007c1"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-en.png?id=f64d7d8e6f83a7792a769039a66007c1 640w,/images/patterns/content/strategy/strategy-comic-1-en-1.5x.png?id=b952bc8d43f5e4e0db81b8e3c62dc205 960w,/images/patterns/content/strategy/strategy-comic-1-en-2x.png?id=7eb14bd7920ad630c1ecf448d40602df 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Various transportation strategies" />
<figcaption><p>Various strategies for getting to
the airport.</p></figcaption>
</figure>

Imagine that you have to get to the airport. You can catch a bus, order
a cab, or get on your bicycle. These are your transportation strategies.
You can pick one of the strategies depending on factors such as budget
or time constraints.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Structure of the Strategy design pattern" /><img
src="../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Structure of the Strategy design pattern" />
</figure>
:::

1.  The **Context** maintains a reference to one of the concrete
    strategies and communicates with this object only via the strategy
    interface.

2.  The **Strategy** interface is common to all concrete strategies. It
    declares a method the context uses to execute a strategy.

3.  **Concrete Strategies** implement different variations of an
    algorithm the context uses.

4.  The context calls the execution method on the linked strategy object
    each time it needs to run the algorithm. The context doesn't know
    what type of strategy it works with or how the algorithm is
    executed.

5.  The **Client** creates a specific strategy object and passes it to
    the context. The context exposes a setter which lets clients replace
    the strategy associated with the context at runtime.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the context uses multiple **strategies** to execute
various arithmetic operations.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The strategy interface declares operations common to all
// supported versions of some algorithm. The context uses this
// interface to call the algorithm defined by the concrete
// strategies.
interface Strategy is
    method execute(a, b)

// Concrete strategies implement the algorithm while following
// the base strategy interface. The interface makes them
// interchangeable in the context.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// The context defines the interface of interest to clients.
class Context is
    // The context maintains a reference to one of the strategy
    // objects. The context doesn&#39;t know the concrete class of a
    // strategy. It should work with all strategies via the
    // strategy interface.
    private strategy: Strategy

    // Usually the context accepts a strategy through the
    // constructor, and also provides a setter so that the
    // strategy can be switched at runtime.
    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    // The context delegates some work to the strategy object
    // instead of implementing multiple versions of the
    // algorithm on its own.
    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// The client code picks a concrete strategy and passes it to
// the context. The client should be aware of the differences
// between strategies in order to make the right choice.
class ExampleApplication is
    method main() is
        Create context object.

        Read first number.
        Read last number.
        Read the desired action from user input.

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        result = context.executeStrategy(First number, Second number)

        Print result.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Applicability

::::::::::: applicability
::: applicability-problem
Use the Strategy pattern when you want to use different variants of an
algorithm within an object and be able to switch from one algorithm to
another during runtime.
:::

::: applicability-solution
The Strategy pattern lets you indirectly alter the object's behavior at
runtime by associating it with different sub-objects which can perform
specific sub-tasks in different ways.
:::

::: applicability-problem
Use the Strategy when you have a lot of similar classes that only differ
in the way they execute some behavior.
:::

::: applicability-solution
The Strategy pattern lets you extract the varying behavior into a
separate class hierarchy and combine the original classes into one,
thereby reducing duplicate code.
:::

::: applicability-problem
Use the pattern to isolate the business logic of a class from the
implementation details of algorithms that may not be as important in the
context of that logic.
:::

::: applicability-solution
The Strategy pattern lets you isolate the code, internal data, and
dependencies of various algorithms from the rest of the code. Various
clients get a simple interface to execute the algorithms and switch them
at runtime.
:::

::: applicability-problem
Use the pattern when your class has a massive conditional statement that
switches between different variants of the same algorithm.
:::

::: applicability-solution
The Strategy pattern lets you do away with such a conditional by
extracting all algorithms into separate classes, all of which implement
the same interface. The original object delegates execution to one of
these objects, instead of implementing all variants of the algorithm.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  In the context class, identify an algorithm that's prone to frequent
    changes. It may also be a massive conditional that selects and
    executes a variant of the same algorithm at runtime.

2.  Declare the strategy interface common to all variants of the
    algorithm.

3.  One by one, extract all algorithms into their own classes. They
    should all implement the strategy interface.

4.  In the context class, add a field for storing a reference to a
    strategy object. Provide a setter for replacing values of that
    field. The context should work with the strategy object only via the
    strategy interface. The context may define an interface which lets
    the strategy access its data.

5.  Clients of the context must associate it with a suitable strategy
    that matches the way they expect the context to perform its primary
    job.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can swap algorithms used inside an object at runtime.
-    You can isolate the implementation details of an algorithm from the
    code that uses it.
-    You can replace inheritance with composition.
-    *Open/Closed Principle*. You can introduce new strategies without
    having to change the context.
:::

::: col-sm-6
-    If you only have a couple of algorithms and they rarely change,
    there's no real reason to overcomplicate the program with new
    classes and interfaces that come along with the pattern.
-    Clients must be aware of the differences between strategies to be
    able to select a proper one.
-    A lot of modern programming languages have functional type support
    that lets you implement different versions of an algorithm inside a
    set of anonymous functions. Then you could use these functions
    exactly as you'd have used the strategy objects, but without
    bloating your code with extra classes and interfaces.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   [Bridge](bridge.html), [State](state.html),
    [Strategy](strategy.html) (and to some degree
    [Adapter](adapter.html)) have very similar structures. Indeed, all
    of these patterns are based on composition, which is delegating work
    to other objects. However, they all solve different problems. A
    pattern isn't just a recipe for structuring your code in a specific
    way. It can also communicate to other developers the problem the
    pattern solves.

-   [Command](command.html) and [Strategy](strategy.html) may look
    similar because you can use both to parameterize an object with some
    action. However, they have very different intents.

    -   You can use *Command* to convert any operation into an object.
        The operation's parameters become fields of that object. The
        conversion lets you defer execution of the operation, queue it,
        store the history of commands, send commands to remote
        services, etc.

    -   On the other hand, *Strategy* usually describes different ways
        of doing the same thing, letting you swap these algorithms
        within a single context class.

-   [Decorator](decorator.html) lets you change the skin of an object,
    while [Strategy](strategy.html) lets you change the guts.

-   [Template Method](template-method.html) is based on inheritance: it
    lets you alter parts of an algorithm by extending those parts in
    subclasses. [Strategy](strategy.html) is based on composition: you
    can alter parts of the object's behavior by supplying it with
    different strategies that correspond to that behavior. *Template
    Method* works at the class level, so it's static. *Strategy* works
    on the object level, letting you switch behaviors at runtime.

-   [State](state.html) can be considered as an extension of
    [Strategy](strategy.html). Both patterns are based on composition:
    they change the behavior of the context by delegating some work to
    helper objects. *Strategy* makes these objects completely
    independent and unaware of each other. However, *State* doesn't
    restrict dependencies between concrete states, letting them alter
    the state of the context at will.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Strategy in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Strategy in C#"){.prog-lang-link}
[![Strategy in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Strategy in C++"){.prog-lang-link}
[![Strategy in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Strategy in Go"){.prog-lang-link}
[![Strategy in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Strategy in Java"){.prog-lang-link}
[![Strategy in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Strategy in PHP"){.prog-lang-link}
[![Strategy in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Strategy in Python"){.prog-lang-link}
[![Strategy in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Strategy in Ruby"){.prog-lang-link}
[![Strategy in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Strategy in Rust"){.prog-lang-link}
[![Strategy in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Strategy in Swift"){.prog-lang-link}
[![Strategy in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Strategy in TypeScript"){.prog-lang-link}
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

[Template Method []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} State](state.html){.btn .btn-default rel="prev"}
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
