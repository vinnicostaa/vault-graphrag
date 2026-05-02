::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Удаление сеттера {#удаление-сеттера .title}

::: aka
Также известен как: [Remove Setting
Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Значение поля должно быть установлено только в момент создания и больше
никогда не меняться.
:::

::: after
### Решение

Удалите методы, устанавливающие значение этого поля.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Remove Setting Method -
Before](../images/refactoring/diagrams/Remove%20Setting%20Method%20-%20Before0679.png?id=6f5c0d77289a8f59a2bb89032d5bc55f){width="190"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Remove Setting Method -
After](../images/refactoring/diagrams/Remove%20Setting%20Method%20-%20Aftercb50.png?id=5e75fe3df982c4da469ddceb96530058){width="190"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Вы хотите сделать значение поля неизменяемым.

### Порядок рефакторинга

1.  Значение поля должно меняться только в конструкторе. Если
    конструктор не содержит параметра для установки значения, нужно его
    добавить.

2.  Найдите все вызовы сеттера.

    -   Если вызов сеттера стоит сразу после вызова конструктора
        текущего класса, переместите его аргумент в вызов конструктора и
        удалите сеттер.

    -   Вызовы сеттера в конструкторе замените на прямой доступ к полю.

3.  Удалите сеттер.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Помогает рефакторингу

:::: dl
::: dt
[Замена ссылки значением](change-reference-to-value.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Сокрытие метода []{.fa .fa-arrow-right}](hide-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена параметров
объектом](introduce-parameter-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
