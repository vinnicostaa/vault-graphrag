:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Спуск поля {#спуск-поля .title}

::: aka
Також відомий як: [Push Down Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поле використовується тільки в деяких підкласах.
:::

::: after
### Рішення

Перемістіть поле в ці підкласи.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Push Down Field -
Before](../images/refactoring/diagrams/Push%20Down%20Field%20-%20Before4e32.png?id=79ff2461236c616fcfbc5b0ab8abf5d9){width="452"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Push Down Field -
After](../images/refactoring/diagrams/Push%20Down%20Field%20-%20After4461.png?id=b48b45a448c27122df61dab2c2451227){width="452"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Поле, яке планували зробити універсальним для усіх класів, по факту,
використовується тільки в деяких підкласах. Така ситуація може
виникнути, коли плановані фічи так і не були реалізовані.

Крім того, така ситуація може виникнути після відокремлення (чи
видалення) частини функціональності з ієрархії класів.

### Переваги

-   Покращує зв'язність усередині класів. Поле знаходиться там, де воно
    реально використовується.

-   При переміщенні в декілька підкласів одночасно, з'являється
    можливість розвивати поля незалежно одне від одного. Правда, така
    дія створює дублювання коду, тому варто спускати поля, тільки якщо
    ви дійсно маєте намір використовувати їх по-різному.

### Порядок рефакторингу

1.  Оголосіть поле в усіх необхідних підкласах.

2.  Видаліть поле з суперкласу.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Підйом поля](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Схожі рефакторинги

:::: dl
::: dt
[Спуск методу](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Витягання підкласу](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Бореться з запахом

:::: dl
::: dt
[Відмова від спадку](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Витягання підкласу []{.fa .fa-arrow-right}](extract-subclass.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Спуск методу](push-down-method.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей рефакторинг є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](refactoring/course.html){.btn .btn-secondary .btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
