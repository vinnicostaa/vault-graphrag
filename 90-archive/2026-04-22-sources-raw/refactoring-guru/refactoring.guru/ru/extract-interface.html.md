::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Извлечение интерфейса {#извлечение-интерфейса .title}

::: aka
Также известен как: [Extract Interface]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Несколько клиентов пользуются одной и той же частью интерфейса класса.
Либо в двух классах часть интерфейса оказалась общей.
:::

::: after
### Решение

Выделите эту общую часть в свой собственный интерфейс.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Extract Interface -
Before](../images/refactoring/diagrams/Extract%20Interface%20-%20Before543f.png?id=fc2bba34908c16e2b86c5c9cc8c424b1){width="209"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Extract Interface -
After](../images/refactoring/diagrams/Extract%20Interface%20-%20After2e06.png?id=2692cc5b979184941c55080bc89226b4){width="209"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

1.  Интерфейсы бывают кстати, когда один и тот же класс может отыгрывать
    различные роли в различных ситуациях. Используйте [извлечение
    интерфейса](extract-interface.html), чтобы явно обозначить каждую из
    ролей.

2.  Ещё один удобный случай возникает, когда требуется описать операции,
    которые класс выполняет на своём сервере. Если в будущем
    предполагается разрешить использование серверов нескольких видов,
    все они должны реализовывать этот интерфейс.

### Полезные факты

Есть некоторое сходство между [извлечением
суперкласса](extract-superclass.html) и [извлечением
интерфейса](extract-interface.html).

Извлечение интерфейса позволяет выделять только общие интерфейсы, но не
общий код. Другими словами, если в классах находится [дублирующий
код](smells/duplicate-code.html), то, применив извлечение интерфейса, вы
никак не избавитесь от этого дублирования.

Тем не менее, эту проблему можно уменьшить, применив [извлечение
класса](extract-class.html) для помещения поведения, содержащего
дублирование в отдельный компонент и делегирования ему всей работы. В
случае если объем общего поведения окажется довольно большим, всегда
можно применить [извлечение суперкласса](extract-superclass.html).
Конечно, это даже проще, но помните, что при этом вы получаете только
один родительский класс.

### Порядок рефакторинга

1.  Создайте пустой интерфейс.

2.  Объявите общие операции в интерфейсе.

3.  Объявите нужные классы как реализующие этот интерфейс.

4.  Измените объявление типов в клиентском коде так, чтобы они
    использовали новый интерфейс.

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
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение суперкласса](extract-superclass.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Свёртывание иерархии []{.fa
.fa-arrow-right}](collapse-hierarchy.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Извлечение
суперкласса](extract-superclass.html){.btn .btn-default rel="prev"}
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
