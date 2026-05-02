:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Replace Constructor with Factory Method {#replace-constructor-with-factory-method .title}

::::: before-after-container
::: before
### Problem

You have a complex constructor that does something more than just
setting parameter values in object fields.
:::

::: after
### Solution

Create a factory method and use it to replace constructor calls.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
class Employee {
  Employee(int type) {
    this.type = type;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
class Employee {
  static Employee create(int type) {
    employee = new Employee(type);
    // do some heavy lifting.
    return employee;
  }
  // ...
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
public class Employee 
{
  public Employee(int type) 
  {
    this.type = type;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
public class Employee
{
  public static Employee Create(int type)
  {
    employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
  }
  // ...
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
class Employee {
  // ...
  public function __construct($type) {
   $this->type = $type;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
class Employee {
  // ...
  static public function create($type) {
    $employee = new Employee($type);
    // do some heavy lifting.
    return $employee;
  }
  // ...
}
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
class Employee {
  constructor(type: number) {
    this.type = type;
  }
  // ...
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
class Employee {
  static create(type: number): Employee {
    let employee = new Employee(type);
    // Do some heavy lifting.
    return employee;
  }
  // ...
}
```
::::
:::::::
:::::::::::::::::::::::

### Why Refactor

The most obvious reason for using this refactoring technique is
related to [Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html).

You have code in which a object was previously created and the value of
the coded type was passed to it. After use of the refactoring method,
several subclasses have appeared and from them you need to create
objects depending on the value of the coded type. Changing the original
constructor to make it return subclass objects is impossible, so
instead we create a static factory method that will return objects of
the necessary classes, after which it replaces all calls to the original
constructor.

Factory methods can be used in other situations as well, when
constructors aren't up to the task. They can be important when
attempting to [Change Value to
Reference](change-value-to-reference.html). They can also be used to set
various creation modes that go beyond the number and types of
parameters.

### Benefits

-   A factory method doesn't necessarily return an object of the
    class in which it was called. Often these could be its subclasses,
    selected based on the arguments given to the method.

-   A factory method can have a better name that describes what and
    how it returns what it does, for example `Troops::GetCrew(myTank)`.

-   A factory method can return an already created object, unlike a
    constructor, which always creates a new instance.

### How to Refactor

1.  Create a factory method. Place a call to the current constructor in
    it.

2.  Replace all constructor calls with calls to the factory method.

3.  Declare the constructor private.

4.  Investigate the constructor code and try to isolate the code not
    directly related to constructing an object of the current class,
    moving such code to the factory method.

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

::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Helps other refactorings

:::: dl
::: dt
[Change Value to Reference](change-value-to-reference.html)
:::
::::

:::: dl
::: dt
[Replace Type Code with
Subclasses](replace-type-code-with-subclasses.html)
:::
::::
:::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Implements design pattern

:::: dl
::: dt
[Metoda wytwórcza](design-patterns/factory-method.html)
:::
::::
:::::
:::::::::::

::: next
#### Czytaj dalej

[Replace Error Code with Exception []{.fa
.fa-arrow-right}](replace-error-code-with-exception.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Hide Method](hide-method.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
