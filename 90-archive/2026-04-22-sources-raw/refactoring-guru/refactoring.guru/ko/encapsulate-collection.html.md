:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Organizing
Data](refactoring/techniques/organizing-data.html){.type}
:::

# Encapsulate Collection {#encapsulate-collection .title}

::::: before-after-container
::: before
### Problem

A class contains a collection field and a simple getter and setter for
working with the collection.
:::

::: after
### Solution

Make the getter-returned value read-only and create methods for
adding/deleting elements of the collection.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Encapsulate Collection -
Before](../images/refactoring/diagrams/Encapsulate%20Collection%20-%20Before4b9f.png?id=0c01860495c763043a13ce47086e49f1){width="227"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Encapsulate Collection -
After](../images/refactoring/diagrams/Encapsulate%20Collection%20-%20After0ac4.png?id=e8f5624c8d63963421680121d49a43de){width="284"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

A class contains a field that contains a collection of objects. This
collection could be an array, list, set or vector. A normal getter and
setter have been created for working with the collection.

But the collections should be used by a protocol that\'s a bit different
from the one used by other data types. The getter method shouldn\'t
return the collection object itself, since this would let clients change
collection contents without the knowledge of the owner class. In
addition, this would show too much of the internal structures of the
object data to clients. The method for getting collection elements
should return a value that doesn\'t allow changing the collection or
disclose excessive data about its structure.

In addition, there shouldn\'t be a method that assigns a value to the
collection. Instead, there should be operations for adding and deleting
elements. Thanks to this, the owner object gains control over addition
and deletion of collection elements.

Such a protocol properly encapsulates a collection, which ultimately
reduces the degree of association between the owner class and the client
code.

### Benefits

-   The collection field is encapsulated inside a class. When the getter
    is called, it returns a copy of the collection, which prevents
    accidental changing or overwriting of the collection elements
    without the knowledge of the class that contains the collection.

-   If collection elements are contained inside a primitive type, such
    as an array, you create more convenient methods for working with the
    collection.

-   If collection elements are contained inside a non-primitive
    container (standard collection class), by encapsulating the
    collection you can restrict access to unwanted standard methods of
    the collection (such as by restricting addition of new elements).

### How to Refactor

1.  Create methods for adding and deleting collection elements. They
    must accept collection elements in their parameters.

2.  Assign an empty collection to the field as the initial value if this
    isn\'t done in the class constructor.

3.  Find the calls of the collection field setter. Change the setter so
    that it uses operations for adding and deleting elements, or make
    these operations call client code.

Note that setters can be used only to replace all collection elements
with other ones. Therefore it may be advisable to change the setter name
([Rename Method](rename-method.html)) to `replace`.

4.  Find all calls of the collection getter after which the collection
    is changed. Change the code so that it uses your new methods for
    adding and deleting elements from the collection.

5.  Change the getter so that it returns a read-only representation of
    the collection.

6.  Inspect the client code that uses the collection for code that would
    look better inside of the collection class itself.

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

:::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Eliminates smell

:::: dl
::: dt
[Data Class](smells/data-class.html)
:::
::::
:::::
::::::

::: next
#### 다음을 보세요

[Replace Type Code with Class []{.fa
.fa-arrow-right}](replace-type-code-with-class.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Encapsulate Field](encapsulate-field.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
