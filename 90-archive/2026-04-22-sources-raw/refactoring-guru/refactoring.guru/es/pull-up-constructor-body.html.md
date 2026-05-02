::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Pull Up Constructor Body {#pull-up-constructor-body .title}

::::: before-after-container
::: before
### Problem

Your subclasses have constructors with code that's mostly identical.
:::

::: after
### Solution

Create a superclass constructor and move the code that's the same in the
subclasses to it. Call the superclass constructor in the subclass
constructors.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
class Manager extends Employee {
  public Manager(String name, String id, int grade) {
    super(name, id);
    this.grade = grade;
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
public class Manager: Employee 
{
  public Manager(string name, string id, int grade) 
  {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
public class Manager: Employee 
{
  public Manager(string name, string id, int grade): base(name, id)
  {
    this.grade = grade;
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
class Manager extends Employee {
  public function __construct($name, $id, $grade) {
    $this->name = $name;
    $this->id = $id;
    $this->grade = $grade;
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
class Manager extends Employee {
  public function __construct($name, $id, $grade) {
    parent::__construct($name, $id);
    $this->grade = $grade;
  }
  // ...
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
class Manager(Employee):
    def __init__(self, name, id, grade):
        self.name = name
        self.id = id
        self.grade = grade
    # ...
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
class Manager(Employee):
    def __init__(self, name, id, grade):
        Employee.__init__(name, id)
        self.grade = grade
    # ...
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    this.name = name;
    this.id = id;
    this.grade = grade;
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
class Manager extends Employee {
  constructor(name: string, id: string, grade: number) {
    super(name, id);
    this.grade = grade;
  }
  // ...
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

How is this refactoring technique different from [Pull Up
Method](pull-up-method.html)?

1.  In Java, subclasses can't inherit a constructor, so you can't simply
    apply [Pull Up Method](pull-up-method.html) to the subclass
    constructor and delete it after removing all the constructor code to
    the superclass. In addition to creating a constructor in the
    superclass it's necessary to have constructors in the subclasses
    with simple delegation to the superclass constructor.

2.  In C++ and Java (if you didn't explicitly call the superclass
    constructor) the superclass constructor is automatically called
    prior to the subclass constructor, which makes it necessary to move
    the common code only from the beginning of the subclass constructors
    (since you won't be able to call the superclass constructor from an
    arbitrary place in a subclass constructor).

3.  In most programming languages, a subclass constructor can have its
    own list of parameters different from the parameters of the
    superclass. Therefore you should create a superclass constructor
    only with the parameters that it truly needs.

### How to Refactor

1.  Create a constructor in a superclass.

2.  Extract the common code from the beginning of the constructor of
    each subclass to the superclass constructor. Before doing so, try to
    move as much common code as possible to the beginning of the
    constructor.

3.  Place the call for the superclass constructor in the first line in
    the subclass constructors.

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

::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

:::: dl
::: dt
[Pull Up Method](pull-up-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::
:::::
:::::::::

::: next
#### Leer siguiente

[Push Down Method []{.fa .fa-arrow-right}](push-down-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Pull Up Method](pull-up-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
