::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Видалення сеттера {#видалення-сеттера .title}

::: aka
Також відомий як: [Remove Setting Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Значення поля має бути встановлене тільки в момент створення і більше
ніколи не мінятися.
:::

::: after
### Рішення

Видаліть методи, що встановлюють значення цього поля.
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
Після
:::

::: ba-container
![Remove Setting Method -
After](../images/refactoring/diagrams/Remove%20Setting%20Method%20-%20Aftercb50.png?id=5e75fe3df982c4da469ddceb96530058){width="190"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Ви хочете зробити значення поля незмінним.

### Порядок рефакторингу

1.  Значення поля повинне мінятися тільки в конструкторі. Якщо
    конструктор не містить параметра для установки значення, треба його
    додати.

2.  Знайдіть усі виклики сеттера.

    -   Якщо виклик сеттера стоїть відразу після виклику конструктора
        поточного класу, перемістіть його аргумент у виклик конструктора
        і видаліть сеттер.

    -   Виклики сеттера в конструкторі замініть на прямий доступ до
        поля.

3.  Видаліть сеттер.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Заміна посилання значенням](change-reference-to-value.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Приховання методу []{.fa .fa-arrow-right}](hide-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна параметрів
об\'єктом](introduce-parameter-object.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
