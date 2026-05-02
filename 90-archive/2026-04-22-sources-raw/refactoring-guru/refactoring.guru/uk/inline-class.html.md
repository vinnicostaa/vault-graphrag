:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Переміщення функцій між
об\'єктами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Вбудовування класу {#вбудовування-класу .title}

::: aka
Також відомий як: [Inline Class]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Клас майже нічого не робить, ні за що не відповідає, і ніякої
відповідальності для цього класу не планується.
:::

::: after
### Рішення

Перемістіть усі фічі з цього класу в інший.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Inline Class -
Before](../images/refactoring/diagrams/Inline%20Class%20-%20Beforebf8b.png?id=717d80baaa902d09acbaa55fd0539638){width="509"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Inline Class -
After](../images/refactoring/diagrams/Inline%20Class%20-%20After5bcb.png?id=7cb7db0fe2ab0d17f067d411f9e98f15){width="209"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Часто цей рефакторинг стає наслідком недавнього «перенесення» частини
фіч з одного класу в інший, після чого від початкового класу мало що
залишилося.

### Переваги

-   Менше даремних класів --- більше вільної оперативної пам'яті, у тому
    числі, і у вас в голові.

### Порядок рефакторингу

1.  Створіть в класі-одержувачі публічні поля і методи, такі ж, як в
    класі-донорі. Методи повинні звертатися до аналогічних методів
    класу-донору.

2.  Замініть усі звернення до класу-донору зверненнями до полів і
    методів класу-одержувача.

3.  Саме час протестувати програму і переконатися, що під час роботи не
    було допущено ніяких помилок. Якщо тести показали, що все працює
    так, як потрібно, починаємо виконувати [переміщення
    методу](move-method.html) і [переміщення поля](move-field.html) для
    того, щоби повністю перемістити весь функціонал в клас-приймач з
    початкового класу. Продовжуємо робити це, поки початковий клас не
    виявиться зовсім порожнім.

4.  Видаліть початковий клас.

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

::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Анти-рефакторинг

:::: dl
::: dt
[Відокремлення класу](extract-class.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Стрільба дробом](smells/shotgun-surgery.html)
:::
::::

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
:::::::::
:::::::::::::

::: next
#### Читаємо далі

[Приховання делегування []{.fa
.fa-arrow-right}](hide-delegate.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відокремлення класу](extract-class.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
