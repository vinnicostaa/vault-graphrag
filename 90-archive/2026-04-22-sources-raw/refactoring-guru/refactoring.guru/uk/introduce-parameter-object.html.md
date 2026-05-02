:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна параметрів об\'єктом {#заміна-параметрів-обєктом .title}

::: aka
Також відомий як: [Introduce Parameter
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У ваших методах зустрічається група параметрів, що повторюється.
:::

::: after
### Рішення

Замініть ці параметри об'єктом.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Introduce Parameter Object -
Before](../images/refactoring/diagrams/Introduce%20Parameter%20Object%20-%20Before9531.png?id=023aa35781e4c8d16e0faf7172646d1e){width="359"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Introduce Parameter Object -
After](../images/refactoring/diagrams/Introduce%20Parameter%20Object%20-%20After803b.png?id=b5ed29b5678759399a83c229b1837730){width="359"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Однакові групи параметрів частенько зустрічаються не в єдиному методі.
Це призводить до дублювання коду, як самих параметрів, так і частих
операцій над ними. Як тільки ви зведете параметри в одному класі, ви
зможете перемістити туди і методи обробки цих даних, очистивши від цього
коду інші методи.

### Переваги

-   Покращує читабельність коду. Замість пачки різноманітних параметрів
    ви бачите один об'єкт із зрозумілою назвою.

-   Однакові групи параметрів тут і там створюють особливий рід
    дублювання коду, при якому начебто не відбувається виклику
    ідентичного коду, але весь час зустрічаються однакові групи
    параметрів і аргументів.

### Недоліки

-   Якщо ви перемістили в новий клас лише дані і не плануєте переміщати
    в нього ніяких операцій над цими даними --- це може тхнути [класами
    даних](smells/data-class.html).

### Порядок рефакторингу

1.  Створіть новий клас, який представлятиме вашу групу параметрів.
    Зробіть так, щоб дані об'єктів цього класу не можна було змінити
    після створення (make classes immutable).

2.  В метод, до якого застосовується рефакторинг, [додайте новий
    параметр](add-parameter.html), у якому передаватиметься ваш
    об'єкт-параметр. В усіх викликах методу передавайте в цей параметр
    об'єкт, що створюється із старих параметрів методу.

3.  Тепер починайте по одному видаляти старі параметри з методу,
    замінюючи їх в коді полями об'єкта-параметра. Тестуйте програму
    після кожної заміни параметра.

4.  По закінченню оцініть, чи є можливість і сенс перенести якусь
    частину методу (а іноді і весь метод) в клас об'єкта-параметра. Якщо
    так, використайте [переміщення методу](move-method.html) або
    [відокремлення методу](extract-method.html), щоб здійснити
    перенесення.

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
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Передача усього об\'єкта](preserve-whole-object.html)
:::
::::
:::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Довгий список параметрів](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Групи даних](smells/data-clumps.html)
:::
::::

:::: dl
::: dt
[Одержимість елементарними типами](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::
:::::::::::
:::::::::::::::

::: next
#### Читаємо далі

[Видалення сеттера []{.fa
.fa-arrow-right}](remove-setting-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна параметра викликом
методу](replace-parameter-with-method-call.html){.btn .btn-default
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
