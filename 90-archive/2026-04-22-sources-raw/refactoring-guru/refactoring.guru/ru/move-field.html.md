::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Перемещение функций
между
объектами](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Перемещение поля {#перемещение-поля .title}

::: aka
Также известен как: [Move Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Поле используется в другом классе больше, чем в собственном.
:::

::: after
### Решение

Создайте поле в новом классе и перенаправьте к нему всех пользователей
старого поля.
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
После
:::

::: ba-container
![Move Field -
After](../images/refactoring/diagrams/Move%20Field%20-%20After4c93.png?id=d7c21af94ec9df17575373bae745e96e){width="134"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Зачастую поля переносятся как часть [извлечение одного класса из
другого](extract-class.html). Решить, в каком из классов должно остаться
поле, бывает непросто. Тем не менее, у нас есть неплохой рецепт ---
**поле должно быть там, где находятся методы, которые его используют**
(либо там, где этих методов больше).

Это правило поможет вам и в других случаях, когда поле попросту
находится не там, где нужно.

### Порядок рефакторинга

1.  Если поле публичное, вам будет намного проще совершить рефакторинг,
    если вы сделаете его приватным и предоставите публичные методы
    доступа (для этого можно использовать рефакторинг [инкапсуляция
    поля](encapsulate-field.html)).

2.  Создайте такое же поле с методами доступа в классе-приёмнике.

3.  Определите, как вы будете обращаться к классу-получателю. Вполне
    возможно, у вас уже есть поле или метод, которые возвращают
    подходящий объект. Если нет --- нужно будет написать новый метод или
    поле, в котором бы хранился объект класса-получателя.

4.  Замените все обращения к старому полю на соответствующие вызовы
    методов в классе-получателе. Если поле не приватное, проделайте это
    и в суперклассе, и в подклассах.

5.  Удалите поле в исходном классе.

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

:::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Перемещение метода](move-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Помогает рефакторингу

:::: dl
::: dt
[Извлечение класса](extract-class.html)
:::
::::

:::: dl
::: dt
[Встраивание класса](inline-class.html)
:::
::::
:::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Стрельба дробью](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Параллельные иерархии
наследования](smells/parallel-inheritance-hierarchies.html)
:::
::::

:::: dl
::: dt
[Неуместная близость](smells/inappropriate-intimacy.html)
:::
::::
:::::::::
::::::::::::::::::

::: next
#### Читаем дальше

[Извлечение класса []{.fa .fa-arrow-right}](extract-class.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Перемещение метода](move-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
