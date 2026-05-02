:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посетитель](../../visitor.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посетитель](../../../../images/patterns/cards/visitor-mini0a1c.png?id=854a35a62963bec1d75eab996918989b){srcset="/images/patterns/cards/visitor-mini-2x.png?id=9b87f3f3b772f434b28a25876829b504 2x"}
:::

# **Посетитель** на C++ {#посетитель-на-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Посетитель** --- это поведенческий паттерн, который позволяет добавить
новую операцию для целой иерархии классов, не изменяя код этих классов.

> Подробней о том, почему Посетитель нельзя заменить простой перегрузкой
> методов читайте в статье [Посетитель и Double
> Dispatch](../../visitor-double-dispatch.html).
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посетитель ](../../visitor.html){.btn .btn-lg
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

**Применимость:** Посетитель нечасто встречается в C++ коде из-за своей
сложности и нюансов реализазации.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Посетитель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-cc .anchor} **main.cc:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Интерфейс Посетителя объявляет набор методов посещения, соответствующих
 * классам компонентов. Сигнатура метода посещения позволяет посетителю
 * определить конкретный класс компонента, с которым он имеет дело.
 */
class ConcreteComponentA;
class ConcreteComponentB;

class Visitor {
 public:
  virtual void VisitConcreteComponentA(const ConcreteComponentA *element) const = 0;
  virtual void VisitConcreteComponentB(const ConcreteComponentB *element) const = 0;
};

/**
 * Интерфейс Компонента объявляет метод accept, который в качестве аргумента
 * может получать любой объект, реализующий интерфейс посетителя.
 */

class Component {
 public:
  virtual ~Component() {}
  virtual void Accept(Visitor *visitor) const = 0;
};

/**
 * Каждый Конкретный Компонент должен реализовать метод accept таким образом,
 * чтобы он вызывал метод посетителя, соответствующий классу компонента.
 */
class ConcreteComponentA : public Component {
  /**
   * Обратите внимание, мы вызываем visitConcreteComponentA, что соответствует
   * названию текущего класса. Таким образом мы позволяем посетителю узнать, с
   * каким классом компонента он работает.
   */
 public:
  void Accept(Visitor *visitor) const override {
    visitor-&gt;VisitConcreteComponentA(this);
  }
  /**
   * Конкретные Компоненты могут иметь особые методы, не объявленные в их
   * базовом классе или интерфейсе. Посетитель всё же может использовать эти
   * методы, поскольку он знает о конкретном классе компонента.
   */
  std::string ExclusiveMethodOfConcreteComponentA() const {
    return &quot;A&quot;;
  }
};

class ConcreteComponentB : public Component {
  /**
   * То же самое здесь: visitConcreteComponentB =&gt; ConcreteComponentB
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
 * Конкретные Посетители реализуют несколько версий одного и того же алгоритма,
 * которые могут работать со всеми классами конкретных компонентов.
 *
 * Максимальную выгоду от паттерна Посетитель вы почувствуете, используя его со
 * сложной структурой объектов, такой как дерево Компоновщика. В этом случае
 * было бы полезно хранить некоторое промежуточное состояние алгоритма при
 * выполнении методов посетителя над различными объектами структуры.
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
 * Клиентский код может выполнять операции посетителя над любым набором
 * элементов, не выясняя их конкретных классов. Операция принятия направляет
 * вызов к соответствующей операции в объекте посетителя.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>The client code works with all visitors via the base Visitor interface:
A + ConcreteVisitor1
B + ConcreteVisitor1

It allows the same client code to work with different types of visitors:
A + ConcreteVisitor2
B + ConcreteVisitor2</code></pre>
</figure>

::: next
#### Читаем дальше

[Паттерны проектирования на Go []{.fa
.fa-arrow-right}](../../go.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод на
C++](../../template-method/cpp/example.html){.btn .btn-default
rel="prev"}
:::

## **Посетитель** на других языках программирования

[![Посетитель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посетитель на Go"){.prog-lang-link}
[![Посетитель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посетитель на PHP"){.prog-lang-link}
[![Посетитель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посетитель на Python"){.prog-lang-link}
[![Посетитель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посетитель на Swift"){.prog-lang-link}
[![Посетитель на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
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
