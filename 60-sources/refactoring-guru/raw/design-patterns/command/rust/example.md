:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[Command](../../command.html){.type} / [Rust](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Command](../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Command** in Rust {#command-in-rust .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Command** is behavioral design pattern that converts requests or
simple operations into objects.

The conversion allows deferred or remote execution of commands, storing
command history, etc.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Learn more about Command ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Text Editor: Commands and Undo](#example-0)
:::

::: en-file
 [command](#example-0--command-rs)
:::

:::: en-inside
::: en-file
  [copy](#example-0--command-copy-rs)
:::
::::

:::: en-inside
::: en-file
  [cut](#example-0--command-cut-rs)
:::
::::

:::: en-inside
::: en-file
  [paste](#example-0--command-paste-rs)
:::
::::

::: en-file
 [main](#example-0--main-rs)
:::
::::::::::::::

In Rust, a command instance should *NOT hold a permanent reference to
global context*, instead the latter should be passed *from top to down
as a mutable parameter* of the "`execute`" method:

<figure class="code">
<pre class="code" lang="rust"><code>fn execute(&amp;mut self, app: &amp;mut cursive::Cursive) -&gt; bool;</code></pre>
</figure>
:::::::::::::::::

[]{#example-0}

## Text Editor: Commands and Undo {#example-0-title}

Key points:

-   Each button runs a separate command.
-   Because a command is represented as an object, it can be pushed into
    a `history` array in order to be undone later.
-   TUI is created with `cursive` crate.

#### []{#example-0--command-rs .anchor} **command.rs:** Command Inteface

<figure class="code">
<pre class="code" lang="rust"><code>mod copy;
mod cut;
mod paste;

pub use copy::CopyCommand;
pub use cut::CutCommand;
pub use paste::PasteCommand;

/// Declares a method for executing (and undoing) a command.
///
/// Each command receives an application context to access
/// visual components (e.g. edit view) and a clipboard.
pub trait Command {
    fn execute(&amp;mut self, app: &amp;mut cursive::Cursive) -&gt; bool;
    fn undo(&amp;mut self, app: &amp;mut cursive::Cursive);
}</code></pre>
</figure>

#### []{#example-0--command-copy-rs .anchor} **command/copy.rs:** Copy Command

<figure class="code">
<pre class="code" lang="rust"><code>use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct CopyCommand;

impl Command for CopyCommand {
    fn execute(&amp;mut self, app: &amp;mut Cursive) -&gt; bool {
        let editor = app.find_name::&lt;EditView&gt;(&quot;Editor&quot;).unwrap();
        let mut context = app.take_user_data::&lt;AppContext&gt;().unwrap();

        context.clipboard = editor.get_content().to_string();

        app.set_user_data(context);
        false
    }

    fn undo(&amp;mut self, _: &amp;mut Cursive) {}
}</code></pre>
</figure>

#### []{#example-0--command-cut-rs .anchor} **command/cut.rs:** Cut Command

<figure class="code">
<pre class="code" lang="rust"><code>use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct CutCommand {
    backup: String,
}

impl Command for CutCommand {
    fn execute(&amp;mut self, app: &amp;mut Cursive) -&gt; bool {
        let mut editor = app.find_name::&lt;EditView&gt;(&quot;Editor&quot;).unwrap();

        app.with_user_data(|context: &amp;mut AppContext| {
            self.backup = editor.get_content().to_string();
            context.clipboard = self.backup.clone();
            editor.set_content(&quot;&quot;.to_string());
        });

        true
    }

    fn undo(&amp;mut self, app: &amp;mut Cursive) {
        let mut editor = app.find_name::&lt;EditView&gt;(&quot;Editor&quot;).unwrap();
        editor.set_content(&amp;self.backup);
    }
}</code></pre>
</figure>

#### []{#example-0--command-paste-rs .anchor} **command/paste.rs:** Paste Command

<figure class="code">
<pre class="code" lang="rust"><code>use cursive::{views::EditView, Cursive};

use super::Command;
use crate::AppContext;

#[derive(Default)]
pub struct PasteCommand {
    backup: String,
}

impl Command for PasteCommand {
    fn execute(&amp;mut self, app: &amp;mut Cursive) -&gt; bool {
        let mut editor = app.find_name::&lt;EditView&gt;(&quot;Editor&quot;).unwrap();

        app.with_user_data(|context: &amp;mut AppContext| {
            self.backup = editor.get_content().to_string();
            editor.set_content(context.clipboard.clone());
        });

        true
    }

    fn undo(&amp;mut self, app: &amp;mut Cursive) {
        let mut editor = app.find_name::&lt;EditView&gt;(&quot;Editor&quot;).unwrap();
        editor.set_content(&amp;self.backup);
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>mod command;

use cursive::{
    traits::Nameable,
    views::{Dialog, EditView},
    Cursive,
};

use command::{Command, CopyCommand, CutCommand, PasteCommand};

/// An application context to be passed into visual component callbacks.
/// It contains a clipboard and a history of commands to be undone.
#[derive(Default)]
struct AppContext {
    clipboard: String,
    history: Vec&lt;Box&lt;dyn Command&gt;&gt;,
}

fn main() {
    let mut app = cursive::default();

    app.set_user_data(AppContext::default());
    app.add_layer(
        Dialog::around(EditView::default().with_name(&quot;Editor&quot;))
            .title(&quot;Type and use buttons&quot;)
            .button(&quot;Copy&quot;, |s| execute(s, CopyCommand))
            .button(&quot;Cut&quot;, |s| execute(s, CutCommand::default()))
            .button(&quot;Paste&quot;, |s| execute(s, PasteCommand::default()))
            .button(&quot;Undo&quot;, undo)
            .button(&quot;Quit&quot;, |s| s.quit()),
    );

    app.run();
}

/// Executes a command and then pushes it to a history array.
fn execute(app: &amp;mut Cursive, mut command: impl Command + &#39;static) {
    if command.execute(app) {
        app.with_user_data(|context: &amp;mut AppContext| {
            context.history.push(Box::new(command));
        });
    }
}

/// Pops the last command and executes an undo action.
fn undo(app: &amp;mut Cursive) {
    let mut context = app.take_user_data::&lt;AppContext&gt;().unwrap();
    if let Some(mut command) = context.history.pop() {
        command.undo(app)
    }
    app.set_user_data(context);
}</code></pre>
</figure>

# Output

<figure class="image">
<img
src="data:image/png;base64,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AAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQBAEMAABAEgQwAAAASRDAAAAAJEEAAwAAkAQBDAAAQBIEMAAAAEkQwAAAACRBAAMAAJAEAQwAAEASBDAAAABJEMAAAAAkQQADAACQgIj/Bwiv3SlPndWAAAAAAElFTkSuQmCC"
data-fetchpriority="high" alt="Text Editor screenshot" />
</figure>

::: next
#### Read next

[Iterator in Rust []{.fa
.fa-arrow-right}](../../iterator/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Chain of Responsibility in
Rust](../../chain-of-responsibility/rust/example.html){.btn .btn-default
rel="prev"}
:::

## **Command** in Other Languages

[![Command in
C#](../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Command in C#"){.prog-lang-link}
[![Command in
C++](../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Command in C++"){.prog-lang-link}
[![Command in
Go](../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Command in Go"){.prog-lang-link}
[![Command in
Java](../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Command in Java"){.prog-lang-link}
[![Command in
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Command in PHP"){.prog-lang-link}
[![Command in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Command in Python"){.prog-lang-link}
[![Command in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Command in Ruby"){.prog-lang-link}
[![Command in
Swift](../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Command in Swift"){.prog-lang-link}
[![Command in
TypeScript](../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Command in TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
