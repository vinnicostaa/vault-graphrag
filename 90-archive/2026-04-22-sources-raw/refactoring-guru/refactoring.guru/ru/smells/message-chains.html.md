:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Опутыватели
связями](../refactoring/smells/couplers.html){.type}
:::

# Цепочка вызовов {#цепочка-вызовов .title}

::: aka
Также известен как: [Message Chains]{style="display:inline-block"}
:::

### Симптомы и признаки

Вы видите в коде цепочки вызовов вроде такой `$a->b()->c()->d()`

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-01e234.png?id=c290ab1d348b3e6ab500c0b949f3d3f8"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-01.png?id=c290ab1d348b3e6ab500c0b949f3d3f8 500w,/images/refactoring/content/smells/message-chains-01-1.5x.png?id=66d768c6d2df06a8744c861b48b5b404 750w,/images/refactoring/content/smells/message-chains-01-2x.png?id=63332ad44f028e0d60f42d3a56e0280a 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Цепочка вызовов появляется тогда, когда клиент запрашивает у одного
объекта другой, в свою очередь этот объект запрашивает ещё один и т. д.
Такие последовательности вызовов означают, что клиент связан с
навигацией по структуре классов. Любые изменения промежуточных связей
означают необходимость модификации клиента.

### Лечение

-   Для удаления цепочки вызовов применяется приём [сокрытие
    делегирования](../hide-delegate.html).

-   Иногда лучше рассмотреть, для чего используется конечный объект.
    Может быть, имеет смысл использовать [извлечение
    метода](../extract-method.html), чтобы извлечь эту функциональность,
    и передвинуть её в самое начало цепи с помощью [перемещения
    метода](../move-method.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-02cbe3.png?id=d348325f450e592900b1a4a2ed960b53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-02.png?id=d348325f450e592900b1a4a2ed960b53 500w,/images/refactoring/content/smells/message-chains-02-1.5x.png?id=277dfdf5360730a6e4babf0c4732cd29 750w,/images/refactoring/content/smells/message-chains-02-2x.png?id=07214cc9363ca16dcbaab4bb6e1edeef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Может уменьшить связность между классами цепочки.

-   Может уменьшить размер кода.

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-03d2e4.png?id=e651ac11f057e3e2e7c7786fc4051a66"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-03.png?id=e651ac11f057e3e2e7c7786fc4051a66 500w,/images/refactoring/content/smells/message-chains-03-1.5x.png?id=4a3dc90e543359c0f886d46d05b5b1e5 750w,/images/refactoring/content/smells/message-chains-03-2x.png?id=2e0e5bdf1e249a09f9c8e67f01de6bd1 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   Если вы перестараетесь в процессе сокрытия делегирования, в коде
    будет довольно сложно понять, где именно осуществляется конкретная
    работа. Другими словами, появится запах
    [Посредник](middle-man.html).

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

[Посредник []{.fa .fa-arrow-right}](middle-man.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Неуместная
близость](inappropriate-intimacy.html){.btn .btn-default rel="prev"}
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
