:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Извлечение суперкласса {#извлечение-суперкласса .title}

::: aka
Также известен как: [Extract Superclass]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть два класса с общими полями и методами.
:::

::: after
### Решение

Создайте для них общий суперкласс и перенесите туда одинаковые поля и
методы.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Extract Superclass -
Before](../images/refactoring/diagrams/Extract%20Superclass%20-%20Beforef473.png?id=e8192774aeefddece5b3c1a7a868127d){width="209"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Extract Superclass -
After](../images/refactoring/diagrams/Extract%20Superclass%20-%20After12ec.png?id=1cff46d95be1632df8af715e14ea88c9){width="509"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Одним из видов дублирования кода является наличие двух классов,
выполняющих сходные задачи одинаковым способом или сходные задачи
разными способами. Объекты предоставляют встроенный механизм для
упрощения такой ситуации с помощью наследования. Однако часто общность
оказывается незамеченной до тех пор, пока не будут созданы какие-то
классы, и тогда появляется необходимость создавать структуру
наследования позднее.

### Достоинства

-   Убирает дублирование кода. Общие поля и методы теперь «живут» только
    в одном месте.

### Когда нельзя применить

-   Вы не можете применить этот рефакторинг к классам, которые уже имеют
    суперкласс.

### Порядок рефакторинга

1.  Создайте абстрактный суперкласс.

2.  Используйте [подъём поля](pull-up-field.html), [подъём
    метода](pull-up-method.html) и [подъём тела
    конструктора](pull-up-constructor-body.html) для перемещения общей
    функциональности в суперкласс. Лучше начинать с полей, так как
    помимо общих полей, вам нужно будет перенести те из них, которые
    используются в общих методах.

3.  Стоит поискать места в клиентском коде, в которых можно заменить
    использование подклассов вашим общим классом (например, в
    объявлениях типов).

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение интерфейса](extract-interface.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Извлечение интерфейса []{.fa
.fa-arrow-right}](extract-interface.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Извлечение
подкласса](extract-subclass.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
