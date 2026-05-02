::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Dealing with
Generalization](refactoring/techniques/dealing-with-generalization.html){.type}
:::

# Replace Delegation with Inheritance {#replace-delegation-with-inheritance .title}

::::: before-after-container
::: before
### Problem

A class contains many simple methods that delegate to all methods of
another class.
:::

::: after
### Solution

Make the class a delegate inheritor, which makes the delegating methods
unnecessary.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Replace Delegation with Inheritance -
Before](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20Before7d55.png?id=b408bcc35ce5a341e675818d6c1124a2){width="453"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Replace Delegation with Inheritance -
After](../images/refactoring/diagrams/Replace%20Delegation%20with%20Inheritance%20-%20After3a88.png?id=633a1c3ff0e87033e10408df57ac6118){width="152"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

Delegation is a more flexible approach than inheritance, since it allows
changing how delegation is implemented and placing other classes there
as well. Nonetheless, delegation stops being beneficial if you delegate
actions to only one class and all of its public methods.

In such a case, if you replace delegation with inheritance, you cleanse
the class of a large number of delegating methods and spare yourself
from needing to create them for each new delegate class method.

### Benefits

-   Reduces code length. All these delegating methods are no longer
    necessary.

### When Not to Use

-   Don\'t use this technique if the class contains delegation to only a
    portion of the public methods of the delegate class. By doing so,
    you would violate the *Liskov substitution principle*.

-   This technique can be used only if the class still doesn\'t have
    parents.

### How to Refactor

1.  Make the class a subclass of the delegate class.

2.  Place the current object in a field containing a reference to the
    delegate object.

3.  Delete the methods with simple delegation one by one. If their names
    were different, use [Rename Method](rename-method.html) to give all
    the methods a single name.

4.  Replace all references to the delegate field with references to the
    current object.

5.  Remove the delegate field.

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

:::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Anti-refactoring

:::: dl
::: dt
[Replace Inheritance with
Delegation](replace-inheritance-with-delegation.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Remove Middle Man](remove-middle-man.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::
:::::
::::::::::::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Replace Inheritance with
Delegation](replace-inheritance-with-delegation.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::
