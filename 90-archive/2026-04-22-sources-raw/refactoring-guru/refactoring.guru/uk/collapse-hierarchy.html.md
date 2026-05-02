::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Задачі
узагальнення
об\'єктів](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Згортання ієрархії {#згортання-ієрархії .title}

::: aka
Також відомий як: [Collapse Hierarchy]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є деяка ієрархія класів, в якій підклас мало чим відрізняється від
суперкласу.
:::

::: after
### Рішення

Злийте підклас і суперклас воєдино.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Collapse Hierarchy -
Before](../images/refactoring/diagrams/Collapse%20Hierarchy%20-%20Beforea922.png?id=e95d97b3a9d564fbdba5ec0b76748f88){width="152"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Collapse Hierarchy -
After](../images/refactoring/diagrams/Collapse%20Hierarchy%20-%20After1803.png?id=db86997e7b68eaac829168048cd02a8b){width="152"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Розвиток програми привів до того, що підклас і суперклас стали дуже мало
відрізнятися один від одного. Якась фіча була прибрана з підкласу,
якийсь метод «переїхав» в суперклас, і ось ви вже маєте два практично
однакових класи.

### Переваги

-   Зменшується складність програми. Менше класів, менше різних речей,
    які треба тримати в голові, менше «рухомих частин», менше
    ймовірність зламати щось при подальших змінах в коді.

-   Навігація за кодом стає простіша, коли методи визначені тільки в
    одному класі. Потрібний метод не доводиться шукати за усією
    ієрархією.

### Коли не слід застосовувати

-   Якщо в ієрархії класів знаходиться більше одного підкласу, то після
    проведення рефакторингу, решта підкласів повинні стати спадкоємцями
    класу, в якому була об'єднана ієрархія.

-   Проте майте на увазі, що це може привести до порушення *принципу
    підстановки Барбари Лісков*. Наприклад, якщо в програмі емуляторі
    міського транспорту невірно згорнути суперклас `Транспорт` в підклас
    `Автомобіль`, клас `Літак` може виявитися спадкоємцем `Автомобіля`,
    а це вже неправильно.

### Порядок рефакторингу

1.  Виберіть, який клас прибрати зручніше: суперклас або підклас.

2.  Використайте [підйом поля](pull-up-field.html) і [підйом
    методу](pull-up-method.html), якщо ви вирішили позбутися від
    підкласу. Використайте [спуск поля](push-down-field.html) і [спуск
    методу](push-down-method.html), якщо прибраний буде суперклас.

3.  Замініть усі використання класу, який буде видалений, класом, в який
    переїжджають поля і методи. Найчастіше це буде код створення класів,
    вказівки типів параметрів і змінних, а також документації в
    коментарях.

4.  Видаліть порожній клас.

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

:::::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

::::: dl
::: dt
[Вбудовування класу](inline-class.html)
:::

::: dd
*Згортання ієрархії* є варіантом [вбудовування
класу](inline-class.html), де всі фічі переїзджають в суперклас або
підклас.
:::
:::::
::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Ледачий клас](smells/lazy-class.html)
:::
::::

:::: dl
::: dt
[Теоретична спільність](smells/speculative-generality.html)
:::
::::
:::::::
::::::::::::

::: next
#### Читаємо далі

[Створення шаблонного методу []{.fa
.fa-arrow-right}](form-template-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відокремлення
інтерфейсу](extract-interface.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::
