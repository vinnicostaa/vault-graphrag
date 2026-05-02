::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Заміна одностороннього зв\'язку двонаправленим {#заміна-одностороннього-звязку-двонаправленим .title}

::: aka
Також відомий як: [Change Unidirectional Association to
Bidirectional]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є два класи, яким потрібно використовувати фічі один одного, але
між ними існує тільки односторонній зв'язок.
:::

::: after
### Рішення

Додайте бракуючий зв'язок у клас, в якому він відсутній.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Change Unidirectional Association to Bidirectional -
Before](../images/refactoring/diagrams/Change%20Unidirectional%20Association%20to%20Bidirectional%20-%20Before914d.png?id=a31d8472bbb2be95836c8b5e2334ac77){width="396"}
:::
:::::

::::: after
::: ba
Після
:::

::: ba-container
![Change Unidirectional Association to Bidirectional -
After](../images/refactoring/diagrams/Change%20Unidirectional%20Association%20to%20Bidirectional%20-%20After01d6.png?id=058ba44a44610bcb9caa4c23f60ad0bc){width="396"}
:::
:::::
:::::::::
::::::::::

### Причини рефакторингу

Між класами спочатку був односторонній зв'язок. Проте з часом
клієнтському коду знадобився доступ в обидві сторони цього зв'язку.

### Переваги

-   Якщо в класі виникає необхідність в зворотньому зв'язку, її можна
    просто вичислити. Проте, якщо такі обчислення виявляються досить
    складними, краще зберігати зворотній зв'язок.

### Недоліки

-   Двосторонні зв'язки набагато складніші в реалізації і підтримці, ніж
    односторонні.

-   Двосторонні зв'язки роблять класи залежними один від одного. При
    односторонньому зв'язку один з них можна було використовувати окремо
    від іншого.

### Порядок рефакторингу

1.  Додайте поле, яке міститиме зворотний зв'язок.

2.  Вирішіть, який клас буде керуючим. Цей клас повинен містити методи,
    які створюють або оновлюють зв'язок при додаванні або зміні
    елементів, встановлюючи зв'язок у своєму класі, а також викликаючи
    службові методи установки зв'язку в зв'язуваному об'єкті.

3.  Створіть службовий метод установки зв'язку в керуючому класі. Цей
    метод повинен заповнювати поле із зв'язком тим, що йому подають в
    параметрах. Назвіть цей метод так, щоби було зрозуміло, що його не
    варто використовувати в інших цілях.

4.  Якщо старі методи управління одностороннім зв'язком знаходилися в
    керуючому класі, доповніть їх викликами службових методів із
    зв'язуваного об'єкта.

5.  Якщо старі методи управління зв'язком знаходилися в керуючому класі,
    створіть методи управління в керуючому класі, викликайте їх і
    делегуйте їм виконання.

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
[Заміна двонаправленого зв\'язку
одностороннім](change-bidirectional-association-to-unidirectional.html)
:::
::::
:::::
::::::

::: next
#### Читаємо далі

[Заміна двонаправленого зв\'язку одностороннім []{.fa
.fa-arrow-right}](change-bidirectional-association-to-unidirectional.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Дублювання видимих
даних](duplicate-observed-data.html){.btn .btn-default rel="prev"}
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
