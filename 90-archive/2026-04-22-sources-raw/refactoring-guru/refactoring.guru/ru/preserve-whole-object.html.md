:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение вызовов
методов](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Передача всего объекта {#передача-всего-объекта .title}

::: aka
Также известен как: [Preserve Whole
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Вы получаете несколько значений из объекта, а затем передаёте их в метод
как параметры.
:::

::: after
### Решение

Вместо этого передавайте весь объект.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int low = daysTempRange.getLow();
int high = daysTempRange.getHigh();
boolean withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
boolean withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
int low = daysTempRange.GetLow();
int high = daysTempRange.GetHigh();
bool withinPlan = plan.WithinRange(low, high);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
bool withinPlan = plan.WithinRange(daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$low = $daysTempRange->getLow();
$high = $daysTempRange->getHigh();
$withinPlan = $plan->withinRange($low, $high);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
$withinPlan = $plan->withinRange($daysTempRange);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
low = daysTempRange.getLow()
high = daysTempRange.getHigh()
withinPlan = plan.withinRange(low, high)
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
withinPlan = plan.withinRange(daysTempRange)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let low = daysTempRange.getLow();
let high = daysTempRange.getHigh();
let withinPlan = plan.withinRange(low, high);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
let withinPlan = plan.withinRange(daysTempRange);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Проблема заключается в том, что перед вызовом вашего метода нужно каждый
раз вызывать методы будущего объекта-параметра. Если способ вызова этих
методов либо количество данных, получаемых для метода, поменяется, то
изменения придётся искать и вносить в дюжину мест по всей программе.

Вместо этого код получения всех нужных данных может храниться одном
месте --- в самом методе.

### Достоинства

-   Вместо пачки разнообразных параметров вы видите один объект с
    понятным названием.

-   Если методу понадобятся ещё какие-то данные из объекта, не нужно
    будет переписывать все места, где вызывается этот метод, а только
    лишь внутренности самого метода.

### Недостатки

-   В некоторых случаях после такого преобразования метод теряет в
    универсальности, так как он мог получать данные из множества разных
    источников, а в результате рефакторинга мы ограничиваем круг его
    применения только для объектов с определённым интерфейсом.

### Порядок рефакторинга

1.  Создайте параметр в методе для объекта, из которого можно получить
    нужные значения.

2.  Теперь начинайте по одному удалять старые параметры из метода,
    заменяя их в коде вызовами соответствующих методов
    объекта-параметра. Тестируйте программу после каждой замены
    параметра.

3.  Удалите код получения значений из объекта-параметра, который стоял
    перед вызовом метода.

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

::::::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Родственные рефакторинги

:::: dl
::: dt
[Замена параметров объектом](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Замена параметра вызовом
метода](replace-parameter-with-method-call.html)
:::
::::
:::::::

::::::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Одержимость элементарными типами](smells/primitive-obsession.html)
:::
::::

:::: dl
::: dt
[Длинный список параметров](smells/long-parameter-list.html)
:::
::::

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Группы данных](smells/data-clumps.html)
:::
::::
:::::::::::
:::::::::::::::::

::: next
#### Читаем дальше

[Замена параметра вызовом метода []{.fa
.fa-arrow-right}](replace-parameter-with-method-call.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена параметра набором специализированных
методов](replace-parameter-with-explicit-methods.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
