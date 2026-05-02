::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Вбудовування змінної {#вбудовування-змінної .title}

::: aka
Також відомий як: [Inline Temp]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є локальна змінна, якій присвоюється результат простого виразу (і
більше нічого).
:::

::: after
### Рішення

Замініть звернення до змінної цим виразом.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  double basePrice = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
boolean hasDiscount(Order order) {
  return order.basePrice() > 1000;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
bool HasDiscount(Order order)
{
  double basePrice = order.BasePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
bool HasDiscount(Order order)
{
  return order.BasePrice() > 1000;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
$basePrice = $anOrder->basePrice();
return $basePrice > 1000;
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
return $anOrder->basePrice() > 1000;
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
def hasDiscount(order):
    basePrice = order.basePrice()
    return basePrice > 1000
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
def hasDiscount(order):
    return order.basePrice() > 1000
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  let basePrice: number = order.basePrice();
  return basePrice > 1000;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
hasDiscount(order: Order): boolean {
  return order.basePrice() > 1000;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Вбудовування локальної змінної майже завжди використовують як частину
[заміни змінної викликом методу](replace-temp-with-query.html) або для
полегшення [відокремлення методу](extract-method.html).

### Переваги

-   Сам по собі цей рефакторинг не несе майже ніякої користі. Проте,
    якщо змінній присвоюється результат виконання якогось методу, у вас
    є можливість трохи поліпшити читабельність програми, позбавившись
    від зайвої змінної.

### Недоліки

-   Іноді, на перший погляд даремні тимчасові змінні потрібні для
    кешування, тобто збереження результату якоїсь дорогої операції, що
    буде використаний в ході роботи кілька разів повторно. Перед тим, як
    здійснитити рефакторинг, переконаєтеся, що у вашому випадку це не
    так.

### Порядок рефакторингу

1.  Знайдіть усі місця, де використовується змінна, і замініть цю змінну
    виразом, який їй присвоювався.

2.  Видаліть оголошення змінної і рядок присвоєння їй значення.

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

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Домогає іншим рефакторингам

:::: dl
::: dt
[Заміна змінної викликом методу](replace-temp-with-query.html)
:::
::::

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::
:::::::
::::::::

::: next
#### Читаємо далі

[Заміна змінної викликом методу []{.fa
.fa-arrow-right}](replace-temp-with-query.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відокремлення
змінної](extract-variable.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
