:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Мёртвый код {#мёртвый-код .title}

::: aka
Также известен как: [Dead Code]{style="display:inline-block"}
:::

### Симптомы и признаки

Переменная, параметр, поле, метод или класс больше не используются (чаще
всего потому, что устарели).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-01fd46.png?id=418685bee5de933c472c48efcb5b67a0"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-01.png?id=418685bee5de933c472c48efcb5b67a0 500w,/images/refactoring/content/smells/dead-code-01-1.5x.png?id=264efc49269071a3781dd96a1994c249 750w,/images/refactoring/content/smells/dead-code-01-2x.png?id=d9981d853e7e855cf0454fc32aab85a6 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Когда требования к программному продукту изменились, либо были внесены
какие-то корректировки, но чистка старого кода так и не была проведена.

Мёртвый код можно обнаружить и в сложном условном коде, где одна из
веток никогда не может быть исполнена (в виду ошибки или другого
стечения обстоятельств).

### Лечение

Лучше всего мёртвый код обнаруживается при помощи хорошей [среды
разработки
(IDE)](https://en.wikipedia.org/wiki/Integrated_development_environment).

-   Удалите неиспользуемый код и лишние файлы.

-   В случае выявления ненужного класса может быть использовано
    [встраивание класса](../inline-class.html). Если у такого класса
    есть подклассы, то поможет [схлопывание
    иерархии](../collapse-hierarchy.html).

-   Для удаления ненужных параметров используйте [удаление
    параметра](../remove-parameter.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/dead-code-025e37.png?id=b368f23b7cc88340933b761cf2ad1954"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/dead-code-02.png?id=b368f23b7cc88340933b761cf2ad1954 500w,/images/refactoring/content/smells/dead-code-02-1.5x.png?id=c84149b6f42ea2e4ab59b653276627c6 750w,/images/refactoring/content/smells/dead-code-02-2x.png?id=bf78ebf643d890b41b60058d8e67d845 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Уменьшает размер кода.

-   Упрощает его поддержку.

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

[Теоретическая общность []{.fa
.fa-arrow-right}](speculative-generality.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Класс данных](data-class.html){.btn .btn-default
rel="prev"}
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
