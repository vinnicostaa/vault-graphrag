:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Опутыватели
связями](../refactoring/smells/couplers.html){.type}
:::

# Неуместная близость {#неуместная-близость .title}

::: aka
Также известен как: [Inappropriate
Intimacy]{style="display:inline-block"}
:::

### Симптомы и признаки

Один класс использует служебные поля и методы другого класса.

<figure class="image">
<img
src="../../images/refactoring/content/smells/inappropriate-intimacy-017f9d.png?id=31bf185f4ff946f13e28d27d377a4b6c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-01.png?id=31bf185f4ff946f13e28d27d377a4b6c 500w,/images/refactoring/content/smells/inappropriate-intimacy-01-1.5x.png?id=c41452986a03eb3fb0af7992184f8b22 750w,/images/refactoring/content/smells/inappropriate-intimacy-01-2x.png?id=1857242bb9cf7b2ca50327897d256711 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Смотрите внимательно за классами, которые проводят слишком много времени
вместе. Хорошие классы должны знать друг о друге как можно меньше. Такие
классы легче поддерживать и повторно использовать.

### Лечение

-   Самый простой выход --- при помощи [перемещения
    метода](../move-method.html) и [перемещения
    поля](../move-field.html) перенести части одного класса в другой (в
    тот, где они используются). Однако это может сработать только в том
    случае, если оригинальный класс не использует перемещаемые поля и
    методы.

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/inappropriate-intimacy-024036.png?id=3f23c8df6eb8cf91b46e39fa912ff85c"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/inappropriate-intimacy-02.png?id=3f23c8df6eb8cf91b46e39fa912ff85c 500w,/images/refactoring/content/smells/inappropriate-intimacy-02-1.5x.png?id=4c3b0852df877ca82d7fe12d61235b0a 750w,/images/refactoring/content/smells/inappropriate-intimacy-02-2x.png?id=f3ac66384b14f074ed36b6d9c3720b20 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Другим решением является [извлечение зависимых частей в отдельный
    класс](../extract-class.html) и [сокрытие
    делегирования](../hide-delegate.html) к этому классу.

-   Если между классами существует обоюдная зависимость, стоит
    прибегнуть к [замене двунаправленной связи на
    однонаправленную](../change-bidirectional-association-to-unidirectional.html).

-   Если близость возникает между подклассом и родительским классом,
    рассмотрите возможность [замены делегирования
    наследованием](../replace-delegation-with-inheritance.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/inappropriate-intimacy-03b31e.png?id=de33e2285073feaabd1a81cffdcd386c"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/inappropriate-intimacy-03.png?id=de33e2285073feaabd1a81cffdcd386c 500w,/images/refactoring/content/smells/inappropriate-intimacy-03-1.5x.png?id=a2b1cf219e0a4a3056c4e7116b973d85 750w,/images/refactoring/content/smells/inappropriate-intimacy-03-2x.png?id=51bfcec36fb123f146857099555dc478 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает организацию кода.

-   Упрощает техническую поддержку и повторное использование кода.

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

[Цепочка вызовов []{.fa .fa-arrow-right}](message-chains.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Завистливые функции](feature-envy.html){.btn
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
