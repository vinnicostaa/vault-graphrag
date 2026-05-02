:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Proxy](../../proxy.html){.type} / [Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Proxy](../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Proxy** in Python {#proxy-in-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Proxy** is a structural design pattern that provides an object that
acts as a substitute for a real service object used by a client. A proxy
receives client requests, does some work (access control, caching, etc.)
and then passes the request to a service object.

The proxy object has the same interface as a service, which makes it
interchangeable with a real object when passed to a client.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Proxy ](../../proxy.html){.btn .btn-lg .btn-primary}
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** While the Proxy pattern isn't a frequent guest in
most Python applications, it's still very handy in some special cases.
It's irreplaceable when you want to add some additional behaviors to an
object of some existing class without changing the client code.

**Identification:** Proxies delegate all of the real work to some other
object. Each proxy method should, in the end, refer to a service object
unless the proxy is a subclass of a service.
:::::::::::

[]{#example-0}

## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Proxy** design pattern.
It focuses on answering these questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

#### []{#example-0--main-py .anchor} **main.py:** Conceptual example

<figure class="code">
<pre class="code" lang="python"><code>from abc import ABC, abstractmethod


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares common operations for both RealSubject and
    the Proxy. As long as the client works with RealSubject using this
    interface, you&#39;ll be able to pass it a proxy instead of a real subject.
    &quot;&quot;&quot;

    @abstractmethod
    def request(self) -&gt; None:
        pass


class RealSubject(Subject):
    &quot;&quot;&quot;
    The RealSubject contains some core business logic. Usually, RealSubjects are
    capable of doing some useful work which may also be very slow or sensitive -
    e.g. correcting input data. A Proxy can solve these issues without any
    changes to the RealSubject&#39;s code.
    &quot;&quot;&quot;

    def request(self) -&gt; None:
        print(&quot;RealSubject: Handling request.&quot;)


class Proxy(Subject):
    &quot;&quot;&quot;
    The Proxy has an interface identical to the RealSubject.
    &quot;&quot;&quot;

    def __init__(self, real_subject: RealSubject) -&gt; None:
        self._real_subject = real_subject

    def request(self) -&gt; None:
        &quot;&quot;&quot;
        The most common applications of the Proxy pattern are lazy loading,
        caching, controlling the access, logging, etc. A Proxy can perform one
        of these things and then, depending on the result, pass the execution to
        the same method in a linked RealSubject object.
        &quot;&quot;&quot;

        if self.check_access():
            self._real_subject.request()
            self.log_access()

    def check_access(self) -&gt; bool:
        print(&quot;Proxy: Checking access prior to firing a real request.&quot;)
        return True

    def log_access(self) -&gt; None:
        print(&quot;Proxy: Logging the time of request.&quot;, end=&quot;&quot;)


def client_code(subject: Subject) -&gt; None:
    &quot;&quot;&quot;
    The client code is supposed to work with all objects (both subjects and
    proxies) via the Subject interface in order to support both real subjects
    and proxies. In real life, however, clients mostly work with their real
    subjects directly. In this case, to implement the pattern more easily, you
    can extend your proxy from the real subject&#39;s class.
    &quot;&quot;&quot;

    # ...

    subject.request()

    # ...


if __name__ == &quot;__main__&quot;:
    print(&quot;Client: Executing the client code with a real subject:&quot;)
    real_subject = RealSubject()
    client_code(real_subject)

    print(&quot;&quot;)

    print(&quot;Client: Executing the same client code with a proxy:&quot;)
    proxy = Proxy(real_subject)
    client_code(proxy)</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: Executing the client code with a real subject:
RealSubject: Handling request.

Client: Executing the same client code with a proxy:
Proxy: Checking access prior to firing a real request.
RealSubject: Handling request.
Proxy: Logging the time of request.</code></pre>
</figure>

::: next
#### Read next

[Chain of Responsibility in Python []{.fa
.fa-arrow-right}](../../chain-of-responsibility/python/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Flyweight in
Python](../../flyweight/python/example.html){.btn .btn-default
rel="prev"}
:::

## **Proxy** in Other Languages

[![Proxy in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Proxy in C#"){.prog-lang-link}
[![Proxy in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Proxy in C++"){.prog-lang-link}
[![Proxy in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Proxy in Go"){.prog-lang-link}
[![Proxy in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Proxy in Java"){.prog-lang-link}
[![Proxy in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Proxy in PHP"){.prog-lang-link}
[![Proxy in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Proxy in Ruby"){.prog-lang-link}
[![Proxy in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Proxy in Rust"){.prog-lang-link}
[![Proxy in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Proxy in Swift"){.prog-lang-link}
[![Proxy in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Proxy in TypeScript"){.prog-lang-link}
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
