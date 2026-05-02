::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Заміна посилання значенням {#заміна-посилання-значенням .title}

::: aka
Також відомий як: [Change Reference to
Value]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є об'єкт-посилання, який занадто маленький і незмінний, щоб
виправдати складнощі по управлінню його життєвим циклом.
:::

::: after
### Рішення

Перетворіть його на об'єкт-значення.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Change Reference to Value -
Before](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20Before2033.png?id=584dcba345161853463376cef73ab205){width="377"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Change Reference to Value -
After](../images/refactoring/diagrams/Change%20Reference%20to%20Value%20-%20After05e8.png?id=08ac04b5ee1d4845fb6ebe409a4a6614){width="377"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Спонукати до переходу від посилання до значення можуть незручності, які
виникають при роботі з об'єктом-посиланням.

Об'єктами-посиланнями необхідно якимось чином управляти:

-   кожного разу доводиться просити в об'єкта-сховища потрібний об'єкт;

-   посилання в пам'яті теж можуть виявитися незручними в роботі;

-   працювати з об'єктами-посиланнями, на відміну від об'єктів-значень,
    особливо складно в розподілених і паралельних системах.

Крім того, об'єкти-значення будуть особливо корисні, якщо вам більше
потрібна незмінність об'єктів, ніж можливість зміни їх стану під час
життя об'єкта.

### Переваги

-   Важлива властивість об'єктів-значень полягає в тому, що вони мають
    бути незмінними. При кожному запиті, що повертає значення одного з
    них, повинен виходити однаковий результат. Якщо це так, то не
    виникає проблем за наявності багатьох об'єктів, що представляють
    одну і ту ж річ.

-   Об'єкти-значення набагато простіші в реалізації.

### Недоліки

-   Якщо значення може змінюватись, вам необхідно забезпечити, щоби при
    зміні будь-якого з об'єктів оновлювалися значення в усіх інших, які
    представляють ту ж саму річ. Це настільки обтяжливо, що простіше для
    цього створити об'єкт-посилання.

### Порядок рефакторингу

1.  Забезпечте незмінність об'єкта. Об'єкт не повинен мати сеттерів або
    інших методів, що міняють його стан і дані (у цьому може допомогти
    [видалення сеттера](remove-setting-method.html)). Єдиним місцем, де
    полям об'єкта-значення присвоюються якісь дані, має бути
    конструктор.

2.  Створіть метод порівняння для того, щоби мати можливість порівняти
    два об'єкти-значення.

3.  Перевірте, чи можливо видалити фабричний метод і зробити конструктор
    об'єкта публічним.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Анти-рефакторинг

:::: dl
::: dt
[Заміна значення посиланням](change-value-to-reference.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Заміна поля-масиву об\'єктом []{.fa
.fa-arrow-right}](replace-array-with-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна значення
посиланням](change-value-to-reference.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
