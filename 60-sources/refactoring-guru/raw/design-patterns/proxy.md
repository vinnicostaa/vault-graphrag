:::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../index.html){.home} / [Design
Patterns](../design-patterns.html){.type} / [Structural
Patterns](structural-patterns.html){.type}
:::

# Proxy {#proxy .title}

::: {.section .intent}
##  Intent

**Proxy** is a structural design pattern that lets you provide a
substitute or placeholder for another object. A proxy controls access to
the original object, allowing you to perform something either before or
after the request gets through to the original object.

<figure class="image">
<img
src="../images/patterns/content/proxy/proxyf258.png?id=efece4647fb11e3f7539291796327666"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/proxy/proxy.png?id=efece4647fb11e3f7539291796327666 640w,/images/patterns/content/proxy/proxy-1.5x.png?id=80a11690fa49172c517470a834289b60 960w,/images/patterns/content/proxy/proxy-2x.png?id=fb3d14e21c210a758d4777f4d93dce09 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Proxy design pattern" />
</figure>
:::

::: {.section .problem}
##  Problem

Why would you want to control access to an object? Here is an example:
you have a massive object that consumes a vast amount of system
resources. You need it from time to time, but not always.

<figure class="image">
<img
src="../images/patterns/diagrams/proxy/problem-en84d1.png?id=b36e65189e939de5dc809636c1946a43"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/problem-en.png?id=b36e65189e939de5dc809636c1946a43 510w,/images/patterns/diagrams/proxy/problem-en-1.5x.png?id=344d31dfcf156ae35b7e4005b0959f28 765w,/images/patterns/diagrams/proxy/problem-en-2x.png?id=50393231365429384bd08c3fe3807f56 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Problem solved by Proxy pattern" />
<figcaption><p>Database queries can be really slow.</p></figcaption>
</figure>

You could implement lazy initialization: create this object only when
it's actually needed. All of the object's clients would need to execute
some deferred initialization code. Unfortunately, this would probably
cause a lot of code duplication.

In an ideal world, we'd want to put this code directly into our object's
class, but that isn't always possible. For instance, the class may be
part of a closed 3rd-party library.
:::

::: {.section .solution}
##  Solution

The Proxy pattern suggests that you create a new proxy class with the
same interface as an original service object. Then you update your app
so that it passes the proxy object to all of the original object's
clients. Upon receiving a request from a client, the proxy creates a
real service object and delegates all the work to it.

<figure class="image">
<img
src="../images/patterns/diagrams/proxy/solution-enc6c3.png?id=ab36b8b03fabf92c7dd10ad87507b78c"
style="aspect-ratio: 3.19;"
srcset="/images/patterns/diagrams/proxy/solution-en.png?id=ab36b8b03fabf92c7dd10ad87507b78c 510w,/images/patterns/diagrams/proxy/solution-en-1.5x.png?id=5b127cfce320b214653a58a5ade5f887 765w,/images/patterns/diagrams/proxy/solution-en-2x.png?id=06d3d96b36666ea5762250dbc8d5e624 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Solution with the Proxy pattern" />
<figcaption><p>The proxy disguises itself as a database object. It can
handle lazy initialization and result caching without the client or the
real database object even knowing.</p></figcaption>
</figure>

But what's the benefit? If you need to execute something either before
or after the primary logic of the class, the proxy lets you do this
without changing that class. Since the proxy implements the same
interface as the original class, it can be passed to any client that
expects a real service object.
:::

::: {.section .analogy}
##  Real-World Analogy {#analogy}

<figure class="image">
<img
src="../images/patterns/diagrams/proxy/live-example66da.png?id=a268c57fdaf073ee81cf4dfc7239eae2"
style="aspect-ratio: 2.57;"
srcset="/images/patterns/diagrams/proxy/live-example.png?id=a268c57fdaf073ee81cf4dfc7239eae2 540w,/images/patterns/diagrams/proxy/live-example-1.5x.png?id=4b8ee7f90cf76180004e9f5d37661ee2 810w,/images/patterns/diagrams/proxy/live-example-2x.png?id=8b8083fa1954d2d92ca84a5a5636ead7 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="A credit card is a proxy for a bundle of cash" />
<figcaption><p>Credit cards can be used for payments just the same
as cash.</p></figcaption>
</figure>

A credit card is a proxy for a bank account, which is a proxy for a
bundle of cash. Both implement the same interface: they can be used for
making a payment. A consumer feels great because there's no need to
carry loads of cash around. A shop owner is also happy since the income
from a transaction gets added electronically to the shop's bank account
without the risk of losing the deposit or getting robbed on the way to
the bank.
:::

::::: {.section .structure-container}
##  Structure

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../images/patterns/diagrams/proxy/structure7fcd.png?id=f2478a82a84e1a1e512a8414bf1abd1c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/proxy/structure.png?id=f2478a82a84e1a1e512a8414bf1abd1c 370w,/images/patterns/diagrams/proxy/structure-1.5x.png?id=0db7bf3c1193b2a1961894349f1e07ad 555w,/images/patterns/diagrams/proxy/structure-2x.png?id=3d54eeca9af4aa373e989a73463539b5 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Structure of the Proxy design pattern" /><img
src="../images/patterns/diagrams/proxy/structure-indexed2b2c.png?id=e56d420f31925b3d41348c5036d3cd64"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.05;"
srcset="/images/patterns/diagrams/proxy/structure-indexed.png?id=e56d420f31925b3d41348c5036d3cd64 410w,/images/patterns/diagrams/proxy/structure-indexed-1.5x.png?id=5b24af632e76d81d24eab0b7c0be1a66 615w,/images/patterns/diagrams/proxy/structure-indexed-2x.png?id=ddf2b3b4807e52330c456c59fc52d504 820w"
sizes="(max-width: 720px) 100vw, 410px" loading="lazy" width="410"
alt="Structure of the Proxy design pattern" />
</figure>
:::

1.  The **Service Interface** declares the interface of the Service. The
    proxy must follow this interface to be able to disguise itself as a
    service object.

2.  The **Service** is a class that provides some useful business logic.

3.  The **Proxy** class has a reference field that points to a service
    object. After the proxy finishes its processing (e.g., lazy
    initialization, logging, access control, caching, etc.), it passes
    the request to the service object.

    Usually, proxies manage the full lifecycle of their service objects.

4.  The **Client** should work with both services and proxies via the
    same interface. This way you can pass a proxy into any code that
    expects a service object.
::::
:::::

::: {.section .pseudocode}
##  Pseudocode

This example illustrates how the **Proxy** pattern can help to introduce
lazy initialization and caching to a 3rd-party YouTube integration
library.

<figure class="image">
<img
src="../images/patterns/diagrams/proxy/example4e80.png?id=ddb1402479562b9e2c776933cc861bed"
style="aspect-ratio: 1.23;"
srcset="/images/patterns/diagrams/proxy/example.png?id=ddb1402479562b9e2c776933cc861bed 490w,/images/patterns/diagrams/proxy/example-1.5x.png?id=9b7608e1ef46a52d4ca1d7d89986e5b0 735w,/images/patterns/diagrams/proxy/example-2x.png?id=40f22d12d183b9285a380e4a072efb16 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Structure of the Proxy pattern example" />
<figcaption><p>Caching results of a service with
a proxy.</p></figcaption>
</figure>

The library provides us with the video downloading class. However, it's
very inefficient. If the client application requests the same video
multiple times, the library just downloads it over and over, instead of
caching and reusing the first downloaded file.

The proxy class implements the same interface as the original downloader
and delegates it all the work. However, it keeps track of the downloaded
files and returns the cached result when the app requests the same video
multiple times.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// The interface of a remote service.
interface ThirdPartyYouTubeLib is
    method listVideos()
    method getVideoInfo(id)
    method downloadVideo(id)

// The concrete implementation of a service connector. Methods
// of this class can request information from YouTube. The speed
// of the request depends on a user&#39;s internet connection as
// well as YouTube&#39;s. The application will slow down if a lot of
// requests are fired at the same time, even if they all request
// the same information.
class ThirdPartyYouTubeClass implements ThirdPartyYouTubeLib is
    method listVideos() is
        // Send an API request to YouTube.

    method getVideoInfo(id) is
        // Get metadata about some video.

    method downloadVideo(id) is
        // Download a video file from YouTube.

// To save some bandwidth, we can cache request results and keep
// them for some time. But it may be impossible to put such code
// directly into the service class. For example, it could have
// been provided as part of a third party library and/or defined
// as `final`. That&#39;s why we put the caching code into a new
// proxy class which implements the same interface as the
// service class. It delegates to the service object only when
// the real requests have to be sent.
class CachedYouTubeClass implements ThirdPartyYouTubeLib is
    private field service: ThirdPartyYouTubeLib
    private field listCache, videoCache
    field needReset

    constructor CachedYouTubeClass(service: ThirdPartyYouTubeLib) is
        this.service = service

    method listVideos() is
        if (listCache == null || needReset)
            listCache = service.listVideos()
        return listCache

    method getVideoInfo(id) is
        if (videoCache == null || needReset)
            videoCache = service.getVideoInfo(id)
        return videoCache

    method downloadVideo(id) is
        if (!downloadExists(id) || needReset)
            service.downloadVideo(id)

// The GUI class, which used to work directly with a service
// object, stays unchanged as long as it works with the service
// object through an interface. We can safely pass a proxy
// object instead of a real service object since they both
// implement the same interface.
class YouTubeManager is
    protected field service: ThirdPartyYouTubeLib

    constructor YouTubeManager(service: ThirdPartyYouTubeLib) is
        this.service = service

    method renderVideoPage(id) is
        info = service.getVideoInfo(id)
        // Render the video page.

    method renderListPanel() is
        list = service.listVideos()
        // Render the list of video thumbnails.

    method reactOnUserInput() is
        renderVideoPage()
        renderListPanel()

// The application can configure proxies on the fly.
class Application is
    method init() is
        aYouTubeService = new ThirdPartyYouTubeClass()
        aYouTubeProxy = new CachedYouTubeClass(aYouTubeService)
        manager = new YouTubeManager(aYouTubeProxy)
        manager.reactOnUserInput()</code></pre>
</figure>
:::

:::::::::::::::: {.section .applicability-container}
##  Applicability

::::::::::::::: applicability
There are dozens of ways to utilize the Proxy pattern. Let's go over the
most popular uses.

::: applicability-problem
Lazy initialization (virtual proxy). This is when you have a heavyweight
service object that wastes system resources by being always up, even
though you only need it from time to time.
:::

::: applicability-solution
Instead of creating the object when the app launches, you can delay the
object's initialization to a time when it's really needed.
:::

::: applicability-problem
Access control (protection proxy). This is when you want only specific
clients to be able to use the service object; for instance, when your
objects are crucial parts of an operating system and clients are various
launched applications (including malicious ones).
:::

::: applicability-solution
The proxy can pass the request to the service object only if the
client's credentials match some criteria.
:::

::: applicability-problem
Local execution of a remote service (remote proxy). This is when the
service object is located on a remote server.
:::

::: applicability-solution
In this case, the proxy passes the client request over the network,
handling all of the nasty details of working with the network.
:::

::: applicability-problem
Logging requests (logging proxy). This is when you want to keep a
history of requests to the service object.
:::

::: applicability-solution
The proxy can log each request before passing it to the service.
:::

::: applicability-problem
Caching request results (caching proxy). This is when you need to cache
results of client requests and manage the life cycle of this cache,
especially if results are quite large.
:::

::: applicability-solution
The proxy can implement caching for recurring requests that always yield
the same results. The proxy may use the parameters of requests as the
cache keys.
:::

::: applicability-problem
Smart reference. This is when you need to be able to dismiss a
heavyweight object once there are no clients that use it.
:::

::: applicability-solution
The proxy can keep track of clients that obtained a reference to the
service object or its results. From time to time, the proxy may go over
the clients and check whether they are still active. If the client list
gets empty, the proxy might dismiss the service object and free the
underlying system resources.

The proxy can also track whether the client had modified the service
object. Then the unchanged objects may be reused by other clients.
:::
:::::::::::::::
::::::::::::::::

::: {.section .checklist}
##  How to Implement {#checklist}

1.  If there's no pre-existing service interface, create one to make
    proxy and service objects interchangeable. Extracting the interface
    from the service class isn't always possible, because you'd need to
    change all of the service's clients to use that interface. Plan B is
    to make the proxy a subclass of the service class, and this way
    it'll inherit the interface of the service.

2.  Create the proxy class. It should have a field for storing a
    reference to the service. Usually, proxies create and manage the
    whole life cycle of their services. On rare occasions, a service is
    passed to the proxy via a constructor by the client.

3.  Implement the proxy methods according to their purposes. In most
    cases, after doing some work, the proxy should delegate the work to
    the service object.

4.  Consider introducing a creation method that decides whether the
    client gets a proxy or a real service. This can be a simple static
    method in the proxy class or a full-blown factory method.

5.  Consider implementing lazy initialization for the service object.
:::

:::::: {.section .pros-cons}
##  Pros and Cons {#pros-cons}

::::: row
::: col-sm-6
-    You can control the service object without clients knowing about
    it.
-    You can manage the lifecycle of the service object when clients
    don't care about it.
-    The proxy works even if the service object isn't ready or is not
    available.
-    *Open/Closed Principle*. You can introduce new proxies without
    changing the service or clients.
:::

::: col-sm-6
-    The code may become more complicated since you need to introduce a
    lot of new classes.
-    The response from the service might get delayed.
:::
:::::
::::::

::: {.section .relations}
##  Relations with Other Patterns {#relations}

-   With [Adapter](adapter.html) you access an existing object via
    different interface. With [Proxy](proxy.html), the interface stays
    the same. With [Decorator](decorator.html) you access the object via
    an enhanced interface.

-   [Facade](facade.html) is similar to [Proxy](proxy.html) in that both
    buffer a complex entity and initialize it on its own. Unlike
    *Facade*, *Proxy* has the same interface as its service object,
    which makes them interchangeable.

-   [Decorator](decorator.html) and [Proxy](proxy.html) have similar
    structures, but very different intents. Both patterns are built on
    the composition principle, where one object is supposed to delegate
    some of the work to another. The difference is that a *Proxy*
    usually manages the life cycle of its service object on its own,
    whereas the composition of *Decorators* is always controlled by the
    client.
:::

::: {.section .implementations}
##  Code Examples {#implementations}

[![Proxy in
C#](../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](proxy/csharp/example.html "Proxy in C#"){.prog-lang-link}
[![Proxy in
C++](../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](proxy/cpp/example.html "Proxy in C++"){.prog-lang-link}
[![Proxy in
Go](../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](proxy/go/example.html "Proxy in Go"){.prog-lang-link}
[![Proxy in
Java](../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](proxy/java/example.html "Proxy in Java"){.prog-lang-link}
[![Proxy in
PHP](../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](proxy/php/example.html "Proxy in PHP"){.prog-lang-link}
[![Proxy in
Python](../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](proxy/python/example.html "Proxy in Python"){.prog-lang-link}
[![Proxy in
Ruby](../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](proxy/ruby/example.html "Proxy in Ruby"){.prog-lang-link}
[![Proxy in
Rust](../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](proxy/rust/example.html "Proxy in Rust"){.prog-lang-link}
[![Proxy in
Swift](../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](proxy/swift/example.html "Proxy in Swift"){.prog-lang-link}
[![Proxy in
TypeScript](../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](proxy/typescript/example.html "Proxy in TypeScript"){.prog-lang-link}
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

[Behavioral Patterns []{.fa
.fa-arrow-right}](behavioral-patterns.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Flyweight](flyweight.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::
