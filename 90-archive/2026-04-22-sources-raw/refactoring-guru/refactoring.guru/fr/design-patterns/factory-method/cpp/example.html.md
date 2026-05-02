:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Fabrique](../../factory-method.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Fabrique](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Fabrique** en C++ {#fabrique-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
La **Fabrique** est un patron de conception de création qui permet de
créer des produits sans avoir à préciser leurs classes concrètes.

La fabrique définit une méthode qui doit être utilisée pour créer des
objets à la place de l'appel au constructeur (opérateur `new`). Les
sous-classes peuvent redéfinir cette méthode pour modifier la classe des
objets qui seront créés.

> Lisez notre [Comparaison des fabriques](../../factory-comparison.html)
> si vous avez du mal à saisir la différence entre les divers concepts
> et patrons.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Fabrique
](../../factory-method.html){.btn .btn-lg .btn-primary}
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

**Exemples d'utilisation :** La fabrique est très largement utilisée en
C++. Elle est très utile lorsque vous avez besoin de flexibilité dans
votre code.

**Identification :** La fabrique peut être reconnue grâce à ses méthodes
de création qui produisent des objets depuis les classes concrètes, mais
les retournent en tant qu'objets d'interface ou de type abstrait.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure du patron de conception
**Fabrique**. Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-cc .anchor} **main.cc:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * The Product interface declares the operations that all concrete products must
 * implement.
 */

class Product {
 public:
  virtual ~Product() {}
  virtual std::string Operation() const = 0;
};

/**
 * Concrete Products provide various implementations of the Product interface.
 */
class ConcreteProduct1 : public Product {
 public:
  std::string Operation() const override {
    return &quot;{Result of the ConcreteProduct1}&quot;;
  }
};
class ConcreteProduct2 : public Product {
 public:
  std::string Operation() const override {
    return &quot;{Result of the ConcreteProduct2}&quot;;
  }
};

/**
 * The Creator class declares the factory method that is supposed to return an
 * object of a Product class. The Creator&#39;s subclasses usually provide the
 * implementation of this method.
 */

class Creator {
  /**
   * Note that the Creator may also provide some default implementation of the
   * factory method.
   */
 public:
  virtual ~Creator(){};
  virtual Product* FactoryMethod() const = 0;
  /**
   * Also note that, despite its name, the Creator&#39;s primary responsibility is
   * not creating products. Usually, it contains some core business logic that
   * relies on Product objects, returned by the factory method. Subclasses can
   * indirectly change that business logic by overriding the factory method and
   * returning a different type of product from it.
   */

  std::string SomeOperation() const {
    // Call the factory method to create a Product object.
    Product* product = this-&gt;FactoryMethod();
    // Now, use the product.
    std::string result = &quot;Creator: The same creator&#39;s code has just worked with &quot; + product-&gt;Operation();
    delete product;
    return result;
  }
};

/**
 * Concrete Creators override the factory method in order to change the
 * resulting product&#39;s type.
 */
class ConcreteCreator1 : public Creator {
  /**
   * Note that the signature of the method still uses the abstract product type,
   * even though the concrete product is actually returned from the method. This
   * way the Creator can stay independent of concrete product classes.
   */
 public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct1();
  }
};

class ConcreteCreator2 : public Creator {
 public:
  Product* FactoryMethod() const override {
    return new ConcreteProduct2();
  }
};

/**
 * The client code works with an instance of a concrete creator, albeit through
 * its base interface. As long as the client keeps working with the creator via
 * the base interface, you can pass it any creator&#39;s subclass.
 */
void ClientCode(const Creator&amp; creator) {
  // ...
  std::cout &lt;&lt; &quot;Client: I&#39;m not aware of the creator&#39;s class, but it still works.\n&quot;
            &lt;&lt; creator.SomeOperation() &lt;&lt; std::endl;
  // ...
}

/**
 * The Application picks a creator&#39;s type depending on the configuration or
 * environment.
 */

int main() {
  std::cout &lt;&lt; &quot;App: Launched with the ConcreteCreator1.\n&quot;;
  Creator* creator = new ConcreteCreator1();
  ClientCode(*creator);
  std::cout &lt;&lt; std::endl;
  std::cout &lt;&lt; &quot;App: Launched with the ConcreteCreator2.\n&quot;;
  Creator* creator2 = new ConcreteCreator2();
  ClientCode(*creator2);

  delete creator;
  delete creator2;
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

::: next
#### Suivant

[Prototype en C++ []{.fa
.fa-arrow-right}](../../prototype/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Monteur en
C++](../../builder/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Fabrique** dans les autres langues

[![Fabrique en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Fabrique en C#"){.prog-lang-link}
[![Fabrique en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Fabrique en Go"){.prog-lang-link}
[![Fabrique en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Fabrique en Java"){.prog-lang-link}
[![Fabrique en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Fabrique en PHP"){.prog-lang-link}
[![Fabrique en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Fabrique en Python"){.prog-lang-link}
[![Fabrique en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Fabrique en Ruby"){.prog-lang-link}
[![Fabrique en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Fabrique en Rust"){.prog-lang-link}
[![Fabrique en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Fabrique en Swift"){.prog-lang-link}
[![Fabrique en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Fabrique en TypeScript"){.prog-lang-link}
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
