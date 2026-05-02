:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Состояние](../../state.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Состояние](../../../../images/patterns/cards/state-mini63fb.png?id=f4018837e0641d1dade756b6678fd4ee){srcset="/images/patterns/cards/state-mini-2x.png?id=7e24398b27a43c7bd286fc0ea54d2a35 2x"}
:::

# **Состояние** на Ruby {#состояние-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Состояние** --- это поведенческий паттерн, позволяющий динамически
изменять поведение объекта при смене его состояния.

Поведения, зависящие от состояния, переезжают в отдельные классы.
Первоначальный класс хранит ссылку на один из таких объектов-состояний и
делегирует ему работу.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Состояние ](../../state.html){.btn .btn-lg
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

**Применимость:** Паттерн Состояние часто используют в Ruby для
превращения в объекты громоздких стейт-машин, построенных на операторах
`switch`.

**Признаки применения паттерна:** Методы класса делегируют работу одному
вложенному объекту.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Состояние**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Контекст определяет интерфейс, представляющий интерес для клиентов. Он также
# хранит ссылку на экземпляр подкласса Состояния, который отображает текущее
# состояние Контекста.
#
# @abstract
class Context
  # Ссылка на текущее состояние Контекста.
  # @return [State]
  attr_accessor :state
  private :state

  # @param [State] state
  def initialize(state)
    transition_to(state)
  end

  # Контекст позволяет изменять объект Состояния во время выполнения.
  #
  # @param [State] state
  def transition_to(state)
    puts &quot;Context: Transition to #{state.class}&quot;
    @state = state
    @state.context = self
  end

  # Контекст делегирует часть своего поведения текущему объекту Состояния.

  def request1
    @state.handle1
  end

  def request2
    @state.handle2
  end
end

# Базовый класс Состояния объявляет методы, которые должны реализовать все
# Конкретные Состояния, а также предоставляет обратную ссылку на объект
# Контекст, связанный с Состоянием. Эта обратная ссылка может использоваться
# Состояниями для передачи Контекста другому Состоянию.
#
# @abstract
class State
  attr_accessor :context

  # @abstract
  def handle1
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  def handle2
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Конкретные Состояния реализуют различные модели поведения, связанные с
# состоянием Контекста.

class ConcreteStateA &lt; State
  def handle1
    puts &#39;ConcreteStateA handles request1.&#39;
    puts &#39;ConcreteStateA wants to change the state of the context.&#39;
    @context.transition_to(ConcreteStateB.new)
  end

  def handle2
    puts &#39;ConcreteStateA handles request2.&#39;
  end
end

class ConcreteStateB &lt; State
  def handle1
    puts &#39;ConcreteStateB handles request1.&#39;
  end

  def handle2
    puts &#39;ConcreteStateB handles request2.&#39;
    puts &#39;ConcreteStateB wants to change the state of the context.&#39;
    @context.transition_to(ConcreteStateA.new)
  end
end

# Клиентский код.

context = Context.new(ConcreteStateA.new)
context.request1
context.request2</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Context: Transition to ConcreteStateA
ConcreteStateA handles request1.
ConcreteStateA wants to change the state of the context.
Context: Transition to ConcreteStateB
ConcreteStateB handles request2.
ConcreteStateB wants to change the state of the context.
Context: Transition to ConcreteStateA</code></pre>
</figure>

## **Состояние** на других языках программирования

[![Состояние на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Состояние на C#"){.prog-lang-link}
[![Состояние на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Состояние на C++"){.prog-lang-link}
[![Состояние на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Состояние на Go"){.prog-lang-link}
[![Состояние на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Состояние на Java"){.prog-lang-link}
[![Состояние на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Состояние на PHP"){.prog-lang-link}
[![Состояние на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Состояние на Python"){.prog-lang-link}
[![Состояние на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Состояние на Rust"){.prog-lang-link}
[![Состояние на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Состояние на Swift"){.prog-lang-link}
[![Состояние на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Состояние на TypeScript"){.prog-lang-link}
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
