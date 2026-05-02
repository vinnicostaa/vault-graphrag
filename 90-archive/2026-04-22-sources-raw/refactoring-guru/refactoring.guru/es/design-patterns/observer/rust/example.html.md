::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Observer](../../observer.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observer](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observer** en Rust {#observer-en-rust .pattern-example-title-block-title}
::::

:::::::::::: pattern-example-body
::: pattern-example-brief
**Observer** es un patrón de diseño de comportamiento que permite a un
objeto notificar a otros objetos sobre cambios en su estado.

El patrón Observer proporciona una forma de suscribirse y cancelar la
subscripción a estos eventos para cualquier objeto que implementa una
interfaz suscriptora.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Observer ](../../observer.html){.btn
.btn-lg .btn-primary}
:::

::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Conceptual example](#example-0)
:::

::: en-file
 [editor](#example-0--editor-rs)
:::

::: en-file
 [observer](#example-0--observer-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::
:::::::::
::::::::::::

[]{#example-0}

## Conceptual example {#example-0-title}

In Rust, a convenient way to define a subscriber is to have a function
as a callable object with complex logic passing it to a event publisher.

In this Observer example, Subscribers are either *a lambda function* or
*an explicit function* subscribed to the event. Explicit function
objects could be also unsubscribed (although, there could be limitations
for some function types).

#### []{#example-0--editor-rs .anchor} **editor.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use crate::observer::{Event, Publisher};

/// Editor has its own logic and it utilizes a publisher
/// to operate with subscribers and events.
#[derive(Default)]
pub struct Editor {
    publisher: Publisher,
    file_path: String,
}

impl Editor {
    pub fn events(&amp;mut self) -&gt; &amp;mut Publisher {
        &amp;mut self.publisher
    }

    pub fn load(&amp;mut self, path: String) {
        self.file_path = path.clone();
        self.publisher.notify(Event::Load, path);
    }

    pub fn save(&amp;self) {
        self.publisher.notify(Event::Save, self.file_path.clone());
    }
}</code></pre>
</figure>

#### []{#example-0--observer-rs .anchor} **observer.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use std::collections::HashMap;

/// An event type.
#[derive(PartialEq, Eq, Hash, Clone)]
pub enum Event {
    Load,
    Save,
}

/// A subscriber (listener) has type of a callable function.
pub type Subscriber = fn(file_path: String);

/// Publisher sends events to subscribers (listeners).
#[derive(Default)]
pub struct Publisher {
    events: HashMap&lt;Event, Vec&lt;Subscriber&gt;&gt;,
}

impl Publisher {
    pub fn subscribe(&amp;mut self, event_type: Event, listener: Subscriber) {
        self.events.entry(event_type.clone()).or_default();
        self.events.get_mut(&amp;event_type).unwrap().push(listener);
    }

    pub fn unsubscribe(&amp;mut self, event_type: Event, listener: Subscriber) {
        self.events
            .get_mut(&amp;event_type)
            .unwrap()
            .retain(|&amp;x| x != listener);
    }

    pub fn notify(&amp;self, event_type: Event, file_path: String) {
        let listeners = self.events.get(&amp;event_type).unwrap();
        for listener in listeners {
            listener(file_path.clone());
        }
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use editor::Editor;
use observer::Event;

mod editor;
mod observer;

fn main() {
    let mut editor = Editor::default();

    editor.events().subscribe(Event::Load, |file_path| {
        let log = &quot;/path/to/log/file.txt&quot;.to_string();
        println!(&quot;Save log to {}: Load file {}&quot;, log, file_path);
    });

    editor.events().subscribe(Event::Save, save_listener);

    editor.load(&quot;test1.txt&quot;.into());
    editor.load(&quot;test2.txt&quot;.into());
    editor.save();

    editor.events().unsubscribe(Event::Save, save_listener);
    editor.save();
}

fn save_listener(file_path: String) {
    let email = &quot;admin@example.com&quot;.to_string();
    println!(&quot;Email to {}: Save file {}&quot;, email, file_path);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Save log to /path/to/log/file.txt: Load file test1.txt
Save log to /path/to/log/file.txt: Load file test2.txt
Email to admin@example.com: Save file test2.txt</code></pre>
</figure>

::: next
#### Leer siguiente

[State en Rust []{.fa
.fa-arrow-right}](../../state/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Memento en
Rust](../../memento/rust/example.html){.btn .btn-default rel="prev"}
:::

## **Observer** en otros lenguajes

[![Observer en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observer en C#"){.prog-lang-link}
[![Observer en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observer en C++"){.prog-lang-link}
[![Observer en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observer en Go"){.prog-lang-link}
[![Observer en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observer en Java"){.prog-lang-link}
[![Observer en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observer en PHP"){.prog-lang-link}
[![Observer en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observer en Python"){.prog-lang-link}
[![Observer en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observer en Ruby"){.prog-lang-link}
[![Observer en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observer en Swift"){.prog-lang-link}
[![Observer en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observer en TypeScript"){.prog-lang-link}
::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::
:::::::::::::::::::::::::
