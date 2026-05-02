::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Переміщення функцій між
об\'єктами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Переміщення поля {#переміщення-поля .title}

::: aka
Також відомий як: [Move Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поле використовується в іншому класі більше, ніж у власному.
:::

::: after
### Рішення

Створіть поле в новому класі і перенаправляйте до нього всіх
користувачів старого поля.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Move Field -
Before](../images/refactoring/diagrams/Move%20Field%20-%20Before2fe2.png?id=b58c81b01a0c4ef8659f92cc64fa51a8){width="134"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Move Field -
After](../images/refactoring/diagrams/Move%20Field%20-%20After4c93.png?id=d7c21af94ec9df17575373bae745e96e){width="134"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Часто поля переносяться як частини [відокремлення одного класу з
іншого](extract-class.html). Вирішити, в якому з класів повинне
залишитися поле, буває непросто. Проте, у нас є непоганий рецепт ---
**поле має бути там, де знаходяться методи, які його використовують**
(або там, де цих методів більше).

Це правило допоможе вам і в інших випадках, коли поле просто знаходиться
не там, де треба.

### Порядок рефакторингу

1.  Якщо поле публічне, вам буде набагато простіше виконати рефакторинг,
    якщо ви зробите його приватним і надасте публічні методи доступу
    (для цього можна використати рефакторинг [інкапсуляція
    поля](encapsulate-field.html)).

2.  Створіть таке ж поле з методами доступу в класі-одержувачі.

3.  Визначте, як ви звертатиметеся до класу-одержувача. Цілком можливо,
    у вас вже є поле або метод, які повертають відповідний об'єкт. Якщо
    ні --- треба буде написати новий метод або поле, в якому б
    зберігався об'єкт класу-одержувача.

4.  Замініть усі звернення до старого поля на відповідні виклики методів
    в класі-одержувачі. Якщо поле не приватне, виконайте це і в
    суперкласі, і в підкласах.

5.  Видаліть поле в класі-донорі.

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

:::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Переміщення методу](move-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Відокремлення класу](extract-class.html)
:::
::::

:::: dl
::: dt
[Вбудовування класу](inline-class.html)
:::
::::
:::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Стрільба дробом](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Паралельні ієрархії
наслідування](smells/parallel-inheritance-hierarchies.html)
:::
::::

:::: dl
::: dt
[Недоречна близькість](smells/inappropriate-intimacy.html)
:::
::::
:::::::::
::::::::::::::::::

::: next
#### Читаємо далі

[Відокремлення класу []{.fa .fa-arrow-right}](extract-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Переміщення методу](move-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
