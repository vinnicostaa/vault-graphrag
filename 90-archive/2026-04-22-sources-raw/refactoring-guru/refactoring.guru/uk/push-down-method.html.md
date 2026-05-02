:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Спуск методу {#спуск-методу .title}

::: aka
Також відомий як: [Push Down Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поведінка, реалізована в суперкласі, використовується тільки одним або
декількома підкласами.
:::

::: after
### Рішення

Перемістіть цю поведінку в підкласи.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Push Down Method -
Before](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Beforea9e3.png?id=ed31b8dcbc3c98fc668ce95cff9f76a8){width="452"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Push Down Method -
After](../images/refactoring/diagrams/Push%20Down%20Method%20-%20Aftere373.png?id=dcf9788fec0c13fb8e60fe494d481529){width="452"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Метод, який планували зробити універсальним для усіх класів, по факту
використовується тільки в одному підкласі. Така ситуація може виникнути,
коли плановані фічи так і не були реалізовані.

Крім того, така ситуація може виникнути після відокремлення (чи
видалення) частини функціональності з ієрархії класів, після якого метод
залишився використовуваним тільки в одному підкласі.

Якщо ви бачите, що метод потрібний більш ніж одному підкласу (але не
всім), можливо, варто створити проміжний підклас і перемістити метод в
нього. Це дозволить уникнути дублювання коду, яке виникло б при спуску
методу в усі підкласи.

### Переваги

-   Покращує зв'язність усередині класів. Метод знаходиться там, де ви
    очікуєте його побачити.

### Порядок рефакторіингу

1.  Оголосіть метод в підкласі і скопіюйте його код з суперкласу.

2.  Видаліть метод з суперкласу.

3.  Знайдіть усі місця, де використовується метод, і переконаєтеся, що
    він викликається з потрібного підкласу.

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
[Підйом методу](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Схожі рефакторинги

:::: dl
::: dt
[Спуск поля](push-down-field.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Витягання підкласу](extract-subclass.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Бореться з запахом

:::: dl
::: dt
[Відмова від спадку](smells/refused-bequest.html)
:::
::::
:::::
:::::::::::::::

::: next
#### Читаємо далі

[Спуск поля []{.fa .fa-arrow-right}](push-down-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Підйом тіла
конструктора](pull-up-constructor-body.html){.btn .btn-default
rel="prev"}
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
