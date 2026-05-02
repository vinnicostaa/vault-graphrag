:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Заплутувальники
зв\'язками](../refactoring/smells/couplers.html){.type}
:::

# Ланцюжок викликів {#ланцюжок-викликів .title}

::: aka
Також відомий як: [Message Chains]{style="display:inline-block"}
:::

### Симптоми і ознаки

Ви бачите в коді ланцюжок викликів на зразок такого `$a->b()->c()->d()`

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-01e234.png?id=c290ab1d348b3e6ab500c0b949f3d3f8"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-01.png?id=c290ab1d348b3e6ab500c0b949f3d3f8 500w,/images/refactoring/content/smells/message-chains-01-1.5x.png?id=66d768c6d2df06a8744c861b48b5b404 750w,/images/refactoring/content/smells/message-chains-01-2x.png?id=63332ad44f028e0d60f42d3a56e0280a 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Ланцюжок викликів з'являється тоді, коли клієнт запитує один об'єкт щодо
іншого, у свою чергу цей об'єкт запитує ще один і т. д. Такі
послідовності викликів означають, що клієнт є пов'язаний з навігацією по
структурі класів. Будь-які зміни проміжних зв'язків означають, що клієнт
потрібно буде модифікувати.

### Лікування

-   Для видалення ланцюжка викликів застосовується прийом [прихованого
    делегування](../hide-delegate.html).

-   Іноді краще розглянути, для чого використовується кінцевий об'єкт.
    Можливо, має сенс використати [відокремлення
    методу](../extract-method.html), щоб витягнути цю функціональність і
    перенести її в самий початок ланцюга за допомогою [переміщення
    методу](../move-method.html).

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-02cbe3.png?id=d348325f450e592900b1a4a2ed960b53"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-02.png?id=d348325f450e592900b1a4a2ed960b53 500w,/images/refactoring/content/smells/message-chains-02-1.5x.png?id=277dfdf5360730a6e4babf0c4732cd29 750w,/images/refactoring/content/smells/message-chains-02-2x.png?id=07214cc9363ca16dcbaab4bb6e1edeef 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Виграш

-   Зменшується зв'язність між класами ланцюжка.

-   Зменшується розмір коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/message-chains-03d2e4.png?id=e651ac11f057e3e2e7c7786fc4051a66"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/message-chains-03.png?id=e651ac11f057e3e2e7c7786fc4051a66 500w,/images/refactoring/content/smells/message-chains-03-1.5x.png?id=4a3dc90e543359c0f886d46d05b5b1e5 750w,/images/refactoring/content/smells/message-chains-03-2x.png?id=2e0e5bdf1e249a09f9c8e67f01de6bd1 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

-   Якщо ви перестараєтеся в процесі приховування делегування, в коді
    буде досить складно зрозуміти, де саме здійснюється конкретна
    робота. Іншими словами, з'явиться запах
    [Посередник](middle-man.html).

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаємо далі

[Посередник []{.fa .fa-arrow-right}](middle-man.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Недоречна
близькість](inappropriate-intimacy.html){.btn .btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей запах коду є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
