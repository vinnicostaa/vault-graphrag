::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Упрощение условных
выражений](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Введение Null-объекта {#введение-null-объекта .title}

::: aka
Также известен как: [Introduce Null
Object]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Из-за того, что некоторые методы возвращают `null` вместо реальных
объектов, у вас в коде присутствует множество проверок на `null`.
:::

::: after
### Решение

Вместо `null` возвращайте Null-объект, который предоставляет поведение
по умолчанию.
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
После
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
После
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
После
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
После
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
После
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

### Причины рефакторинга

Десятки проверок на `null` усложняют и засоряют код.

### Недостатки

-   За отказ от условных операторов вы платите ещё одним новым классом.

### Порядок рефакторинга

1.  Из интересующего вас класса создайте подкласс, который будет
    выполнять роль Null-объекта.

2.  В обоих классах создайте метод `isNull()`, который будет возвращать
    `true` для Null-объекта и `false` для реального класса.

3.  Найдите все места, где код может вернуть `null` вместо реального
    объекта. Измените этот код так, чтобы он возвращал Null-объект.

4.  Найдите все места, где переменные реального класса сравниваются с
    `null`. Замените такие проверки вызовом метода `isNull()`.

5.  -   Если в этих условных операторах при значении не равном `null`
        выполняются методы исходного класса, переопределите эти методы в
        Null-классе и вставьте туда код из `else` части условия. После
        этого условный оператор можно будет вообще удалить, а разное
        поведение будет осуществляться за счёт полиморфизма.
    -   Если не все так просто, и методы переопределить не получается,
        посмотрите, можно ли просто выделите операции, которые должны
        были выполняться при значении равном `null` в новые методы
        Null-объекта. Вызывайте эти методы вместо старого кода в `else`
        как операции по умолчанию.

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
### Родственные рефакторинги

:::: dl
::: dt
[Замена условного оператора
полиморфизмом](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Реализует паттерн проектирования

:::: dl
::: dt
Null-object
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Борется с запахом

:::: dl
::: dt
[Операторы switch](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Временное поле](smells/temporary-field.html)
:::
::::
:::::::
::::::::::::::

::: next
#### Читаем дальше

[Введение проверки утверждения []{.fa
.fa-arrow-right}](introduce-assertion.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена условного оператора
полиморфизмом](replace-conditional-with-polymorphism.html){.btn
.btn-default rel="prev"}
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
