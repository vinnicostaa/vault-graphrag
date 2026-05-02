::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Удаление управляющего флага {#удаление-управляющего-флага .title}

::: aka
Также известен как: [Remove Control Flag]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть булевская переменная, которая играет роль управляющего флага
для нескольких булевских выражений.
:::

::: after
### Решение

Используйте `break`, `continue` и `return` вместо этой переменной.
:::
:::::

### Причины рефакторинга

Управляющие флаги пришли к нам из тех «бородатых» дней, когда хорошим
стилем программирования считалось иметь в функции одну входную точку
(строку объявления функции) и одну выходную точку (в самом конце
функции).

В современных языках программирования этот подход устарел, так как у нас
появились специальные операторы для управления ходом программы в циклах
и других сложных конструкциях:

-   `break`: останавливает выполнение цикла;

-   `continue`: останавливает выполнение текущего витка цикла и
    переходит к проверке условия цикла и следующей итерации;

-   `return`: останавливает выполнение всей функции и возвращает её
    результат, если он подан в этом операторе.

### Достоинства

-   Код с управляющим флагом зачастую получается значительно более
    запутанным, чем при использовании операторов управления выполнением.

### Порядок рефакторинга

1.  Найдите присваивание значения управляющему флагу, которое приводит к
    выходу из цикла или текущей итерации.

2.  Замените его на `break`, если это выход из цикла, или `continue`,
    если это выход из итерации, или `return`, если нужно вернуть это
    значение из функции.

3.  Уберите весь остальной код и проверки, связанные с управляющим
    флагом.

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

::: next
#### Читаем дальше

[Замена вложенных условных операторов граничным оператором []{.fa
.fa-arrow-right}](replace-nested-conditional-with-guard-clauses.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Объединение дублирующихся фрагментов в условных
операторах](consolidate-duplicate-conditional-fragments.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::

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
::::::::::::::::::
:::::::::::::::::::
