:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Компоновщик](../../composite.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компоновщик](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компоновщик** на C++ {#компоновщик-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Компоновщик** --- это структурный паттерн, который позволяет создавать
дерево объектов и работать с ним так же, как и с единичным объектом.

Компоновщик давно стал синонимом всех задач, связанных с построением
дерева объектов. Все операции компоновщика основаны на рекурсии и
«суммировании» результатов на ветвях дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Компоновщик ](../../composite.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн Компоновщик встречается в любых задачах,
которые связаны с построением дерева. Самый простой пример --- составные
элементы GUI, которые тоже можно рассматривать как дерево.

**Признаки применения паттерна:** Если из объектов строится древовидная
структура, и со всеми объектами дерева, как и с самим деревом работают
через общий интерфейс.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Компоновщик**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-cc .anchor} **main.cc:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="cpp"><code>#include &lt;algorithm&gt;
#include &lt;iostream&gt;
#include &lt;list&gt;
#include &lt;string&gt;
/**
 * Базовый класс Компонент объявляет общие операции как для простых, так и для
 * сложных объектов структуры.
 */
class Component {
  /**
   * @var Component
   */
 protected:
  Component *parent_;
  /**
   * При необходимости базовый Компонент может объявить интерфейс для установки
   * и получения родителя компонента в древовидной структуре. Он также может
   * предоставить некоторую реализацию по умолчанию для этих методов.
   */
 public:
  virtual ~Component() {}
  void SetParent(Component *parent) {
    this-&gt;parent_ = parent;
  }
  Component *GetParent() const {
    return this-&gt;parent_;
  }
  /**
   * В некоторых случаях целесообразно определить операции управления потомками
   * прямо в базовом классе Компонент. Таким образом, вам не нужно будет
   * предоставлять конкретные классы компонентов клиентскому коду, даже во время
   * сборки дерева объектов. Недостаток такого подхода в том, что эти методы
   * будут пустыми для компонентов уровня листа.
   */
  virtual void Add(Component *component) {}
  virtual void Remove(Component *component) {}
  /**
   * Вы можете предоставить метод, который позволит клиентскому коду понять,
   * может ли компонент иметь вложенные объекты.
   */
  virtual bool IsComposite() const {
    return false;
  }
  /**
   * Базовый Компонент может сам реализовать некоторое поведение по умолчанию
   * или поручить это конкретным классам, объявив метод, содержащий поведение
   * абстрактным.
   */
  virtual std::string Operation() const = 0;
};
/**
 * Класс Лист представляет собой конечные объекты структуры. Лист не может иметь
 * вложенных компонентов.
 *
 * Обычно объекты Листьев выполняют фактическую работу, тогда как объекты
 * Контейнера лишь делегируют работу своим подкомпонентам.
 */
class Leaf : public Component {
 public:
  std::string Operation() const override {
    return &quot;Leaf&quot;;
  }
};
/**
 * Класс Контейнер содержит сложные компоненты, которые могут иметь вложенные
 * компоненты. Обычно объекты Контейнеры делегируют фактическую работу своим
 * детям, а затем «суммируют» результат.
 */
class Composite : public Component {
  /**
   * @var \SplObjectStorage
   */
 protected:
  std::list&lt;Component *&gt; children_;

 public:
  /**
   * Объект контейнера может как добавлять компоненты в свой список вложенных
   * компонентов, так и удалять их, как простые, так и сложные.
   */
  void Add(Component *component) override {
    this-&gt;children_.push_back(component);
    component-&gt;SetParent(this);
  }
  /**
   * Have in mind that this method removes the pointer to the list but doesn&#39;t
   * frees the
   *     memory, you should do it manually or better use smart pointers.
   */
  void Remove(Component *component) override {
    children_.remove(component);
    component-&gt;SetParent(nullptr);
  }
  bool IsComposite() const override {
    return true;
  }
  /**
   * Контейнер выполняет свою основную логику особым образом. Он проходит
   * рекурсивно через всех своих детей, собирая и суммируя их результаты.
   * Поскольку потомки контейнера передают эти вызовы своим потомкам и так
   * далее, в результате обходится всё дерево объектов.
   */
  std::string Operation() const override {
    std::string result;
    for (const Component *c : children_) {
      if (c == children_.back()) {
        result += c-&gt;Operation();
      } else {
        result += c-&gt;Operation() + &quot;+&quot;;
      }
    }
    return &quot;Branch(&quot; + result + &quot;)&quot;;
  }
};
/**
 * Клиентский код работает со всеми компонентами через базовый интерфейс.
 */
void ClientCode(Component *component) {
  // ...
  std::cout &lt;&lt; &quot;RESULT: &quot; &lt;&lt; component-&gt;Operation();
  // ...
}

/**
 * Благодаря тому, что операции управления потомками объявлены в базовом классе
 * Компонента, клиентский код может работать как с простыми, так и со сложными
 * компонентами, вне зависимости от их конкретных классов.
 */
void ClientCode2(Component *component1, Component *component2) {
  // ...
  if (component1-&gt;IsComposite()) {
    component1-&gt;Add(component2);
  }
  std::cout &lt;&lt; &quot;RESULT: &quot; &lt;&lt; component1-&gt;Operation();
  // ...
}

/**
 * Таким образом, клиентский код может поддерживать простые компоненты-листья...
 */

int main() {
  Component *simple = new Leaf;
  std::cout &lt;&lt; &quot;Client: I&#39;ve got a simple component:\n&quot;;
  ClientCode(simple);
  std::cout &lt;&lt; &quot;\n\n&quot;;
  /**
   * ...а также сложные контейнеры.
   */

  Component *tree = new Composite;
  Component *branch1 = new Composite;

  Component *leaf_1 = new Leaf;
  Component *leaf_2 = new Leaf;
  Component *leaf_3 = new Leaf;
  branch1-&gt;Add(leaf_1);
  branch1-&gt;Add(leaf_2);
  Component *branch2 = new Composite;
  branch2-&gt;Add(leaf_3);
  tree-&gt;Add(branch1);
  tree-&gt;Add(branch2);
  std::cout &lt;&lt; &quot;Client: Now I&#39;ve got a composite tree:\n&quot;;
  ClientCode(tree);
  std::cout &lt;&lt; &quot;\n\n&quot;;

  std::cout &lt;&lt; &quot;Client: I don&#39;t need to check the components classes even when managing the tree:\n&quot;;
  ClientCode2(tree, simple);
  std::cout &lt;&lt; &quot;\n&quot;;

  delete simple;
  delete tree;
  delete branch1;
  delete branch2;
  delete leaf_1;
  delete leaf_2;
  delete leaf_3;

  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: Leaf

Client: Now I&#39;ve got a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>

::: next
#### Читаем дальше

[Декоратор на C++ []{.fa
.fa-arrow-right}](../../decorator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мост на C++](../../bridge/cpp/example.html){.btn
.btn-default rel="prev"}
:::

## **Компоновщик** на других языках программирования

[![Компоновщик на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Компоновщик на C#"){.prog-lang-link}
[![Компоновщик на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Компоновщик на Go"){.prog-lang-link}
[![Компоновщик на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компоновщик на Java"){.prog-lang-link}
[![Компоновщик на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Компоновщик на PHP"){.prog-lang-link}
[![Компоновщик на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компоновщик на Python"){.prog-lang-link}
[![Компоновщик на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компоновщик на Ruby"){.prog-lang-link}
[![Компоновщик на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компоновщик на Rust"){.prog-lang-link}
[![Компоновщик на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Компоновщик на Swift"){.prog-lang-link}
[![Компоновщик на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компоновщик на TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
