::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type}
:::

# Memento {#memento .title}

::: aka
Also known as: [Snapshot]{style="display:inline-block"}
:::

::: {.section .intent}
##  Intent

**Memento** is a behavioral design pattern that lets you save and
restore the previous state of an object without revealing the details of
its implementation.

<figure class="image">
<img
src="../images/patterns/content/memento/memento-en3e3c.png?id=e51abf6a98a5b1f91e0f3a000f113e1a"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-en.png?id=e51abf6a98a5b1f91e0f3a000f113e1a 640w,/images/patterns/content/memento/memento-en-1.5x.png?id=d3bfbeb3e1e04874273c05d4b04334b9 960w,/images/patterns/content/memento/memento-en-2x.png?id=6782c5bbd8e036321dfd86558b540dda 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Memento design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Imagine that you're creating a text editor app. In addition to simple
text editing, your editor can format text, insert inline images, etc.

At some point, you decided to let users undo any operations carried out
on the text. This feature has become so common over the years that
nowadays people expect every app to have it. For the implementation, you
chose to take the direct approach. Before performing any operation, the
app records the state of all objects and saves it in some storage.
Later, when a user decides to revert an action, the app fetches the
latest snapshot from the history and uses it to restore the state of all
objects.

<figure class="image">
<img
src="../images/patterns/diagrams/memento/problem1-ene323.png?id=e0d795264bcc87b62a27e41d89d54899"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-en.png?id=e0d795264bcc87b62a27e41d89d54899 470w,/images/patterns/diagrams/memento/problem1-en-1.5x.png?id=1d6ba45c84e02ceea3aa855e875ea98f 705w,/images/patterns/diagrams/memento/problem1-en-2x.png?id=566e033ea5d3ab01e47fc4fc8697f994 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Reverting operations in the editor" />
<figcaption><p>Before executing an operation, the app saves a snapshot
of the objects’ state, which can later be used to restore objects to
their previous state.</p></figcaption>
</figure>

Let's think about those state snapshots. How exactly would you produce
one? You'd probably need to go over all the fields in an object and copy
their values into storage. However, this would only work if the object
had quite relaxed access restrictions to its contents. Unfortunately,
most real objects won't let others peek inside them that easily, hiding
all significant data in private fields.

Ignore that problem for now and let's assume that our objects behave
like hippies: preferring open relations and keeping their state public.
While this approach would solve the immediate problem and let you
produce snapshots of objects' states at will, it still has some serious
issues. In the future, you might decide to refactor some of the editor
classes, or add or remove some of the fields. Sounds easy, but this
would also require changing the classes responsible for copying the
state of the affected objects.

<figure class="image">
<img
src="../images/patterns/diagrams/memento/problem2-en9522.png?id=a1c8258832f35d27f1a566322b34bf78"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/memento/problem2-en.png?id=a1c8258832f35d27f1a566322b34bf78 300w,/images/patterns/diagrams/memento/problem2-en-1.5x.png?id=9776088e19f7bd740210d9349184adb2 450w,/images/patterns/diagrams/memento/problem2-en-2x.png?id=0fa1afcb7eaf54093e64722a2f187f8c 600w"
sizes="(max-width: 720px) 100vw, 300px" loading="lazy" width="300"
alt="How to make a copy of the object&#39;s private state?" />
<figcaption><p>How to make a copy of the object’s
private state?</p></figcaption>
</figure>

But there's more. Let's consider the actual "snapshots" of the editor's
state. What data does it contain? At a bare minimum, it must contain the
actual text, cursor coordinates, current scroll position, etc. To make a
snapshot, you'd need to collect these values and put them into some kind
of container.

Most likely, you're going to store lots of these container objects
inside some list that would represent the history. Therefore the
containers would probably end up being objects of one class. The class
would have almost no methods, but lots of fields that mirror the
editor's state. To allow other objects to write and read data to and
from a snapshot, you'd probably need to make its fields public. That
would expose all the editor's states, private or not. Other classes
would become dependent on every little change to the snapshot class,
which would otherwise happen within private fields and methods without
affecting outer classes.

It looks like we've reached a dead end: you either expose all internal
details of classes, making them too fragile, or restrict access to their
state, making it impossible to produce snapshots. Is there any other way
to implement the \"undo\"?
:::

::: {.section .solution}
##  Solution

All problems that we've just experienced are caused by broken
encapsulation. Some objects try to do more than they are supposed to. To
collect the data required to perform some action, they invade the
private space of other objects instead of letting these objects perform
the actual action.

The Memento pattern delegates creating the state snapshots to the actual
owner of that state, the *originator* object. Hence, instead of other
objects trying to copy the editor's state from the "outside," the editor
class itself can make the snapshot since it has full access to its own
state.

The pattern suggests storing the copy of the object's state in a special
object called *memento*. The contents of the memento aren't accessible
to any other object except the one that produced it. Other objects must
communicate with mementos using a limited interface which may allow
fetching the snapshot's metadata (creation time, the name of the
performed operation, etc.), but not the original object's state
contained in the snapshot.

<figure class="image">
<img
src="../images/patterns/diagrams/memento/solution-encd00.png?id=b8adab0fc6d22cf8d630e560b9a8fafd"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-en.png?id=b8adab0fc6d22cf8d630e560b9a8fafd 610w,/images/patterns/diagrams/memento/solution-en-1.5x.png?id=e4f138c0113407c6e41e2ece6dd149a9 915w,/images/patterns/diagrams/memento/solution-en-2x.png?id=cfba1b6b9f65659c29178f7399e30b49 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="The originator has full access to the memento, whereas the caretaker can only access the metadata" />
<figcaption><p>The originator has full access to the memento, whereas
the caretaker can only access the metadata.</p></figcaption>
</figure>

Such a restrictive policy lets you store mementos inside other objects,
usually called *caretakers*. Since the caretaker works with the memento
only via the limited interface, it's not able to tamper with the state
stored inside the memento. At the same time, the originator has access
to all fields inside the memento, allowing it to restore its previous
state at will.

In our text editor example, we can create a separate history class to
act as the caretaker. A stack of mementos stored inside the caretaker
will grow each time the editor is about to execute an operation. You
could even render this stack within the app's UI, displaying the history
of previously performed operations to a user.

When a user triggers the undo, the history grabs the most recent memento
from the stack and passes it back to the editor, requesting a roll-back.
Since the editor has full access to the memento, it changes its own
state with the values taken from the memento.
:::

:::::::::: {.section .structure-container}
##  Structure

::::::::: structure
#### Implementation based on nested classes

The classic implementation of the pattern relies on support for nested
classes, available in many popular programming languages (such as C++,
C#, and Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento based on nested classes" /><img
src="../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Memento based on nested classes" />
</figure>
:::
::::

1.  The **Originator** class can produce snapshots of its own state, as
    well as restore its state from snapshots when needed.

2.  The **Memento** is a value object that acts as a snapshot of the
    originator's state. It's a common practice to make the memento
    immutable and pass it the data only once, via the constructor.

3.  The **Caretaker** knows not only "when" and "why" to capture the
    originator's state, but also when the state should be restored.

    A caretaker can keep track of the originator's history by storing a
    stack of mementos. When the originator has to travel back in
    history, the caretaker fetches the topmost memento from the stack
    and passes it to the originator's restoration method.

4.  In this implementation, the memento class is nested inside the
    originator. This lets the originator access the fields and methods
    of the memento, even though they're declared private. On the other
    hand, the caretaker has very limited access to the memento's fields
    and methods, which lets it store mementos in a stack but not tamper
    with their state.

#### Implementation based on an intermediate interface

There's an alternative implementation, suitable for programming
languages that don't support nested classes (yeah, PHP, I'm talking
about you).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento without nested classes" /><img
src="../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Memento without nested classes" />
</figure>
:::
::::

1.  In the absence of nested classes, you can restrict access to the
    memento's fields by establishing a convention that caretakers can
    work with a memento only through an explicitly declared intermediary
    interface, which would only declare methods related to the memento's
    metadata.

2.  On the other hand, originators can work with a memento object
    directly, accessing fields and methods declared in the memento
    class. The downside of this approach is that you need to declare all
    members of the memento public.

#### Implementation with even stricter encapsulation

There's another implementation which is useful when you don't want to
leave even the slightest chance of other classes accessing the state of
the originator through the memento.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento with strict encapsulation" /><img
src="../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Memento with strict encapsulation" />
</figure>
:::
::::

1.  This implementation allows having multiple types of originators and
    mementos. Each originator works with a corresponding memento class.
    Neither originators nor mementos expose their state to anyone.

2.  Caretakers are now explicitly restricted from changing the state
    stored in mementos. Moreover, the caretaker class becomes
    independent from the originator because the restoration method is
    now defined in the memento class.

3.  Each memento becomes linked to the originator that produced it. The
    originator passes itself to the memento's constructor, along with
    the values of its state. Thanks to the close relationship between
    these classes, a memento can restore the state of its originator,
    given that the latter has defined the appropriate setters.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Pseudocode

This example uses the Memento pattern alongside the
[Command](command.html) pattern for storing snapshots of the complex
text editor's state and restoring an earlier state from these snapshots
when needed.

<figure class="image">
<img
src="../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Structure of the Memento example" />
<figcaption><p>Saving snapshots of the text
editor’s state.</p></figcaption>
</figure>

The command objects act as caretakers. They fetch the editor's memento
before executing operations related to commands. When a user attempts to
undo the most recent command, the editor can use the memento stored in
that command to revert itself to the previous state.

The memento class doesn't declare any public fields, getters or setters.
Therefore no object can alter its contents. Mementos are linked to the
editor object that created them. This lets a memento restore the linked
editor's state by passing the data via setters on the editor object.
Since mementos are linked to specific editor objects, you can make your
app support several independent editor windows with a centralized undo
stack.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The originator holds some important data that may change over
// time. It also defines a method for saving its state inside a
// memento and another method for restoring the state from it.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    // Saves the current state inside a memento.
    method createSnapshot():Snapshot is
        // Memento is an immutable object; that&#39;s why the
        // originator passes its state to the memento&#39;s
        // constructor parameters.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// The memento class stores the past state of the editor.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // At some point, a previous state of the editor can be
    // restored using a memento object.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// A command object can act as a caretaker. In that case, the
// command gets a memento just before it changes the
// originator&#39;s state. When undo is requested, it restores the
// originator&#39;s state from a memento.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Applicability

::::::: applicability
::: applicability-problem
Use the Memento pattern when you want to produce snapshots of the
object's state to be able to restore a previous state of the object.
:::

::: applicability-solution
The Memento pattern lets you make full copies of an object's state,
including private fields, and store them separately from the object.
While most people remember this pattern thanks to the "undo" use case,
it's also indispensable when dealing with transactions (i.e., if you
need to roll back an operation on error).
:::

::: applicability-problem
Use the pattern when direct access to the object's
fields/getters/setters violates its encapsulation.
:::

::: applicability-solution
The Memento makes the object itself responsible for creating a snapshot
of its state. No other object can read the snapshot, making the original
object's state data safe and secure.
:::
:::::::
::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  Determine what class will play the role of the originator. It's
    important to know whether the program uses one central object of
    this type or multiple smaller ones.

2.  Create the memento class. One by one, declare a set of fields that
    mirror the fields declared inside the originator class.

3.  Make the memento class immutable. A memento should accept the data
    just once, via the constructor. The class should have no setters.

4.  If your programming language supports nested classes, nest the
    memento inside the originator. If not, extract a blank interface
    from the memento class and make all other objects use it to refer to
    the memento. You may add some metadata operations to the interface,
    but nothing that exposes the originator's state.

5.  Add a method for producing mementos to the originator class. The
    originator should pass its state to the memento via one or multiple
    arguments of the memento's constructor.

    The return type of the method should be of the interface you
    extracted in the previous step (assuming that you extracted it at
    all). Under the hood, the memento-producing method should work
    directly with the memento class.

6.  Add a method for restoring the originator's state to its class. It
    should accept a memento object as an argument. If you extracted an
    interface in the previous step, make it the type of the parameter.
    In this case, you need to typecast the incoming object to the
    memento class, since the originator needs full access to that
    object.

7.  The caretaker, whether it represents a command object, a history, or
    something entirely different, should know when to request new
    mementos from the originator, how to store them and when to restore
    the originator with a particular memento.

8.  The link between caretakers and originators may be moved into the
    memento class. In this case, each memento must be connected to the
    originator that had created it. The restoration method would also
    move to the memento class. However, this would all make sense only
    if the memento class is nested into originator or the originator
    class provides sufficient setters for overriding its state.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can produce snapshots of the object's state without violating
    its encapsulation.
-    You can simplify the originator's code by letting the caretaker
    maintain the history of the originator's state.
:::

::: col-sm-6
-    The app might consume lots of RAM if clients create mementos too
    often.
-    Caretakers should track the originator's lifecycle to be able to
    destroy obsolete mementos.
-    Most dynamic programming languages, such as PHP, Python and
    JavaScript, can't guarantee that the state within the memento stays
    untouched.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   You can use [Command](command.html) and [Memento](memento.html)
    together when implementing "undo". In this case, commands are
    responsible for performing various operations over a target object,
    while mementos save the state of that object just before a command
    gets executed.

-   You can use [Memento](memento.html) along with
    [Iterator](iterator.html) to capture the current iteration state and
    roll it back if necessary.

-   Sometimes [Prototype](prototype.html) can be a simpler alternative
    to [Memento](memento.html). This works if the object, the state of
    which you want to store in the history, is fairly straightforward
    and doesn't have links to external resources, or the links are easy
    to re-establish.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Memento in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Memento in C#"){.prog-lang-link}
[![Memento in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Memento in C++"){.prog-lang-link}
[![Memento in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Memento in Go"){.prog-lang-link}
[![Memento in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Memento in Java"){.prog-lang-link}
[![Memento in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Memento in PHP"){.prog-lang-link}
[![Memento in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Memento in Python"){.prog-lang-link}
[![Memento in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Memento in Ruby"){.prog-lang-link}
[![Memento in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Memento in Rust"){.prog-lang-link}
[![Memento in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Memento in Swift"){.prog-lang-link}
[![Memento in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Memento in TypeScript"){.prog-lang-link}
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

[Observer []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Mediator](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
