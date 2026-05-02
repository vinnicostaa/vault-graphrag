:::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Рефакторинг](../../refactoring.html){.type} / [Запахи
коду](../smells.html){.type}
:::

# Ускладнювачі змін {#ускладнювачі-змін .title}

Ці запахи призводять до того, що при необхідності щось поміняти в одному
місці програми, вам доводиться вносити безліч змін в інших місцях. Це
серйозно ускладнює і здорожує розвиток програми.

:::::::::::: {.relations .link-list}
::::: dl
::: dt
[Divergent Change](../../smells/divergent-change.html)
:::

::: dd
При внесенні змін до класу доводиться змінювати багато різноманітних
методів. Наприклад, для додавання нового виду товару вам буде потрібно
змінити методи пошуку, відображення і замовлення товарів.
:::
:::::

::::: dl
::: dt
[Shotgun Surgery](../../smells/shotgun-surgery.html)
:::

::: dd
При виконанні будь-яких модифікацій доводиться вносити безліч дрібних
змін у купу класів.
:::
:::::

::::: dl
::: dt
[Parallel Inheritance
Hierarchies](../../smells/parallel-inheritance-hierarchies.html)
:::

::: dd
Всякий раз при створенні підкласу якогось класу доводиться створювати ще
один підклас для іншого класу.
:::
:::::
::::::::::::

::: next
#### Читаємо далі

[Розбіжні модифікації []{.fa
.fa-arrow-right}](../../smells/divergent-change.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Альтернативні класи з різними
інтерфейсами](../../smells/alternative-classes-with-different-interfaces.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::
:::::::::::::::::
::::::::::::::::::
