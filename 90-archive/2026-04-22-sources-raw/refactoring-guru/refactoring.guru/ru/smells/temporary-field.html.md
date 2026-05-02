:::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::: {.main-content-container .center-content-container}
:::::::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Рефакторинг](../refactoring.html){.type}
/ [Запахи кода](../refactoring/smells.html){.type} / [Нарушители
объектно-ориентированного
дизайна](../refactoring/smells/oo-abusers.html){.type}
:::

# Временное поле {#временное-поле .title}

::: aka
Также известен как: [Temporary Field]{style="display:inline-block"}
:::

### Симптомы и признаки

Временные поля --- это поля, которые нужны объекту только при
определённых обстоятельствах. Только тогда они заполняются какими-то
значениями, оставаясь пустыми в остальное время.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-01e767.png?id=5e30a8144171693ee4894091762b9742"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-01.png?id=5e30a8144171693ee4894091762b9742 500w,/images/refactoring/content/smells/temporary-field-01-1.5x.png?id=5079cd815f19d0a8938bc64c40f763ab 750w,/images/refactoring/content/smells/temporary-field-01-2x.png?id=1cf05ea67f1e3bd1c1f634af7408f67c 1000w"
sizes="(max-width: 720px) 100vw, 500px" data-fetchpriority="high"
width="500" height="300" />
</figure>

### Причины появления

Зачастую временные поля создаются для использования в алгоритме, который
требует большого числа входных данных. Так, вместо создания большого
числа параметров в таком методе, программист решает создать для этих
данных поля в классе. Эти поля используются только в данном алгоритме, а
в остальное время простаивают.

Такой код очень трудно понять. Вы ожидаете увидеть данные в полях
объекта, а они почему-то пустуют почти все время.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-02bc27.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-02.png?id=2cf98e02ffb0c1d36a98d48ba7bc5c45 500w,/images/refactoring/content/smells/temporary-field-02-1.5x.png?id=d29dd17bc354ec725ad3092a67cecfd8 750w,/images/refactoring/content/smells/temporary-field-02-2x.png?id=6c8e4384d9029cd3ec84cd330d5871eb 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Лечение

-   Временные поля и весь код, работающий с ними, можно поместить в свой
    собственный класс с помощью [извлечения
    класса](../extract-class.html). По сути, вы создадите
    [объект-метод](../replace-method-with-method-object.html).

-   [Введите Null-объект](../introduce-null-object.html) и встройте его
    вместо кода проверок на наличие значений во временных полях.

<figure class="image">
<img
src="../../images/refactoring/content/smells/temporary-field-032ad0.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc"
style="aspect-ratio: 1.67;"
srcset="/images/refactoring/content/smells/temporary-field-03.png?id=cf0e1c1e2a19745d23ca9e1d917d45cc 500w,/images/refactoring/content/smells/temporary-field-03-1.5x.png?id=09f1df24d41f5f39b6b298b8a71701b7 750w,/images/refactoring/content/smells/temporary-field-03-2x.png?id=c633fd664958307bf296b09de72959dd 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
height="300" />
</figure>

### Выигрыш

-   Улучшает понятность и организацию кода.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

::: next
#### Читаем дальше

[Отказ от наследства []{.fa .fa-arrow-right}](refused-bequest.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Операторы switch](switch-statements.html){.btn
.btn-default rel="prev"}
:::
::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот запах кода --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::
::::::::::::::::
