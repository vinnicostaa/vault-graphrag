:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Раздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Группы данных {#группы-данных .title}

::: aka
Также известен как: [Data Clumps]{style="display:inline-block"}
:::

### Симптомы и признаки

Иногда в разных частях кода встречаются одинаковые группы переменных
(например, параметры подключения к базе данных). Такие группы следует
превращать в самостоятельные классы.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-018c21.png?id=9d8a38ce942720cee728797eba695a2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-01.png?id=9d8a38ce942720cee728797eba695a2f 500w,/images/refactoring/content/smells/data-clumps-01-1.5x.png?id=148bb6bd87a02e66d83bc11bda3ae7e6 750w,/images/refactoring/content/smells/data-clumps-01-2x.png?id=64c7f4113e3c06f10dbec825833fa190 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Появление групп данных является следствием плохой структурированности
программы или программирования методом копирования-вставки.

Чтобы определить группу данных, достаточно удалить одно из значений
данных и проверить, сохранят ли смысл остальные. Если нет, это верный
признак того, что группа переменных напрашивается на объединение их в
объект.

### Лечение

-   Если повторяющиеся данные являются полями какого-то класса,
    используйте [извлечение класса](../extract-class.html) для
    перемещения полей в собственный класс.

-   Если те же группы данных передаются в параметрах методов,
    используйте [замену параметров
    объектом](../introduce-parameter-object.html) чтобы выделить их в
    общий класс.

-   Если некоторые из этих данных передаются в другие методы, подумайте
    о возможности передачи в метод всего объекта данных вместо отдельных
    полей (в этом поможет [передача всего
    объекта](../preserve-whole-object.html)).

-   Посмотрите на код, который использует эти поля. Возможно, имеет
    смысл перенести этот код в класс данных.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-02ec1e.png?id=cfb0a8fa64a983473dd31527e28ae158"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-02.png?id=cfb0a8fa64a983473dd31527e28ae158 500w,/images/refactoring/content/smells/data-clumps-02-1.5x.png?id=2dbf0e8e5960f6797d32abbbc1579d3f 750w,/images/refactoring/content/smells/data-clumps-02-2x.png?id=195f24da42dbd21508aa9ef46a57abba 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает понимание и организацию кода. Операции над определёнными
    данными теперь собраны в одном месте, и их не надо искать по всему
    коду.

-   Уменьшает размер кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/data-clumps-038987.png?id=c170bbdea77b7d4a26947ef328b70adc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/data-clumps-03.png?id=c170bbdea77b7d4a26947ef328b70adc 500w,/images/refactoring/content/smells/data-clumps-03-1.5x.png?id=88875edb97b4047aa9bfb2fd319c0f19 750w,/images/refactoring/content/smells/data-clumps-03-2x.png?id=2b4a70e09a6236715a9bc4bd4663508b 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Передача всего объекта в параметрах метода вместо передачи его
    значений (элементарных типов) может создать нежелательную
    зависимость между двумя классами.

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

[Нарушители объектно-ориентированного дизайна []{.fa
.fa-arrow-right}](../refactoring/smells/oo-abusers.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Длинный список
параметров](long-parameter-list.html){.btn .btn-default rel="prev"}
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
