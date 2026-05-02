:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Заміна змінної викликом методу {#заміна-змінної-викликом-методу .title}

::: aka
Також відомий як: [Replace Temp with
Query]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Ви розміщуєте результат якогось виразу в локальній змінній, щоб
використати її далі в коді.
:::

::: after
### Рішення

Виділіть цей вираз в окремий метод і повертайте результат з нього.
Замініть використання вашої змінної викликом методу. Новий метод може
бути використаний і в інших методах.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double calculateTotal() {
  double basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
double calculateTotal() {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
double basePrice() {
  return quantity * itemPrice;
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
double CalculateTotal() 
{
  double basePrice = quantity * itemPrice;
  
  if (basePrice > 1000) 
  {
    return basePrice * 0.95;
  }
  else 
  {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
double CalculateTotal() 
{
  if (BasePrice() > 1000) 
  {
    return BasePrice() * 0.95;
  }
  else 
  {
    return BasePrice() * 0.98;
  }
}
double BasePrice() 
{
  return quantity * itemPrice;
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
$basePrice = $this->quantity * $this->itemPrice;
if ($basePrice > 1000) {
  return $basePrice * 0.95;
} else {
  return $basePrice * 0.98;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
if ($this->basePrice() > 1000) {
  return $this->basePrice() * 0.95;
} else {
  return $this->basePrice() * 0.98;
}

...

function basePrice() {
  return $this->quantity * $this->itemPrice;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
def calculateTotal():
    basePrice = quantity * itemPrice
    if basePrice > 1000:
        return basePrice * 0.95
    else:
        return basePrice * 0.98
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
def calculateTotal():
    if basePrice() > 1000:
        return basePrice() * 0.95
    else:
        return basePrice() * 0.98

def basePrice():
    return quantity * itemPrice
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
 calculateTotal(): number {
  let basePrice = quantity * itemPrice;
  if (basePrice > 1000) {
    return basePrice * 0.95;
  }
  else {
    return basePrice * 0.98;
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
calculateTotal(): number {
  if (basePrice() > 1000) {
    return basePrice() * 0.95;
  }
  else {
    return basePrice() * 0.98;
  }
}
basePrice(): number {
  return quantity * itemPrice;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Застосування цього рефакторингу може бути підготовчим етапом для
застосування [виділення методу](extract-method.html) для якоїсь частини
дуже довгого методу.

Крім того, іноді можна знайти цей же вираз і в інших методах, що змушує
замислитися над створенням спільного методу для його отримання.

### Переваги

-   Покращує читабельність коду. Набагато простіше зрозуміти, що робить
    метод `getTax()`, ніж рядок `orderPrice() * -2`.

-   Допомагає прибрати дублювання коду, якщо замінюваний рядок
    використовується більш ніж в одному методі.

### Корисні факти

#### Питання швидкодії

При використанні цього рефакторингу може виникнути питання, чи не
позначиться результат рефакторингу погано на швидкодії програми. Чесна
відповідь --- так, результуючий код може отримати додаткове навантаження
за рахунок виклику нового методу. Проте в наш час швидких процесорів і
хороших компіляторів таке навантаження навряд чи буде помітне. Натомість
ми отримуємо кращу читабельність коду і можливість використати новий
метод в інших місцях програми.

Проте, якщо ваша тимчасова змінна служить для кешування результату
дійсно важкого і великого виразу, має сенс зупинити цей рефакторинг
після виділення виразу в новий метод.

### Порядок рефакторингу

1.  Переконайтеся, що змінній в межах методу присвоюється значення
    тільки один раз. Якщо це не так, використайте [розділення
    змінної](split-temporary-variable.html) для того, щоб гарантувати,
    що змінна буде використана тільки для зберігання результату вашого
    виразу.

2.  Використайте [відокремлення методу](extract-method.html) для того,
    щоби перемістити вираз, що цікавить вас, в новий метод.
    Переконайтеся, що цей метод тільки повертає значення і не міняє стан
    об'єкта. Якщо він якось впливає на видимий стан об'єкта,
    використайте [розділення запиту і
    модифікатора](separate-query-from-modifier.html).

3.  Замініть використання змінної викликом вашого нового методу.

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

::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

:::: dl
::: dt
[Відокремлення методу](extract-method.html)
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::
:::::::
:::::::::::

::: next
#### Читаємо далі

[Розщеплювання змінної []{.fa
.fa-arrow-right}](split-temporary-variable.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Вбудовування змінної](inline-temp.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::
