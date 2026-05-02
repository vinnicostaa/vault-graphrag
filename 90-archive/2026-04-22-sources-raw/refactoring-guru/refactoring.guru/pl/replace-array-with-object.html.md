::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Replace Array with Object {#replace-array-with-object .title}

> This refactoring technique is a special case of [Replace Data Value
> with Object](replace-data-value-with-object.html).

::::: before-after-container
::: before
### Problem

You have an array that contains various types of data.
:::

::: after
### Solution

Replace the array with an object that will have separate fields for each
element.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
String[] row = new String[2];
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
Performance row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```
::::
:::::::

::::::: {.before-after .before-after-container .csharp data-lang="csharp"}
:::: before
::: ba
Before
:::

``` {.code lang="csharp"}
string[] row = new string[2];
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
Performance row = new Performance();
row.SetName("Liverpool");
row.SetWins("15");
```
::::
:::::::

::::::: {.before-after .before-after-container .php data-lang="php"}
:::: before
::: ba
Before
:::

``` {.code lang="php"}
$row = [];
$row[0] = "Liverpool";
$row[1] = 15;
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
$row = new Performance;
$row->setName("Liverpool");
$row->setWins(15);
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
row = [None * 2]
row[0] = "Liverpool"
row[1] = "15"
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
row = Performance()
row.setName("Liverpool")
row.setWins("15")
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
let row = new Array(2);
row[0] = "Liverpool";
row[1] = "15";
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
let row = new Performance();
row.setName("Liverpool");
row.setWins("15");
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Arrays are an excellent tool for storing data and collections of a
single type. But if you use an array like post office boxes, storing the
username in box 1 and the user's address in box 14, you will someday be
very unhappy that you did. This approach leads to catastrophic failures
when somebody puts something in the wrong "box" and also requires your
time for figuring out which data is stored where.

### Benefits

-   In the resulting class, you can place all associated behaviors that
    had been previously stored in the main class or elsewhere.

-   The fields of a class are much easier to document than the
    elements of an array.

### How to Refactor

1.  Create the new class that will contain the data from the array.
    Place the array itself in the class as a public field.

2.  Create a field for storing the object of this class in the original
    class. Don't forget to also create the object itself in the place
    where you initiated the data array.

3.  In the new class, create access methods one by one for each of the
    array elements. Give them self-explanatory names that indicate what
    they do. At the same time, replace each use of an array element in
    the main code with the corresponding access method.

4.  When access methods have been created for all elements, make the
    array private.

5.  For each element of the array, create a private field in the class
    and then change the access methods so that they use this field
    instead of the array.

6.  When all data has been moved, delete the array.

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
[Replace Data Value with Object](replace-data-value-with-object.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Eliminates smell

:::: dl
::: dt
[Primitive Obsession](smells/primitive-obsession.html)
:::
::::
:::::
:::::::::

::: next
#### Czytaj dalej

[Duplicate Observed Data []{.fa
.fa-arrow-right}](duplicate-observed-data.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Change Reference to
Value](change-reference-to-value.html){.btn .btn-default rel="prev"}
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
