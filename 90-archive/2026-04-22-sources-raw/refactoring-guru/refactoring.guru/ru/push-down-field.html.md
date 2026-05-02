:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Спуск поля {#спуск-поля .title}

::: aka
Также известен как: [Push Down Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поле используется только в некоторых подклассах.
:::

::: after
### Решение

Переместите поле в эти подклассы.
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
После
:::

::: ba-container
![Push Down Field -
After](../images/refactoring/diagrams/Push%20Down%20Field%20-%20After4461.png?id=b48b45a448c27122df61dab2c2451227){width="452"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Поле, которое планировали сделать универсальным для всех классов, по
факту используется только в некоторых подклассах. Такая ситуация может
возникнуть, когда планируемые фичи так и не были реализованы.

Кроме того, такая ситуация может возникнуть после извлечения (или
удаления) части функциональности из иерархии классов.

### Достоинства

-   Улучшает связность внутри классов. Поле находится там, где оно
    реально используется.

-   При перемещении в несколько подклассов одновременно, появляется
    возможность развивать поля независимо друг от друга. Правда, такое
    действие создаёт дублирование кода, поэтому стоит спускать поля,
    только если вы действительно намерены использовать их по-разному.

### Порядок рефакторинга

1.  Объявите поле во всех необходимых подклассах.

2.  Удалите поле из суперкласса.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Подъём поля](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Родственные рефакторинги

:::: dl
::: dt
[Спуск метода](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Помогает рефакторингу

:::: dl
::: dt
[Извлечение подкласса](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Борется с запахом

:::: dl
::: dt
[Отказ от наследства](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Извлечение подкласса []{.fa
.fa-arrow-right}](extract-subclass.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Спуск метода](push-down-method.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
