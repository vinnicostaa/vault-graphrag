:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} /
[Команда](../../command.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Команда](../../../../images/patterns/cards/command-mini8482.png?id=b149eda017c0583c1e92343b83cfb1eb){srcset="/images/patterns/cards/command-mini-2x.png?id=e5f6332e057f6d352a209da963a8fc54 2x"}
:::

# **Команда** на Go {#команда-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
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

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [button](#example-0--button-go)
:::

::: en-file
 [command](#example-0--command-go)
:::

::: en-file
 [on­Command](#example-0--onCommand-go)
:::

::: en-file
 [off­Command](#example-0--offCommand-go)
:::

::: en-file
 [device](#example-0--device-go)
:::

::: en-file
 [tv](#example-0--tv-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Концептуальний приклад {#example-0-title}

Давайте розглянемо патерн Команда на прикладі телевізора. ТV може бути
увімкнено двома способами:

-   кнопка УВІМК на пульті дистанційного керування;
-   кнопка ВИМК на самому телевізорі.

Ми можемо почати з реалізації об'єкта команди УВІМК з телевізором у ролі
одержувача. Коли на цю команду викликається метод `execute`, вона, у
свою чергу, викликає функцію `TV.on`. Вищевказане визначає об'єкт
виклику. Насправді ми матимемо два об'єкта виклику: пульт і сам ТВ.
Обидва будуть містити об'єкт команди ВКЛ.

Зауважте, що ми обернули один і той же запит в кілька об'єкті виклику.
Це ж саме можна робити і з іншими командами. Перевагою створення окремих
об'єктів команд є відділення логіки користувацького інтерфейсу від
внутрішньої бізнес-логіки. Немає потреби розробляти окремих виконавців
для кожного об'єкта виклику --- сама команда містить всю інформацію,
необхідну для її виконання. Відповідно, її можна використовувати для
відстроченого виконання завдання.

#### []{#example-0--button-go .anchor} **button.go:** Відправник

<figure class="code">
<pre class="code" lang="go"><code>package main

type Button struct {
    command Command
}

func (b *Button) press() {
    b.command.execute()
}</code></pre>
</figure>

#### []{#example-0--command-go .anchor} **command.go:** Інтерфейс команд

<figure class="code">
<pre class="code" lang="go"><code>package main

type Command interface {
    execute()
}</code></pre>
</figure>

#### []{#example-0--onCommand-go .anchor} **onCommand.go:** Конкретна команда

<figure class="code">
<pre class="code" lang="go"><code>package main

type OnCommand struct {
    device Device
}

func (c *OnCommand) execute() {
    c.device.on()
}</code></pre>
</figure>

#### []{#example-0--offCommand-go .anchor} **offCommand.go:** Конкретна команда

<figure class="code">
<pre class="code" lang="go"><code>package main

type OffCommand struct {
    device Device
}

func (c *OffCommand) execute() {
    c.device.off()
}</code></pre>
</figure>

#### []{#example-0--device-go .anchor} **device.go:** Інтерфейс отримувача

<figure class="code">
<pre class="code" lang="go"><code>package main

type Device interface {
    on()
    off()
}</code></pre>
</figure>

#### []{#example-0--tv-go .anchor} **tv.go:** Конкретний отримувач

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Tv struct {
    isRunning bool
}

func (t *Tv) on() {
    t.isRunning = true
    fmt.Println(&quot;Turning tv on&quot;)
}

func (t *Tv) off() {
    t.isRunning = false
    fmt.Println(&quot;Turning tv off&quot;)
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    tv := &amp;Tv{}

    onCommand := &amp;OnCommand{
        device: tv,
    }

    offCommand := &amp;OffCommand{
        device: tv,
    }

    onButton := &amp;Button{
        command: onCommand,
    }
    onButton.press()

    offButton := &amp;Button{
        command: offCommand,
    }
    offButton.press()
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Turning tv on
Turning tv off</code></pre>
</figure>

::: next
#### Читаємо далі

[Ітератор на Go []{.fa
.fa-arrow-right}](../../iterator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ланцюжок обов\'язків на
Go](../../chain-of-responsibility/go/example.html){.btn .btn-default
rel="prev"}
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
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
