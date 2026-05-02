::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Подъём поля {#подъём-поля .title}

::: aka
Также известен как: [Pull Up Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Два класса имеют одно и то же поле.
:::

::: after
### Решение

Переместите поле в суперкласс, убрав его из подклассов.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Pull Up Field -
Before](../images/refactoring/diagrams/Pull%20Up%20Field%20-%20Before41a3.png?id=8174b96aa69ce1541271a14b97cc3e33){width="452"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Pull Up Field -
After](../images/refactoring/diagrams/Pull%20Up%20Field%20-%20After12cb.png?id=c97f930072aba9f00c03ba140797de19){width="452"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Подклассы развивались независимо друг от друга. Это привело к созданию
одинаковых (или очень похожих) полей и методов.

### Достоинства

-   Убивает дублирование полей в подклассах.

-   Облегчает дальнейший перенос дублирующих методов из подклассов в
    суперкласс, если они есть.

### Порядок рефакторинга

1.  Проверьте, что оба поля используются для одинаковых нужд в
    подклассах.

2.  Если поля имеют разные названия, дайте им общее имя и замените все
    обращения к полям в существующем коде.

3.  Создайте поле с таким же именем в суперклассе. Обратите внимание на
    то, что если поля были приватные (private), поле в суперклассе
    должно быть защищённым (protected).

4.  Удалите поля из подклассов.

5.  Возможно, имеет смысл использовать [самоинкапсуляцию
    поля](self-encapsulate-field.html) для нового поля, чтобы скрыть его
    за методами доступа.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Спуск поля](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Подъём метода](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
::::::::::::

::: next
#### Читаем дальше

[Подъём метода []{.fa .fa-arrow-right}](pull-up-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
