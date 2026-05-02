:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Replace Method with Method Object {#replace-method-with-method-object .title}

::::: before-after-container
::: before
### Problem

You have a long method in which the local variables are so intertwined
that you can't apply *Extract Method*.
:::

::: after
### Solution

Transform the method into a separate class so that the local variables
become fields of the class. Then you can split the method into several
methods within the same class.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
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
After
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
Before
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
After
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
Before
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
After
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
Before
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
After
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
Before
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
After
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

### Why Refactor

A method is too long and you can't separate it due to tangled masses of
local variables that are hard to isolate from each other.

The first step is to isolate the entire method into a separate class and
turn its local variables into fields of the class.

Firstly, this allows isolating the problem at the class level. Secondly,
it paves the way for splitting a large and unwieldy method into smaller
ones that wouldn't fit with the purpose of the original class anyway.

### Benefits

-   Isolating a long method in its own class allows stopping a method
    from ballooning in size. This also allows splitting it into
    submethods within the class, without polluting the original class
    with utility methods.

### Drawbacks

-   Another class is added, increasing the overall complexity of the
    program.

### How to Refactor

1.  Create a new class. Name it based on the purpose of the method that
    you're refactoring.

2.  In the new class, create a private field for storing a reference to
    an instance of the class in which the method was previously located.
    It could be used to get some required data from the original class
    if needed.

3.  Create a separate private field for each local variable of the
    method.

4.  Create a constructor that accepts as parameters the values of all
    local variables of the method and also initializes the corresponding
    private fields.

5.  Declare the main method and copy the code of the original method to
    it, replacing the local variables with private fields.

6.  Replace the body of the original method in the original class by
    creating a method object and calling its main method.

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

::: next
#### Suivant

[Substitute Algorithm []{.fa
.fa-arrow-right}](substitute-algorithm.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Remove Assignments to
Parameters](remove-assignments-to-parameters.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
