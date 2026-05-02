::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
коду](../smells.html){.type}
:::

# Порушники об\'єктно-орієнтованого дизайну {#порушники-обєктно-орієнтованого-дизайну .title}

Всі ці запахи являють собою неповне або неправильне використання
можливостей об'єктно-орієнтованого програмування.

::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Switch Statements](../../smells/switch-statements.html)
:::

::: dd
У вас є складний оператор `switch` або послідовність `if `-ів.
:::
:::::

::::: dl
::: dt
[Temporary Field](../../smells/temporary-field.html)
:::

::: dd
Тимчасові поля --- це поля, які потрібні об'єкту лише час від часу.
Тільки тоді вони заповнюються якимись значеннями, залишаючись порожніми
решту часу.
:::
:::::

::::: dl
::: dt
[Refused Bequest](../../smells/refused-bequest.html)
:::

::: dd
Якщо підклас використовує лише малу частину успадкованих методів і
властивостей суперкласа, це є ознакою неправильної ієрархії. При цьому
зайві методи можуть просто не використовуватися або бути перевизначеними
і викидати виключення.
:::
:::::

::::: dl
::: dt
[Alternative Classes with Different
Interfaces](../../smells/alternative-classes-with-different-interfaces.html)
:::

::: dd
Два класи виконують однакові функції, але мають різні назви методів.
:::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Оператори switch []{.fa
.fa-arrow-right}](../../smells/switch-statements.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Групи даних](../../smells/data-clumps.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::
::::::::::::::::::::
:::::::::::::::::::::
