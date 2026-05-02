:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Замена поля-массива объектом {#замена-поля-массива-объектом .title}

::: aka
Также известен как: [Replace Array with
Object]{style="display:inline-block"}
:::

> Этот рефакторинг является особым случаем [замены простого поля
> объектом](replace-data-value-with-object.html).

::::: before-after-container
::: before
### Проблема

У вас есть массив, в котором хранятся разнотипные данные.
:::

::: after
### Решение

Замените массив объектом, который будет иметь отдельные поля для каждого
элемента.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
String[] row = new String[2];
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
Performance row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
string[] row = new string[2];
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
Performance row = new Performance();
row.SetName("Liverpool");
row.SetWins("15");
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$row = [];
$row[0] = "Liverpool";
$row[1] = 15;
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
$row = new Performance;
$row->setName("Liverpool");
$row->setWins(15);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
row = [None * 2]
row[0] = "Liverpool"
row[1] = "15"
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
row = Performance()
row.setName("Liverpool")
row.setWins("15")
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let row = new Array(2);
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
let row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Массивы --- отличный инструмент для хранения однотипных данных и
коллекций. Но ждите беды, если вы используете массив в качестве
банковских ячеек, например, храните в ячейке №1 --- имя пользователя, а
в ячейке №14  --- его адрес. Такой подход не только может привести к
фатальным последствиям, когда кто-то положит что-то не в ту ячейку, но
также требует затраты огромного количества времени на запоминание того,
где какие данные хранятся.

### Достоинства

-   В образовавшийся класс можно переместить все связанные поведения,
    которые раньше хранились в основном классе или в других местах.

-   Поля класса гораздо проще документировать, чем ячейки массива.

### Порядок рефакторинга

1.  Создайте новый класс, который будет содержать данные из массива.
    Поместите в него сам массив как публичное поле.

2.  Создайте поле для хранения объекта этого класса в исходном классе.
    Не забудьте создать также сам объект в том месте, где вы
    инициировали массив данных.

3.  В новом классе один за другим создавайте методы доступа для всех
    элементов массива. Давайте им понятные названия, которые отражают
    суть того, что они хранят. В тоже время изменяйте каждый случай
    использования ячейки массива в основном коде соответствующими
    методами доступа к этой ячейке.

4.  Когда методы доступа будут созданы для всех ячеек, сделайте массив
    приватным.

5.  Для каждого элемента массива создайте приватное поле в классе, после
    чего измените методы доступа так, чтобы они использовали это поле
    вместо массива.

6.  Когда все данные будут перемещены, удалите массив.

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
### Родственные рефакторинги

:::: dl
::: dt
[Замена простого поля объектом](replace-data-value-with-object.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Борется с запахом

:::: dl
::: dt
[Одержимость элементарными типами](smells/primitive-obsession.html)
:::
::::
:::::
:::::::::

::: next
#### Читаем дальше

[Дублирование видимых данных []{.fa
.fa-arrow-right}](duplicate-observed-data.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена ссылки
значением](change-reference-to-value.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::
