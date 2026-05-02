:::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Замісник](../../proxy.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Замісник](../../../../images/patterns/cards/proxy-minid8d7.png?id=25890b11e7dc5af29625ccd0678b63a8){srcset="/images/patterns/cards/proxy-mini-2x.png?id=8638fac9dc08c992852492f9cb29d9c6 2x"}
:::

# **Замісник** на Rust {#замісник-на-rust .pattern-example-title-block-title}
::::

::::::::::::::: pattern-example-body
::: pattern-example-brief
**Замісник** --- це об'єкт, який виступає прошарком між клієнтом та
реальним сервісним об'єктом. Замісник отримує виклики від клієнта,
виконує свою функцію (контроль доступу, кешування, зміна запиту та
інше), а потім передає виклик сервісному об'єктові.

Замісник має той самий інтерфейс, що і реальний об'єкт, тому для клієнта
немає різниці --- працювати з реальним об'єктом безпосередньо, чи за
допомогою замісника.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Замісник ](../../proxy.html){.btn .btn-lg .btn-primary}
:::

:::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Conceptual Example: Nginx Proxy](#example-0)
:::

::: en-file
 [server](#example-0--server-rs)
:::

:::: en-inside
::: en-file
  [application](#example-0--server-application-rs)
:::
::::

:::: en-inside
::: en-file
  [nginx](#example-0--server-nginx-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::::::
:::::::::::::::

[]{#example-0}

## Conceptual Example: Nginx Proxy {#example-0-title}

A web server such as Nginx can act as a proxy for your application
server:

-   It provides controlled access to your application server.
-   It can do rate limiting.
-   It can do request caching.

#### []{#example-0--server-rs .anchor} **server.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod application;
mod nginx;

pub use nginx::NginxServer;

pub trait Server {
    fn handle_request(&amp;mut self, url: &amp;str, method: &amp;str) -&gt; (u16, String);
}</code></pre>
</figure>

#### []{#example-0--server-application-rs .anchor} **server/application.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::Server;

pub struct Application;

impl Server for Application {
    fn handle_request(&amp;mut self, url: &amp;str, method: &amp;str) -&gt; (u16, String) {
        if url == &quot;/app/status&quot; &amp;&amp; method == &quot;GET&quot; {
            return (200, &quot;Ok&quot;.into());
        }

        if url == &quot;/create/user&quot; &amp;&amp; method == &quot;POST&quot; {
            return (201, &quot;User Created&quot;.into());
        }

        (404, &quot;Not Ok&quot;.into())
    }
}</code></pre>
</figure>

#### []{#example-0--server-nginx-rs .anchor} **server/nginx.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use std::collections::HashMap;

use super::{application::Application, Server};

/// NGINX server is a proxy to an application server.
pub struct NginxServer {
    application: Application,
    max_allowed_requests: u32,
    rate_limiter: HashMap&lt;String, u32&gt;,
}

impl NginxServer {
    pub fn new() -&gt; Self {
        Self {
            application: Application,
            max_allowed_requests: 2,
            rate_limiter: HashMap::default(),
        }
    }

    pub fn check_rate_limiting(&amp;mut self, url: &amp;str) -&gt; bool {
        let rate = self.rate_limiter.entry(url.to_string()).or_insert(1);

        if *rate &gt; self.max_allowed_requests {
            return false;
        }

        *rate += 1;
        true
    }
}

impl Server for NginxServer {
    fn handle_request(&amp;mut self, url: &amp;str, method: &amp;str) -&gt; (u16, String) {
        if !self.check_rate_limiting(url) {
            return (403, &quot;Not Allowed&quot;.into());
        }

        self.application.handle_request(url, method)
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>mod server;

use crate::server::{NginxServer, Server};

fn main() {
    let app_status = &amp;&quot;/app/status&quot;.to_string();
    let create_user = &amp;&quot;/create/user&quot;.to_string();

    let mut nginx = NginxServer::new();

    let (code, body) = nginx.handle_request(app_status, &quot;GET&quot;);
    println!(&quot;Url: {}\nHttpCode: {}\nBody: {}\n&quot;, app_status, code, body);

    let (code, body) = nginx.handle_request(app_status, &quot;GET&quot;);
    println!(&quot;Url: {}\nHttpCode: {}\nBody: {}\n&quot;, app_status, code, body);

    let (code, body) = nginx.handle_request(app_status, &quot;GET&quot;);
    println!(&quot;Url: {}\nHttpCode: {}\nBody: {}\n&quot;, app_status, code, body);

    let (code, body) = nginx.handle_request(create_user, &quot;POST&quot;);
    println!(&quot;Url: {}\nHttpCode: {}\nBody: {}\n&quot;, create_user, code, body);

    let (code, body) = nginx.handle_request(create_user, &quot;GET&quot;);
    println!(&quot;Url: {}\nHttpCode: {}\nBody: {}\n&quot;, create_user, code, body);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 200
Body: Ok

Url: /app/status
HttpCode: 403
Body: Not Allowed

Url: /create/user
HttpCode: 201
Body: User Created

Url: /create/user
HttpCode: 404
Body: Not Ok</code></pre>
</figure>

::: next
#### Читаємо далі

[Ланцюжок обов\'язків на Rust []{.fa
.fa-arrow-right}](../../chain-of-responsibility/rust/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Легковаговик на
Rust](../../flyweight/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Замісник** іншими мовами програмування

[![Замісник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Замісник на C#"){.prog-lang-link}
[![Замісник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Замісник на C++"){.prog-lang-link}
[![Замісник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Замісник на Go"){.prog-lang-link}
[![Замісник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Замісник на Java"){.prog-lang-link}
[![Замісник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Замісник на PHP"){.prog-lang-link}
[![Замісник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Замісник на Python"){.prog-lang-link}
[![Замісник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Замісник на Ruby"){.prog-lang-link}
[![Замісник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Замісник на Swift"){.prog-lang-link}
[![Замісник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Замісник на TypeScript"){.prog-lang-link}
:::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::
::::::::::::::::::::::::::::
