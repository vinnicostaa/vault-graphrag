:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Підйом методу {#підйом-методу .title}

::: aka
Також відомий як: [Pull Up Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Підкласи мають методи, які роблять схожу роботу.
:::

::: after
### Рішення

В цьому випадку треба зробити методи ідентичними, а потім перемістити їх
в суперклас.
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
Після
:::

::: ba-container
![Pull Up Method -
After](../images/refactoring/diagrams/Pull%20Up%20Method%20-%20After9072.png?id=57193e5593115571e969abe471dfd9aa){width="452"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Підкласи розвивалися незалежно один від одного. Це привело до створення
однакових (чи дуже схожих) полів і методів.

### Переваги

-   Прибирає дублювання коду. Якщо вам треба внести зміни в метод, краще
    зробити це в одному місці, ніж шукати усі дублікати цього методу в
    підкласах.

-   Також цей рефакторинг можна використати, якщо підклас навіщось
    перевизначає метод суперкласу, але, по суті, робить ту ж роботу.

### Порядок рефакторингу

1.  Обстежте схожі методи в суперкласах. Якщо вони не однакові,
    приведіть їх до одного і того ж виду.

2.  Якщо методи використовують різний набір параметрів, приведіть ці
    параметри до того виду, який ви хочете бачити в суперкласі.

3.  Скопіюйте метод в суперклас. Тут ви можете зіткнутися з тим, що код
    методу використовує поля і методи, які є тільки в підкласах, а тому
    недоступні в суперкласі. Щоб розв'язати цю проблему, вам треба:

    -   Для полів: або [підніміть потрібні поля в
        суперклас](pull-up-field.html), або використайте
        [самоінкапсуляцію поля](self-encapsulate-field.html) для
        створення геттерів і сеттерів в підкласах, а потім оголосіть ці
        геттери абстрактним методом в суперкласі.

    -   Для методів: або [підніміть потрібні методи в
        суперклас](pull-up-method.html), або оголосіть для них
        абстрактні методи в суперкласі (зверніть увагу, ваш клас стане
        абстрактним, якщо не був таким до цього часу).

4.  Видаліть методи в підкласах.

5.  Перевірте місця, в яких викликається метод. Можливо, в деяких з них
    використання підкласу можна замінити суперкласом.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Анти-рефакторинг

:::: dl
::: dt
[Спуск методу](push-down-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Схожі рефакторинги

:::: dl
::: dt
[Підйом поля](pull-up-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Створення шаблонного методу](form-template-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Підйом тіла конструктора []{.fa
.fa-arrow-right}](pull-up-constructor-body.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Підйом поля](pull-up-field.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей рефакторинг є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](refactoring/course.html){.btn .btn-secondary .btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
