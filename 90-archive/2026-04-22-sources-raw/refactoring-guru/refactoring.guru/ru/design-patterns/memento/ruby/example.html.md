:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Снимок](../../memento.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Снимок](../../../../images/patterns/cards/memento-mini723d.png?id=8b2ea4dc2c5d15775a654808cc9de099){srcset="/images/patterns/cards/memento-mini-2x.png?id=1d7cba189261dd84b11369a6838b1055 2x"}
:::

# **Снимок** на Ruby {#снимок-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Снимок** --- это поведенческий паттерн, позволяющий делать снимки
внутреннего состояния объектов, а затем восстанавливать их.

При этом Снимок не раскрывает подробностей реализации объектов, и клиент
не имеет доступа к защищённой информации объекта.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Снимок ](../../memento.html){.btn .btn-lg
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

**Применимость:** Снимок на Ruby чаще всего реализуют с помощью
сериализации. Но это не единственный, да и не самый эффективный метод
сохранения состояния объектов во время выполнения программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Снимок**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Создатель содержит некоторое важное состояние, которое может со временем
# меняться. Он также объявляет метод сохранения состояния внутри снимка и метод
# восстановления состояния из него.
class Originator
  # Для удобства состояние создателя хранится внутри одной переменной.
  attr_accessor :state
  private :state

  # @param [String] state
  def initialize(state)
    @state = state
    puts &quot;Originator: My initial state is: #{@state}&quot;
  end

  # Бизнес-логика Создателя может повлиять на его внутреннее состояние. Поэтому
  # клиент должен выполнить резервное копирование состояния с помощью метода
  # save перед запуском методов бизнес-логики.
  def do_something
    puts &#39;Originator: I\&#39;m doing something important.&#39;
    @state = generate_random_string(30)
    puts &quot;Originator: and my state has changed to: #{@state}&quot;
  end

  private def generate_random_string(length = 10)
    ascii_letters = [*&#39;a&#39;..&#39;z&#39;, *&#39;A&#39;..&#39;Z&#39;]
    (0...length).map { ascii_letters.sample }.join
  end

  # Сохраняет текущее состояние внутри снимка.
  #
  # @return [Memento]
  def save
    ConcreteMemento.new(@state)
  end

  # Восстанавливает состояние Создателя из объекта снимка.
  #
  # @param [Memento] memento
  def restore(memento)
    @state = memento.state
    puts &quot;Originator: My state has changed to: #{@state}&quot;
  end
end

# Интерфейс Снимка предоставляет способ извлечения метаданных снимка, таких как
# дата создания или название. Однако он не раскрывает состояние Создателя.
#
# @abstract
class Memento
  # @abstract
  #
  # @return [String]
  def name
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # @abstract
  #
  # @return [String]
  def date
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

class ConcreteMemento &lt; Memento
  # @param [String] state
  def initialize(state)
    @state = state
    @date = Time.now.strftime(&#39;%F %T&#39;)
  end

  # Создатель использует этот метод, когда восстанавливает своё состояние.
  #
  # @return [String]
  attr_reader :state

  # Остальные методы используются Опекуном для отображения метаданных.
  #
  # @return [String]
  def name
    &quot;#{@date} / (#{@state[0, 9]}...)&quot;
  end

  # @return [String]
  attr_reader :date
end

# Опекун не зависит от класса Конкретного Снимка. Таким образом, он не имеет
# доступа к состоянию создателя, хранящемуся внутри снимка. Он работает со всеми
# снимками через базовый интерфейс Снимка.
class Caretaker
  # @param [Originator] originator
  def initialize(originator)
    @mementos = []
    @originator = originator
  end

  def backup
    puts &quot;\nCaretaker: Saving Originator&#39;s state...&quot;
    @mementos &lt;&lt; @originator.save
  end

  def undo
    return if @mementos.empty?

    memento = @mementos.pop
    puts &quot;Caretaker: Restoring state to: #{memento.name}&quot;

    begin
      @originator.restore(memento)
    rescue StandardError
      undo
    end
  end

  def show_history
    puts &#39;Caretaker: Here\&#39;s the list of mementos:&#39;

    @mementos.each { |memento| puts memento.name }
  end
end

originator = Originator.new(&#39;Super-duper-super-puper-super.&#39;)
caretaker = Caretaker.new(originator)

caretaker.backup
originator.do_something

caretaker.backup
originator.do_something

caretaker.backup
originator.do_something

puts &quot;\n&quot;
caretaker.show_history

puts &quot;\nClient: Now, let&#39;s rollback!\n&quot;
caretaker.undo

puts &quot;\nClient: Once more!\n&quot;
caretaker.undo</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Originator: My initial state is: Super-duper-super-puper-super.

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: CHYzYSIWbqvWkCzIHOqTyEJWfQlFMn

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: vbkhwCeAEQBpLwQLlhmpcvUnwzxVnT

Caretaker: Saving Originator&#39;s state...
Originator: I&#39;m doing something important.
Originator: and my state has changed to: SBWlQnAEPLsitiOQAZbGlXHZAeWBoW

Caretaker: Here&#39;s the list of mementos:
2023-08-11 15:02:35 / (Super-dup...)
2023-08-11 15:02:35 / (CHYzYSIWb...)
2023-08-11 15:02:35 / (vbkhwCeAE...)

Client: Now, let&#39;s rollback!
Caretaker: Restoring state to: 2023-08-11 15:02:35 / (vbkhwCeAE...)
Originator: My state has changed to: vbkhwCeAEQBpLwQLlhmpcvUnwzxVnT

Client: Once more!
Caretaker: Restoring state to: 2023-08-11 15:02:35 / (CHYzYSIWb...)
Originator: My state has changed to: CHYzYSIWbqvWkCzIHOqTyEJWfQlFMn</code></pre>
</figure>

## **Снимок** на других языках программирования

[![Снимок на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Снимок на C#"){.prog-lang-link}
[![Снимок на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Снимок на C++"){.prog-lang-link}
[![Снимок на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Снимок на Go"){.prog-lang-link}
[![Снимок на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Снимок на Java"){.prog-lang-link}
[![Снимок на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Снимок на PHP"){.prog-lang-link}
[![Снимок на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Снимок на Python"){.prog-lang-link}
[![Снимок на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Снимок на Rust"){.prog-lang-link}
[![Снимок на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Снимок на Swift"){.prog-lang-link}
[![Снимок на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Снимок на TypeScript"){.prog-lang-link}
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
