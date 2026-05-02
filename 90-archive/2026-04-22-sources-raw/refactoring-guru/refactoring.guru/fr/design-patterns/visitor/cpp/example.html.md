:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Visiteur](../../visitor.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Visiteur](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Visiteur** en C++ {#visiteur-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **Visiteur** est un patron de conception comportemental qui permet
d'ajouter de nouveaux comportements à une hiérarchie de classes sans
modifier l'existant.

> Découvrez pourquoi les visiteurs ne peuvent pas être remplacés par la
> surcharge de méthodes dans notre article [Visiteur et double
> répartition](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Visiteur ](../../visitor.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** Le visiteur n'est pas un patron très
répandu en C++ à cause de sa complexité et de la rareté de ses cas
d'utilisation.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Visiteur**. Nous
allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-cc .anchor} **main.cc:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Visitor Interface declares a set of visiting methods that correspond to
 * component classes. The signature of a visiting method allows the visitor to
 * identify the exact class of the component that it&#39;s dealing with.
 */
class ConcreteComponentA;
class ConcreteComponentB;

class Visitor {
 public:
  virtual void VisitConcreteComponentA(const ConcreteComponentA *element) const = 0;
  virtual void VisitConcreteComponentB(const ConcreteComponentB *element) const = 0;
};

/**
 * The Component interface declares an `accept` method that should take the base
 * visitor interface as an argument.
 */

class Component {
 public:
  virtual ~Component() {}
  virtual void Accept(Visitor *visitor) const = 0;
};

/**
 * Each Concrete Component must implement the `Accept` method in such a way that
 * it calls the visitor&#39;s method corresponding to the component&#39;s class.
 */
class ConcreteComponentA : public Component {
  /**
   * Note that we&#39;re calling `visitConcreteComponentA`, which matches the
   * current class name. This way we let the visitor know the class of the
   * component it works with.
   */
 public:
  void Accept(Visitor *visitor) const override {
    visitor-&gt;VisitConcreteComponentA(this);
  }
  /**
   * Concrete Components may have special methods that don&#39;t exist in their base
   * class or interface. The Visitor is still able to use these methods since
   * it&#39;s aware of the component&#39;s concrete class.
   */
  std::string ExclusiveMethodOfConcreteComponentA() const {
    return &quot;A&quot;;
  }
};

class ConcreteComponentB : public Component {
  /**
   * Same here: visitConcreteComponentB =&gt; ConcreteComponentB
   */
 public:
  void Accept(Visitor *visitor) const override {
    visitor-&gt;VisitConcreteComponentB(this);
  }
  std::string SpecialMethodOfConcreteComponentB() const {
    return &quot;B&quot;;
  }
};

/**
 * Concrete Visitors implement several versions of the same algorithm, which can
 * work with all concrete component classes.
 *
 * You can experience the biggest benefit of the Visitor pattern when using it
 * with a complex object structure, such as a Composite tree. In this case, it
 * might be helpful to store some intermediate state of the algorithm while
 * executing visitor&#39;s methods over various objects of the structure.
 */
class ConcreteVisitor1 : public Visitor {
 public:
  void VisitConcreteComponentA(const ConcreteComponentA *element) const override {
    std::cout &lt;&lt; element-&gt;ExclusiveMethodOfConcreteComponentA() &lt;&lt; &quot; + ConcreteVisitor1\n&quot;;
  }

  void VisitConcreteComponentB(const ConcreteComponentB *element) const override {
    std::cout &lt;&lt; element-&gt;SpecialMethodOfConcreteComponentB() &lt;&lt; &quot; + ConcreteVisitor1\n&quot;;
  }
};

class ConcreteVisitor2 : public Visitor {
 public:
  void VisitConcreteComponentA(const ConcreteComponentA *element) const override {
    std::cout &lt;&lt; element-&gt;ExclusiveMethodOfConcreteComponentA() &lt;&lt; &quot; + ConcreteVisitor2\n&quot;;
  }
  void VisitConcreteComponentB(const ConcreteComponentB *element) const override {
    std::cout &lt;&lt; element-&gt;SpecialMethodOfConcreteComponentB() &lt;&lt; &quot; + ConcreteVisitor2\n&quot;;
  }
};
/**
 * The client code can run visitor operations over any set of elements without
 * figuring out their concrete classes. The accept operation directs a call to
 * the appropriate operation in the visitor object.
 */
void ClientCode(std::array&lt;const Component *, 2&gt; components, Visitor *visitor) {
  // ...
  for (const Component *comp : components) {
    comp-&gt;Accept(visitor);
  }
  // ...
}

int main() {
  std::array&lt;const Component *, 2&gt; components = {new ConcreteComponentA, new ConcreteComponentB};
  std::cout &lt;&lt; &quot;The client code works with all visitors via the base Visitor interface:\n&quot;;
  ConcreteVisitor1 *visitor1 = new ConcreteVisitor1;
  ClientCode(components, visitor1);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;It allows the same client code to work with different types of visitors:\n&quot;;
  ConcreteVisitor2 *visitor2 = new ConcreteVisitor2;
  ClientCode(components, visitor2);

  for (const Component *comp : components) {
    delete comp;
  }
  delete visitor1;
  delete visitor2;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1

It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>

::: next
#### Suivant

[Les patrons de conception en Go []{.fa
.fa-arrow-right}](../../go.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Patron de méthode en
C++](../../template-method/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## **Visiteur** dans les autres langues

[![Visiteur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Visiteur en C#"){.prog-lang-link}
[![Visiteur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Visiteur en Go"){.prog-lang-link}
[![Visiteur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Visiteur en Java"){.prog-lang-link}
[![Visiteur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Visiteur en PHP"){.prog-lang-link}
[![Visiteur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Visiteur en Python"){.prog-lang-link}
[![Visiteur en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Visiteur en Ruby"){.prog-lang-link}
[![Visiteur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Visiteur en Rust"){.prog-lang-link}
[![Visiteur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Visiteur en Swift"){.prog-lang-link}
[![Visiteur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Visiteur en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
