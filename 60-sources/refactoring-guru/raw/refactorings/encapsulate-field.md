::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Encapsulate Field {#encapsulate-field .title}

::::: before-after-container
::: before
### Problem

You have a public field.
:::

::: after
### Solution

Make the field private and create access methods for it.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
class Person {
  public String name;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
class Person {
  private String name;

  public String getName() {
    return name;
  }
  public void setName(String arg) {
    name = arg;
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
class Person 
{
  public string name;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
class Person 
{
  private string name;

  public string Name
  {
    get { return name; }
    set { name = value; }
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
public $name;
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
private $name;

public getName() {
  return $this->name;
}

public setName($arg) {
  $this->name = $arg;
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
class Person {
  name: string;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
class Person {
  private _name: string;

  get name() {
    return this._name;
  }
  setName(arg: string): void {
    this._name = arg;
  }
}
```
::::
:::::::
:::::::::::::::::::::::

### Why Refactor

One of the pillars of object-oriented programming is *Encapsulation*,
the ability to conceal object data. Otherwise, all objects would be
public and other objects could get and modify the data of your object
without any checks and balances! Data is separated from the behaviors
associated with this data, modularity of program sections is
compromised, and maintenance becomes complicated.

### Benefits

-   If the data and behavior of a component are closely interrelated and
    are in the same place in the code, it's much easier for you to
    maintain and develop this component.

-   You can also perform complicated operations related to access to
    object fields.

### When Not to Use

-   In some cases, encapsulation is ill-advised due to performance
    considerations. These cases are rare but when they happen, this
    circumstance is very important.

    Say that you have a graphical editor that contains objects
    possessing x- and y-coordinates. These fields are unlikely to change
    in the future. What's more, the program involves a great many
    different objects in which these fields are present. So accessing
    the coordinate fields directly saves significant CPU cycles that
    would otherwise be taken up by calling access methods.

    As an example of this unusual case, there's the
    [Point](http://docs.oracle.com/javase/7/docs/api/java/awt/Point.html)
    class in Java. All fields of this class are public.

### How to Refactor

1.  Create a getter and setter for the field.

2.  Find all invocations of the field. Replace receipt of the field
    value with the getter, and replace setting of new field values with
    the setter.

3.  After all field invocations have been replaced, make the field
    private.

#### Next Steps

*Encapsulate Field* is only the first step in bringing data and the
behaviors involving this data closer together. After you create simple
methods for access fields, you should recheck the places where these
methods are called. It's quite possible that the code in these areas
would look more appropriate in the access methods.

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

[ Let\'s see...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Similar refactorings

::::: dl
::: dt
[Self Encapsulate Field](self-encapsulate-field.html)
:::

::: dd
Create getters and setters for a field instead of direct access *within
the class\' methods*.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Data Class](smells/data-class.html)
:::
::::
:::::
::::::::::

::: next
#### Read next

[Encapsulate Collection []{.fa
.fa-arrow-right}](encapsulate-collection.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Return

[[]{.fa .fa-arrow-left} Replace Magic Number with Symbolic
Constant](replace-magic-number-with-symbolic-constant.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-en" position="sidebar"}
::::: ban
::: {.image .product-image}
[Sale!]{.banner-discount}
[![](images/refactoring/course/course-cover-en1b63.jpg?id=ac6ad836a27c4a8d579244f15c48a8b4){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-en}
This refactoring is part of the much bigger **Refactoring Course**.

[ Learn more...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::
