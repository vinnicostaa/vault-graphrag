::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Приховання методу {#приховання-методу .title}

::: aka
Також відомий як: [Hide Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Метод не використовується іншими класами або використовується тільки
всередині своєї ієрархії класів.
:::

::: after
### Рішення

Зробіть метод приватним або захищеним.
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
Після
:::

::: ba-container
![Hide Method -
After](../images/refactoring/diagrams/Hide%20Method%20-%20After5926.png?id=2f7f1435a959a0a611778513e5bc4365){width="134"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Дуже часто потреба в прихованні методів отримання та установки значень
виникає в зв'язку з розробкою більш багатого інтерфейсу, що надає
додаткову поведінку. Особливо це проявляється, якщо ви починали з класу
який не мав ніяких методів, крім геттерів та сеттерів.

Під час вбудовування в клас нової поведінки, може виявитися, що у
відкритих геттерах та сеттерах більше немає потреби, і тоді можна їх
приховати. І якщо надалі використовувати лише прямий доступ до поля,
його методи доступу можна взагалі видалити.

### Переваги

-   Приховання методів спрощує еволюційні зміни у вашому коді. Змінюючи
    приватний метод, вам треба буде піклуватися тільки про те, щоб не
    зламати поточний клас, оскільки цей метод не може бути використаний
    десь ще.

-   Роблячи методи приватними, ви підкреслюєте важливість публічного
    інтерфейсу класу, іншими словами, тих методів, які залишилися
    публічними.

### Порядок рефактірингу

1.  Регулярно робіть спроби знайти методи, які можна зробити приватними.
    Статичний аналіз коду і хороше покриття unit-тестами може дуже
    допомогти в цьому.

2.  Робіть кожен метод настільки приватним, наскільки це можливо.

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
### Бореться з запахом

:::: dl
::: dt
[Клас даних](smells/data-class.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Заміна конструктора фабричним методом []{.fa
.fa-arrow-right}](replace-constructor-with-factory-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Видалення
сеттера](remove-setting-method.html){.btn .btn-default rel="prev"}
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
