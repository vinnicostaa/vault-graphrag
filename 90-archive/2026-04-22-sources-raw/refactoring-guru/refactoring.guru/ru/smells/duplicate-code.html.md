:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} /
[Замусориватели](../refactoring/smells/dispensables.html){.type}
:::

# Дублирование кода {#дублирование-кода .title}

::: aka
Также известен как: [Duplicate Code]{style="display:inline-block"}
:::

### Симптомы и признаки

Два фрагмента кода выглядят почти одинаковыми.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-013568.png?id=16fe591195fa50073551852b3d44844e"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-01.png?id=16fe591195fa50073551852b3d44844e 500w,/images/refactoring/content/smells/duplicate-code-01-1.5x.png?id=fc62042bc009d4890e7b1d37fe1d5817 750w,/images/refactoring/content/smells/duplicate-code-01-2x.png?id=9e462bc4bd52927cf45cfc7dbc5907af 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

В большинстве случаев дублирование возникает тогда, когда в проекте
работает несколько человек, причём над разными его частями. Они работают
над похожими задачами, но не знают, что коллега уже написал похожий код,
который можно использовать вместо написания своего.

Встречается и косвенное дублирование, когда конкретные участки кода
отличаются внешне, хотя и выполняют одну и ту же задачу. Такое
дублирование бывает довольно сложно обнаружить и исправить.

В отдельных случаях дублирование создаётся намеренно. Зачастую, в
спешке, когда поджимают сроки сдачи проекта. Начинающий программист
видит в уже написанном коде фрагмент, выглядящий «почти так, как нужно»
и не может устоять перед соблазном просто скопировать код куда-то в
другое место (и так десяток раз).

А в самых запущенных случаях программист просто слишком ленив, чтобы
избавить код от дублирования.

### Лечение

-   Один и тот же участок кода присутствует в двух методах одного и того
    же класса: необходимо применить [извлечение
    метода](../extract-method.html) и вызывать код созданного метода из
    обоих участков.

    <figure class="image">
    <img
    src="../../images/refactoring/content/smells/duplicate-code-020123.png?id=50d92af3defe2c2688f66cde102c9c09"
    style="aspect-ratio: 1.67;"
    srcset="/images/refactoring/content/smells/duplicate-code-02.png?id=50d92af3defe2c2688f66cde102c9c09 500w,/images/refactoring/content/smells/duplicate-code-02-1.5x.png?id=224c1f9b44e12cfc3ef7b39e6230d53c 750w,/images/refactoring/content/smells/duplicate-code-02-2x.png?id=5b9325ca1b0369ec3423808380fa9022 1000w"
    sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
    height="300" />
    </figure>

-   Один и тот же участок кода присутствует в двух подклассах,
    находящихся на одном уровне:

    -   Необходимо применить [извлечение метода](../extract-method.html)
        для обоих классов с последующим [подъёмом
        поля](../pull-up-field.html) для полей, которые используются в
        поднятом методе.

    -   Если общий код находится в конструкторе, следует использовать
        [подъём тела конструктора](../pull-up-constructor-body.html).

    -   Если участки кода похожи, но не совпадают полностью, нужно
        пользоваться [созданием шаблонного
        метода](../form-template-method.html).

    -   Если оба метода делают одно и то же, но с помощью разных
        алгоритмов, можно выбрать более чёткий из этих алгоритмов и
        применить [замещение алгоритма](../substitute-algorithm.html).

-   Дублирующийся код находится в двух разных классах:

    -   Если эти классы не являются частью какой-то иерархии, следует
        использовать [извлечение
        суперкласса](../extract-superclass.html), чтобы создать для
        интересующих классов один суперкласс, содержащий всю общую
        функциональность.

    -   Если создание суперкласса нежелательно или невозможно, следует
        применить [извлечение класса](../extract-class.html) в одном
        классе, а затем использовать новый компонент в другом.

-   Присутсвует череда условных операторов, которые исполняют один и тот
    же код и отличаются только условиями, следует объединить эти
    операторы в один с общим условием с помощью [объединения условных
    операторов](../consolidate-conditional-expression.html), а также
    применить [извлечение метода](../extract-method.html), чтобы вынести
    это условие в отдельный метод с понятным названием.

-   Один и тот же код выполняется во всех ветках условного оператора:
    необходимо вынести одинаковый код за пределы условного оператора с
    помощью [объединения дублирующихся фрагментов в условных
    операторах](../consolidate-duplicate-conditional-fragments.html).

### Выигрыш

-   Объединение дублирующего кода позволяет улучшить структуру кода и
    уменьшить его объём.

-   Это, в свою очередь, ведёт к упрощению и удешевлению поддержки кода
    в будущем.

<figure class="image">
<img
src="../../images/refactoring/content/smells/duplicate-code-038497.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/duplicate-code-03.png?id=bd88b98ff5e5e1b5a4019cb0a50df9f5 500w,/images/refactoring/content/smells/duplicate-code-03-1.5x.png?id=6c3566b366c660b727083df7f491a38d 750w,/images/refactoring/content/smells/duplicate-code-03-2x.png?id=33df6a84eddb7c888f6757d4d80d5e20 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Не стоит трогать, если\...

-   В очень редких случаях объединение двух одинаковых участков кода
    может сделать код менее очевидным и понятным.

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

[Ленивый класс []{.fa .fa-arrow-right}](lazy-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Комментарии](comments.html){.btn .btn-default
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
