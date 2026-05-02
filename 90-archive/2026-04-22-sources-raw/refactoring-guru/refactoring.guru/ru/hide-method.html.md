::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Сокрытие метода {#сокрытие-метода .title}

::: aka
Также известен как: [Hide Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод не используется другими классами либо используется только внутри
своей иерархии классов.
:::

::: after
### Решение

Сделайте метод приватным или защищённым.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Hide Method -
Before](../images/refactoring/diagrams/Hide%20Method%20-%20Before37bb.png?id=598219cd5a4b26974b091534b2dc89ae){width="134"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Hide Method -
After](../images/refactoring/diagrams/Hide%20Method%20-%20After5926.png?id=2f7f1435a959a0a611778513e5bc4365){width="134"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Очень часто потребность в сокрытии методов получения и установки
значений возникает в связи с разработкой более богатого интерфейса,
предоставляющего дополнительное поведение. Особенно это проявляется в
случае, когда вы начинали с класса в котором не было никаких методов,
кроме геттеров и сеттеров.

По мере встраивания в класс нового поведения может обнаружиться, что в
открытых геттерах и сеттерах более нет надобности, и тогда можно их
скрыть. А после этого, если использовать только прямой доступ к полю,
его методы доступа можно вообще удалить.

### Достоинства

-   Сокрытие методов упрощает эволюционные изменения в вашем коде.
    Изменяя приватный метод, вам нужно будет заботиться только о том,
    чтобы не сломать текущий класс, так как этот метод не может быть
    использован где-то ещё.

-   Делая методы приватными, вы подчёркиваете важность публичного
    интерфейса класса, другими словами, тех методов, которые остались
    публичными.

### Порядок рефакторинга

1.  Регулярно делайте попытки найти методы, которые можно сделать
    приватными. Статический анализ кода и хорошее покрытие unit-тестами
    может очень помочь в этом.

2.  Делайте каждый метод настолько приватным, насколько это возможно.

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
### Борется с запахом

:::: dl
::: dt
[Класс данных](smells/data-class.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Замена конструктора фабричным методом []{.fa
.fa-arrow-right}](replace-constructor-with-factory-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Удаление
сеттера](remove-setting-method.html){.btn .btn-default rel="prev"}
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
