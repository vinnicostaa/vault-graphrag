::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Заміна методу об\'єктом методів {#заміна-методу-обєктом-методів .title}

::: aka
Також відомий як: [Replace Method with Method
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є довгий метод, в якому локальні змінні так сильно переплетені, що
це робить неможливим застосування *відокремлення методу*.
:::

::: after
### Рішення

Перетворіть метод в окремий клас так, щоб локальні змінні стали полями
цього класу. Після цього можна без проблем розділити метод на частини.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Order {
  // ...
  public double price() {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
class Order {
  // ...
  public double price() {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) {
    // Copy relevant information from the
    // order object.
  }
  
  public double compute() {
    // Perform long computation.
  }
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
public class Order 
{
  // ...
  public double Price() 
  {
    double primaryBasePrice;
    double secondaryBasePrice;
    double tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
public class Order 
{
  // ...
  public double Price() 
  {
    return new PriceCalculator(this).Compute();
  }
}

public class PriceCalculator 
{
  private double primaryBasePrice;
  private double secondaryBasePrice;
  private double tertiaryBasePrice;
  
  public PriceCalculator(Order order) 
  {
    // Copy relevant information from the
    // order object.
  }
  
  public double Compute() 
  {
    // Perform long computation.
  }
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
class Order {
  // ...
  public function price() {
    $primaryBasePrice = 10;
    $secondaryBasePrice = 20;
    $tertiaryBasePrice = 30;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
class Order {
  // ...
  public function price() {
    return (new PriceCalculator($this))->compute();
  }
}

class PriceCalculator {
  private $primaryBasePrice;
  private $secondaryBasePrice;
  private $tertiaryBasePrice;
  
  public function __construct(Order $order) {
      // Copy relevant information from the
      // order object.
  }
  
  public function compute() {
    // Perform long computation.
  }
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
class Order:
    # ...
    def price(self):
        primaryBasePrice = 0
        secondaryBasePrice = 0
        tertiaryBasePrice = 0
        # Perform long computation.
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
class Order:
    # ...
    def price(self):
        return PriceCalculator(self).compute()


class PriceCalculator:
    def __init__(self, order):
        self._primaryBasePrice = 0
        self._secondaryBasePrice = 0
        self._tertiaryBasePrice = 0
        # Copy relevant information from the
        # order object.

    def compute(self):
        # Perform long computation.
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Order {
  // ...
  price(): number {
    let primaryBasePrice;
    let secondaryBasePrice;
    let tertiaryBasePrice;
    // Perform long computation.
  }
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
class Order {
  // ...
  price(): number {
    return new PriceCalculator(this).compute();
  }
}

class PriceCalculator {
  private _primaryBasePrice: number;
  private _secondaryBasePrice: number;
  private _tertiaryBasePrice: number;
  
  constructor(order: Order) {
    // Copy relevant information from the
    // order object.
  }
  
  compute(): number {
    // Perform long computation.
  }
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Метод занадто довгий, і ви не можете його розділити через переплетення
локальних змінних, які складно ізолювати одну від одної.

Першим кроком до вирішення проблеми буде виділення всього цього методу в
окремий клас і перетворення його локальних змінних на поля.

По-перше, це дозволить вам ізолювати проблему в межах цього класу, а
по-друге, розчистить дорогу для розділення великого методу на менші за
розміром методи, які, до того ж, не підходили б до контексту
оригінального класу.

### Переваги

-   Ізоляція довгого методу у власному класі дозволяє зупинити
    безконтрольне зростання методу. Крім того, дає можливість ділити
    його на підметоди в рамках свого класу, не засмічуючи службовими
    методами оригінальний клас.

### Недоліки

-   Створюється ще один клас, підвищуючи загальну складність програми.

### Порядок рефакторингу

1.  Створіть новий клас. Дайте йому назву, ґрунтуючись на призначенні
    методу, над яким проводиться рефакторинг.

2.  У новому класі створіть приватне поле для зберігання посилання на
    екземпляр класу, в якому раніше знаходився метод. Це посилання
    згодом можна використати, щоб взяти з оригінального об'єкта якісь
    допоміжні дані.

3.  Створіть окреме приватне поле для кожної локальної змінної методу.

4.  Створіть конструктор, який приймає в параметрах значення всіх
    локальних змінних методу, а також ініціалізує відповідні приватні
    поля.

5.  Оголосіть основний метод і скопіюйте в нього код оригінального
    методу, замінивши локальні змінні приватними полями.

6.  Замініть тіло оригінального методу в початковому класі створенням
    об'єкта-методу і викликом його основного методу.

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

:::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

::::: dl
::: dt
[Заміна простого поля об\'єктом](replace-data-value-with-object.html)
:::

::: dd
Робить те ж саме, але з полем.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::
:::::
::::::::::

::: next
#### Читаємо далі

[Заміна алгоритму []{.fa
.fa-arrow-right}](substitute-algorithm.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Видалення присвоювань
параметрам](remove-assignments-to-parameters.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::
