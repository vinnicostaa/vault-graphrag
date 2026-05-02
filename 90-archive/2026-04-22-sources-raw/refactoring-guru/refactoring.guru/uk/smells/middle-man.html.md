:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи коду](../refactoring/smells.html){.type} / [Заплутувальники
зв\'язками](../refactoring/smells/couplers.html){.type}
:::

# Посередник {#посередник .title}

::: aka
Також відомий як: [Middle Man]{style="display:inline-block"}
:::

### Симптоми і ознаки

Якщо клас виконує тільки одну дію --- делегує роботу іншому класу -
варто замислитись, навіщо він взагалі існує.

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-01d4ce.png?id=14c65845c4e0cf03e7e9e48108090c98"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-01.png?id=14c65845c4e0cf03e7e9e48108090c98 500w,/images/refactoring/content/smells/middle-man-01-1.5x.png?id=9fd227d0b8d3e082fef36f02df56d907 750w,/images/refactoring/content/smells/middle-man-01-2x.png?id=a1a99f8b475b719d9f894aa613515761 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причини появи

Цей запах може бути результатом фанатичної боротьби з [ланцюжками
викликів](message-chains.html).

Також іноді буває, що все корисне навантаження класу поступово
переноситься в інші класи, в результаті окрім делегуючих методів в ньому
нічого не залишається.

### Лікування

-   Якщо більшу частину методів клас делегує іншому класу, треба
    скористатися [видаленням посередника](../remove-middle-man.html).

### Виграш

-   Зменшення розміру коду.

<figure class="image">
<img
src="../../images/refactoring/content/smells/middle-man-02455a.png?id=f507c0fd9a7bde8df8c22b9027d0a404"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/middle-man-02.png?id=f507c0fd9a7bde8df8c22b9027d0a404 500w,/images/refactoring/content/smells/middle-man-02-1.5x.png?id=4ff17bc08cd0dd6f84473f971873f154 750w,/images/refactoring/content/smells/middle-man-02-2x.png?id=41869f090e8263d46e708778fe64059c 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не варто чіпати, якщо\...

Не видаляйте посередників, які були створені свідомо:

-   Посередник міг бути введений для позбавлення від небажаної
    залежності між класами.

-   Деякі патерни проектування навмисно створюють посередників
    (наприклад [Замісник](../design-patterns/proxy.html) чи
    [Декоратор](../design-patterns/decorator.html)).

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

[Інші запахи []{.fa
.fa-arrow-right}](../refactoring/smells/other.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ланцюжок викликів](message-chains.html){.btn
.btn-default rel="prev"}
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
