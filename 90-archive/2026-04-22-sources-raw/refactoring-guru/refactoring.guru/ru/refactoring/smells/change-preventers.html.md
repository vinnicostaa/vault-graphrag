:::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
кода](../smells.html){.type}
:::

# Утяжелители изменений {#утяжелители-изменений .title}

Эти запахи приводят к тому, что при необходимости что-то поменять в
одном месте программы, вам приходится вносить множество изменений в
других местах. Это серьезно осложняет и удорожает развитие программы.

:::::::::::: {.relations .link-list}
::::: dl
::: dt
[Divergent Change](../../smells/divergent-change.html)
:::

::: dd
При внесении изменений в класс приходится изменять большое число
различных методов. Например, для добавления нового вида товара вам нужно
изменить методы поиска, отображения и заказа товаров.
:::
:::::

::::: dl
::: dt
[Shotgun Surgery](../../smells/shotgun-surgery.html)
:::

::: dd
При выполнении любых модификаций приходится вносить множество мелких
изменений в большое число классов.
:::
:::::

::::: dl
::: dt
[Parallel Inheritance
Hierarchies](../../smells/parallel-inheritance-hierarchies.html)
:::

::: dd
Всякий раз при создании подкласса какого-то класса приходится создавать
ещё один подкласс для другого класса.
:::
:::::
::::::::::::

::: next
#### Читаем дальше

[Расходящиеся модификации []{.fa
.fa-arrow-right}](../../smells/divergent-change.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Альтернативные классы с разными
интерфейсами](../../smells/alternative-classes-with-different-interfaces.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::
:::::::::::::::::
::::::::::::::::::
