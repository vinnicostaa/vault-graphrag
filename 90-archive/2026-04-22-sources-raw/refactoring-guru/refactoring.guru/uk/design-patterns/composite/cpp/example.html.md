:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Компонувальник](../../composite.html){.type} /
[C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Компонувальник](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Компонувальник** на C++ {#компонувальник-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Компонувальник** --- це структурний патерн, який дозволяє створювати
дерево об'єктів та працювати з ним так само, як і з одиничним об'єктом.

Компонувальник давно став синонімом всіх завдань, пов'язаних з побудовою
дерева об'єктів. Всі операції компонувальника базуються на рекурсії та
«підсумовуванні» результатів на гілках дерева.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Компонувальник ](../../composite.html){.btn .btn-lg
.btn-primary}
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

**Застосування:** Патерн Компонувальник зустрічається в будь-яких
завданнях, які пов'язані з побудовою дерева. Найпростіший приклад ---
складені елементи GUI, які теж можна розглядати як дерево.

**Ознаки застосування патерна:** Якщо з об'єктів будується деревовидна
структура та з усіма об'єктами дерева, як і з самим деревом, працюють
через загальний інтерфейс.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Компонувальник**, а саме --- з
яких класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-cc .anchor} **main.cc:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="cpp"><code>#include &lt;algorithm&gt;
#include &lt;iostream&gt;
#include &lt;list&gt;
#include &lt;string&gt;
/**
 * The base Component class declares common operations for both simple and
 * complex objects of a composition.
 */
class Component {
  /**
   * @var Component
   */
 protected:
  Component *parent_;
  /**
   * Optionally, the base Component can declare an interface for setting and
   * accessing a parent of the component in a tree structure. It can also
   * provide some default implementation for these methods.
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
   * In some cases, it would be beneficial to define the child-management
   * operations right in the base Component class. This way, you won&#39;t need to
   * expose any concrete component classes to the client code, even during the
   * object tree assembly. The downside is that these methods will be empty for
   * the leaf-level components.
   */
  virtual void Add(Component *component) {}
  virtual void Remove(Component *component) {}
  /**
   * You can provide a method that lets the client code figure out whether a
   * component can bear children.
   */
  virtual bool IsComposite() const {
    return false;
  }
  /**
   * The base Component may implement some default behavior or leave it to
   * concrete classes (by declaring the method containing the behavior as
   * &quot;abstract&quot;).
   */
  virtual std::string Operation() const = 0;
};
/**
 * The Leaf class represents the end objects of a composition. A leaf can&#39;t have
 * any children.
 *
 * Usually, it&#39;s the Leaf objects that do the actual work, whereas Composite
 * objects only delegate to their sub-components.
 */
class Leaf : public Component {
 public:
  std::string Operation() const override {
    return &quot;Leaf&quot;;
  }
};
/**
 * The Composite class represents the complex components that may have children.
 * Usually, the Composite objects delegate the actual work to their children and
 * then &quot;sum-up&quot; the result.
 */
class Composite : public Component {
  /**
   * @var \SplObjectStorage
   */
 protected:
  std::list&lt;Component *&gt; children_;

 public:
  /**
   * A composite object can add or remove other components (both simple or
   * complex) to or from its child list.
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
   * The Composite executes its primary logic in a particular way. It traverses
   * recursively through all its children, collecting and summing their results.
   * Since the composite&#39;s children pass these calls to their children and so
   * forth, the whole object tree is traversed as a result.
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
 * The client code works with all of the components via the base interface.
 */
void ClientCode(Component *component) {
  // ...
  std::cout &lt;&lt; &quot;RESULT: &quot; &lt;&lt; component-&gt;Operation();
  // ...
}

/**
 * Thanks to the fact that the child-management operations are declared in the
 * base Component class, the client code can work with any component, simple or
 * complex, without depending on their concrete classes.
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
 * This way the client code can support the simple leaf components...
 */

int main() {
  Component *simple = new Leaf;
  std::cout &lt;&lt; &quot;Client: I&#39;ve got a simple component:\n&quot;;
  ClientCode(simple);
  std::cout &lt;&lt; &quot;\n\n&quot;;
  /**
   * ...as well as the complex composites.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: Leaf

Client: Now I&#39;ve got a composite tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf))

Client: I don&#39;t need to check the components classes even when managing the tree:
RESULT: Branch(Branch(Leaf+Leaf)+Branch(Leaf)+Leaf)</code></pre>
</figure>

::: next
#### Читаємо далі

[Декоратор на C++ []{.fa
.fa-arrow-right}](../../decorator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Міст на C++](../../bridge/cpp/example.html){.btn
.btn-default rel="prev"}
:::

## **Компонувальник** іншими мовами програмування

[![Компонувальник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Компонувальник на C#"){.prog-lang-link}
[![Компонувальник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Компонувальник на Go"){.prog-lang-link}
[![Компонувальник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Компонувальник на Java"){.prog-lang-link}
[![Компонувальник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Компонувальник на PHP"){.prog-lang-link}
[![Компонувальник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Компонувальник на Python"){.prog-lang-link}
[![Компонувальник на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Компонувальник на Ruby"){.prog-lang-link}
[![Компонувальник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Компонувальник на Rust"){.prog-lang-link}
[![Компонувальник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Компонувальник на Swift"){.prog-lang-link}
[![Компонувальник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Компонувальник на TypeScript"){.prog-lang-link}
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
