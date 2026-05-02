:::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Переименование метода {#переименование-метода .title}

::: aka
Также известен как: [Rename Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Название метода не раскрывает суть того, что он делает.
:::

::: after
### Решение

Измените название метода.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Rename Method -
Before](../images/refactoring/diagrams/Rename%20Method%20-%20Before175a.png?id=7943798ae9db6b5b232860eed6262462){width="171"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Rename Method -
After](../images/refactoring/diagrams/Rename%20Method%20-%20Aftere5be.png?id=62b4e6747951bbbacba3ede379fef200){width="171"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Метод мог получить неудачное название с самого начала. Например, кто-то
создал метод впопыхах, не придал должного значения хорошему названию.

С другой стороны, метод мог быть назван удачно изначально, но ввиду
расширения его функциональности, имя метода перестало быть актуальным.

### Достоинства

-   Улучшает читабельность кода. Постарайтесь дать новому методу такое
    название, которое бы отражало суть того, что он делает. Например,
    `createOrder()`, `renderCustomerInfo()` и т. д.

### Порядок рефакторинга

1.  Проверьте, не определён ли метод в суперклассе или подклассе. Если
    так, нужно будет повторить все шаги и в этих классах.

2.  Следующий шаг важен, чтобы сохранить работоспособность программы во
    время рефакторинга. Итак, создайте новый метод с новыми именем.
    Скопируйте туда код старого метода. Удалите весь код в старом методе
    и вставьте вместо него вызов нового метода.

3.  Найдите все обращения к старому методу и замените их обращениями к
    новому.

4.  Удалите старый метод. Этот шаг неосуществим, если старый метод
    является частью публичного интерфейса. В этом случае, старый метод
    нужно пометить как устаревший (`deprecated`).

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

::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Добавление параметра](add-parameter.html)
:::
::::

:::: dl
::: dt
[Удаление параметра](remove-parameter.html)
:::
::::
:::::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Альтернативные классы с разными
интерфейсами](smells/alternative-classes-with-different-interfaces.html)
:::
::::

:::: dl
::: dt
[Комментарии](smells/comments.html)
:::
::::
:::::::
:::::::::::::

::: next
#### Читаем дальше

[Добавление параметра []{.fa .fa-arrow-right}](add-parameter.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::
