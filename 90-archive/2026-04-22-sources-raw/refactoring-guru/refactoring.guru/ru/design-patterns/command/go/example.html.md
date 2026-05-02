:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
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
**Команда** --- это поведенческий паттерн, позволяющий заворачивать
запросы или простые операции в отдельные объекты.

Это позволяет откладывать выполнение команд, выстраивать их в очереди, а
также хранить историю и делать отмену.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Команда ](../../command.html){.btn .btn-lg
.btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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

## Концептуальный пример {#example-0-title}

Давайте рассмотрим паттерн Команда на примере телевизора. ТV может быть
включен двумя способами:

-   кнопка ВКЛ на пульте дистанционного управления;
-   кнопка ВКЛ на самом телевизоре.

Мы можем начать с реализации объекта команды ВКЛ с телевизором в роли
получателя. Когда на эту команду вызывается метод `execute`, она, в свою
очередь, вызывает функцию `TV.on`. Вышеуказанное определяет вызывающий
объект. На самом деле мы будем иметь два вызывающих объекта: пульт и сам
ТВ. Оба будут содержать объект команды ВКЛ.

Заметьте, что мы обернули один и тот же запрос в несколько вызывающих
объектов. Это же можно делать и с другими командами. Преимуществом
создания отдельных объектов команд является отделение логики
пользовательского интерфейса от внутренней бизнес-логики. Нет нужды
разрабатывать отдельные исполнители для каждого вызывающего объекта --
сама команда содержит всю информацию, необходимую для ее исполнения.
Соответственно, ее можно использовать для отсроченного выполнения
задачи.

#### []{#example-0--button-go .anchor} **button.go:** Отправитель

<figure class="code">
<pre class="code" lang="go"><code>package main

type Button struct {
    command Command
}

func (b *Button) press() {
    b.command.execute()
}</code></pre>
</figure>

#### []{#example-0--command-go .anchor} **command.go:** Интерфейс команды

<figure class="code">
<pre class="code" lang="go"><code>package main

type Command interface {
    execute()
}</code></pre>
</figure>

#### []{#example-0--onCommand-go .anchor} **onCommand.go:** Конкретная команда

<figure class="code">
<pre class="code" lang="go"><code>package main

type OnCommand struct {
    device Device
}

func (c *OnCommand) execute() {
    c.device.on()
}</code></pre>
</figure>

#### []{#example-0--offCommand-go .anchor} **offCommand.go:** Конкретная команда

<figure class="code">
<pre class="code" lang="go"><code>package main

type OffCommand struct {
    device Device
}

func (c *OffCommand) execute() {
    c.device.off()
}</code></pre>
</figure>

#### []{#example-0--device-go .anchor} **device.go:** Интерфейс получателя

<figure class="code">
<pre class="code" lang="go"><code>package main

type Device interface {
    on()
    off()
}</code></pre>
</figure>

#### []{#example-0--tv-go .anchor} **tv.go:** Конкретный получатель

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

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Turning tv on
Turning tv off</code></pre>
</figure>

::: next
#### Читаем дальше

[Итератор на Go []{.fa
.fa-arrow-right}](../../iterator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Цепочка обязанностей на
Go](../../chain-of-responsibility/go/example.html){.btn .btn-default
rel="prev"}
:::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
