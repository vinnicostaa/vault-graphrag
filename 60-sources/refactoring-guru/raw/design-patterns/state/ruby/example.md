:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** in Ruby {#state-in-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**State** is a behavioral design pattern that allows an object to change
the behavior when its internal state changes.

The pattern extracts state-related behaviors into separate state classes
and forces the original object to delegate the work to an instance of
these classes, instead of acting on its own.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about State ](../../state.html){.btn .btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual Example](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The State pattern is commonly used in Ruby to
convert massive `switch`-base state machines into objects.

**Identification:** State pattern can be recognized by methods that
change their behavior depending on the objects' state, controlled
externally.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **State** design pattern.
It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-rb .anchor} **main.rb:** Conceptual example

<figure class="code">
<pre class="code" lang="ruby"><code># The Context defines the interface of interest to clients. It also maintains a
# reference to an instance of a State subclass, which represents the current
# state of the Context.
class Context
  # A reference to the current state of the Context.
  attr_accessor :state
  private :state

  # @param [State] state
  def initialize(state)
    transition_to(state)
  end

  # The Context allows changing the State object at runtime.
  def transition_to(state)
    puts &quot;Context: Transition to #{state.class}&quot;
    @state = state
    @state.context = self
  end

  # The Context delegates part of its behavior to the current State object.

  def request1
    @state.handle1
  end

  def request2
    @state.handle2
  end
end

# The base State class declares methods that all Concrete State should implement
# and also provides a backreference to the Context object, associated with the
# State. This backreference can be used by States to transition the Context to
# another State.
class State
  attr_accessor :context

  # @abstract
  def handle1
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def handle2
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete States implement various behaviors, associated with a state of the
# Context.

class ConcreteStateA &lt; State
  def handle1
    puts &#39;ConcreteStateA handles request1.&#39;
    puts &#39;ConcreteStateA wants to change the state of the context.&#39;
    @context.transition_to(ConcreteStateB.new)
  end

  def handle2
    puts &#39;ConcreteStateA handles request2.&#39;
  end
end

class ConcreteStateB &lt; State
  def handle1
    puts &#39;ConcreteStateB handles request1.&#39;
  end

  def handle2
    puts &#39;ConcreteStateB handles request2.&#39;
    puts &#39;ConcreteStateB wants to change the state of the context.&#39;
    @context.transition_to(ConcreteStateA.new)
  end
end

# The client code.

context = Context.new(ConcreteStateA.new)
context.request1
context.request2</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA</code></pre>
</figure>

## **State** in Other Languages

[![State in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State in C#"){.prog-lang-link}
[![State in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State in C++"){.prog-lang-link}
[![State in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "State in Go"){.prog-lang-link}
[![State in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State in Java"){.prog-lang-link}
[![State in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State in PHP"){.prog-lang-link}
[![State in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State in Python"){.prog-lang-link}
[![State in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State in Rust"){.prog-lang-link}
[![State in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "State in Swift"){.prog-lang-link}
[![State in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State in TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-en" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive with examples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Buy the eBook **Dive Into Design Patterns** and get the access to
archive with dozens of detailed examples that can be opened right in
your IDE.

[ Learn more...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
