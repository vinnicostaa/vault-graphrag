:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Опутыватели
связями](../refactoring/smells/couplers.html){.type}
:::

# Завистливые функции {#завистливые-функции .title}

::: aka
Также известен как: [Feature Envy]{style="display:inline-block"}
:::

### Симптомы и признаки

Метод обращается к данным другого объекта чаще, чем к собственным
данным.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-010982.png?id=f520a24562e3f4b7848eca94792c329f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-01.png?id=f520a24562e3f4b7848eca94792c329f 500w,/images/refactoring/content/smells/feature-envy-01-1.5x.png?id=8f02ea7aa51cd62bef6763bcecabec9b 750w,/images/refactoring/content/smells/feature-envy-01-2x.png?id=4329322703e5af5b3ef7faefd17c4750 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Этот запах может появиться после перемещения каких-то полей в класс
данных. В этом случае операции с данными, возможно, также следует
переместить в этот класс.

### Лечение

Следует придерживаться такого правила: то, что изменяется одновременно,
нужно хранить в одном месте. Обычно данные и функции, использующие эти
данные, также изменяются вместе (хотя бывают исключения).

-   Если метод явно следует перенести в другое место, примените
    [перемещение метода](../move-method.html).

-   Если только часть метода обращается к данным другого объекта,
    примените [извлечение метода](../extract-method.html) к этой части.

-   Если метод использует функции нескольких других классов, нужно
    сначала определить, в каком классе находится больше всего
    используемых данных. Затем следует переместить метод в этот класс
    вместе с остальными данными. Как альтернатива, с помощью [извлечения
    метода](../extract-method.html) метод разбивается на несколько
    частей, и они помещаются в разные места в других классах.

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-025fc9.png?id=a90a3545498c7c22e605ceeb1f23d005"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-02.png?id=a90a3545498c7c22e605ceeb1f23d005 500w,/images/refactoring/content/smells/feature-envy-02-1.5x.png?id=56b1c3639436656391810a4122cc488c 750w,/images/refactoring/content/smells/feature-envy-02-2x.png?id=641faecaeb0d422232c0bcc6346352c5 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Уменьшение дублирования кода (если код работы с данными переехал в
    одно общее место).

-   Улучшение организации кода (так как методы работы с данными
    находятся возле этих данных).

<figure class="image">
<img
src="../../images/refactoring/content/smells/feature-envy-03a90e.png?id=ea63eeab9eda1910348d0930c8592780"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/feature-envy-03.png?id=ea63eeab9eda1910348d0930c8592780 500w,/images/refactoring/content/smells/feature-envy-03-1.5x.png?id=9556986c5ae21dd7e2367d44d28e3258 750w,/images/refactoring/content/smells/feature-envy-03-2x.png?id=d8d24af45285db63e68560788e6240bc 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Бывают случаи, когда поведение намеренно отделяется от класса,
    содержащего данные. Чаще всего это делают для того, чтобы иметь
    возможность динамически менять это поведение (паттерны
    [Стратегия](../design-patterns/strategy.html),
    [Посетитель](../design-patterns/visitor.html) и т. д.).

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

[Неуместная близость []{.fa
.fa-arrow-right}](inappropriate-intimacy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Опутыватели
связями](../refactoring/smells/couplers.html){.btn .btn-default
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
