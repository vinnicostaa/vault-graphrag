:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [Java](../../java.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** en Java {#state-en-java .pattern-example-title-block-title}
::::

::::::::::::::::::::::::: pattern-example-body
::: pattern-example-brief
**State** es un patrón de diseño de comportamiento que permite a un
objeto cambiar de comportamiento cuando cambia su estado interno.

El patrón extrae comportamientos relacionados con el estado, los coloca
dentro de clases de estado separadas y fuerza al objeto original a
delegar el trabajo de una instancia de esas clases, en lugar de actuar
por su cuenta.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón State ](../../state.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Interfaz de un reproductor multimedia](#example-0)
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

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón State se utiliza habitualmente en Java
para convertir las enormes máquinas de estados basadas en declaraciones
`switch`, en objetos.

Aquí tienes algunos ejemplos del patrón State en las principales
bibliotecas Java:

-   [`javax.faces.lifecycle.LifeCycle#execute()`](http://docs.oracle.com/javaee/7/api/javax/faces/lifecycle/Lifecycle.html#execute-javax.faces.context.FacesContext-)
    (controlado por
    [`FacesServlet`](http://docs.oracle.com/javaee/7/api/javax/faces/webapp/FacesServlet.html):
    el comportamiento depende de la fase actual (estado) del ciclo de
    vida de las JSF)

**Identificación:** El patrón State se puede reconocer por métodos que
cambian su comportamiento dependiendo del estado del objeto, controlado
externamente.
:::::::::::::::::::::::::

[]{#example-0}

## Interfaz de un reproductor multimedia {#example-0-title}

En este ejemplo, el patrón State permite a los mismos controles del
reproductor multimedia comportarse de forma diferente, dependiendo del
estado actual de reproducción. La clase principal del reproductor
contiene una referencia a un objeto de estado que realiza la mayor parte
del trabajo para el reproductor. Algunas acciones pueden acabar
sustituyendo el objeto de estado por otro, lo cual cambia la forma en la
que el reproductor reacciona a las interacciones del usuario.

### []{#example-0--states .anchor} **states** {#states .example-title}

#### []{#example-0--states-State-java .anchor} **states/State.java:** Interfaz común de estado

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

#### []{#example-0--ui-Player-java .anchor} **ui/Player.java:** Código principal del reproductor

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

#### []{#example-0--ui-UI-java .anchor} **ui/UI.java:** GUI del reproductor

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

#### []{#example-0--Demo-java .anchor} **Demo.java:** Código de inicialización

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

#### []{#example-0--OutputDemo-png .anchor} **OutputDemo.png:** Captura de pantalla

![](../../../../images/patterns/examples/java/state/OutputDemo11c7.png?id=3fb480159b14df6213408fa1ddaa68be)

::: next
#### Leer siguiente

[Strategy en Java []{.fa
.fa-arrow-right}](../../strategy/java/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Observer en
Java](../../observer/java/example.html){.btn .btn-default rel="prev"}
:::

## **State** en otros lenguajes

[![State en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State en C#"){.prog-lang-link}
[![State en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "State en C++"){.prog-lang-link}
[![State en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "State en Go"){.prog-lang-link}
[![State en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "State en PHP"){.prog-lang-link}
[![State en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "State en Python"){.prog-lang-link}
[![State en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "State en Ruby"){.prog-lang-link}
[![State en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "State en Rust"){.prog-lang-link}
[![State en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "State en Swift"){.prog-lang-link}
[![State en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "State en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
