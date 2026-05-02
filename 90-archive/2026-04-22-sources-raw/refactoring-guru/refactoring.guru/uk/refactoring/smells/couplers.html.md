::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
коду](../smells.html){.type}
:::

# Заплутувальники зв\'язками {#заплутувальники-звязками .title}

Всі запахи з цієї групи призводять до надлишкової зв'язаності між
класами, або показують, що буває якщо тісна зв'язаність заміщується
постійним делегуванням.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Feature Envy](../../smells/feature-envy.html)
:::

::: dd
Метод звертається до даних іншого об'єкта частіше, ніж до власних даних.
:::
:::::

::::: dl
::: dt
[Inappropriate Intimacy](../../smells/inappropriate-intimacy.html)
:::

::: dd
Один клас використовує службові поля і методи іншого класу.
:::
:::::

::::: dl
::: dt
[Message Chains](../../smells/message-chains.html)
:::

::: dd
Ви бачите в коді ланцюжок викликів на зразок такого `$a->b()->c()->d()`
:::
:::::

::::: dl
::: dt
[Middle Man](../../smells/middle-man.html)
:::

::: dd
Якщо клас виконує тільки одну дію --- делегує роботу іншому класу -
варто замислитись, навіщо він взагалі існує.
:::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Заздрісні функції []{.fa
.fa-arrow-right}](../../smells/feature-envy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Теоретична
спільність](../../smells/speculative-generality.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
