:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Flyweight](../../flyweight.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Flyweight](../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Flyweight** in C# {#flyweight-in-c .pattern-example-title-block-title}
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
 [Program](#example-0--Program-cs)
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

#### []{#example-0--Program-cs .anchor} **Program.cs:** Conceptual example

<figure class="code">
<pre class="code" lang="csharp"><code>using System;
using System.Collections.Generic;
using System.Linq;
using System.Text.Json;
// Use Json.NET library, you can download it from NuGet Package Manager

namespace RefactoringGuru.DesignPatterns.Flyweight.Conceptual
{
    // The Flyweight stores a common portion of the state (also called intrinsic
    // state) that belongs to multiple real business entities. The Flyweight
    // accepts the rest of the state (extrinsic state, unique for each entity)
    // via its method parameters.
    public class Flyweight
    {
        private Car _sharedState;

        public Flyweight(Car car)
        {
            this._sharedState = car;
        }

        public void Operation(Car uniqueState)
        {
            string s = JsonSerializer.Serialize(this._sharedState);
            string u = JsonSerializer.Serialize(uniqueState);
            Console.WriteLine($&quot;Flyweight: Displaying shared {s} and unique {u} state.&quot;);
        }
    }

    // The Flyweight Factory creates and manages the Flyweight objects. It
    // ensures that flyweights are shared correctly. When the client requests a
    // flyweight, the factory either returns an existing instance or creates a
    // new one, if it doesn&#39;t exist yet.
    public class FlyweightFactory
    {
        private List&lt;Tuple&lt;Flyweight, string&gt;&gt; flyweights = new List&lt;Tuple&lt;Flyweight, string&gt;&gt;();

        public FlyweightFactory(params Car[] args)
        {
            foreach (var elem in args)
            {
                flyweights.Add(new Tuple&lt;Flyweight, string&gt;(new Flyweight(elem), this.getKey(elem)));
            }
        }

        // Returns a Flyweight&#39;s string hash for a given state.
        public string getKey(Car key)
        {
            List&lt;string&gt; elements = new List&lt;string&gt;();

            elements.Add(key.Model);
            elements.Add(key.Color);
            elements.Add(key.Company);

            if (key.Owner != null &amp;&amp; key.Number != null)
            {
                elements.Add(key.Number);
                elements.Add(key.Owner);
            }

            elements.Sort();

            return string.Join(&quot;_&quot;, elements);
        }

        // Returns an existing Flyweight with a given state or creates a new
        // one.
        public Flyweight GetFlyweight(Car sharedState)
        {
            string key = this.getKey(sharedState);

            if (flyweights.Where(t =&gt; t.Item2 == key).Count() == 0)
            {
                Console.WriteLine(&quot;FlyweightFactory: Can&#39;t find a flyweight, creating new one.&quot;);
                this.flyweights.Add(new Tuple&lt;Flyweight, string&gt;(new Flyweight(sharedState), key));
            }
            else
            {
                Console.WriteLine(&quot;FlyweightFactory: Reusing existing flyweight.&quot;);
            }
            return this.flyweights.Where(t =&gt; t.Item2 == key).FirstOrDefault().Item1;
        }

        public void listFlyweights()
        {
            var count = flyweights.Count;
            Console.WriteLine($&quot;\nFlyweightFactory: I have {count} flyweights:&quot;);
            foreach (var flyweight in flyweights)
            {
                Console.WriteLine(flyweight.Item2);
            }
        }
    }

    public class Car
    {
        public string Owner { get; set; }

        public string Number { get; set; }

        public string Company { get; set; }

        public string Model { get; set; }

        public string Color { get; set; }
    }

    class Program
    {
        static void Main(string[] args)
        {
            // The client code usually creates a bunch of pre-populated
            // flyweights in the initialization stage of the application.
            var factory = new FlyweightFactory(
                new Car { Company = &quot;Chevrolet&quot;, Model = &quot;Camaro2018&quot;, Color = &quot;pink&quot; },
                new Car { Company = &quot;Mercedes Benz&quot;, Model = &quot;C300&quot;, Color = &quot;black&quot; },
                new Car { Company = &quot;Mercedes Benz&quot;, Model = &quot;C500&quot;, Color = &quot;red&quot; },
                new Car { Company = &quot;BMW&quot;, Model = &quot;M5&quot;, Color = &quot;red&quot; },
                new Car { Company = &quot;BMW&quot;, Model = &quot;X6&quot;, Color = &quot;white&quot; }
            );
            factory.listFlyweights();

            addCarToPoliceDatabase(factory, new Car {
                Number = &quot;CL234IR&quot;,
                Owner = &quot;James Doe&quot;,
                Company = &quot;BMW&quot;,
                Model = &quot;M5&quot;,
                Color = &quot;red&quot;
            });

            addCarToPoliceDatabase(factory, new Car {
                Number = &quot;CL234IR&quot;,
                Owner = &quot;James Doe&quot;,
                Company = &quot;BMW&quot;,
                Model = &quot;X1&quot;,
                Color = &quot;red&quot;
            });

            factory.listFlyweights();
        }

        public static void addCarToPoliceDatabase(FlyweightFactory factory, Car car)
        {
            Console.WriteLine(&quot;\nClient: Adding a car to database.&quot;);

            var flyweight = factory.GetFlyweight(new Car {
                Color = car.Color,
                Model = car.Model,
                Company = car.Company
            });

            // The client code either stores or calculates extrinsic state and
            // passes it to the flyweight&#39;s methods.
            flyweight.Operation(car);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Camaro2018_Chevrolet_pink
black_C300_Mercedes Benz
C500_Mercedes Benz_red
BMW_M5_red
BMW_white_X6

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared {&quot;Owner&quot;:null,&quot;Number&quot;:null,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;M5&quot;,&quot;Color&quot;:&quot;red&quot;} and unique {&quot;Owner&quot;:&quot;James Doe&quot;,&quot;Number&quot;:&quot;CL234IR&quot;,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;M5&quot;,&quot;Color&quot;:&quot;red&quot;} state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared {&quot;Owner&quot;:null,&quot;Number&quot;:null,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;X1&quot;,&quot;Color&quot;:&quot;red&quot;} and unique {&quot;Owner&quot;:&quot;James Doe&quot;,&quot;Number&quot;:&quot;CL234IR&quot;,&quot;Company&quot;:&quot;BMW&quot;,&quot;Model&quot;:&quot;X1&quot;,&quot;Color&quot;:&quot;red&quot;} state.

FlyweightFactory: I have 6 flyweights:
Camaro2018_Chevrolet_pink
black_C300_Mercedes Benz
C500_Mercedes Benz_red
BMW_M5_red
BMW_white_X6
BMW_red_X1</code></pre>
</figure>

::: next
#### Read next

[Proxy in C# []{.fa
.fa-arrow-right}](../../proxy/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Facade in
C#](../../facade/csharp/example.html){.btn .btn-default rel="prev"}
:::

## **Flyweight** in Other Languages

[![Flyweight in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Flyweight in C++"){.prog-lang-link}
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
