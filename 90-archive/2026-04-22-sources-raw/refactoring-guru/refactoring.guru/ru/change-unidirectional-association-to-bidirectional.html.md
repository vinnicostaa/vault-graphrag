::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Замена однонаправленной связи двунаправленной {#замена-однонаправленной-связи-двунаправленной .title}

::: aka
Также известен как: [Change Unidirectional Association to
Bidirectional]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть два класса, которым нужно использовать фичи друг друга, но
между ними существует только односторонняя связь.
:::

::: after
### Решение

Добавьте недостающую связь в класс, в котором она отсутствует.
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
После
:::

::: ba-container
![Change Unidirectional Association to Bidirectional -
After](../images/refactoring/diagrams/Change%20Unidirectional%20Association%20to%20Bidirectional%20-%20After01d6.png?id=058ba44a44610bcb9caa4c23f60ad0bc){width="396"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Между классами изначально была одностороння связь. Однако с течением
времени клиентскому коду потребовался доступ в обе стороны этой связи.

### Достоинства

-   Если в классе возникает необходимость в обратной связи, её можно
    попросту вычислить. Однако, если такие вычисления оказываются
    довольно сложными, лучше хранить обратную связь.

### Недостатки

-   Двусторонние связи гораздо сложнее в реализации и поддержке, чем
    односторонние.

-   Двусторонние связи делают классы зависимыми друг от друга. При
    односторонней связи один из них можно было использовать отдельно от
    другого.

### Порядок рефакторинга

1.  Добавьте поле, которое будет содержать обратную связь.

2.  Решите, какой класс будет «управляющим». Этот класс должен содержать
    методы, которые создают или обновляют связь при добавлении или
    изменении элементов, устанавливая связь в своём классе, а также
    вызывая служебные методы установки связи в связываемом объекте.

3.  Создайте служебный метод установки связи в «не управляющем классе».
    Этот метод должен заполнять поле со связью тем, что ему подают в
    параметрах. Назовите этот метод так, чтобы было очевидно, что его не
    стоит использовать в других целях.

4.  Если старые методы управления однонаправленной связью находились в
    «управляющем классе», дополните их вызовами служебных методов из
    связываемого объекта.

5.  Если старые методы управления связью находились в «не управляющем
    классе», создайте управляющие методы в «управляющем классе»,
    вызывайте их и делегируйте им выполнение.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Анти-рефакторинг

:::: dl
::: dt
[Замена двунаправленной связи
однонаправленной](change-bidirectional-association-to-unidirectional.html)
:::
::::
:::::
::::::

::: next
#### Читаем дальше

[Замена двунаправленной связи однонаправленной []{.fa
.fa-arrow-right}](change-bidirectional-association-to-unidirectional.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Дублирование видимых
данных](duplicate-observed-data.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::
