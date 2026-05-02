:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Посредник](../../mediator.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Посредник](../../../../images/patterns/cards/mediator-mini1561.png?id=a7e43ee8e17e4474737b1fcb3201d7ba){srcset="/images/patterns/cards/mediator-mini-2x.png?id=d288d7c73f5581ae09701d61200ae09e 2x"}
:::

# **Посредник** на Ruby {#посредник-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Посредник** --- это поведенческий паттерн, который упрощает
коммуникацию между компонентами системы.

Посредник убирает прямые связи между отдельными компонентами, заставляя
их общаться друг с другом через себя.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Посредник ](../../mediator.html){.btn .btn-lg
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

**Применимость:** Пожалуй, самое популярное применение Посредника в
Ruby-коде --- это связь нескольких компонентов GUI одной программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Посредник**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Интерфейс Посредника предоставляет метод, используемый компонентами для
# уведомления посредника о различных событиях. Посредник может реагировать на
# эти события и передавать исполнение другим компонентам.
#
# @abstract
class Mediator
  # @abstract
  #
  # @param [Object] sender
  # @param [String] event
  def notify(_sender, _event)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

class ConcreteMediator &lt; Mediator
  # @param [Component1] component1
  # @param [Component2] component2
  def initialize(component1, component2)
    @component1 = component1
    @component1.mediator = self
    @component2 = component2
    @component2.mediator = self
  end

  # @param [Object] sender
  # @param [String] event
  def notify(_sender, event)
    if event == &#39;A&#39;
      puts &#39;Mediator reacts on A and triggers following operations:&#39;
      @component2.do_c
    elsif event == &#39;D&#39;
      puts &#39;Mediator reacts on D and triggers following operations:&#39;
      @component1.do_b
      @component2.do_c
    end
  end
end

# Базовый Компонент обеспечивает базовую функциональность хранения экземпляра
# посредника внутри объектов компонентов.
class BaseComponent
  # @return [Mediator]
  attr_accessor :mediator

  # @param [Mediator] mediator
  def initialize(mediator = nil)
    @mediator = mediator
  end
end

# Конкретные Компоненты реализуют различную функциональность. Они не зависят от
# других компонентов. Они также не зависят от каких-либо конкретных классов
# посредников.
class Component1 &lt; BaseComponent
  def do_a
    puts &#39;Component 1 does A.&#39;
    @mediator.notify(self, &#39;A&#39;)
  end

  def do_b
    puts &#39;Component 1 does B.&#39;
    @mediator.notify(self, &#39;B&#39;)
  end
end

class Component2 &lt; BaseComponent
  def do_c
    puts &#39;Component 2 does C.&#39;
    @mediator.notify(self, &#39;C&#39;)
  end

  def do_d
    puts &#39;Component 2 does D.&#39;
    @mediator.notify(self, &#39;D&#39;)
  end
end

# Клиентский код.
c1 = Component1.new
c2 = Component2.new
ConcreteMediator.new(c1, c2)

puts &#39;Client triggers operation A.&#39;
c1.do_a

puts &quot;\n&quot;

puts &#39;Client triggers operation D.&#39;
c2.do_d</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Client triggers operation A.
Component 1 does A.
Mediator reacts on A and triggers following operations:
Component 2 does C.

Client triggers operation D.
Component 2 does D.
Mediator reacts on D and triggers following operations:
Component 1 does B.
Component 2 does C.</code></pre>
</figure>

## **Посредник** на других языках программирования

[![Посредник на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Посредник на C#"){.prog-lang-link}
[![Посредник на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Посредник на C++"){.prog-lang-link}
[![Посредник на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Посредник на Go"){.prog-lang-link}
[![Посредник на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Посредник на Java"){.prog-lang-link}
[![Посредник на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Посредник на PHP"){.prog-lang-link}
[![Посредник на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Посредник на Python"){.prog-lang-link}
[![Посредник на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Посредник на Rust"){.prog-lang-link}
[![Посредник на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Посредник на Swift"){.prog-lang-link}
[![Посредник на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Посредник на TypeScript"){.prog-lang-link}
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
