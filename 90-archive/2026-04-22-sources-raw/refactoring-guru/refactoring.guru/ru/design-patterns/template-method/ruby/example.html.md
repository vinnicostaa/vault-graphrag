:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Шаблонный
метод](../../template-method.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Шаблонный
метод](../../../../images/patterns/cards/template-method-mini56e3.png?id=9f200248d88026d8e79d0f3dae411ab4){srcset="/images/patterns/cards/template-method-mini-2x.png?id=178bf56e39b3a1f548dd636076209c98 2x"}
:::

# **Шаблонный метод** на Ruby {#шаблонный-метод-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Шаблонный метод** --- это поведенческий паттерн, задающий скелет
алгоритма в суперклассе и заставляющий подклассы реализовать конкретные
шаги этого алгоритма.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Шаблонный метод
](../../template-method.html){.btn .btn-lg .btn-primary}
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Шаблонные методы можно встретить во многих
библиотечных классах Ruby. Разработчики создают их, чтобы позволить
клиентам легко и быстро расширять стандартный код при помощи
наследования.

**Признаки применения паттерна:** Класс заставляет своих потомков
реализовать методы-шаги, но самостоятельно реализует структуру
алгоритма.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Шаблонный метод**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Абстрактный Класс определяет шаблонный метод, содержащий скелет некоторого
# алгоритма, состоящего из вызовов (обычно) абстрактных примитивных операций.
#
# Конкретные подклассы должны реализовать эти операции, но оставить сам
# шаблонный метод без изменений.
#
# @abstract
class AbstractClass
  # Шаблонный метод определяет скелет алгоритма.
  def template_method
    base_operation1
    required_operations1
    base_operation2
    hook1
    required_operations2
    base_operation3
    hook2
  end

  # Эти операции уже имеют реализации.

  def base_operation1
    puts &#39;AbstractClass says: I am doing the bulk of the work&#39;
  end

  def base_operation2
    puts &#39;AbstractClass says: But I let subclasses override some operations&#39;
  end

  def base_operation3
    puts &#39;AbstractClass says: But I am doing the bulk of the work anyway&#39;
  end

  # А эти операции должны быть реализованы в подклассах.
  #
  # @abstract
  def required_operations1
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def required_operations2
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Это «хуки». Подклассы могут переопределять их, но это не обязательно,
  # поскольку у хуков уже есть стандартная (но пустая) реализация. Хуки
  # предоставляют дополнительные точки расширения в некоторых критических местах
  # алгоритма.

  def hook1; end

  def hook2; end
end

# Конкретные классы должны реализовать все абстрактные операции базового класса.
# Они также могут переопределить некоторые операции с реализацией по умолчанию.
class ConcreteClass1 &lt; AbstractClass
  def required_operations1
    puts &#39;ConcreteClass1 says: Implemented Operation1&#39;
  end

  def required_operations2
    puts &#39;ConcreteClass1 says: Implemented Operation2&#39;
  end
end

# Обычно конкретные классы переопределяют только часть операций базового класса.
class ConcreteClass2 &lt; AbstractClass
  def required_operations1
    puts &#39;ConcreteClass2 says: Implemented Operation1&#39;
  end

  def required_operations2
    puts &#39;ConcreteClass2 says: Implemented Operation2&#39;
  end

  def hook1
    puts &#39;ConcreteClass2 says: Overridden Hook1&#39;
  end
end

# Клиентский код вызывает шаблонный метод для выполнения алгоритма. Клиентский
# код не должен знать конкретный класс объекта, с которым работает, при условии,
# что он работает с объектами через интерфейс их базового класса.
#
# @param [AbstractClass] abstract_class
def client_code(abstract_class)
  # ...
  abstract_class.template_method
  # ...
end

puts &#39;Same client code can work with different subclasses:&#39;
client_code(ConcreteClass1.new)
puts &quot;\n&quot;

puts &#39;Same client code can work with different subclasses:&#39;
client_code(ConcreteClass2.new)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

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

## **Шаблонный метод** на других языках программирования

[![Шаблонный метод на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Шаблонный метод на C#"){.prog-lang-link}
[![Шаблонный метод на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Шаблонный метод на C++"){.prog-lang-link}
[![Шаблонный метод на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Шаблонный метод на Go"){.prog-lang-link}
[![Шаблонный метод на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Шаблонный метод на Java"){.prog-lang-link}
[![Шаблонный метод на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Шаблонный метод на PHP"){.prog-lang-link}
[![Шаблонный метод на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Шаблонный метод на Python"){.prog-lang-link}
[![Шаблонный метод на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Шаблонный метод на Rust"){.prog-lang-link}
[![Шаблонный метод на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Шаблонный метод на Swift"){.prog-lang-link}
[![Шаблонный метод на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Шаблонный метод на TypeScript"){.prog-lang-link}
:::::::::::::::

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
:::::::::::::::::::::
::::::::::::::::::::::
