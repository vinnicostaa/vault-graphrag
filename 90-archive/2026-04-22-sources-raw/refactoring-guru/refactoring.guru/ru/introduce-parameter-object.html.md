:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Замена параметров объектом {#замена-параметров-объектом .title}

::: aka
Также известен как: [Introduce Parameter
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

В ваших методах встречается повторяющаяся группа параметров.
:::

::: after
### Решение

Замените эти параметры объектом.
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
После
:::

::: ba-container
![Introduce Parameter Object -
After](../images/refactoring/diagrams/Introduce%20Parameter%20Object%20-%20After803b.png?id=b5ed29b5678759399a83c229b1837730){width="359"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Одинаковые группы параметров зачастую встречаются не в единственном
методе. Это приводит к дублированию кода, как самих параметров, так и
частых операций над ними. Как только вы сведёте параметры в одном
классе, вы сможете переместить туда и методы обработки этих данных,
очистив от этого кода другие методы.

### Достоинства

-   Улучшает читабельность кода. Вместо пачки разнообразных параметров
    вы видите один объект с понятным названием.

-   Одинаковые группы параметров тут и там создают особый род
    дублирования кода, при котором вроде бы не происходит вызова
    идентичного кода, но все время встречаются одинаковые группы
    параметров и аргументов.

### Недостатки

-   Если вы переместили в новый класс только данные и не планируете
    перемещать в него никакие поведения и операции над этими данными,
    это может попахивать [классами данных](smells/data-class.html).

### Порядок рефакторинга

1.  Создайте новый класс, который будет представлять вашу группу
    параметров. Сделайте так, чтобы данные объектов этого класса нельзя
    было изменить после создания.

2.  В методе, к которому применяем рефакторинг, [добавьте новый
    параметр](add-parameter.html), в котором будет передаваться ваш
    объект-параметр. Во всех вызовах метода передавайте в этот параметр
    объект, создаваемый из старых параметров метода.

3.  Теперь начинайте по одному удалять старые параметры из метода,
    заменяя их в коде полями объекта-параметра. Тестируйте программу
    после каждой замены параметра.

4.  По окончанию оцените, есть ли возможность и смысл перенести какую-то
    часть метода (а иногда и весь метод) в класс объекта-параметра. Если
    так, используйте [перемещение метода](move-method.html) или
    [извлечение метода](extract-method.html), чтобы осуществить перенос.

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

::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Передача всего объекта](preserve-whole-object.html)
:::
::::
:::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Длинный список параметров](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Группы данных](smells/data-clumps.html)
:::
::::

:::: dl
::: dt
[Одержимость элементарными типами](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::
:::::::::::
:::::::::::::::

::: next
#### Читаем дальше

[Удаление сеттера []{.fa
.fa-arrow-right}](remove-setting-method.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена параметра вызовом
метода](replace-parameter-with-method-call.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
