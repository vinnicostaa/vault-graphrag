:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Перемещение функций
между
объектами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Сокрытие делегирования {#сокрытие-делегирования .title}

::: aka
Также известен как: [Hide Delegate]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Клиент получает объект B из поля или метода объекта А. Затем клиент
вызывает какой-то метод объекта B.
:::

::: after
### Решение

Создайте новый метод в классе А, который бы делегировал вызов объекту B.
Таким образом, клиент перестанет знать о классе В и зависеть от него.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Hide Delegate -
Before](../images/refactoring/diagrams/Hide%20Delegate%20-%20Before0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Hide Delegate -
After](../images/refactoring/diagrams/Hide%20Delegate%20-%20After921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Для начала следует определиться с названиями:

-   *Сервер* --- это объект, к которому клиент имеет непосредственный
    доступ.

-   *Делегат* --- это конечный объект, который содержит
    функциональность, нужную клиенту.

Цепочка вызовов появляется тогда, когда клиент запрашивает у одного
объекта другой, потом второй объект запрашивает еще один и т. д. Такие
последовательности вызовов означают, что клиент связан с навигацией по
структуре классов. Любые изменения промежуточных связей означают
необходимость модификации клиента.

### Достоинства

-   Скрывает делегирование от клиента. Чем меньше клиентский код знает
    подробностей о связях между объектами, тем проще будет впоследствии
    вносить изменения в программу.

### Недостатки

-   Если требуется создать слишком много делегирующих методов,
    *класс-сервер* рискует превратиться в лишнее промежуточное звено и
    привести к запашку [посредник](smells/middle-man.html).

### Порядок рефакторинга

1.  Для каждого метода *класса-делегата*, вызываемого клиентом, нужно
    создать метод в *классе-сервере*, который бы делегировал вызов
    *классу-делегату*.

2.  Измените код клиента так, чтобы он вызывал методы *класса-сервера*.

3.  Если после всех изменений клиент больше не нуждается в
    *классе-делегате*, можно убрать метод доступа к *классу-делегату* из
    *класса-сервера* (тот метод, который использовался изначально для
    получения *класса-делегата*).

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Удаление посредника](remove-middle-man.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Цепочка вызовов](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Неуместная близость](smells/inappropriate-intimacy.html)
:::
::::
:::::::
:::::::::::

::: next
#### Читаем дальше

[Удаление посредника []{.fa
.fa-arrow-right}](remove-middle-man.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Встраивание класса](inline-class.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот рефакторинг --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
