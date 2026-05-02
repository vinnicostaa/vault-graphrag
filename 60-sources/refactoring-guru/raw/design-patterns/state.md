:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# State {#state .title}

::: {.section .intent}
##  Intent

**State** is a behavioral design pattern that lets an object alter its
behavior when its internal state changes. It appears as if the object
changed its class.

<figure class="image">
<img
src="../images/patterns/content/state/state-en8864.png?id=c323fb8c54e2d57bebf4806c087afb07"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-en.png?id=c323fb8c54e2d57bebf4806c087afb07 640w,/images/patterns/content/state/state-en-1.5x.png?id=35e85880bc501216ca374fe53eefeb1d 960w,/images/patterns/content/state/state-en-2x.png?id=dfd427a938223ae880291c2850f3e34a 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="State Design Pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

The State pattern is closely related to the concept of a *Finite-State
Machine* []{.pop-i}[Finite-State Machine:
[https://refactoring.guru/fsm](https://en.wikipedia.org/wiki/Finite-state_machine)]{.pop-content}.

<figure class="image">
<img
src="../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Finite-State Machine" />
<figcaption><p>Finite-State Machine.</p></figcaption>
</figure>

The main idea is that, at any given moment, there's a *finite* number of
*states* which a program can be in. Within any unique state, the program
behaves differently, and the program can be switched from one state to
another instantaneously. However, depending on a current state, the
program may or may not switch to certain other states. These switching
rules, called *transitions*, are also finite and predetermined.

You can also apply this approach to objects. Imagine that we have a
`Document` class. A document can be in one of three states: `Draft`,
`Moderation` and `Published`. The `publish` method of the document works
a little bit differently in each state:

-   In `Draft`, it moves the document to moderation.
-   In `Moderation`, it makes the document public, but only if the
    current user is an administrator.
-   In `Published`, it doesn't do anything at all.

<figure class="image">
<img
src="../images/patterns/diagrams/state/problem2-en7fee.png?id=8a3cb79b309a9d83276278244cdcb610"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-en.png?id=8a3cb79b309a9d83276278244cdcb610 560w,/images/patterns/diagrams/state/problem2-en-1.5x.png?id=b434789130221879f4320d60fc0ceba8 840w,/images/patterns/diagrams/state/problem2-en-2x.png?id=916d1d80335d02b94bec972db93fd94b 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Possible states of a document object" />
<figcaption><p>Possible states and transitions of a
document object.</p></figcaption>
</figure>

State machines are usually implemented with lots of conditional
statements (`if` or `switch`) that select the appropriate behavior
depending on the current state of the object. Usually, this "state" is
just a set of values of the object's fields. Even if you've never heard
about finite-state machines before, you've probably implemented a state
at least once. Does the following code structure ring a bell?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Do nothing.
                break
    // ...</code></pre>
</figure>

The biggest weakness of a state machine based on conditionals reveals
itself once we start adding more and more states and state-dependent
behaviors to the `Document` class. Most methods will contain monstrous
conditionals that pick the proper behavior of a method according to the
current state. Code like this is very difficult to maintain because any
change to the transition logic may require changing state conditionals
in every method.

The problem tends to get bigger as a project evolves. It's quite
difficult to predict all possible states and transitions at the design
stage. Hence, a lean state machine built with a limited set of
conditionals can grow into a bloated mess over time.
:::

::: {.section .solution}
##  Solution

The State pattern suggests that you create new classes for all possible
states of an object and extract all state-specific behaviors into these
classes.

Instead of implementing all behaviors on its own, the original object,
called *context*, stores a reference to one of the state objects that
represents its current state, and delegates all the state-related work
to that object.

<figure class="image">
<img
src="../images/patterns/diagrams/state/solution-en9de4.png?id=2db312e603c026421063dddef065b170"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-en.png?id=2db312e603c026421063dddef065b170 490w,/images/patterns/diagrams/state/solution-en-1.5x.png?id=8e41b0084588290f2768775c21258680 735w,/images/patterns/diagrams/state/solution-en-2x.png?id=73ae9e51f65b2ee61d4abe481dcfc430 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Document delegates the work to a state object" />
<figcaption><p>Document delegates the work to a
state object.</p></figcaption>
</figure>

To transition the context into another state, replace the active state
object with another object that represents that new state. This is
possible only if all state classes follow the same interface and the
context itself works with these objects through that interface.

This structure may look similar to the [Strategy](strategy.html)
pattern, but there's one key difference. In the State pattern, the
particular states may be aware of each other and initiate transitions
from one state to another, whereas strategies almost never know about
each other.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

The buttons and switches in your smartphone behave differently depending
on the current state of the device:

-   When the phone is unlocked, pressing buttons leads to executing
    various functions.
-   When the phone is locked, pressing any button leads to the unlock
    screen.
-   When the phone's charge is low, pressing any button shows the
    charging screen.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/state/structure-en8828.png?id=38c5cc3a610a201e5bc26a441c63d327"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-en.png?id=38c5cc3a610a201e5bc26a441c63d327 540w,/images/patterns/diagrams/state/structure-en-1.5x.png?id=921e29000461174f83c90ba03100f09f 810w,/images/patterns/diagrams/state/structure-en-2x.png?id=69d9c6da31574e2aeafcf39abdd6b74d 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure of the State design pattern" /><img
src="../images/patterns/diagrams/state/structure-en-indexedb9b1.png?id=303874f78151b9aebdc585080e98d773"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-en-indexed.png?id=303874f78151b9aebdc585080e98d773 540w,/images/patterns/diagrams/state/structure-en-indexed-1.5x.png?id=06f3aec92ab2ba08682230003ba3fd9f 810w,/images/patterns/diagrams/state/structure-en-indexed-2x.png?id=d9b987cad93ddb1dfa48e1abe85b0971 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Structure of the State design pattern" />
</figure>
:::

1.  **Context** stores a reference to one of the concrete state objects
    and delegates to it all state-specific work. The context
    communicates with the state object via the state interface. The
    context exposes a setter for passing it a new state object.

2.  The **State** interface declares the state-specific methods. These
    methods should make sense for all concrete states because you don't
    want some of your states to have useless methods that will never be
    called.

3.  **Concrete States** provide their own implementations for the
    state-specific methods. To avoid duplication of similar code across
    multiple states, you may provide intermediate abstract classes that
    encapsulate some common behavior.

    State objects may store a backreference to the context object.
    Through this reference, the state can fetch any required info from
    the context object, as well as initiate state transitions.

4.  Both context and concrete states can set the next state of the
    context and perform the actual state transition by replacing the
    state object linked to the context.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

In this example, the **State** pattern lets the same controls of the
media player behave differently, depending on the current playback
state.

<figure class="image">
<img
src="../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Structure of the State pattern example" />
<figcaption><p>Example of changing object behavior with
state objects.</p></figcaption>
</figure>

The main object of the player is always linked to a state object that
performs most of the work for the player. Some actions replace the
current state object of the player with another, which changes the way
the player reacts to user interactions.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The AudioPlayer class acts as a context. It also maintains a
// reference to an instance of one of the state classes that
// represents the current state of the audio player.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Context delegates handling user input to a state
        // object. Naturally, the outcome depends on what state
        // is currently active, since each state can handle the
        // input differently.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Other objects must be able to switch the audio player&#39;s
    // active state.
    method changeState(state: State) is
        this.state = state

    // UI methods delegate execution to the active state.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // A state may call some service methods on the context.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...


// The base state class declares methods that all concrete
// states should implement and also provides a backreference to
// the context object associated with the state. States can use
// the backreference to transition the context to another state.
abstract class State is
    protected field player: AudioPlayer

    // Context passes itself through the state constructor. This
    // may help a state fetch some useful context data if it&#39;s
    // needed.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Concrete states implement various behaviors associated with a
// state of the context.
class LockedState extends State is

    // When you unlock a locked player, it may assume one of two
    // states.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Locked, so do nothing.

    method clickNext() is
        // Locked, so do nothing.

    method clickPrevious() is
        // Locked, so do nothing.


// They can also trigger state transitions in the context.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Applicability

::::::::: applicability
::: applicability-problem
Use the State pattern when you have an object that behaves differently
depending on its current state, the number of states is enormous, and
the state-specific code changes frequently.
:::

::: applicability-solution
The pattern suggests that you extract all state-specific code into a set
of distinct classes. As a result, you can add new states or change
existing ones independently of each other, reducing the maintenance
cost.
:::

::: applicability-problem
Use the pattern when you have a class polluted with massive conditionals
that alter how the class behaves according to the current values of the
class's fields.
:::

::: applicability-solution
The State pattern lets you extract branches of these conditionals into
methods of corresponding state classes. While doing so, you can also
clean temporary fields and helper methods involved in state-specific
code out of your main class.
:::

::: applicability-problem
Use State when you have a lot of duplicate code across similar states
and transitions of a condition-based state machine.
:::

::: applicability-solution
The State pattern lets you compose hierarchies of state classes and
reduce duplication by extracting common code into abstract base classes.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Decide what class will act as the context. It could be an existing
    class which already has the state-dependent code; or a new class, if
    the state-specific code is distributed across multiple classes.

2.  Declare the state interface. Although it may mirror all the methods
    declared in the context, aim only for those that may contain
    state-specific behavior.

3.  For every actual state, create a class that derives from the state
    interface. Then go over the methods of the context and extract all
    code related to that state into your newly created class.

    While moving the code to the state class, you might discover that it
    depends on private members of the context. There are several
    workarounds:

    -   Make these fields or methods public.
    -   Turn the behavior you're extracting into a public method in the
        context and call it from the state class. This way is ugly but
        quick, and you can always fix it later.
    -   Nest the state classes into the context class, but only if your
        programming language supports nesting classes.

4.  In the context class, add a reference field of the state interface
    type and a public setter that allows overriding the value of that
    field.

5.  Go over the method of the context again and replace empty state
    conditionals with calls to corresponding methods of the state
    object.

6.  To switch the state of the context, create an instance of one of the
    state classes and pass it to the context. You can do this within the
    context itself, or in various states, or in the client. Wherever
    this is done, the class becomes dependent on the concrete state
    class that it instantiates.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    *Single Responsibility Principle*. Organize the code related to
    particular states into separate classes.
-    *Open/Closed Principle*. Introduce new states without changing
    existing state classes or the context.
-    Simplify the code of the context by eliminating bulky state machine
    conditionals.
:::

::: col-sm-6
-    Applying the pattern can be overkill if a state machine has only a
    few states or rarely changes.
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

[![State in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "State in C#"){.prog-lang-link}
[![State in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "State in C++"){.prog-lang-link}
[![State in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "State in Go"){.prog-lang-link}
[![State in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "State in Java"){.prog-lang-link}
[![State in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "State in PHP"){.prog-lang-link}
[![State in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "State in Python"){.prog-lang-link}
[![State in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "State in Ruby"){.prog-lang-link}
[![State in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "State in Rust"){.prog-lang-link}
[![State in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "State in Swift"){.prog-lang-link}
[![State in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "State in TypeScript"){.prog-lang-link}
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

[Strategy []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Observer](observer.html){.btn .btn-default
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
