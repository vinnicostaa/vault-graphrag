::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Adapter](../../adapter.html){.type} / [Swift](../../swift.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Adapter](../../../images/patterns/cards/adapter-minid866.png?id=b2ee4f681fb589be5a0685b94692aebb){srcset="/images/patterns/cards/adapter-mini-2x.png?id=8274d99afbbe9c63bfbfd0d68ceeffc7 2x"}
:::

# **Adapter** in Swift {#adapter-in-swift .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Adapter** is a structural design pattern, which allows incompatible
objects to collaborate.

The Adapter acts as a wrapper between two objects. It catches calls for
one object and transforms them to format and interface recognizable by
the second object.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Adapter ](../../adapter.html){.btn .btn-lg
.btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [Example](#example-0--Example-swift)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::

::: en-intro
 [Real World Example](#example-1)
:::

::: en-file
 [Example](#example-1--Example-swift)
:::

::: en-file
 [Output](#example-1--Output-txt)
:::
:::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The Adapter pattern is pretty common in Swift code.
It's very often used in systems based on some legacy code. In such
cases, Adapters make legacy code work with modern classes.

**Identification:** Adapter is recognizable by a constructor which takes
an instance of a different abstract/interface type. When adapter
receives a call to any of its methods, it translates parameters to
appropriate format and then directs the call to one or several methods
of the wrapped object.
::::::::::::::

::::: {style="padding: 1rem; border: solid 4px #e35e5e; border-radius: 1rem; text-align: center"}
::: {style="margin-bottom: 0.5rem;"}
The following examples are available on [Swift
Playgrounds](https://www.alemohamad.com/playgrounds){style="text-decoration: underline; display: inline-block"
rel="nofollow"}.
:::

::: {style="font-size: 13px;"}
Kudos to [Alejandro
Mohamad](https://www.alemohamad.com/){style="text-decoration: underline;"
rel="nofollow"} for creating the Playground version.
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World Example](#example-1){#example-1-tab
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Conceptual Example {#example-0-title}

This example illustrates the structure of the **Adapter** design pattern
and focuses on the following questions:

-   What classes does it consist of?
-   What roles do these classes play?
-   In what way the elements of the pattern are related?

After learning about the pattern's structure it'll be easier for you to
grasp the following example, based on a real-world Swift use case.

#### []{#example-0--Example-swift .anchor} **Example.swift:** Conceptual example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest

/// The Target defines the domain-specific interface used by the client code.
class Target {

    func request() -&gt; String {
        return &quot;Target: The default target&#39;s behavior.&quot;
    }
}

/// The Adaptee contains some useful behavior, but its interface is incompatible
/// with the existing client code. The Adaptee needs some adaptation before the
/// client code can use it.
class Adaptee {

    public func specificRequest() -&gt; String {
        return &quot;.eetpadA eht fo roivaheb laicepS&quot;
    }
}

/// The Adapter makes the Adaptee&#39;s interface compatible with the Target&#39;s
/// interface.
class Adapter: Target {

    private var adaptee: Adaptee

    init(_ adaptee: Adaptee) {
        self.adaptee = adaptee
    }

    override func request() -&gt; String {
        return &quot;Adapter: (TRANSLATED) &quot; + adaptee.specificRequest().reversed()
    }
}

/// The client code supports all classes that follow the Target interface.
class Client {
    // ...
    static func someClientCode(target: Target) {
        print(target.request())
    }
    // ...
}

/// Let&#39;s see how it all works together.
class AdapterConceptual: XCTestCase {

    func testAdapterConceptual() {
        print(&quot;Client: I can work just fine with the Target objects:&quot;)
        Client.someClientCode(target: Target())

        let adaptee = Adaptee()
        print(&quot;Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:&quot;)
        print(&quot;Adaptee: &quot; + adaptee.specificRequest())

        print(&quot;Client: But I can work with it via the Adapter:&quot;)
        Client.someClientCode(target: Adapter(adaptee))
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Client: I can work just fine with the Target objects:
Target: The default target&#39;s behavior.
Client: The Adaptee class has a weird interface. See, I don&#39;t understand it:
Adaptee: .eetpadA eht fo roivaheb laicepS
Client: But I can work with it via the Adapter:
Adapter: (TRANSLATED) Special behavior of the Adaptee.</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Real World Example {#example-1-title}

#### []{#example-1--Example-swift .anchor} **Example.swift:** Real world example

<figure class="code">
<pre class="code" lang="swift"><code>import XCTest
import UIKit

/// Adapter Design Pattern
///
/// Intent: Convert the interface of a class into the interface clients expect.
/// Adapter lets classes work together that couldn&#39;t work otherwise because of
/// incompatible interfaces.

class AdapterRealWorld: XCTestCase {

    /// Example. Let&#39;s assume that our app perfectly works with Facebook
    /// authorization. However, users ask you to add sign in via Twitter.
    ///
    /// Unfortunately, Twitter SDK has a different authorization method.
    ///
    /// Firstly, you have to create the new protocol &#39;AuthService&#39; and insert
    /// the authorization method of Facebook SDK.
    ///
    /// Secondly, write an extension for Twitter SDK and implement methods of
    /// AuthService protocol, just a simple redirect.
    ///
    /// Thirdly, write an extension for Facebook SDK. You should not write any
    /// code at this point as methods already implemented by Facebook SDK.
    ///
    /// It just tells a compiler that both SDKs have the same interface.

    func testAdapterRealWorld() {

        print(&quot;Starting an authorization via Facebook&quot;)
        startAuthorization(with: FacebookAuthSDK())

        print(&quot;Starting an authorization via Twitter.&quot;)
        startAuthorization(with: TwitterAuthSDK())
    }

    func startAuthorization(with service: AuthService) {

        /// The current top view controller of the app
        let topViewController = UIViewController()

        service.presentAuthFlow(from: topViewController)
    }
}

protocol AuthService {

    func presentAuthFlow(from viewController: UIViewController)
}

class FacebookAuthSDK {

    func presentAuthFlow(from viewController: UIViewController) {
        /// Call SDK methods and pass a view controller
        print(&quot;Facebook WebView has been shown.&quot;)
    }
}

class TwitterAuthSDK {

    func startAuthorization(with viewController: UIViewController) {
        /// Call SDK methods and pass a view controller
        print(&quot;Twitter WebView has been shown. Users will be happy :)&quot;)
    }
}

extension TwitterAuthSDK: AuthService {

    /// This is an adapter
    ///
    /// Yeah, we are able to not create another class and just extend an
    /// existing one

    func presentAuthFlow(from viewController: UIViewController) {
        print(&quot;The Adapter is called! Redirecting to the original method...&quot;)
        self.startAuthorization(with: viewController)
    }
}

extension FacebookAuthSDK: AuthService {
    /// This extension just tells a compiler that both SDKs have the same
    /// interface.
}</code></pre>
</figure>

#### []{#example-1--Output-txt .anchor} **Output.txt:** Execution result

<figure class="code">
<pre class="code" lang="output"><code>Starting an authorization via Facebook
Facebook WebView has been shown
///
Starting an authorization via Twitter
The Adapter is called! Redirecting to the original method...
Twitter WebView has been shown. Users will be happy :)</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Conceptual Example](#example-0){#example-0-tab-bottom .nav-item
.nav-link .active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Real World
Example](#example-1){#example-1-tab-bottom .nav-item .nav-link
toggle="tab" role="tab" aria-controls="example-1" aria-selected="true"}
:::

::: next
#### Read next

[Bridge in Swift []{.fa
.fa-arrow-right}](../../bridge/swift/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Singleton in
Swift](../../singleton/swift/example.html){.btn .btn-default rel="prev"}
:::

## **Adapter** in Other Languages

[![Adapter in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Adapter in C#"){.prog-lang-link}
[![Adapter in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Adapter in C++"){.prog-lang-link}
[![Adapter in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Adapter in Go"){.prog-lang-link}
[![Adapter in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Adapter in Java"){.prog-lang-link}
[![Adapter in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Adapter in PHP"){.prog-lang-link}
[![Adapter in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Adapter in Python"){.prog-lang-link}
[![Adapter in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Adapter in Ruby"){.prog-lang-link}
[![Adapter in
Rust](../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Adapter in Rust"){.prog-lang-link}
[![Adapter in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Adapter in TypeScript"){.prog-lang-link}
::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
