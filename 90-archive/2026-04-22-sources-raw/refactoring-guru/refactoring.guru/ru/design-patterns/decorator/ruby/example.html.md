:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Декоратор](../../decorator.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Декоратор](../../../../images/patterns/cards/decorator-mini6adb.png?id=d30458908e315af195cb183bc52dbef9){srcset="/images/patterns/cards/decorator-mini-2x.png?id=3b58e540d7d28523080cad341ed9b2e9 2x"}
:::

# **Декоратор** на Ruby {#декоратор-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Декоратор** --- это структурный паттерн, который позволяет добавлять
объектам новые поведения на лету, помещая их в объекты-обёртки.

Декоратор позволяет оборачивать объекты бесчисленное количество раз
благодаря тому, что и обёртки, и реальные оборачиваемые объекты имеют
общий интерфейс.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Декоратор ](../../decorator.html){.btn .btn-lg
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
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в Ruby-коде, особенно в
коде, работающем с потоками данных.

**Признаки применения паттерна:** Декоратор можно распознать по
создающим методам, которые принимают в параметрах объекты того же
абстрактного типа или интерфейса, что и текущий класс.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Декоратор**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Базовый интерфейс Компонента определяет поведение, которое изменяется
# декораторами.
class Component
  # @return [String]
  def operation
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Конкретные Компоненты предоставляют реализации поведения по умолчанию. Может
# быть несколько вариаций этих классов.
class ConcreteComponent &lt; Component
  # @return [String]
  def operation
    &#39;ConcreteComponent&#39;
  end
end

# Базовый класс Декоратора следует тому же интерфейсу, что и другие компоненты.
# Основная цель этого класса - определить интерфейс обёртки для всех конкретных
# декораторов. Реализация кода обёртки по умолчанию может включать в себя поле
# для хранения завёрнутого компонента и средства его инициализации.
class Decorator &lt; Component
  attr_accessor :component

  # @param [Component] component
  def initialize(component)
    @component = component
  end

  # Декоратор делегирует всю работу обёрнутому компоненту.
  #
  # @return [String]
  def operation
    @component.operation
  end
end

# Конкретные Декораторы вызывают обёрнутый объект и изменяют его результат
# некоторым образом.
class ConcreteDecoratorA &lt; Decorator
  # Декораторы могут вызывать родительскую реализацию операции, вместо того,
  # чтобы вызвать обёрнутый объект напрямую. Такой подход упрощает расширение
  # классов декораторов.
  #
  # @return [String]
  def operation
    &quot;ConcreteDecoratorA(#{@component.operation})&quot;
  end
end

# Декораторы могут выполнять своё поведение до или после вызова обёрнутого
# объекта.
class ConcreteDecoratorB &lt; Decorator
  # @return [String]
  def operation
    &quot;ConcreteDecoratorB(#{@component.operation})&quot;
  end
end

# Клиентский код работает со всеми объектами, используя интерфейс Компонента.
# Таким образом, он остаётся независимым от конкретных классов компонентов, с
# которыми работает.
#
# @param [Component] component
def client_code(component)
  # ...

  print &quot;RESULT: #{component.operation}&quot;

  # ...
end

# Таким образом, клиентский код может поддерживать как простые компоненты...
simple = ConcreteComponent.new
puts &#39;Client: I\&#39;ve got a simple component:&#39;
client_code(simple)
puts &quot;\n\n&quot;

# ...так и декорированные.
#
# Обратите внимание, что декораторы могут обёртывать не только простые
# компоненты, но и другие декораторы.
decorator1 = ConcreteDecoratorA.new(simple)
decorator2 = ConcreteDecoratorB.new(decorator1)
puts &#39;Client: Now I\&#39;ve got a decorated component:&#39;
client_code(decorator2)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: I&#39;ve got a simple component:
RESULT: ConcreteComponent

Client: Now I&#39;ve got a decorated component:
RESULT: ConcreteDecoratorB(ConcreteDecoratorA(ConcreteComponent))</code></pre>
</figure>

## **Декоратор** на других языках программирования

[![Декоратор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Декоратор на C#"){.prog-lang-link}
[![Декоратор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Декоратор на C++"){.prog-lang-link}
[![Декоратор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Декоратор на Go"){.prog-lang-link}
[![Декоратор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Декоратор на Java"){.prog-lang-link}
[![Декоратор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Декоратор на PHP"){.prog-lang-link}
[![Декоратор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Декоратор на Python"){.prog-lang-link}
[![Декоратор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Декоратор на Rust"){.prog-lang-link}
[![Декоратор на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Декоратор на Swift"){.prog-lang-link}
[![Декоратор на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Декоратор на TypeScript"){.prog-lang-link}
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
