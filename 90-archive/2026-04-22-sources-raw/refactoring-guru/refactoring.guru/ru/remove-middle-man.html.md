:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Перемещение функций
между
объектами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Удаление посредника {#удаление-посредника .title}

::: aka
Также известен как: [Remove Middle Man]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Класс имеет слишком много методов, которые просто делегируют работу
другим объектам.
:::

::: after
### Решение

Удалите эти методы и заставьте клиента вызывать конечные методы
напрямую.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Remove Middle Man -
Before](../images/refactoring/diagrams/Remove%20Middle%20Man%20-%20Before921c.png?id=f51110f3e0d4423b3f9088e92fc3dce4){width="153"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Remove Middle Man -
After](../images/refactoring/diagrams/Remove%20Middle%20Man%20-%20After0412.png?id=f7de1016e76545f7c51af09463ce5f4c){width="415"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

В этом рефакторинге мы будем использовать названия из [сокрытия
делегирования](hide-delegate.html), а именно:

-   *Сервер* --- это объект, к которому клиент имеет непосредственный
    доступ.

-   *Делегат* --- это конечный объект, который содержит
    функциональность, нужную клиенту.

Существует два вида проблем:

1.  *Класс-сервер* ничего не делает сам по себе, создавая бесполезную
    сложность. В этом случае стоит задуматься, нужен ли этот класс
    вообще.

2.  Каждый раз, когда в *делегате* появляется новая фича, для нее нужно
    создавать делегирующий метод в *классе-сервере*. Это бывает накладно
    при большом количестве изменений.

### Порядок рефакторинга

1.  Создайте геттер для доступа к объекту *класса-делегата* из объекта
    *класса-сервера*.

2.  Замените вызовы делегирующих методов *класса-сервера* прямыми
    вызовами методов *класса-делегата*.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Сокрытие делегирования](hide-delegate.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Посредник](smells/middle-man.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Введение внешнего метода []{.fa
.fa-arrow-right}](introduce-foreign-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Сокрытие делегирования](hide-delegate.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
