:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Комментарии {#комментарии .title}

::: aka
Также известен как: [Comments]{style="display:inline-block"}
:::

### Симптомы и признаки

Метод содержит множество поясняющих комментариев.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-01cb86.png?id=584958123f3b902e0ad0895d879509b9"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-01.png?id=584958123f3b902e0ad0895d879509b9 500w,/images/refactoring/content/smells/comments-01-1.5x.png?id=c6c70d9f1f12d481d8cabfc726a7fbf7 750w,/images/refactoring/content/smells/comments-01-2x.png?id=15fe22a84b974b19a752ad169ae999ae 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Зачастую комментарии создаются с хорошими намерениями, когда автор и сам
понимает, что его код недостаточно очевиден и понятен. В таких случаях
комментарии играют роль «дезодоранта», т.е. пытаются заглушить «дурной
запах» недостаточно проработанного кода.

> Самый лучший комментарий --- это хорошее название метода или класса.

Если вы чувствуете, что фрагмент кода будет непонятным без комментария,
попробуйте изменить структуру кода так, чтобы любые комментарии стали
излишними.

### Лечение

-   Если комментарий предназначен для того, чтобы объяснить сложное
    выражение, возможно, это выражение лучше разбить на понятные
    подвыражения с помощью [извлечения
    переменной](../extract-variable.html).

-   Если комментарий поясняет целый блок кода, возможно, этот блок можно
    извлечь в отдельный метод с помощью [извлечения
    метода](../extract-method.html). Название нового метода вам, скорей
    всего, подскажет сам комментарий.

-   Если метод уже выделен, но для объяснения его действия по-прежнему
    нужен комментарий, дайте методу новое не требующее комментария
    название. Используйте для этого [переименование
    метода](../rename-method.html).

-   Если требуется описать какие-то правила, касающиеся правильной
    работы метода, попробуйте рефакторинг [введение
    утверждения](../introduce-assertion.html).

### Выигрыш

-   Код становится более очевидным и понятным.

<figure class="image">
<img
src="../../images/refactoring/content/smells/comments-02b499.png?id=266f82bb7081957d409ae690c2c66483"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/comments-02.png?id=266f82bb7081957d409ae690c2c66483 500w,/images/refactoring/content/smells/comments-02-1.5x.png?id=6451f6937bda39e8f101367df8dec883 750w,/images/refactoring/content/smells/comments-02-2x.png?id=57a83d2b705347aa0d0a6d197a1f9d3c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

Иногда комментарии бывают полезными:

-   Те, которые объясняют **почему** что-то выполняется именно таким
    образом.

-   Те, которые объясняют сложные алгоритмы (когда все иные средства
    упростить алгоритм уже были испробованы).

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаем дальше

[Дублирование кода []{.fa .fa-arrow-right}](duplicate-code.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa
.fa-arrow-left} Замусориватели](../refactoring/smells/dispensables.html){.btn
.btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот запах кода --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
