::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [디자인
패턴들](../../../design-patterns.html){.type} / [팩토리
메서드](../../factory-method.html){.type} /
[러스트](../../rust.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![팩토리
메서드](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# 러스트로 작성된 **팩토리 메서드** {#러스트로-작성된-팩토리-메서드 .pattern-example-title-block-title}
::::

::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**팩토리 메서드**는 제품 객체들의 구상 클래스들을 지정하지 않고 해당
제품 객체들을 생성할 수 있도록 하는 생성 디자인 패턴입니다.

팩토리 메서드는 메서드를 정의하며, 이 메서드는 직접 생성자 호출​(`new`
연산자)​을 사용하여 객체를 생성하는 대신 객체 생성에 사용되여야 합니다.
자식 클래스들은 이 메서드를 오버라이드하여 생성될 객체들의 클래스를
변경할 수 있습니다.

> 다양한 팩토리 패턴들과 개념들의 차이점을 이해하지 못하셨다면 [팩토리
> 비교](../../factory-comparison.html)를 읽어보세요.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ 팩토리 메서드에 대하여 더 자세히 알아보세요
](../../factory-method.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
내비게이션
:::

::: en-intro
 [소개](#)
:::

::: en-intro
 [Dialog Rendering](#example-0)
:::

::: en-file
 [gui](#example-0--gui-rs)
:::

::: en-file
 [html_gui](#example-0--html_gui-rs)
:::

::: en-file
 [windows_gui](#example-0--windows_gui-rs)
:::

::: en-file
 [init](#example-0--init-rs)
:::

::: en-file
 [main](#example-0--main-rs)
:::

::: en-intro
 [Maze Game](#example-1)
:::

::: en-file
 [game](#example-1--game-rs)
:::

::: en-file
 [magic_maze](#example-1--magic_maze-rs)
:::

::: en-file
 [ordinary_maze](#example-1--ordinary_maze-rs)
:::

::: en-file
 [main](#example-1--main-rs)
:::
::::::::::::::::
:::::::::::::::::::

::: {#nav-tab .nav .nav-tabs .nav-justified role="tablist"}
[Dialog Rendering](#example-0){#example-0-tab .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Maze Game](#example-1){#example-1-tab .nav-item
.nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::::: {#nav-tabContent .pattern-example-tab-content .tab-content}
::: {#example-0 .tab-pane .show .active role="tabpanel" aria-labelledby="example-0-tab" style="padding-top: 24px"}
## Dialog Rendering {#example-0-title}

This example illustrates how to organize a GUI framework into
independent modules using **dynamic dispatch**:

1.  The `gui` module defines interfaces for all the components.\
    It has no external dependencies.
2.  The `html_gui` module provides HTML implementation of the base GUI.\
    Depends on `gui`.
3.  The `windows_gui` module provides Windows implementation of the base
    GUI.\
    Depends on `gui`.

The `app` is a client application that can use several implementations
of the GUI framework, depending on the current environment or
configuration. However, most of the app code doesn\'t depend on specific
types of GUI elements. All client code works with GUI elements through
abstract interfaces defined by the `gui` module.

The [Abstract Factory
example](../../../../design-patterns/abstract-factory/rust/example.html)
demonstrates an even greater separation of a factory interface and its
implementations.

#### []{#example-0--gui-rs .anchor} **gui.rs:** Product & Creator

<figure class="code">
<pre class="code" lang="rust"><code>pub trait Button {
    fn render(&amp;self);
    fn on_click(&amp;self);
}

/// Dialog has a factory method `create_button`.
///
/// It creates different buttons depending on a factory implementation.
pub trait Dialog {
    /// The factory method. It must be overridden with a concrete implementation.
    fn create_button(&amp;self) -&gt; Box&lt;dyn Button&gt;;

    fn render(&amp;self) {
        let button = self.create_button();
        button.render();
    }

    fn refresh(&amp;self) {
        println!(&quot;Dialog - Refresh&quot;);
    }
}</code></pre>
</figure>

#### []{#example-0--html_gui-rs .anchor} **html_gui.rs:** Concrete creator

<figure class="code">
<pre class="code" lang="rust"><code>use crate::gui::{Button, Dialog};

pub struct HtmlButton;

impl Button for HtmlButton {
    fn render(&amp;self) {
        println!(&quot;&lt;button&gt;Test Button&lt;/button&gt;&quot;);
        self.on_click();
    }

    fn on_click(&amp;self) {
        println!(&quot;Click! Button says - &#39;Hello World!&#39;&quot;);
    }
}

pub struct HtmlDialog;

impl Dialog for HtmlDialog {
    /// Creates an HTML button.
    fn create_button(&amp;self) -&gt; Box&lt;dyn Button&gt; {
        Box::new(HtmlButton)
    }
}</code></pre>
</figure>

#### []{#example-0--windows_gui-rs .anchor} **windows_gui.rs:** Another concrete creator

<figure class="code">
<pre class="code" lang="rust"><code>use crate::gui::{Button, Dialog};

pub struct WindowsButton;

impl Button for WindowsButton {
    fn render(&amp;self) {
        println!(&quot;Drawing a Windows button&quot;);
        self.on_click();
    }

    fn on_click(&amp;self) {
        println!(&quot;Click! Hello, Windows!&quot;);
    }
}

pub struct WindowsDialog;

impl Dialog for WindowsDialog {
    /// Creates a Windows button.
    fn create_button(&amp;self) -&gt; Box&lt;dyn Button&gt; {
        Box::new(WindowsButton)
    }
}</code></pre>
</figure>

#### []{#example-0--init-rs .anchor} **init.rs:** Initialization code

<figure class="code">
<pre class="code" lang="rust"><code>use crate::gui::Dialog;
use crate::html_gui::HtmlDialog;
use crate::windows_gui::WindowsDialog;

pub fn initialize() -&gt; &amp;&#39;static dyn Dialog {
    // The dialog type is selected depending on the environment settings or configuration.
    if cfg!(windows) {
        println!(&quot;-- Windows detected, creating Windows GUI --&quot;);
        &amp;WindowsDialog
    } else {
        println!(&quot;-- No OS detected, creating the HTML GUI --&quot;);
        &amp;HtmlDialog
    }
}</code></pre>
</figure>

#### []{#example-0--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>mod gui;
mod html_gui;
mod init;
mod windows_gui;

use init::initialize;

fn main() {
    // The rest of the code doesn&#39;t depend on specific dialog types, because
    // it works with all dialog objects via the abstract `Dialog` trait
    // which is defined in the `gui` module.
    let dialog = initialize();
    dialog.render();
    dialog.refresh();
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>&lt;button&gt;Test Button&lt;/button&gt;
Click! Button says - &#39;Hello World!&#39;
Dialog - Refresh</code></pre>
</figure>
:::

::: {#example-1 .tab-pane role="tabpanel" aria-labelledby="example-1-tab" style="padding-top: 24px"}
## Maze Game {#example-1-title}

This example illustrates how to implement the Factory Method pattern
using **static dispatch** (generics).

*Inspired by the Factory Method [example from the GoF
book](https://en.wikipedia.org/wiki/Factory_method_pattern).*

#### []{#example-1--game-rs .anchor} **game.rs**

<figure class="code">
<pre class="code" lang="rust"><code>/// Maze room that is going to be instantiated with a factory method.
pub trait Room {
    fn render(&amp;self);
}

/// Maze game has a factory method producing different rooms.
pub trait MazeGame {
    type RoomImpl: Room;

    /// A factory method.
    fn rooms(&amp;self) -&gt; Vec&lt;Self::RoomImpl&gt;;

    fn play(&amp;self) {
        for room in self.rooms() {
            room.render();
        }
    }
}

/// The client code initializes resources and does other preparations
/// then it uses a factory to construct and run the game.
pub fn run(maze_game: impl MazeGame) {
    println!(&quot;Loading resources...&quot;);
    println!(&quot;Starting the game...&quot;);

    maze_game.play();
}</code></pre>
</figure>

#### []{#example-1--magic_maze-rs .anchor} **magic_maze.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::game::{MazeGame, Room};

#[derive(Clone)]
pub struct MagicRoom {
    title: String,
}

impl MagicRoom {
    pub fn new(title: String) -&gt; Self {
        Self { title }
    }
}

impl Room for MagicRoom {
    fn render(&amp;self) {
        println!(&quot;Magic Room: {}&quot;, self.title);
    }
}

pub struct MagicMaze {
    rooms: Vec&lt;MagicRoom&gt;,
}

impl MagicMaze {
    pub fn new() -&gt; Self {
        Self {
            rooms: vec![
                MagicRoom::new(&quot;Infinite Room&quot;.into()),
                MagicRoom::new(&quot;Red Room&quot;.into()),
            ],
        }
    }
}

impl MazeGame for MagicMaze {
    type RoomImpl = MagicRoom;

    fn rooms(&amp;self) -&gt; Vec&lt;Self::RoomImpl&gt; {
        self.rooms.clone()
    }
}</code></pre>
</figure>

#### []{#example-1--ordinary_maze-rs .anchor} **ordinary_maze.rs**

<figure class="code">
<pre class="code" lang="rust"><code>use super::game::{MazeGame, Room};

#[derive(Clone)]
pub struct OrdinaryRoom {
    id: u32,
}

impl OrdinaryRoom {
    pub fn new(id: u32) -&gt; Self {
        Self { id }
    }
}

impl Room for OrdinaryRoom {
    fn render(&amp;self) {
        println!(&quot;Ordinary Room: #{}&quot;, self.id);
    }
}

pub struct OrdinaryMaze {
    rooms: Vec&lt;OrdinaryRoom&gt;,
}

impl OrdinaryMaze {
    pub fn new() -&gt; Self {
        Self {
            rooms: vec![OrdinaryRoom::new(1), OrdinaryRoom::new(2)],
        }
    }
}

impl MazeGame for OrdinaryMaze {
    type RoomImpl = OrdinaryRoom;

    fn rooms(&amp;self) -&gt; Vec&lt;Self::RoomImpl&gt; {
        let mut rooms = self.rooms.clone();
        rooms.reverse();
        rooms
    }
}</code></pre>
</figure>

#### []{#example-1--main-rs .anchor} **main.rs:** Client code

<figure class="code">
<pre class="code" lang="rust"><code>mod game;
mod magic_maze;
mod ordinary_maze;

use magic_maze::MagicMaze;
use ordinary_maze::OrdinaryMaze;

/// The game runs with different mazes depending on the concrete factory type:
/// it&#39;s either an ordinary maze or a magic maze.
///
/// For demonstration purposes, both mazes are used to construct the game.
fn main() {
    // Option 1: The game starts with an ordinary maze.
    let ordinary_maze = OrdinaryMaze::new();
    game::run(ordinary_maze);

    // Option 2: The game starts with a magic maze.
    let magic_maze = MagicMaze::new();
    game::run(magic_maze);
}</code></pre>
</figure>

### Output

<figure class="code">
<pre class="code"><code>Loading resources...
Starting the game...
Magic Room: Infinite Room
Magic Room: Red Room
Loading resources...
Starting the game...
Ordinary Room: #2
Ordinary Room: #1</code></pre>
</figure>
:::
:::::

::: {#nav-tab .nav .nav-tabs .nav-tabs-bottom .nav-justified role="tablist"}
[Dialog Rendering](#example-0){#example-0-tab-bottom .nav-item .nav-link
.active toggle="tab" role="tab" aria-controls="example-0"
aria-selected="true"} [Maze Game](#example-1){#example-1-tab-bottom
.nav-item .nav-link toggle="tab" role="tab" aria-controls="example-1"
aria-selected="true"}
:::

::: next
#### 다음을 보세요

[러스트로 작성된 프로토타입 []{.fa
.fa-arrow-right}](../../prototype/rust/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} 러스트로 작성된
빌더](../../builder/rust/example.html){.btn .btn-default rel="prev"}
:::

## 다른 언어로 작성된 **팩토리 메서드**

[![C#으로 작성된 팩토리
메서드](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "C#으로 작성된 팩토리 메서드"){.prog-lang-link}
[![C++로 작성된 팩토리
메서드](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "C++로 작성된 팩토리 메서드"){.prog-lang-link}
[![Go로 작성된 팩토리
메서드](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Go로 작성된 팩토리 메서드"){.prog-lang-link}
[![자바로 작성된 팩토리
메서드](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "자바로 작성된 팩토리 메서드"){.prog-lang-link}
[![PHP로 작성된 팩토리
메서드](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "PHP로 작성된 팩토리 메서드"){.prog-lang-link}
[![파이썬으로 작성된 팩토리
메서드](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "파이썬으로 작성된 팩토리 메서드"){.prog-lang-link}
[![루비로 작성된 팩토리
메서드](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "루비로 작성된 팩토리 메서드"){.prog-lang-link}
[![스위프트로 작성된 팩토리
메서드](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "스위프트로 작성된 팩토리 메서드"){.prog-lang-link}
[![타입스크립트로 작성된 팩토리
메서드](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "타입스크립트로 작성된 팩토리 메서드"){.prog-lang-link}
::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ko" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ 예시들을 포함한 저장소](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
전자책 **디자인 패턴에 뛰어들기**를 구매하면 IDE에서 바로 열 수 있는
수십 가지의 상세한 코드 예시들이 포함된 저장소를 사용할 수 있습니다.

[ 더 알아보기...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
