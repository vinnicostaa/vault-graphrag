::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
кода](../smells.html){.type}
:::

# Опутыватели связями {#опутыватели-связями .title}

Все запахи из этой группы приводят к избыточной связанности между
классами, либо показывают, что бывает, если тесная связанность
заменяется постоянным делегированием.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Feature Envy](../../smells/feature-envy.html)
:::

::: dd
Метод обращается к данным другого объекта чаще, чем к собственным
данным.
:::
:::::

::::: dl
::: dt
[Inappropriate Intimacy](../../smells/inappropriate-intimacy.html)
:::

::: dd
Один класс использует служебные поля и методы другого класса.
:::
:::::

::::: dl
::: dt
[Message Chains](../../smells/message-chains.html)
:::

::: dd
Вы видите в коде цепочки вызовов вроде такой `$a->b()->c()->d()`
:::
:::::

::::: dl
::: dt
[Middle Man](../../smells/middle-man.html)
:::

::: dd
Если класс выполняет одно действие --- делегирует работу другому
классу --- стоит задуматься, зачем он вообще существует.
:::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Завистливые функции []{.fa
.fa-arrow-right}](../../smells/feature-envy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Теоретическая
общность](../../smells/speculative-generality.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
