:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Behavioral
Patterns](behavioral-patterns.html){.type} /
[Visitor](visitor.html){.type}
:::

# Visitor and Double Dispatch {#visitor-and-double-dispatch .title}

Let's take a look at following class hierarchy of geometric shapes
(beware the pseudocode):

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ...

class Dot extends Shape is
    field x, y
    method draw()
    // ...

class Circle extends Dot is
    field radius
    method draw()
    // ...

class Rectangle extends Shape is
    field width, height
    method draw()
    // ...

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ...</code></pre>
</figure>

The code works fine and the app is in production. But one day you
decided to make an export feature. The export code would look alien if
placed in these classes. So instead of adding export to all classes of
this hierarchy you decided to create a new class, external to the
hierarchy and put all the export logic inside. The class would get
methods for exporting public state of each object into XML strings:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exporting shape&quot;)
    method export(d: Dot)
        print(&quot;Exporting dot&quot;)
    method export(c: Circle)
        print(&quot;Exporting circle&quot;)
    method export(r: Rectangle)
        print(&quot;Exporting rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exporting compound&quot;)</code></pre>
</figure>

The code looks good, but let's try it out:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Unfortunatelly, this will output &quot;Exporting shape&quot;.</code></pre>
</figure>

Wait! Why?!

## Thinking as a compiler

Note: the following information is true for the most modern
object-oriented programming languages (Java, C#, PHP, and others).

### Late/dynamic binding

Pretend that you're a compiler. You have to decide how to compile the
following code:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Let's see\... the `draw` method defined in `Shape` class. Wait a sec,
but there are also four subclasses that override this method. Can we
safely decide which of the implementations to call here? It doesn't look
so. The only way to know for sure is to launch the program and check the
class of an object passed to the method. The only thing we know for sure
is that object **will have** implementation of the `draw` method.

So the resulting machine code will be checking class of the object
passed to the `shape` parameter and picking the `draw` implementation
from the appropriate class.

Such a dynamic type check is called late (or dynamic) binding:

-   **Late**, because we link object and its implementation after
    compilation, at runtime.
-   **Dynamic**, because every new object might need to be linked to a
    different implementation.

### Early/static binding

Now, let's "compile" following code:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

Everything is clear with the second line: the `Exporter` class doesn't
have a custom constructor, so we just instantiate an object. What's
about the `export` call? The `Exporter` has five methods with the same
name that differ with parameter types. Which one to call? Looks like
we're going to need a dynamic binding here as well.

But there's another problem. What if there's a shape class that doesn't
have appropriate `export` method in `Exporter` class? For instance, an
`Ellipse` object. The compiler can't guarantee that the appropriate
overloaded method exists in contrast with overridden methods. The
ambiguous situation arises which a compiler can't allow.

Therefore, compiler developers use a safe path and use the early (or
static) binding for overloaded methods:

-   **Early** because it happens at compile time, before the program is
    launched.
-   **Static** because it can't be altered at runtime.

Let's return to our example. We're sure that the incoming argument will
be of `Shape` hierarchy: either the `Shape` class or one of its
subclasses. We also know that `Exporter` class has a basic
implementation of the export that supports `Shape` class:
`export(s: Shape)`.

That's the only implementation that can be safely linked to a given code
without making things ambiguous. That's why even if we pass a
`Rectangle` object into `exportShape`, the exporter will still call an
`export(s: Shape)` method.

## Double dispatch

**Double dispatch** is a trick that allows using dynamic binding
alongside with overloaded methods. Here how it's done:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Visited shape&quot;)
    method visit(d: Dot)
        print(&quot;Visited dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // Compiler knows for sure that `this` is a `Shape`.
        // Which means that the `visit(s: Shape)` can be safely called.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Compiler knows that `this` is a `Dot`.
        // Which means that the `visit(s: Dot)` can be safely called.
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// The `accept` method is overriden, not overloaded. Compiler binds it
// dynamically. Therefore the `accept` will be executed on a class that
// corresponds to an object calling a method (in our case, the `Dot` class).
g.accept(v);

// Output: &quot;Visited dot&quot;</code></pre>
</figure>

### Afterword

Even though the [Visitor](visitor.html) pattern is built on the double
dispatch principle, that's not its primary purpose. Visitor lets you add
"external" operations to a whole class hierarchy without changing the
existing code of these classes.

::: next
#### Read next

[Design Patterns in C# []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Visitor](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
