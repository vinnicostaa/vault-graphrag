:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Раздувальщики](../refactoring/smells/bloaters.html){.type}
:::

# Длинный список параметров {#длинный-список-параметров .title}

::: aka
Также известен как: [Long Parameter List]{style="display:inline-block"}
:::

### Симптомы и признаки

Количество параметров метода больше трёх-четырёх.

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-015e37.png?id=06fad4adaf485cfaa569e66c20f268eb"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-01.png?id=06fad4adaf485cfaa569e66c20f268eb 500w,/images/refactoring/content/smells/long-parameter-list-01-1.5x.png?id=7e78b50f369819e051b3aa450a183659 750w,/images/refactoring/content/smells/long-parameter-list-01-2x.png?id=d964f68180e89b6312726c7a5719e35d 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Длинный список параметров может появиться после объединения нескольких
вариантов алгоритмов в одном методе. В этом случае может быть создан
длинный список параметров, контролирующих то, какая из вариаций будет
выполнена и как.

Появление длинного списка параметров также может быть связано с попыткой
программиста уменьшить связанность между классами. Например, код
создания конкретных объектов, нужных в методе, переместили из самого
метода в код вызова этого метода, причём созданные объекты передаются в
метод как параметры. Таким образом, оригинальный класс перестал знать о
связях между объектами, и связность уменьшилась.

Но если таких объектов нужно создать несколько, под каждый из них
потребуется свой параметр, что приводит к разрастанию списка параметров.

В длинных списках параметров трудно разбираться, они становятся
противоречивыми и сложными в использовании. Вместо длинного списка
параметров метод может использовать данные своего собственного объекта.
Если всех необходимых данных в текущем объекте нет, в качестве параметра
метода можно передать другой объект, который получит недостающие данные.

### Лечение

-   Если данные, передаваемые в метод, можно получить путём вызова
    метода другого объекта, применяем [замену параметра вызовом
    метода](../replace-parameter-with-method-call.html). Этот объект
    может быть помещён в поле собственного класса либо передан как
    параметр метода.

-   Вместо того чтобы передавать группу данных, полученных из другого
    объекта в качестве параметров, в метод можно передать сам объект,
    используя [передачу всего объекта](../preserve-whole-object.html).

-   Если есть несколько несвязанных элементов данных, иногда их можно
    объединить в один объект-параметр, применив [замену параметров
    объектом](../introduce-parameter-object.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/long-parameter-list-020f28.png?id=7571291fcaea939ed137400cbe0f3c02"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/long-parameter-list-02.png?id=7571291fcaea939ed137400cbe0f3c02 500w,/images/refactoring/content/smells/long-parameter-list-02-1.5x.png?id=f933042e101fee9bb5424c308c62daeb 750w,/images/refactoring/content/smells/long-parameter-list-02-2x.png?id=90514c8d76f4eae7b76439778b82c778 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Повышает читабельность кода, уменьшает его размер.

-   В процессе рефакторинга вы можете обнаружить дублирование кода,
    которое ранее было незаметно.

### Не стоит трогать, если\...

-   Не стоит избавляться от параметров, если при этом появляется
    нежелательная связанность между классами.

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

[Группы данных []{.fa .fa-arrow-right}](data-clumps.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Одержимость элементарными
типами](primitive-obsession.html){.btn .btn-default rel="prev"}
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
