:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** in C++ {#flyweight-in-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Flyweight** is a structural design pattern that allows programs to
support vast quantities of objects by keeping their memory consumption
low.

The pattern achieves it by sharing parts of object state between
multiple objects. In other words, the Flyweight saves RAM by caching the
same data used by different objects.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Flyweight ](../../flyweight.html){.btn .btn-lg
.btn-primary}
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Flyweight pattern has a single purpose:
minimizing memory intake. If your program doesn't struggle with a
shortage of RAM, then you might just ignore this pattern for a while.

**Identification:** Flyweight can be recognized by a creation method
that returns cached objects instead of creating new.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Flyweight** design
pattern. It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-cc .anchor} **main.cc:** Conceptual example

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Flyweight Design Pattern
 *
 * Intent: Lets you fit more objects into the available amount of RAM by sharing
 * common parts of state between multiple objects, instead of keeping all of the
 * data in each object.
 */

struct SharedState
{
    std::string brand_;
    std::string model_;
    std::string color_;

    SharedState(const std::string &amp;brand, const std::string &amp;model, const std::string &amp;color)
        : brand_(brand), model_(model), color_(color)
    {
    }

    friend std::ostream &amp;operator&lt;&lt;(std::ostream &amp;os, const SharedState &amp;ss)
    {
        return os &lt;&lt; &quot;[ &quot; &lt;&lt; ss.brand_ &lt;&lt; &quot; , &quot; &lt;&lt; ss.model_ &lt;&lt; &quot; , &quot; &lt;&lt; ss.color_ &lt;&lt; &quot; ]&quot;;
    }
};

struct UniqueState
{
    std::string owner_;
    std::string plates_;

    UniqueState(const std::string &amp;owner, const std::string &amp;plates)
        : owner_(owner), plates_(plates)
    {
    }

    friend std::ostream &amp;operator&lt;&lt;(std::ostream &amp;os, const UniqueState &amp;us)
    {
        return os &lt;&lt; &quot;[ &quot; &lt;&lt; us.owner_ &lt;&lt; &quot; , &quot; &lt;&lt; us.plates_ &lt;&lt; &quot; ]&quot;;
    }
};

/**
 * The Flyweight stores a common portion of the state (also called intrinsic
 * state) that belongs to multiple real business entities. The Flyweight accepts
 * the rest of the state (extrinsic state, unique for each entity) via its
 * method parameters.
 */
class Flyweight
{
private:
    SharedState *shared_state_;

public:
    Flyweight(const SharedState *shared_state) : shared_state_(new SharedState(*shared_state))
    {
    }
    Flyweight(const Flyweight &amp;other) : shared_state_(new SharedState(*other.shared_state_))
    {
    }
    ~Flyweight()
    {
        delete shared_state_;
    }
    SharedState *shared_state() const
    {
        return shared_state_;
    }
    void Operation(const UniqueState &amp;unique_state) const
    {
        std::cout &lt;&lt; &quot;Flyweight: Displaying shared (&quot; &lt;&lt; *shared_state_ &lt;&lt; &quot;) and unique (&quot; &lt;&lt; unique_state &lt;&lt; &quot;) state.\n&quot;;
    }
};
/**
 * The Flyweight Factory creates and manages the Flyweight objects. It ensures
 * that flyweights are shared correctly. When the client requests a flyweight,
 * the factory either returns an existing instance or creates a new one, if it
 * doesn&#39;t exist yet.
 */
class FlyweightFactory
{
    /**
     * @var Flyweight[]
     */
private:
    std::unordered_map&lt;std::string, Flyweight&gt; flyweights_;
    /**
     * Returns a Flyweight&#39;s string hash for a given state.
     */
    std::string GetKey(const SharedState &amp;ss) const
    {
        return ss.brand_ + &quot;_&quot; + ss.model_ + &quot;_&quot; + ss.color_;
    }

public:
    FlyweightFactory(std::initializer_list&lt;SharedState&gt; share_states)
    {
        for (const SharedState &amp;ss : share_states)
        {
            this-&gt;flyweights_.insert(std::make_pair&lt;std::string, Flyweight&gt;(this-&gt;GetKey(ss), Flyweight(&amp;ss)));
        }
    }

    /**
     * Returns an existing Flyweight with a given state or creates a new one.
     */
    Flyweight GetFlyweight(const SharedState &amp;shared_state)
    {
        std::string key = this-&gt;GetKey(shared_state);
        if (this-&gt;flyweights_.find(key) == this-&gt;flyweights_.end())
        {
            std::cout &lt;&lt; &quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.\n&quot;;
            this-&gt;flyweights_.insert(std::make_pair(key, Flyweight(&amp;shared_state)));
        }
        else
        {
            std::cout &lt;&lt; &quot;FlyweightFactory: Reusing existing flyweight.\n&quot;;
        }
        return this-&gt;flyweights_.at(key);
    }
    void ListFlyweights() const
    {
        size_t count = this-&gt;flyweights_.size();
        std::cout &lt;&lt; &quot;\nFlyweightFactory: I have &quot; &lt;&lt; count &lt;&lt; &quot; flyweights:\n&quot;;
        for (std::pair&lt;std::string, Flyweight&gt; pair : this-&gt;flyweights_)
        {
            std::cout &lt;&lt; pair.first &lt;&lt; &quot;\n&quot;;
        }
    }
};

// ...
void AddCarToPoliceDatabase(
    FlyweightFactory &amp;ff, const std::string &amp;plates, const std::string &amp;owner,
    const std::string &amp;brand, const std::string &amp;model, const std::string &amp;color)
{
    std::cout &lt;&lt; &quot;\nClient: Adding a car to database.\n&quot;;
    const Flyweight &amp;flyweight = ff.GetFlyweight({brand, model, color});
    // The client code either stores or calculates extrinsic state and passes it
    // to the flyweight&#39;s methods.
    flyweight.Operation({owner, plates});
}

/**
 * The client code usually creates a bunch of pre-populated flyweights in the
 * initialization stage of the application.
 */

int main()
{
    FlyweightFactory *factory = new FlyweightFactory({{&quot;Chevrolet&quot;, &quot;Camaro2018&quot;, &quot;pink&quot;}, {&quot;Mercedes Benz&quot;, &quot;C300&quot;, &quot;black&quot;}, {&quot;Mercedes Benz&quot;, &quot;C500&quot;, &quot;red&quot;}, {&quot;BMW&quot;, &quot;M5&quot;, &quot;red&quot;}, {&quot;BMW&quot;, &quot;X6&quot;, &quot;white&quot;}});
    factory-&gt;ListFlyweights();

    AddCarToPoliceDatabase(*factory,
                            &quot;CL234IR&quot;,
                            &quot;James Doe&quot;,
                            &quot;BMW&quot;,
                            &quot;M5&quot;,
                            &quot;red&quot;);

    AddCarToPoliceDatabase(*factory,
                            &quot;CL234IR&quot;,
                            &quot;James Doe&quot;,
                            &quot;BMW&quot;,
                            &quot;X1&quot;,
                            &quot;red&quot;);
    factory-&gt;ListFlyweights();
    delete factory;

    return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
BMW_X6_white
Mercedes Benz_C500_red
Mercedes Benz_C300_black
BMW_M5_red
Chevrolet_Camaro2018_pink

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([ BMW , M5 , red ]) and unique ([ CL234IR , James Doe ]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([ BMW , X1 , red ]) and unique ([ CL234IR , James Doe ]) state.

FlyweightFactory: I have 6 flyweights:
BMW_X1_red
Mercedes Benz_C300_black
BMW_X6_white
Mercedes Benz_C500_red
BMW_M5_red
Chevrolet_Camaro2018_pink</code></pre>
</figure>

::: next
#### Read next

[Proxy in C++ []{.fa
.fa-arrow-right}](../../proxy/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Facade in
C++](../../facade/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Flyweight** in Other Languages

[![Flyweight in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Flyweight in C#"){.prog-lang-link}
[![Flyweight in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Flyweight in Go"){.prog-lang-link}
[![Flyweight in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Flyweight in Java"){.prog-lang-link}
[![Flyweight in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Flyweight in PHP"){.prog-lang-link}
[![Flyweight in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Flyweight in Python"){.prog-lang-link}
[![Flyweight in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Flyweight in Ruby"){.prog-lang-link}
[![Flyweight in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Flyweight in Rust"){.prog-lang-link}
[![Flyweight in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Flyweight in Swift"){.prog-lang-link}
[![Flyweight in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Flyweight in TypeScript"){.prog-lang-link}
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
