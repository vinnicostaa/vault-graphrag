::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Замена ссылки значением {#замена-ссылки-значением .title}

::: aka
Также известен как: [Change Reference to
Value]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть объект-ссылка, который слишком маленький и неизменяемый,
чтобы оправдать сложности по управлению его жизненным циклом.
:::

::: after
### Решение

Превратите его в объект-значение.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Change Reference to Value -
Before](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20Before2033.png?id=584dcba345161853463376cef73ab205){width="377"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Change Reference to Value -
After](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20After05e8.png?id=08ac04b5ee1d4845fb6ebe409a4a6614){width="377"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Побудить к переходу от ссылки к значению могут возникшие при работе с
объектом-ссылкой неудобства.

Объектами-ссылками необходимо некоторым образом управлять:

-   всегда приходится запрашивать у объекта-хранилища нужный объект;

-   ссылки в памяти тоже могут оказаться неудобными в работе;

-   работать с объектами-ссылками, в отличие от объектов-значений,
    особенно сложно в распределённых и параллельных системах.

Кроме того, объекты-значения будут особенно полезны, если вам больше
подходит неизменяемость объектов, чем возможность изменения их состояния
во время жизни объекта.

### Достоинства

-   Важное свойство объектов-значений заключается в том, что они должны
    быть неизменяемыми. При каждом запросе, возвращающем значение одного
    из них, должен получаться одинаковый результат. Если это так, то не
    возникает проблем при наличии многих объектов, представляющих одну и
    ту же вещь.

-   Объекты-значения гораздо проще в реализации.

### Недостатки

-   Если значение изменяемое, вам необходимо обеспечить, чтобы при
    изменении любого из объектов обновлялись значения во всех остальных,
    которые представляют ту же самую сущность. Это настолько
    обременительно, что проще для этого создать объект-ссылку.

### Порядок рефакторинга

1.  Обеспечьте неизменяемость объекта. Объект не должен иметь сеттеров
    или других методов, меняющих его состояние и данные (в этом может
    помочь [удаление сеттера](remove-setting-method.html)). Единственное
    место, где полям объекта-значения присваиваются какие-то данные,
    должен быть конструктор.

2.  Создайте метод сравнения для того, чтобы иметь возможность сравнить
    два объекта-значения.

3.  Проверьте, возможно ли удалить фабричный метод и сделать конструктор
    объекта публичным.

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
### Анти-рефакторинг

:::: dl
::: dt
[Замена значения ссылкой](change-value-to-reference.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Замена поля-массива объектом []{.fa
.fa-arrow-right}](replace-array-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена значения
ссылкой](change-value-to-reference.html){.btn .btn-default rel="prev"}
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
