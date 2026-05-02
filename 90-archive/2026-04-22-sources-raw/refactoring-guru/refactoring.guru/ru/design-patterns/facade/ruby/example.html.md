:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Фасад](../../facade.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Фасад](../../../../images/patterns/cards/facade-mini14fd.png?id=71ad6fa98b168c11cb3a1a9517dedf78){srcset="/images/patterns/cards/facade-mini-2x.png?id=d4cc6a5d81a31143cc665f7ac1481ac8 2x"}
:::

# **Фасад** на Ruby {#фасад-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Фасад** --- это структурный паттерн, который предоставляет простой (но
урезанный) интерфейс к сложной системе объектов, библиотеке или
фреймворку.

Кроме того, что Фасад позволяет снизить общую сложность программы, он
также помогает вынести код, зависимый от внешней системы в единственное
место.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Фасад ](../../facade.html){.btn .btn-lg
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

**Применимость:** Паттерн часто встречается в клиентских приложениях,
написанных на Ruby, которые используют классы-фасады для упрощения
работы со сложными библиотеки или API.

**Признаки применения паттерна:** Фасад угадывается в классе, который
имеет простой интерфейс, но делегирует основную часть работы другим
классам. Чаще всего, фасады сами следят за жизненным циклом объектов
сложной системы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Фасад**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Класс Фасада предоставляет простой интерфейс для сложной логики одной или
# нескольких подсистем. Фасад делегирует запросы клиентов соответствующим
# объектам внутри подсистемы. Фасад также отвечает за управление их жизненным
# циклом. Все это защищает клиента от нежелательной сложности подсистемы.
class Facade
  # В зависимости от потребностей вашего приложения вы можете предоставить
  # Фасаду существующие объекты подсистемы или заставить Фасад создать их
  # самостоятельно.
  #
  # @param [Subsystem1] subsystem1
  # @param [Subsystem2] subsystem2
  def initialize(subsystem1, subsystem2)
    @subsystem1 = subsystem1 || Subsystem1.new
    @subsystem2 = subsystem2 || Subsystem2.new
  end

  # Методы Фасада удобны для быстрого доступа к сложной функциональности
  # подсистем. Однако клиенты получают только часть возможностей подсистемы.
  #
  # @return [String]
  def operation
    results = []
    results.append(&#39;Facade initializes subsystems:&#39;)
    results.append(@subsystem1.operation1)
    results.append(@subsystem2.operation1)
    results.append(&#39;Facade orders subsystems to perform the action:&#39;)
    results.append(@subsystem1.operation_n)
    results.append(@subsystem2.operation_z)
    results.join(&quot;\n&quot;)
  end
end

# Подсистема может принимать запросы либо от фасада, либо от клиента напрямую. В
# любом случае, для Подсистемы Фасад – это ещё один клиент, и он не является
# частью Подсистемы.
class Subsystem1
  # @return [String]
  def operation1
    &#39;Subsystem1: Ready!&#39;
  end

  # ...

  # @return [String]
  def operation_n
    &#39;Subsystem1: Go!&#39;
  end
end

# Некоторые фасады могут работать с разными подсистемами одновременно.
class Subsystem2
  # @return [String]
  def operation1
    &#39;Subsystem2: Get ready!&#39;
  end

  # ...

  # @return [String]
  def operation_z
    &#39;Subsystem2: Fire!&#39;
  end
end

# Клиентский код работает со сложными подсистемами через простой интерфейс,
# предоставляемый Фасадом. Когда фасад управляет жизненным циклом подсистемы,
# клиент может даже не знать о существовании подсистемы. Такой подход позволяет
# держать сложность под контролем.
#
# @param [Facade] facade
def client_code(facade)
  print facade.operation
end

# В клиентском коде могут быть уже созданы некоторые объекты подсистемы. В этом
# случае может оказаться целесообразным инициализировать Фасад с этими объектами
# вместо того, чтобы позволить Фасаду создавать новые экземпляры.
subsystem1 = Subsystem1.new
subsystem2 = Subsystem2.new
facade = Facade.new(subsystem1, subsystem2)
client_code(facade)</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Facade initializes subsystems:
Subsystem1: Ready!
Subsystem2: Get ready!
Facade orders subsystems to perform the action:
Subsystem1: Go!
Subsystem2: Fire!</code></pre>
</figure>

## **Фасад** на других языках программирования

[![Фасад на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Фасад на C#"){.prog-lang-link}
[![Фасад на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Фасад на C++"){.prog-lang-link}
[![Фасад на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Фасад на Go"){.prog-lang-link}
[![Фасад на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Фасад на Java"){.prog-lang-link}
[![Фасад на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Фасад на PHP"){.prog-lang-link}
[![Фасад на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Фасад на Python"){.prog-lang-link}
[![Фасад на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Фасад на Rust"){.prog-lang-link}
[![Фасад на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Фасад на Swift"){.prog-lang-link}
[![Фасад на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Фасад на TypeScript"){.prog-lang-link}
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
