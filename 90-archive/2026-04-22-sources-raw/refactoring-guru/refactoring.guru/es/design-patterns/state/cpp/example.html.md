:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[State](../../state.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![State](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **State** en C++ {#state-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
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

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón State se utiliza habitualmente en C++
para convertir las enormes máquinas de estados basadas en `switch`, en
objetos.

**Identificación:** El patrón State se puede reconocer por métodos que
cambian su comportamiento dependiendo del estado del objeto, controlado
externamente.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **State**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-cc .anchor} **main.cc:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="cpp"><code>#include &lt;iostream&gt;
#include &lt;typeinfo&gt;
/**
 * The base State class declares methods that all Concrete State should
 * implement and also provides a backreference to the Context object, associated
 * with the State. This backreference can be used by States to transition the
 * Context to another State.
 */

class Context;

class State {
  /**
   * @var Context
   */
 protected:
  Context *context_;

 public:
  virtual ~State() {
  }

  void set_context(Context *context) {
    this-&gt;context_ = context;
  }

  virtual void Handle1() = 0;
  virtual void Handle2() = 0;
};

/**
 * The Context defines the interface of interest to clients. It also maintains a
 * reference to an instance of a State subclass, which represents the current
 * state of the Context.
 */
class Context {
  /**
   * @var State A reference to the current state of the Context.
   */
 private:
  State *state_;

 public:
  Context(State *state) : state_(nullptr) {
    this-&gt;TransitionTo(state);
  }
  ~Context() {
    delete state_;
  }
  /**
   * The Context allows changing the State object at runtime.
   */
  void TransitionTo(State *state) {
    std::cout &lt;&lt; &quot;Context: Transition to &quot; &lt;&lt; typeid(*state).name() &lt;&lt; &quot;.\n&quot;;
    if (this-&gt;state_ != nullptr)
      delete this-&gt;state_;
    this-&gt;state_ = state;
    this-&gt;state_-&gt;set_context(this);
  }
  /**
   * The Context delegates part of its behavior to the current State object.
   */
  void Request1() {
    this-&gt;state_-&gt;Handle1();
  }
  void Request2() {
    this-&gt;state_-&gt;Handle2();
  }
};

/**
 * Concrete States implement various behaviors, associated with a state of the
 * Context.
 */

class ConcreteStateA : public State {
 public:
  void Handle1() override;

  void Handle2() override {
    std::cout &lt;&lt; &quot;ConcreteStateA handles request2.\n&quot;;
  }
};

class ConcreteStateB : public State {
 public:
  void Handle1() override {
    std::cout &lt;&lt; &quot;ConcreteStateB handles request1.\n&quot;;
  }
  void Handle2() override {
    std::cout &lt;&lt; &quot;ConcreteStateB handles request2.\n&quot;;
    std::cout &lt;&lt; &quot;ConcreteStateB wants to change the state of the context.\n&quot;;
    this-&gt;context_-&gt;TransitionTo(new ConcreteStateA);
  }
};

void ConcreteStateA::Handle1() {
  {
    std::cout &lt;&lt; &quot;ConcreteStateA handles request1.\n&quot;;
    std::cout &lt;&lt; &quot;ConcreteStateA wants to change the state of the context.\n&quot;;

    this-&gt;context_-&gt;TransitionTo(new ConcreteStateB);
  }
}

/**
 * The client code.
 */
void ClientCode() {
  Context *context = new Context(new ConcreteStateA);
  context-&gt;Request1();
  context-&gt;Request2();
  delete context;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to 14ConcreteStateA.
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to 14ConcreteStateB.
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to 14ConcreteStateA.
  </code></pre>
</figure>

::: next
#### Leer siguiente

[Strategy en C++ []{.fa
.fa-arrow-right}](../../strategy/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Observer en
C++](../../observer/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **State** en otros lenguajes

[![State en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "State en C#"){.prog-lang-link}
[![State en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "State en Go"){.prog-lang-link}
[![State en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "State en Java"){.prog-lang-link}
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
:::::::::::::::::

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
:::::::::::::::::::::::
::::::::::::::::::::::::
