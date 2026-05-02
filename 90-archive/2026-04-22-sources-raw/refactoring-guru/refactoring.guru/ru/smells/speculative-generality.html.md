:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Теоретическая общность {#теоретическая-общность .title}

::: aka
Также известен как: [Speculative
Generality]{style="display:inline-block"}
:::

### Симптомы и признаки

Класс, метод, поле или параметр не используются.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-01d899.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-01.png?id=c804fce5c6c5c34b4d9389fcb2aa60aa 500w,/images/refactoring/content/smells/speculative-generality-01-1.5x.png?id=475f4713f626920ce866b378e350abc8 750w,/images/refactoring/content/smells/speculative-generality-01-2x.png?id=7875dde405dfa433685fa9040da39b5e 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Иногда код создаётся «про запас», чтобы поддерживать какой-то возможный
будущий функционал, который в итоге так и не реализуется. В результате
этот код становится труднее понимать и сопровождать.

### Лечение

-   Для удаления незадействованных абстрактных классов используйте
    [сворачивание иерархии](../collapse-hierarchy.html).

-   Ненужное делегирование функциональности другому классу может быть
    удалено с помощью [встраивания класса](../inline-class.html).

-   От неиспользуемых методов можно избавиться с помощью [встраивания
    метода](../inline-method.html).

-   Методы с неиспользуемыми параметрами должны быть подвергнуты
    [удалению параметров](../remove-parameter.html).

-   Неиспользуемые поля можно просто удалить.

<figure class="image">
<img
src="../../images/refactoring/content/smells/speculative-generality-02bab3.png?id=e9d0e8a6170b6d0d0be9cca44175fe44"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/speculative-generality-02.png?id=e9d0e8a6170b6d0d0be9cca44175fe44 500w,/images/refactoring/content/smells/speculative-generality-02-1.5x.png?id=f9c73fb909b254c3451fe73b2dde4548 750w,/images/refactoring/content/smells/speculative-generality-02-2x.png?id=017235ad164cb220c99f21f201872d29 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Уменьшение размера кода.

-   Упрощение поддержки.

### Не стоит трогать, если\...

-   В случаях, когда вы работаете над фреймворком, создание
    функциональности, не используемой самим фреймворком, вполне
    оправдано. Главное, чтобы она была полезна пользователям фреймворка.

-   Перед удалением элементов, стоит удостовериться, не используются ли
    они в юнит-тестах. Такое бывает, если в тестах необходим способ
    получения какой-то служебной информации класса или осуществления
    каких-то специальных тестовых действий.

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

[Опутыватели связями []{.fa
.fa-arrow-right}](../refactoring/smells/couplers.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Мёртвый код](dead-code.html){.btn .btn-default
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
