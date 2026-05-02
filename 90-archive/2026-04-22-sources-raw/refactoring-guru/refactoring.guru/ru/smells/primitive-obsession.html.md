:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Раздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Одержимость элементарными типами {#одержимость-элементарными-типами .title}

::: aka
Также известен как: [Primitive Obsession]{style="display:inline-block"}
:::

### Симптомы и признаки

-   Использование элементарных типов вместо маленьких объектов для
    небольших задач (например, валюта, диапазоны, специальные строки для
    телефонных номеров и т. п.)

-   Использование констант для кодирования какой-то информации
    (например, константа `USER_ADMIN_ROLE = 1` для обозначения
    пользователей с ролью администратора).

-   Использование строковых констант в качестве названий полей в
    массивах.

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-0177ec.png?id=73e1c5b08ea68a7ad7a66832644e6696"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-01.png?id=73e1c5b08ea68a7ad7a66832644e6696 500w,/images/refactoring/content/smells/primitive-obsession-01-1.5x.png?id=128163a863560b18aa19640c18468bfb 750w,/images/refactoring/content/smells/primitive-obsession-01-2x.png?id=e13be298e4b8d9d4a987972dfc777f4b 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Как и большинство других запахов, этот начинается с маленькой слабости.
Программисту понадобилось поле для хранения каких-то данных. Он подумал,
что создать поле элементарного типа куда проще, чем заводить новый
класс. Это и было сделано. Потом понадобилось другое поле, и оно было
добавлено схожим образом. Не успели оглянуться, как класс уже разросся
до грандиозных размеров.

Примитивные типы нередко используются для «симуляции» типов. Это когда
вместо отдельного типа данных вы имеете набор чисел или строк, который
составляет список допустимых значений для какой-то сущности. Зачастую
этим конкретным числам и строкам даются понятные имена с помощью
констант, что и является причиной их широкого распространения.

Ещё одним плохим способом использования примитивных типов является
«симуляция» полей. При этом класс содержит большой массив разнообразных
данных, а в роли индексов массива для получения этих данных используются
строковые константы, заданные в классе.

### Лечение

-   Если вы имеете множество разнообразных полей примитивных типов,
    возможно, некоторые из них можно логически сгруппировать и перенести
    в свой собственный класс. Ещё лучше, если в этот класс вы сможете
    перенести и поведения, связанные с этими данными. Справиться с этой
    проблемой поможет [замена значения данных
    объектом](../replace-data-value-with-object.html).

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/primitive-obsession-02edeb.png?id=69dfd06eec8d6053e9d88c03ecc78044"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/primitive-obsession-02.png?id=69dfd06eec8d6053e9d88c03ecc78044 500w,/images/refactoring/content/smells/primitive-obsession-02-1.5x.png?id=6531a2995361be154aa01d81aa9340a0 750w,/images/refactoring/content/smells/primitive-obsession-02-2x.png?id=255bbb1e0b340ce3b62a5898e8edc75a 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Если значения этих примитивных полей используются в параметрах
    методов, используйте [замену параметров
    объектом](../introduce-parameter-object.html) или [передачу всего
    объекта](../preserve-whole-object.html).

-   В случаях, когда в переменных закодированы какие-то сложные данные,
    используйте [замену кодирования типа
    классом](../replace-type-code-with-class.html), [замену кодирования
    типа подклассами](../replace-type-code-with-subclasses.html) или
    [замену кодирования типа
    состоянием/стратегией](../replace-type-code-with-state-strategy.html).

-   Если среди переменных есть массивы, используйте [замену массива
    объектом](../replace-array-with-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/primitive-obsession-03b918.png?id=ab0a8c7b6188265bb9890424e679af2f"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/primitive-obsession-03.png?id=ab0a8c7b6188265bb9890424e679af2f 500w,/images/refactoring/content/smells/primitive-obsession-03-1.5x.png?id=c3dd77a95422b4727db752c6634cf57f 750w,/images/refactoring/content/smells/primitive-obsession-03-2x.png?id=bec1080bcf2be14eeda69b0090d5d3fb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Повышает гибкость кода ввиду использования объектов вместо
    примитивных типов.

-   Улучшает понимание и организацию кода. Операции над определёнными
    данными теперь собраны в одном месте, и их не надо искать по всему
    коду. Теперь не нужно догадываться, зачем созданы все эти странные
    константы и почему поля содержатся в массиве.

-   Может вскрыть факты дублирования кода.

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

[Длинный список параметров []{.fa
.fa-arrow-right}](long-parameter-list.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Большой класс](large-class.html){.btn
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
