:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Фабричний
метод](../../factory-method.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фабричний
метод](../../../../images/patterns/cards/factory-method-minid7f6.png?id=72619e9527893374b98a5913779ac167){srcset="/images/patterns/cards/factory-method-mini-2x.png?id=fa9d4a8d61a67cc3822e52b9daf69dad 2x"}
:::

# **Фабричний метод** на C++ {#фабричний-метод-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Фабричний метод** --- це породжуючий патерн проектування, який вирішує
проблему створення різних продуктів, без прив'язки коду до конкретних
класів продуктів.

Фабричний метод задає метод, який необхідно використовувати замість
виклику оператора `new` для створення об'єктів-продуктів. Підкласи
можуть перевизначити цей метод, щоб змінювати тип створюваних продуктів.

> Якщо ви вже чули про *Фабрику*, *Фабричний метод* чи *Абстрактну
> фабрику*, але вам все одно важко їх розрізняти, то прочитайте нашу
> статтю [Порівняння фабрик](../../factory-comparison.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Фабричний метод ](../../factory-method.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн можна часто зустріти в будь-якому C++ коді,
коли потрібна гнучкість при створенні продуктів.

**Ознаки застосування патерна:** Фабричний метод можна визначити за
створюючими методами, які повертають об'єкти продуктів через абстрактні
типи або інтерфейси. Це дозволяє змінювати типи створюваних продуктів у
підкласах.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Фабричний метод**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-cc .anchor} **main.cc:** Приклад структури патерна

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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>App: Launched with the ConcreteCreator1.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct1}

App: Launched with the ConcreteCreator2.
Client: I&#39;m not aware of the creator&#39;s class, but it still works.
Creator: The same creator&#39;s code has just worked with {Result of the ConcreteProduct2}</code></pre>
</figure>

::: next
#### Читаємо далі

[Прототип на C++ []{.fa
.fa-arrow-right}](../../prototype/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Будівельник на
C++](../../builder/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Фабричний метод** іншими мовами програмування

[![Фабричний метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фабричний метод на C#"){.prog-lang-link}
[![Фабричний метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фабричний метод на Go"){.prog-lang-link}
[![Фабричний метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Фабричний метод на Java"){.prog-lang-link}
[![Фабричний метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фабричний метод на PHP"){.prog-lang-link}
[![Фабричний метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фабричний метод на Python"){.prog-lang-link}
[![Фабричний метод на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Фабричний метод на Ruby"){.prog-lang-link}
[![Фабричний метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фабричний метод на Rust"){.prog-lang-link}
[![Фабричний метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фабричний метод на Swift"){.prog-lang-link}
[![Фабричний метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фабричний метод на TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
