:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Абстрактная
фабрика](../../abstract-factory.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Абстрактная
фабрика](../../../../images/patterns/cards/abstract-factory-minid1c5.png?id=4c3927c446313a38ce77dfee38111e27){srcset="/images/patterns/cards/abstract-factory-mini-2x.png?id=22236aaa65ff52cbde1c713216d52c1f 2x"}
:::

# **Абстрактная фабрика** на Ruby {#абстрактная-фабрика-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Абстрактная фабрика** --- это порождающий паттерн проектирования,
который решает проблему создания целых семейств связанных продуктов, без
указания конкретных классов продуктов.

Абстрактная фабрика задаёт интерфейс создания всех доступных типов
продуктов, а каждая конкретная реализация фабрики порождает продукты
одной из вариаций. Клиентский код вызывает методы фабрики для получения
продуктов, вместо самостоятельного создания с помощью оператора `new`.
При этом фабрика сама следит за тем, чтобы создать продукт нужной
вариации.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Абстрактная фабрика
](../../abstract-factory.html){.btn .btn-lg .btn-primary}
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

**Применимость:** Паттерн можно часто встретить в Ruby-коде, особенно
там, где требуется создание семейств продуктов (например, внутри
фреймворков).

**Признаки применения паттерна:** Паттерн можно определить по методам,
возвращающим фабрику, которая, в свою очередь, используется для создания
конкретных продуктов, возвращая их через абстрактные типы или
интерфейсы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Абстрактная фабрика**, а
именно --- из каких классов он состоит, какие роли эти классы выполняют
и как они взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Интерфейс Абстрактной Фабрики объявляет набор методов, которые возвращают
# различные абстрактные продукты. Эти продукты называются семейством и связаны
# темой или концепцией высокого уровня. Продукты одного семейства обычно могут
# взаимодействовать между собой. Семейство продуктов может иметь несколько
# вариаций, но продукты одной вариации несовместимы с продуктами другой.
#
# @abstract
class AbstractFactory
  # @abstract
  def create_product_a
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def create_product_b
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Конкретная Фабрика производит семейство продуктов одной вариации. Фабрика
# гарантирует совместимость полученных продуктов. Обратите внимание, что
# сигнатуры методов Конкретной Фабрики возвращают абстрактный продукт, в то
# время как внутри метода создается экземпляр конкретного продукта.
class ConcreteFactory1 &lt; AbstractFactory
  def create_product_a
    ConcreteProductA1.new
  end

  def create_product_b
    ConcreteProductB1.new
  end
end

# Каждая Конкретная Фабрика имеет соответствующую вариацию продукта.
class ConcreteFactory2 &lt; AbstractFactory
  def create_product_a
    ConcreteProductA2.new
  end

  def create_product_b
    ConcreteProductB2.new
  end
end

# Каждый отдельный продукт семейства продуктов должен иметь базовый интерфейс.
# Все вариации продукта должны реализовывать этот интерфейс.
#
# @abstract
class AbstractProductA
  # @abstract
  #
  # @return [String]
  def useful_function_a
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Конкретные продукты создаются соответствующими Конкретными Фабриками.
class ConcreteProductA1 &lt; AbstractProductA
  def useful_function_a
    &#39;The result of the product A1.&#39;
  end
end

class ConcreteProductA2 &lt; AbstractProductA
  def useful_function_a
    &#39;The result of the product A2.&#39;
  end
end

# Базовый интерфейс другого продукта. Все продукты могут взаимодействовать друг
# с другом, но правильное взаимодействие возможно только между продуктами одной
# и той же конкретной вариации.
#
# @abstract
class AbstractProductB
  # Продукт B способен работать самостоятельно...
  #
  # @abstract
  def useful_function_b
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # ...а также взаимодействовать с Продуктами A той же вариации.
  #
  # Абстрактная Фабрика гарантирует, что все продукты, которые она создает,
  # имеют одинаковую вариацию и, следовательно, совместимы.
  #
  # @abstract
  #
  # @param [AbstractProductA] collaborator
  def another_useful_function_b(_collaborator)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Конкретные Продукты создаются соответствующими Конкретными Фабриками.
class ConcreteProductB1 &lt; AbstractProductB
  # @return [String]
  def useful_function_b
    &#39;The result of the product B1.&#39;
  end

  # Продукт B1 может корректно работать только с Продуктом A1. Тем не менее, он
  # принимает любой экземпляр Абстрактного Продукта А в качестве аргумента.
  #
  # @param [AbstractProductA] collaborator
  #
  # @return [String]
  def another_useful_function_b(collaborator)
    result = collaborator.useful_function_a
    &quot;The result of the B1 collaborating with the (#{result})&quot;
  end
end

class ConcreteProductB2 &lt; AbstractProductB
  # @return [String]
  def useful_function_b
    &#39;The result of the product B2.&#39;
  end

  # Продукт B2 может корректно работать только с Продуктом A2. Тем не менее, он
  # принимает любой экземпляр Абстрактного Продукта А в качестве аргумента.
  #
  # @param [AbstractProductA] collaborator
  def another_useful_function_b(collaborator)
    result = collaborator.useful_function_a
    &quot;The result of the B2 collaborating with the (#{result})&quot;
  end
end

# Клиентский код работает с фабриками и продуктами только через абстрактные
# типы: Абстрактная Фабрика и Абстрактный Продукт. Это позволяет передавать
# любой подкласс фабрики или продукта клиентскому коду, не нарушая его.
#
# @param [AbstractFactory] factory
def client_code(factory)
  product_a = factory.create_product_a
  product_b = factory.create_product_b

  puts product_b.useful_function_b
  puts product_b.another_useful_function_b(product_a)
end

# Клиентский код может работать с любым конкретным классом фабрики.
puts &#39;Client: Testing client code with the first factory type:&#39;
client_code(ConcreteFactory1.new)

puts &quot;\n&quot;

puts &#39;Client: Testing the same client code with the second factory type:&#39;
client_code(ConcreteFactory2.new)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client: Testing client code with the first factory type:
The result of the product B1.
The result of the B1 collaborating with the (The result of the product A1.)

Client: Testing the same client code with the second factory type:
The result of the product B2.
The result of the B2 collaborating with the (The result of the product A2.)</code></pre>
</figure>

## **Абстрактная фабрика** на других языках программирования

[![Абстрактная фабрика на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Абстрактная фабрика на C#"){.prog-lang-link}
[![Абстрактная фабрика на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Абстрактная фабрика на C++"){.prog-lang-link}
[![Абстрактная фабрика на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Абстрактная фабрика на Go"){.prog-lang-link}
[![Абстрактная фабрика на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Абстрактная фабрика на Java"){.prog-lang-link}
[![Абстрактная фабрика на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Абстрактная фабрика на PHP"){.prog-lang-link}
[![Абстрактная фабрика на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Абстрактная фабрика на Python"){.prog-lang-link}
[![Абстрактная фабрика на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Абстрактная фабрика на Rust"){.prog-lang-link}
[![Абстрактная фабрика на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Абстрактная фабрика на Swift"){.prog-lang-link}
[![Абстрактная фабрика на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Абстрактная фабрика на TypeScript"){.prog-lang-link}
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
