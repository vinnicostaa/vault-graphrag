:::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying
Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Introduce Null Object {#introduce-null-object .title}

::::: before-after-container
::: before
### Problem

Since some methods return `null` instead of real objects, you have many
checks for `null` in your code.
:::

::: after
### Solution

Instead of `null`, return a null object that exhibits the default
behavior.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
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
After
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
Before
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
After
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
Before
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
After
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
Before
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
After
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
Before
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
After
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

### Why Refactor

Dozens of checks for `null` make your code longer and uglier.

### Drawbacks

-   The price of getting rid of conditionals is creating yet another new
    class.

### How to Refactor

1.  From the class in question, create a subclass that will perform the
    role of null object.

2.  In both classes, create the method `isNull()`, which will return
    `true` for a null object and `false` for a real class.

3.  Find all places where the code may return `null` instead of a real
    object. Change the code so that it returns a null object.

4.  Find all places where the variables of the real class are compared
    with `null`. Replace these checks with a call for `isNull()`.

5.  -   If methods of the original class are run in these conditionals
        when a value doesn\'t equal `null`, redefine these methods in
        the null class and insert the code from the `else` part of the
        condition there. Then you can delete the entire conditional and
        differing behavior will be implemented via polymorphism.
    -   If things aren\'t so simple and the methods can\'t be redefined,
        see if you can simply extract the operators that were supposed
        to be performed in the case of a `null` value to new methods of
        the null object. Call these methods instead of the old code in
        `else` as the operations by default.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-en" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Tired of reading? {#tired-of-reading .title}

No wonder, it takes [7 hours]{.blue} to read all of the text we have
here.

Try our interactive course on refactoring. It offers a less tedious
approach to learning new stuff.

[ Let\'s see...](../refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Replace Conditional with
Polymorphism](replace-conditional-with-polymorphism.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Implements design pattern

:::: dl
::: dt
Null-object
:::
::::
:::::

::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Switch Statements](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Temporary Field](smells/temporary-field.html)
:::
::::
:::::::
::::::::::::::

::: next
#### 次を読む

[Introduce Assertion []{.fa
.fa-arrow-right}](introduce-assertion.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Replace Conditional with
Polymorphism](replace-conditional-with-polymorphism.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](../images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](../refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](../refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::
