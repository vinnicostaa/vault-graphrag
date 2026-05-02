:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Команда](../../command.html){.type} / [Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Команда](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Команда** на Ruby {#команда-на-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Команда** --- это поведенческий паттерн, позволяющий заворачивать
запросы или простые операции в отдельные объекты.

Это позволяет откладывать выполнение команд, выстраивать их в очереди, а
также хранить историю и делать отмену.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Команда ](../../command.html){.btn .btn-lg
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

**Применимость:** Паттерн можно часто встретить в Ruby-коде, особенно
когда нужно откладывать выполнение команд, выстраивать их в очереди, а
также хранить историю и делать отмену.

**Признаки применения паттерна:** Классы команд построены вокруг одного
действия и имеют очень узкий контекст. Объекты команд часто подаются в
обработчики событий элементов GUI. Практически любая реализация отмены
использует принципа команд.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Команда**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--main-rb .anchor} **main.rb:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="ruby"><code># Интерфейс Команды объявляет метод для выполнения команд.
#
# @abstract
class Command
  # @abstract
  def execute
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Некоторые команды способны выполнять простые операции самостоятельно.
class SimpleCommand &lt; Command
  # @param [String] payload
  def initialize(payload)
    @payload = payload
  end

  def execute
    puts &quot;SimpleCommand: See, I can do simple things like printing (#{@payload})&quot;
  end
end

# Но есть и команды, которые делегируют более сложные операции другим объектам,
# называемым «получателями».
class ComplexCommand &lt; Command
  # Сложные команды могут принимать один или несколько объектов-получателей
  # вместе с любыми данными о контексте через конструктор.
  #
  # @param [Receiver] receiver
  # @param [String] a
  # @param [String] b
  def initialize(receiver, a, b)
    @receiver = receiver
    @a = a
    @b = b
  end

  # Команды могут делегировать выполнение любым методам получателя.
  def execute
    print &#39;ComplexCommand: Complex stuff should be done by a receiver object&#39;
    @receiver.do_something(@a)
    @receiver.do_something_else(@b)
  end
end

# Классы Получателей содержат некую важную бизнес-логику. Они умеют выполнять
# все виды операций, связанных с выполнением запроса. Фактически, любой класс
# может выступать Получателем.
class Receiver
  # @param [String] a
  def do_something(a)
    print &quot;\nReceiver: Working on (#{a}.)&quot;
  end

  # @param [String] b
  def do_something_else(b)
    print &quot;\nReceiver: Also working on (#{b}.)&quot;
  end
end

# Отправитель связан с одной или несколькими командами. Он отправляет запрос
# команде.
class Invoker
  # Инициализация команд.

  # @param [Command] command
  def on_start=(command)
    @on_start = command
  end

  # @param [Command] command
  def on_finish=(command)
    @on_finish = command
  end

  # Отправитель не зависит от классов конкретных команд и получателей.
  # Отправитель передаёт запрос получателю косвенно, выполняя команду.
  def do_something_important
    puts &#39;Invoker: Does anybody want something done before I begin?&#39;
    @on_start.execute if @on_start.is_a? Command

    puts &#39;Invoker: ...doing something really important...&#39;

    puts &#39;Invoker: Does anybody want something done after I finish?&#39;
    @on_finish.execute if @on_finish.is_a? Command
  end
end

# Клиентский код может параметризовать отправителя любыми командами.
invoker = Invoker.new
invoker.on_start = SimpleCommand.new(&#39;Say Hi!&#39;)
receiver = Receiver.new
invoker.on_finish = ComplexCommand.new(receiver, &#39;Send email&#39;, &#39;Save report&#39;)

invoker.do_something_important</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

## **Команда** на других языках программирования

[![Команда на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Команда на C#"){.prog-lang-link}
[![Команда на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Команда на C++"){.prog-lang-link}
[![Команда на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Команда на Go"){.prog-lang-link}
[![Команда на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Команда на Java"){.prog-lang-link}
[![Команда на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Команда на PHP"){.prog-lang-link}
[![Команда на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Команда на Python"){.prog-lang-link}
[![Команда на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Команда на Rust"){.prog-lang-link}
[![Команда на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Команда на Swift"){.prog-lang-link}
[![Команда на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Команда на TypeScript"){.prog-lang-link}
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
