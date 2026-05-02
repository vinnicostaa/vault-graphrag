:::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Решение задач
обобщения](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Создание шаблонного метода {#создание-шаблонного-метода .title}

::: aka
Также известен как: [Form Template Method]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

В подклассах реализованы алгоритмы, содержащие похожие шаги и одинаковый
порядок выполнения этих шагов.
:::

::: after
### Решение

Вынесите структуру алгоритма и одинаковые шаги в суперкласс, а в
подклассах оставьте реализацию отличающихся шагов.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
До
:::

::: ba-container
![Form Template Method -
Before](../images/refactoring/diagrams/Form%20Template%20Method%20-%20Beforee8c2.png?id=858de51c181ba956e5ecf63e3a88fda2){width="696"}
:::
:::::

::::: after
::: ba
После
:::

::: ba-container
![Form Template Method -
After](../images/refactoring/diagrams/Form%20Template%20Method%20-%20After9b6d.png?id=7a2977c70d9fcd80c96307a877343828){width="640"}
:::
:::::
:::::::::
::::::::::

### Причины рефакторинга

Подклассы развиваются параллельно. Иногда разными людьми, что приводит к
дублированию кода и ошибок, а также к усложнению поддержки, так как
каждое изменение приходится проводить во всех подклассах.

### Достоинства

-   Когда мы говорим о дублировании кода, не всегда имеется в виду
    программирование методом копирования-вставки. Нередко дублирование
    возникает на более абстрактном уровне. Например, у вас есть метод
    сортировки чисел и метод сортировки коллекции объектов. При этом,
    единственное, чем они отличаются --- это сравнение элементов.
    Создание шаблонного метода позволяет справиться с таким
    дублированием, объединив общие шаги алгоритма в суперклассе и
    оставив различия для подклассов.

-   Создание шаблонного метода реализует *принцип
    открытости/закрытости*. При появлении новой версии алгоритма, вам
    нужно будет всего лишь создать новый подкласс, не меняя существующий
    код.

### Порядок рефакторинга

1.  Разбейте алгоритмы в подклассах на составные части, описанные в
    отдельных методах. В этом может помочь [извлечение
    метода](extract-method.html).

2.  Получившиеся методы, одинаковые для всех подклассов, можете смело
    перемещать в суперкласс, используя [подъём
    метода](pull-up-method.html).

3.  Отличающиеся методы приведите к единым названиям с помощью
    [переименования метода](rename-method.html).

4.  Поместите сигнатуры отличающихся методов в суперкласс как
    абстрактные с помощью [подъёма метода](pull-up-method.html). Их
    реализации оставьте в подклассах.

5.  И наконец, поднимите основной метод алгоритма в суперкласс. Он
    теперь должен работать с методами-шагами, описанными в
    суперклассе --- реальными или абстрактными.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Реализует паттерн проектирования

:::: dl
::: dt
[Шаблонный метод](design-patterns/template-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Замена наследования делегированием []{.fa
.fa-arrow-right}](replace-inheritance-with-delegation.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Свёртывание
иерархии](collapse-hierarchy.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::
