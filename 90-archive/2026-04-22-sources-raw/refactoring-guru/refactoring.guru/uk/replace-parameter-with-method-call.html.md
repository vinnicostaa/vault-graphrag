::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
викликів
методів](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Заміна параметра викликом методу {#заміна-параметра-викликом-методу .title}

::: aka
Також відомий як: [Replace Parameter with Method
Call]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Викликаємо метод і передаємо його результати в параметри іншого методу.
При цьому значення параметрів могли б бути отримані і всередині
викликаного методу.
:::

::: after
### Рішення

Замість передачі значення через аргументи, спробуйте перемістити код
отримання значення всередину самого методу.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.getSeasonalDiscount();
double fees = this.getFees();
double finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
int basePrice = quantity * itemPrice;
double finalPrice = discountedPrice(basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double seasonDiscount = this.GetSeasonalDiscount();
double fees = this.GetFees();
double finalPrice = DiscountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
int basePrice = quantity * itemPrice;
double finalPrice = DiscountedPrice(basePrice);
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
$seasonDiscount = $this->getSeasonalDiscount();
$fees = $this->getFees();
$finalPrice = $this->discountedPrice($basePrice, $seasonDiscount, $fees);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
$basePrice = $this->quantity * $this->itemPrice;
$finalPrice = $this->discountedPrice($basePrice);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
seasonalDiscount = self.getSeasonalDiscount()
fees = self.getFees()
finalPrice = discountedPrice(basePrice, seasonalDiscount, fees)
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
basePrice = quantity * itemPrice
finalPrice = discountedPrice(basePrice)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
const seasonDiscount = this.getSeasonalDiscount();
const fees = this.getFees();
const finalPrice = discountedPrice(basePrice, seasonDiscount, fees);
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
let basePrice = quantity * itemPrice;
let finalPrice = discountedPrice(basePrice);
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

У довгому списку параметрів зазвичай вкрай складно орієнтуватися. Крім
того, виклики таких методів часто перетворюються на цілу низку обчислень
значень, які передаватимуться в метод. Ось чому якщо значення параметра
може бути вираховано за допомогою виклику якогось методу, це слід
зробити усередині самого методу, а від параметра позбутися.

### Переваги

-   Позбавляємося від зайвих параметрів, спрощуючи виклики методів. Ці
    параметри частенько створюються як заділ на майбутнє (яке може так і
    не настати).

### Недоліки

-   Параметр може знадобитися завтра для якихось інших цілей і метод
    доведеться переписати.

### Порядок рефакторингу

1.  Переконайтеся, що код отримання значення не використовує параметрів
    з поточного методу. Це важливо, оскільки параметри будуть недоступні
    усередині іншого методу, через що перенесення стане неможливим.

2.  Якщо код отримання значення складніший, ніж один виклик якогось
    методу або функції, застосуйте [відокремлення
    методу](extract-method.html), щоб виділити цей код в новий метод і
    зробити виклик простішим.

3.  У коді головного методу замініть усі звернення до замінюваного
    параметра викликами методу отримання значення.

4.  Використайте [видалення параметра](remove-parameter.html), щоб
    видалити невживаний тепер параметр.

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

::: next
#### Читаємо далі

[Заміна параметрів об\'єктом []{.fa
.fa-arrow-right}](introduce-parameter-object.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Передача усього
об\'єкта](preserve-whole-object.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
