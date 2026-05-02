:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Заміна поля-масиву об\'єктом {#заміна-поля-масиву-обєктом .title}

::: aka
Також відомий як: [Replace Array with
Object]{style="display:inline-block"}
:::

> Цей рефакторинг є особливим випадком [заміни простого поля
> об'єктом](replace-data-value-with-object.html).

::::: before-after-container
::: before
### Проблема

У вас є масив, в якому зберігаються різнотипні дані.
:::

::: after
### Рішення

Замініть масив об'єктом, який матиме окремі поля для кожного елементу.
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
Після
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
Після
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
Після
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
Після
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
Після
:::

``` {.code lang="typescript"}
let row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Масиви --- відмінний інструмент для зберігання однотипних даних і
колекцій. Але чекайте біди, якщо ви використовуєте масив як банківські
скриньки, наприклад, зберігаєте в елементі №1 --- ім'я користувача, а в
елементі №14 --- його адресу. Такий варіант не лише може привести до
фатальних наслідків, коли хтось покладе щось не в той елемент, але також
вимагає витрати величезної кількості часу на запам'ятовування того, де
які дані зберігаються.

### Переваги

-   У клас, що утворився, можна перенести всю пов'язану поведінку, яка
    раніше зберігалася в основному класі або в інших місцях.

-   Поля класу набагато простіше документувати, ніж елементи масиву.

### Порядок рефакторингу

1.  Створіть новий клас, який міститиме дані з масиву. Помістіть в нього
    сам масив як публічне поле.

2.  Створіть поле для зберігання об'єкта цього класу в початковому
    класі. Не забудьте створити також сам об'єкт в тому місці, де ви
    ініціювали масив даних.

3.  У новому класі один за іншим створюйте методи доступу для всіх
    елементів масиву. Давайте їм зрозумілі назви, які відображають суть
    того, що вони зберігають. У той же час змінюйте кожен випадок
    використання елементу масиву в основному коді відповідними методами
    доступу до цього елементу.

4.  Коли методи доступу будуть створені для всіх елементів, зробіть
    масив приватним.

5.  Для кожного елементу масиву створіть приватне поле в класі, після
    чого змініть методи доступу так, щоб вони використовували це поле
    замість масиву.

6.  Коли всі дані будуть переміщені, видаліть масив.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Заміна простого поля об\'єктом](replace-data-value-with-object.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Одержимість елементарними типами](smells/primitive-obsession.html)
:::
::::
:::::
:::::::::

::: next
#### Читаємо далі

[Дублювання видимих даних []{.fa
.fa-arrow-right}](duplicate-observed-data.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна посилання
значенням](change-reference-to-value.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::
