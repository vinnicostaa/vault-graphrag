:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Перемещение функций
между
объектами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Встраивание класса {#встраивание-класса .title}

::: aka
Также известен как: [Inline Class]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Класс почти ничего не делает, ни за что не отвечает, и никакой
ответственности для этого класса не планируется.
:::

::: after
### Решение

Переместите все фичи из описанного класса в другой.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Inline Class -
Before](../images/refactoring/diagrams/Inline%20Class%20-%20Beforebf8b.png?id=717d80baaa902d09acbaa55fd0539638){width="509"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Inline Class -
After](../images/refactoring/diagrams/Inline%20Class%20-%20After5bcb.png?id=7cb7db0fe2ab0d17f067d411f9e98f15){width="209"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Часто этот рефакторинг оказывается следствием недавнего «переселения»
части фич класса в другие, после чего от исходного класса мало что
осталось.

### Достоинства

-   Меньше бесполезных классов --- больше свободной оперативной памяти,
    в том числе, и у вас в голове.

### Порядок рефакторинга

1.  Создайте в классе-приёмнике публичные поля и методы, которые есть в
    классе-доноре. Методы должны обращаться к аналогичным методам
    класса-донора.

2.  Замените все обращения к классу-донору обращениями к полям и методам
    класса-приёмника.

3.  Самое время протестировать программу и убедиться, что во время
    работы не было допущено никаких ошибок. Если тесты показали, что все
    работает так, как должно, начинаем использовать [перемещение
    метода](move-method.html) и [перемещение поля](move-field.html) для
    того, чтобы полностью переместить все функциональности в
    класс-приёмник из исходного класса. Продолжаем делать это, пока
    исходный класс не окажется совсем пустым.

4.  Удалите исходный класс.

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

::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Извлечение класса](extract-class.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Стрельба дробью](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Ленивый класс](smells/lazy-class.html)
:::
::::

:::: dl
::: dt
[Теоретическая общность](smells/speculative-generality.html)
:::
::::
:::::::::
:::::::::::::

::: next
#### Читаем дальше

[Сокрытие делегирования []{.fa
.fa-arrow-right}](hide-delegate.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Извлечение класса](extract-class.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
