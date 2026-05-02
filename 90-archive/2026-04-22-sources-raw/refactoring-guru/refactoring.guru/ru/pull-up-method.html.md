:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Подъём метода {#подъём-метода .title}

::: aka
Также известен как: [Pull Up Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Подклассы имеют методы, которые делают схожую работу.
:::

::: after
### Решение

В этом случае нужно сделать методы идентичными, а затем переместить их в
суперкласс.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Pull Up Method -
Before](../images/refactoring/diagrams/Pull%20Up%20Method%20-%20Beforee533.png?id=1af4e8b520779e6b72d6d91115f6373f){width="452"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Pull Up Method -
After](../images/refactoring/diagrams/Pull%20Up%20Method%20-%20After9072.png?id=57193e5593115571e969abe471dfd9aa){width="452"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Подклассы развивались независимо друг от друга. Это привело к созданию
одинаковых (или очень похожих) полей и методов.

### Достоинства

-   Убирает дублирование кода. Если вам нужно внести изменения в метод,
    лучше сделать это в одном месте, чем искать все дубликаты этого
    метода в подклассах.

-   Также этот рефакторинг можно использовать и в случае, если подкласс
    зачем-то переопределяет метод суперкласса, но, по сути, делает ту же
    работу.

### Порядок рефакторинга

1.  Обследовать похожие методы в суперклассах. Если они не одинаковы,
    привести их к одному и тому же виду.

2.  Если методы используют разный набор параметров, привести эти
    параметры к тому виду, который вы хотите видеть в суперклассе.

3.  Скопируйте метод в суперкласс. Здесь вы можете столкнуться с тем,
    что код метода использует поля и методы, которые есть только в
    подклассах, а посему недоступны в суперклассе. Чтобы решить эту
    проблему, вам нужно:

    -   Для полей: либо [поднимите нужные поля в
        суперкласс](pull-up-field.html), либо используйте
        [самоинкапсуляцию поля](self-encapsulate-field.html) для
        создания геттеров и сеттеров в подклассах, а затем объявите эти
        геттеры абстрактным методом в суперклассе.

    -   Для методов: либо [поднимите нужные методы в
        суперкласс](pull-up-method.html), либо объявите для них
        абстрактные методы в суперклассе (обратите внимание, ваш класс
        станет абстрактным, если не был таким до этого).

4.  Удалите методы в подклассах.

5.  Проверьте места, в которых вызывается метод. Возможно, в некоторых
    из них использование подкласса можно заменить суперклассом.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Спуск метода](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Родственные рефакторинги

:::: dl
::: dt
[Подъём поля](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Помогает рефакторингу

:::: dl
::: dt
[Создание шаблонного метода](form-template-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаем дальше

[Подъём тела конструктора []{.fa
.fa-arrow-right}](pull-up-constructor-body.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Подъём поля](pull-up-field.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
