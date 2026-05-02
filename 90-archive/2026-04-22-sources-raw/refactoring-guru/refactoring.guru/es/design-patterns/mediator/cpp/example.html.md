:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Mediator](../../mediator.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Mediator](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Mediator** en C++ {#mediator-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Mediator** es un patrón de diseño de comportamiento que reduce el
acoplamiento entre los componentes de un programa haciendo que se
comuniquen indirectamente a través de un objeto mediador especial.

El patrón Mediator facilita la modificación, extensión y reutilización
de componentes individuales porque ya no son dependientes de todas las
demás clases.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Mediator ](../../mediator.html){.btn
.btn-lg .btn-primary}
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

**Ejemplos de uso:** El uso más popular del patrón Mediator en código
C++ es para facilitar las comunicaciones entre los componentes GUI de
una aplicación. El sinónimo del patrón Mediator es la parte del
Controlador del patrón MVC.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Mediator**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-cc .anchor} **main.cc:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="cpp"><code>#include &lt;iostream&gt;
#include &lt;string&gt;
/**
 * The Mediator interface declares a method used by components to notify the
 * mediator about various events. The Mediator may react to these events and
 * pass the execution to other components.
 */
class BaseComponent;
class Mediator {
 public:
  virtual void Notify(BaseComponent *sender, std::string event) const = 0;
};

/**
 * The Base Component provides the basic functionality of storing a mediator&#39;s
 * instance inside component objects.
 */
class BaseComponent {
 protected:
  Mediator *mediator_;

 public:
  BaseComponent(Mediator *mediator = nullptr) : mediator_(mediator) {
  }
  void set_mediator(Mediator *mediator) {
    this-&gt;mediator_ = mediator;
  }
};

/**
 * Concrete Components implement various functionality. They don&#39;t depend on
 * other components. They also don&#39;t depend on any concrete mediator classes.
 */
class Component1 : public BaseComponent {
 public:
  void DoA() {
    std::cout &lt;&lt; &quot;Component 1 does A.\n&quot;;
    this-&gt;mediator_-&gt;Notify(this, &quot;A&quot;);
  }
  void DoB() {
    std::cout &lt;&lt; &quot;Component 1 does B.\n&quot;;
    this-&gt;mediator_-&gt;Notify(this, &quot;B&quot;);
  }
};

class Component2 : public BaseComponent {
 public:
  void DoC() {
    std::cout &lt;&lt; &quot;Component 2 does C.\n&quot;;
    this-&gt;mediator_-&gt;Notify(this, &quot;C&quot;);
  }
  void DoD() {
    std::cout &lt;&lt; &quot;Component 2 does D.\n&quot;;
    this-&gt;mediator_-&gt;Notify(this, &quot;D&quot;);
  }
};

/**
 * Concrete Mediators implement cooperative behavior by coordinating several
 * components.
 */
class ConcreteMediator : public Mediator {
 private:
  Component1 *component1_;
  Component2 *component2_;

 public:
  ConcreteMediator(Component1 *c1, Component2 *c2) : component1_(c1), component2_(c2) {
    this-&gt;component1_-&gt;set_mediator(this);
    this-&gt;component2_-&gt;set_mediator(this);
  }
  void Notify(BaseComponent *sender, std::string event) const override {
    if (event == &quot;A&quot;) {
      std::cout &lt;&lt; &quot;Mediator reacts on A and triggers following operations:\n&quot;;
      this-&gt;component2_-&gt;DoC();
    }
    if (event == &quot;D&quot;) {
      std::cout &lt;&lt; &quot;Mediator reacts on D and triggers following operations:\n&quot;;
      this-&gt;component1_-&gt;DoB();
      this-&gt;component2_-&gt;DoC();
    }
  }
};

/**
 * The client code.
 */

void ClientCode() {
  Component1 *c1 = new Component1;
  Component2 *c2 = new Component2;
  ConcreteMediator *mediator = new ConcreteMediator(c1, c2);
  std::cout &lt;&lt; &quot;Client triggers operation A.\n&quot;;
  c1-&gt;DoA();
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;Client triggers operation D.\n&quot;;
  c2-&gt;DoD();

  delete c1;
  delete c2;
  delete mediator;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

::: next
#### Leer siguiente

[Memento en C++ []{.fa
.fa-arrow-right}](../../memento/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Iterator en
C++](../../iterator/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Mediator** en otros lenguajes

[![Mediator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Mediator en C#"){.prog-lang-link}
[![Mediator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Mediator en Go"){.prog-lang-link}
[![Mediator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Mediator en Java"){.prog-lang-link}
[![Mediator en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Mediator en PHP"){.prog-lang-link}
[![Mediator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Mediator en Python"){.prog-lang-link}
[![Mediator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Mediator en Ruby"){.prog-lang-link}
[![Mediator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Mediator en Rust"){.prog-lang-link}
[![Mediator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Mediator en Swift"){.prog-lang-link}
[![Mediator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Mediator en TypeScript"){.prog-lang-link}
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
