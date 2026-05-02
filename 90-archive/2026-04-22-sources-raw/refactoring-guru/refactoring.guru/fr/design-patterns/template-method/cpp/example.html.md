:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Patron de
méthode](../../template-method.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Patron de
méthode](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Patron de méthode** en C++ {#patron-de-méthode-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
Le **Patron de méthode** est un patron de conception comportemental qui
permet de définir le squelette d'un algorithme dans la classe de base,
et laisse les sous-classes redéfinir les étapes sans modifier la
structure globale de l'algorithme.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Patron de méthode
](../../template-method.html){.btn .btn-lg .btn-primary}
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

**Exemples d'utilisation :** Le patron de méthode est assez répandu en
C++. Les développeurs l'emploient souvent pour fournir un framework qui
permet aux utilisateurs d'étendre des fonctionnalités standards avec
l'héritage.

**Identification :** Le patron de méthode peut être reconnu par ses
méthodes comportementales qui ont déjà un comportement « par défaut »
défini dans la classe de base.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du **Patron de
méthode**. Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-cc .anchor} **main.cc:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Abstract Class defines a template method that contains a skeleton of some
 * algorithm, composed of calls to (usually) abstract primitive operations.
 *
 * Concrete subclasses should implement these operations, but leave the template
 * method itself intact.
 */
class AbstractClass {
  /**
   * The template method defines the skeleton of an algorithm.
   */
 public:
  void TemplateMethod() const {
    this-&gt;BaseOperation1();
    this-&gt;RequiredOperations1();
    this-&gt;BaseOperation2();
    this-&gt;Hook1();
    this-&gt;RequiredOperation2();
    this-&gt;BaseOperation3();
    this-&gt;Hook2();
  }
  /**
   * These operations already have implementations.
   */
 protected:
  void BaseOperation1() const {
    std::cout &lt;&lt; &quot;AbstractClass says: I am doing the bulk of the work\n&quot;;
  }
  void BaseOperation2() const {
    std::cout &lt;&lt; &quot;AbstractClass says: But I let subclasses override some operations\n&quot;;
  }
  void BaseOperation3() const {
    std::cout &lt;&lt; &quot;AbstractClass says: But I am doing the bulk of the work anyway\n&quot;;
  }
  /**
   * These operations have to be implemented in subclasses.
   */
  virtual void RequiredOperations1() const = 0;
  virtual void RequiredOperation2() const = 0;
  /**
   * These are &quot;hooks.&quot; Subclasses may override them, but it&#39;s not mandatory
   * since the hooks already have default (but empty) implementation. Hooks
   * provide additional extension points in some crucial places of the
   * algorithm.
   */
  virtual void Hook1() const {}
  virtual void Hook2() const {}
};
/**
 * Concrete classes have to implement all abstract operations of the base class.
 * They can also override some operations with a default implementation.
 */
class ConcreteClass1 : public AbstractClass {
 protected:
  void RequiredOperations1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass1 says: Implemented Operation1\n&quot;;
  }
  void RequiredOperation2() const override {
    std::cout &lt;&lt; &quot;ConcreteClass1 says: Implemented Operation2\n&quot;;
  }
};
/**
 * Usually, concrete classes override only a fraction of base class&#39; operations.
 */
class ConcreteClass2 : public AbstractClass {
 protected:
  void RequiredOperations1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Implemented Operation1\n&quot;;
  }
  void RequiredOperation2() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Implemented Operation2\n&quot;;
  }
  void Hook1() const override {
    std::cout &lt;&lt; &quot;ConcreteClass2 says: Overridden Hook1\n&quot;;
  }
};
/**
 * The client code calls the template method to execute the algorithm. Client
 * code does not have to know the concrete class of an object it works with, as
 * long as it works with objects through the interface of their base class.
 */
void ClientCode(AbstractClass *class_) {
  // ...
  class_-&gt;TemplateMethod();
  // ...
}

int main() {
  std::cout &lt;&lt; &quot;Same client code can work with different subclasses:\n&quot;;
  ConcreteClass1 *concreteClass1 = new ConcreteClass1;
  ClientCode(concreteClass1);
  std::cout &lt;&lt; &quot;\n&quot;;
  std::cout &lt;&lt; &quot;Same client code can work with different subclasses:\n&quot;;
  ConcreteClass2 *concreteClass2 = new ConcreteClass2;
  ClientCode(concreteClass2);
  delete concreteClass1;
  delete concreteClass2;
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass1 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass1 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway

Same client code can work with different subclasses:
AbstractClass says: I am doing the bulk of the work
ConcreteClass2 says: Implemented Operation1
AbstractClass says: But I let subclasses override some operations
ConcreteClass2 says: Overridden Hook1
ConcreteClass2 says: Implemented Operation2
AbstractClass says: But I am doing the bulk of the work anyway</code></pre>
</figure>

::: next
#### Suivant

[Visiteur en C++ []{.fa
.fa-arrow-right}](../../visitor/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Stratégie en
C++](../../strategy/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Patron de méthode** dans les autres langues

[![Patron de méthode en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Patron de méthode en C#"){.prog-lang-link}
[![Patron de méthode en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Patron de méthode en Go"){.prog-lang-link}
[![Patron de méthode en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Patron de méthode en Java"){.prog-lang-link}
[![Patron de méthode en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Patron de méthode en PHP"){.prog-lang-link}
[![Patron de méthode en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Patron de méthode en Python"){.prog-lang-link}
[![Patron de méthode en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Patron de méthode en Ruby"){.prog-lang-link}
[![Patron de méthode en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Patron de méthode en Rust"){.prog-lang-link}
[![Patron de méthode en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Patron de méthode en Swift"){.prog-lang-link}
[![Patron de méthode en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Patron de méthode en TypeScript"){.prog-lang-link}
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
