:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Команда](../../command.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Команда](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Команда** на Python {#команда-на-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Команда** --- це поведінковий патерн, що дозволяє загортати запити або
прості операції в окремі об'єкти.

Це дозволяє відкладати виконання команд, вибудовувати їх у чергу, а
також зберігати історію і робити скасування.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Команда ](../../command.html){.btn .btn-lg
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
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Складність:** []{.fa-stars}

**Популярність:** []{.fa-stars}

**Застосування:** Патерн часто зустрічається в Python-коді, особливо
якщо потрібно відкладати виконання команд, вибудовувати їх у чергу, а
також зберігати історію та робити скасування.

**Ознаки застосування патерна:** Класи команд побудовані навколо однієї
дії та мають дуже вузький контекст. Об'єкти команд часто подаються в
обробники подій елементів GUI. Практично будь-яка реалізація скасування
використовує принцип команд.
:::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Цей приклад показує структуру патерна **Команда**, а саме --- з яких
класів він складається, які ролі ці класи виконують і як вони
взаємодіють один з одним.

#### []{#example-0--main-py .anchor} **main.py:** Приклад структури патерна

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod


class Command(ABC):
    &quot;&quot;&quot;
    The Command interface declares a method for executing a command.
    &quot;&quot;&quot;

    @abstractmethod
    def execute(self) -&gt; None:
        pass


class SimpleCommand(Command):
    &quot;&quot;&quot;
    Some commands can implement simple operations on their own.
    &quot;&quot;&quot;

    def __init__(self, payload: str) -&gt; None:
        self._payload = payload

    def execute(self) -&gt; None:
        print(f&quot;SimpleCommand: See, I can do simple things like printing&quot;
              f&quot;({self._payload})&quot;)


class ComplexCommand(Command):
    &quot;&quot;&quot;
    However, some commands can delegate more complex operations to other
    objects, called &quot;receivers.&quot;
    &quot;&quot;&quot;

    def __init__(self, receiver: Receiver, a: str, b: str) -&gt; None:
        &quot;&quot;&quot;
        Complex commands can accept one or several receiver objects along with
        any context data via the constructor.
        &quot;&quot;&quot;

        self._receiver = receiver
        self._a = a
        self._b = b

    def execute(self) -&gt; None:
        &quot;&quot;&quot;
        Commands can delegate to any methods of a receiver.
        &quot;&quot;&quot;

        print(&quot;ComplexCommand: Complex stuff should be done by a receiver object&quot;, end=&quot;&quot;)
        self._receiver.do_something(self._a)
        self._receiver.do_something_else(self._b)


class Receiver:
    &quot;&quot;&quot;
    The Receiver classes contain some important business logic. They know how to
    perform all kinds of operations, associated with carrying out a request. In
    fact, any class may serve as a Receiver.
    &quot;&quot;&quot;

    def do_something(self, a: str) -&gt; None:
        print(f&quot;\nReceiver: Working on ({a}.)&quot;, end=&quot;&quot;)

    def do_something_else(self, b: str) -&gt; None:
        print(f&quot;\nReceiver: Also working on ({b}.)&quot;, end=&quot;&quot;)


class Invoker:
    &quot;&quot;&quot;
    The Invoker is associated with one or several commands. It sends a request
    to the command.
    &quot;&quot;&quot;

    _on_start = None
    _on_finish = None

    &quot;&quot;&quot;
    Initialize commands.
    &quot;&quot;&quot;

    def set_on_start(self, command: Command):
        self._on_start = command

    def set_on_finish(self, command: Command):
        self._on_finish = command

    def do_something_important(self) -&gt; None:
        &quot;&quot;&quot;
        The Invoker does not depend on concrete command or receiver classes. The
        Invoker passes a request to a receiver indirectly, by executing a
        command.
        &quot;&quot;&quot;

        print(&quot;Invoker: Does anybody want something done before I begin?&quot;)
        if isinstance(self._on_start, Command):
            self._on_start.execute()

        print(&quot;Invoker: ...doing something really important...&quot;)

        print(&quot;Invoker: Does anybody want something done after I finish?&quot;)
        if isinstance(self._on_finish, Command):
            self._on_finish.execute()


if __name__ == &quot;__main__&quot;:
    &quot;&quot;&quot;
    The client code can parameterize an invoker with any commands.
    &quot;&quot;&quot;

    invoker = Invoker()
    invoker.set_on_start(SimpleCommand(&quot;Say Hi!&quot;))
    receiver = Receiver()
    invoker.set_on_finish(ComplexCommand(
        receiver, &quot;Send email&quot;, &quot;Save report&quot;))

    invoker.do_something_important()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Invoker: Does anybody want something done before I begin?
SimpleCommand: See, I can do simple things like printing (Say Hi!)
Invoker: ...doing something really important...
Invoker: Does anybody want something done after I finish?
ComplexCommand: Complex stuff should be done by a receiver object
Receiver: Working on (Send email.)
Receiver: Also working on (Save report.)</code></pre>
</figure>

::: next
#### Читаємо далі

[Ітератор на Python []{.fa
.fa-arrow-right}](../../iterator/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ланцюжок обов\'язків на
Python](../../chain-of-responsibility/python/example.html){.btn
.btn-default rel="prev"}
:::

## **Команда** іншими мовами програмування

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
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Команда на Ruby"){.prog-lang-link}
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
