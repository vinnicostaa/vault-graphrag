::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Спрощення
умовних
виразів](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Введення Null-об\'єкта {#введення-null-обєкта .title}

::: aka
Також відомий як: [Introduce Null Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Через те, що деякі методи повертають `null` замість реальних об'єктів, у
вас в коді присутня безліч перевірок на `null`.
:::

::: after
### Рішення

Замість `null` повертайте Null-об'єкт, який надає поведінку за
умовчанням.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
if (customer == null) {
  plan = BillingPlan.basic();
}
else {
  plan = customer.getPlan();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
class NullCustomer extends Customer {
  boolean isNull() {
    return true;
  }
  Plan getPlan() {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
customer = (order.customer != null) ?
  order.customer : new NullCustomer();

// Use Null-object as if it's normal subclass.
plan = customer.getPlan();
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
До
:::

``` {.code lang="csharp"}
if (customer == null) 
{
  plan = BillingPlan.Basic();
}
else 
{
  plan = customer.GetPlan();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
public sealed class NullCustomer: Customer 
{
  public override bool IsNull 
  {
    get { return true; }
  }
  
  public override Plan GetPlan() 
  {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
customer = order.customer ?? new NullCustomer();

// Use Null-object as if it's normal subclass.
plan = customer.GetPlan();
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
До
:::

``` {.code lang="php"}
if ($customer === null) {
  $plan = BillingPlan::basic();
} else {
  $plan = $customer->getPlan();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
class NullCustomer extends Customer {
  public function isNull() {
    return true;
  }
  public function getPlan() {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
$customer = ($order->customer !== null) ?
  $order->customer :
  new NullCustomer;

// Use Null-object as if it's normal subclass.
$plan = $customer->getPlan();
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
До
:::

``` {.code lang="python"}
if customer is None:
    plan = BillingPlan.basic()
else:
    plan = customer.getPlan()
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="python"}
class NullCustomer(Customer):

    def isNull(self):
        return True

    def getPlan(self):
        return self.NullPlan()

    # Some other NULL functionality.

# Replace null values with Null-object.
customer = order.customer or NullCustomer()

# Use Null-object as if it's normal subclass.
plan = customer.getPlan()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
if (customer == null) {
  plan = BillingPlan.basic();
}
else {
  plan = customer.getPlan();
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
class NullCustomer extends Customer {
  isNull(): boolean {
    return true;
  }
  getPlan(): Plan {
    return new NullPlan();
  }
  // Some other NULL functionality.
}

// Replace null values with Null-object.
let customer = (order.customer != null) ?
  order.customer : new NullCustomer();

// Use Null-object as if it's normal subclass.
plan = customer.getPlan();
```
::::
:::::::
::::::::::::::::::::::::::::

### Причини рефакторингу

Безліч перевірок на `null` ускладнюють і засмічують код.

### Недоліки

-   За відмову від умовних операторів ви розплачуєтесь ще одним новим
    класом.

### Порядок рефакторингу

1.  З потрібного вам класу створіть підклас, який виконуватиме роль
    Null-об'єкта.

2.  В обох класах створіть метод `isNull()`, який повертатиме `true` для
    Null-об'єкта і `false` для об'єкта реального класу.

3.  Знайдіть усі місця, де код може повернути `null` замість реального
    об'єкта. Змініть цей код так, щоб він повертав Null-об'єкт.

4.  Знайдіть усі місця, де змінні реального класу порівнюються з `null`.
    Замініть такі перевірки викликом методу `isNull()`.

5.  -   Якщо в цих умовних операторах при значенні, не рівному \`null\`,
        виконуються методи початкового класу, перевизначить ці методи в
        Null-класі і вставте туди код з частини умови `else`. Після
        цього умовний оператор можна буде взагалі видалити, а різна
        поведінка здійснюватиметься за рахунок поліморфізму.
    -   Якщо не все так просто, і методи перевизначити не виходить,
        подивіться, чи можна просто виділити операції, які повинні були
        виконуватися при значенні рівному `null` в нові методи
        Null-об'єкта. Викликайте ці методи замість старого коду в `else`
        як операції за умовчанням.

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

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Схожі рефакторинги

:::: dl
::: dt
[Заміна умовного оператора
поліморфізмом](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Реалізує паттерн проектування

:::: dl
::: dt
Null-object
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Бореться з запахом

:::: dl
::: dt
[Оператори switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Тимчасове поле](smells/temporary-field.html)
:::
::::
:::::::
::::::::::::::

::: next
#### Читаємо далі

[Введення перевірки твердження []{.fa
.fa-arrow-right}](introduce-assertion.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна умовного оператора
поліморфізмом](replace-conditional-with-polymorphism.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::
