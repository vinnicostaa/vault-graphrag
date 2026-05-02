:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../index.html){.home} / [Design
Patterns](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** in Java {#state-in-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
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

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Interface of a media player](#example-0)
:::

::: en-file
 states
:::

:::: en-inside
::: en-file
  [State](#example-0--states-State-java)
:::
::::

:::: en-inside
::: en-file
  [Locked­State](#example-0--states-LockedState-java)
:::
::::

:::: en-inside
::: en-file
  [Ready­State](#example-0--states-ReadyState-java)
:::
::::

:::: en-inside
::: en-file
  [Playing­State](#example-0--states-PlayingState-java)
:::
::::

::: en-file
 ui
:::

:::: en-inside
::: en-file
  [Player](#example-0--ui-Player-java)
:::
::::

:::: en-inside
::: en-file
  [UI](#example-0--ui-UI-java)
:::
::::

::: en-file
 [Demo](#example-0--Demo-java)
:::

::: en-file
 [Output­Demo](#example-0--OutputDemo-png)
:::
::::::::::::::::::::::

**Complexity:** []{.fa-stars}

**Popularity:** []{.fa-stars}

**Usage examples:** The State pattern is commonly used in Java to
convert massive `switch`-base state machines into objects.

Here are some examples of the State pattern in core Java libraries:

-   [`javax.faces.lifecycle.LifeCycle#execute()`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html#execute-javax.faces.context.FacesContext-)
    (controlled by the
    [`FacesServlet`](http://docs.oracle.com/javaee/7/api/javax/faces/webapp/FacesServlet.html):
    behavior is dependent on current phase (state) of JSF lifecycle)

**Identification:** The State pattern can be recognized by methods that
change their behavior depending on the objects' state. You can confirm
identification if you see that this state can be controlled or replaced
by other objects, including state objects themselves.
:::::::::::::::::::::::::

[]{#example-0}

## Interface of a media player {#example-0-title}

In this example, the State pattern lets the same media player controls
behave differently, depending on the current playback state. The main
class of the player contains a reference to a state object, which
performs most of the work for the player. Some actions may end-up
replacing the state object with another, which changes the way the
player reacts to the user interactions.

### []{#example-0--states .anchor} **states** {#states .example-title}

#### []{#example-0--states-State-java .anchor} **states/State.java:** Common state interface

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.states;

import refactoring_guru.state.example.ui.Player;

/**
 * Common interface for all states.
 */
public abstract class State {
    Player player;

    /**
     * Context passes itself through the state constructor. This may help a
     * state to fetch some useful context data if needed.
     */
    State(Player player) {
        this.player = player;
    }

    public abstract String onLock();
    public abstract String onPlay();
    public abstract String onNext();
    public abstract String onPrevious();
}</code></pre>
</figure>

#### []{#example-0--states-LockedState-java .anchor} **states/LockedState.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.states;

import refactoring_guru.state.example.ui.Player;

/**
 * Concrete states provide the special implementation for all interface methods.
 */
public class LockedState extends State {

    LockedState(Player player) {
        super(player);
        player.setPlaying(false);
    }

    @Override
    public String onLock() {
        if (player.isPlaying()) {
            player.changeState(new ReadyState(player));
            return &quot;Stop playing&quot;;
        } else {
            return &quot;Locked...&quot;;
        }
    }

    @Override
    public String onPlay() {
        player.changeState(new ReadyState(player));
        return &quot;Ready&quot;;
    }

    @Override
    public String onNext() {
        return &quot;Locked...&quot;;
    }

    @Override
    public String onPrevious() {
        return &quot;Locked...&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--states-ReadyState-java .anchor} **states/ReadyState.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.states;

import refactoring_guru.state.example.ui.Player;

/**
 * They can also trigger state transitions in the context.
 */
public class ReadyState extends State {

    public ReadyState(Player player) {
        super(player);
    }

    @Override
    public String onLock() {
        player.changeState(new LockedState(player));
        return &quot;Locked...&quot;;
    }

    @Override
    public String onPlay() {
        String action = player.startPlayback();
        player.changeState(new PlayingState(player));
        return action;
    }

    @Override
    public String onNext() {
        return &quot;Locked...&quot;;
    }

    @Override
    public String onPrevious() {
        return &quot;Locked...&quot;;
    }
}</code></pre>
</figure>

#### []{#example-0--states-PlayingState-java .anchor} **states/PlayingState.java**

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.states;

import refactoring_guru.state.example.ui.Player;

public class PlayingState extends State {

    PlayingState(Player player) {
        super(player);
    }

    @Override
    public String onLock() {
        player.changeState(new LockedState(player));
        player.setCurrentTrackAfterStop();
        return &quot;Stop playing&quot;;
    }

    @Override
    public String onPlay() {
        player.changeState(new ReadyState(player));
        return &quot;Paused...&quot;;
    }

    @Override
    public String onNext() {
        return player.nextTrack();
    }

    @Override
    public String onPrevious() {
        return player.previousTrack();
    }
}</code></pre>
</figure>

### []{#example-0--ui .anchor} **ui** {#ui .example-title}

#### []{#example-0--ui-Player-java .anchor} **ui/Player.java:** Player primary code

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.ui;

import refactoring_guru.state.example.states.ReadyState;
import refactoring_guru.state.example.states.State;

import java.util.ArrayList;
import java.util.List;

public class Player {
    private State state;
    private boolean playing = false;
    private List&lt;String&gt; playlist = new ArrayList&lt;&gt;();
    private int currentTrack = 0;

    public Player() {
        this.state = new ReadyState(this);
        setPlaying(true);
        for (int i = 1; i &lt;= 12; i++) {
            playlist.add(&quot;Track &quot; + i);
        }
    }

    public void changeState(State state) {
        this.state = state;
    }

    public State getState() {
        return state;
    }

    public void setPlaying(boolean playing) {
        this.playing = playing;
    }

    public boolean isPlaying() {
        return playing;
    }

    public String startPlayback() {
        return &quot;Playing &quot; + playlist.get(currentTrack);
    }

    public String nextTrack() {
        currentTrack++;
        if (currentTrack &gt; playlist.size() - 1) {
            currentTrack = 0;
        }
        return &quot;Playing &quot; + playlist.get(currentTrack);
    }

    public String previousTrack() {
        currentTrack--;
        if (currentTrack &lt; 0) {
            currentTrack = playlist.size() - 1;
        }
        return &quot;Playing &quot; + playlist.get(currentTrack);
    }

    public void setCurrentTrackAfterStop() {
        this.currentTrack = 0;
    }
}</code></pre>
</figure>

#### []{#example-0--ui-UI-java .anchor} **ui/UI.java:** Player's GUI

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example.ui;

import javax.swing.*;
import java.awt.*;

public class UI {
    private Player player;
    private static JTextField textField = new JTextField();

    public UI(Player player) {
        this.player = player;
    }

    public void init() {
        JFrame frame = new JFrame(&quot;Test player&quot;);
        frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        JPanel context = new JPanel();
        context.setLayout(new BoxLayout(context, BoxLayout.Y_AXIS));
        frame.getContentPane().add(context);
        JPanel buttons = new JPanel(new FlowLayout(FlowLayout.CENTER));
        context.add(textField);
        context.add(buttons);

        // Context delegates handling user&#39;s input to a state object. Naturally,
        // the outcome will depend on what state is currently active, since all
        // states can handle the input differently.
        JButton play = new JButton(&quot;Play&quot;);
        play.addActionListener(e -&gt; textField.setText(player.getState().onPlay()));
        JButton stop = new JButton(&quot;Stop&quot;);
        stop.addActionListener(e -&gt; textField.setText(player.getState().onLock()));
        JButton next = new JButton(&quot;Next&quot;);
        next.addActionListener(e -&gt; textField.setText(player.getState().onNext()));
        JButton prev = new JButton(&quot;Prev&quot;);
        prev.addActionListener(e -&gt; textField.setText(player.getState().onPrevious()));
        frame.setVisible(true);
        frame.setSize(300, 100);
        buttons.add(play);
        buttons.add(stop);
        buttons.add(next);
        buttons.add(prev);
    }
}</code></pre>
</figure>

#### []{#example-0--Demo-java .anchor} **Demo.java:** Initialization code

<figure class="code">
<pre class="code" lang="java"><code>package refactoring_guru.state.example;

import refactoring_guru.state.example.ui.Player;
import refactoring_guru.state.example.ui.UI;

/**
 * Demo class. Everything comes together here.
 */
public class Demo {
    public static void main(String[] args) {
        Player player = new Player();
        UI ui = new UI(player);
        ui.init();
    }
}</code></pre>
</figure>

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Screenshot

![](../../../images/patterns/examples/java/state/OutputDemo11c7.png?id=3fb480159b14df6213408fa1ddaa68be)

::: next
#### Read next

[Strategy in Java []{.fa
.fa-arrow-right}](../../strategy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Observer in
Java](../../observer/java/example.html){.btn .btn-default rel="prev"}
:::

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
PHP](../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State in PHP"){.prog-lang-link}
[![State in
Python](../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State in Python"){.prog-lang-link}
[![State in
Ruby](../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State in Ruby"){.prog-lang-link}
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
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
