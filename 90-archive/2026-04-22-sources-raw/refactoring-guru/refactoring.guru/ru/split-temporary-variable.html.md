::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Расщепление переменной {#расщепление-переменной .title}

::: aka
Также известен как: [Split Temporary
Variable]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас есть локальная переменная, которая используется для хранения
разных промежуточных значений внутри метода (за исключением переменных
циклов).
:::

::: after
### Решение

Используйте разные переменные для разных значений. Каждая переменная
должна отвечать только за одну определённую вещь.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double temp = 2 * (height + width);
System.out.println(temp);
temp = height * width;
System.out.println(temp);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
final double perimeter = 2 * (height + width);
System.out.println(perimeter);
final double area = height * width;
System.out.println(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
double temp = 2 * (height + width);
Console.WriteLine(temp);
temp = height * width;
Console.WriteLine(temp);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
readonly double perimeter = 2 * (height + width);
Console.WriteLine(perimeter);
readonly double area = height * width;
Console.WriteLine(area);
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$temp = 2 * ($this->height + $this->width);
echo $temp;
$temp = $this->height * $this->width;
echo $temp;
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
$perimeter = 2 * ($this->height + $this->width);
echo $perimeter;
$area = $this->height * $this->width;
echo $area;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
temp = 2 * (height + width)
print(temp)
temp = height * width
print(temp)
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
perimeter = 2 * (height + width)
print(perimeter)
area = height * width
print(area)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Если вы «экономите» переменные внутри функции, повторно используете их
для различных несвязанных целей, у вас обязательно начнутся проблемы в
тот момент, когда потребуется внести какие-то изменения в код,
содержащий эти переменные. Вам придётся перепроверять все случаи
использования переменной, чтобы удостовериться в отсутствии ошибки в
коде.

### Достоинства

-   Каждый элемент программы должен отвечать только за одну вещь. Это
    сильно упрощает поддержку кода в будущем, так как вы можете спокойно
    заменить этот элемент, не опасаясь побочных эффектов.

-   Улучшается читабельность кода. Если переменная создавалась очень
    давно, да еще и в спешке, она могла получить элементарное название,
    которое не объясняет сути хранимого значения, например, `k`, `a2`,
    `value` и т. д. У вас есть шанс исправить ситуацию, назначив новым
    переменным хорошие названия, отражающие суть хранимых значений.
    Например, `customerTaxValue`, `cityUnemploymentRate`,
    `clientSalutationString` и т. д.

-   Этот рефакторинг помогает в дальнейшем [выделить повторяющиеся
    участки кода в отдельные методы](extract-method.html).

### Порядок рефакторинга

1.  Найдите место в коде, где переменная в первый раз заполняется
    каким-то значением. В этом месте переименуйте ее, причем новое
    название должно соответствовать присваиваемому значению.

2.  Подставьте её новое название вместо старого в тех местах, где
    использовалось это значение переменной.

3.  Повторите операцию для случаев, где переменной присваивается новое
    значение.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Анти-рефакторинг

:::: dl
::: dt
[Встраивание переменной](inline-temp.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Родственные рефакторинги

:::: dl
::: dt
[Извлечение переменной](extract-variable.html)
:::
::::

:::: dl
::: dt
[Удаление присваиваний
параметрам](remove-assignments-to-parameters.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Помогает рефакторингу

:::: dl
::: dt
[Извлечение метода](extract-method.html)
:::
::::
:::::
::::::::::::::

::: next
#### Читаем дальше

[Удаление присваиваний параметрам []{.fa
.fa-arrow-right}](remove-assignments-to-parameters.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена переменной вызовом
метода](replace-temp-with-query.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::
