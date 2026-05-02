:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Move Method {#move-method .title}

::::: before-after-container
::: before
### Problem

A method is used more in another class than in its own class.
:::

::: after
### Solution

Create a new method in the class that uses the method the most, then
move code from the old method to there. Turn the code of the original
method into a reference to the new method in the other class or else
remove it entirely.
:::
:::::

:::::::::: examples
::::::::: {.before-after .before-after-container}
::::: before
::: ba
Before
:::

::: ba-container
![Move Method -
Before](../images/refactoring/diagrams/Move%20Method%20-%20Before2a62.png?id=4abd2ccfc636cdcffe30f5593868bbfa){width="134"}
:::
:::::

::::: after
::: ba
After
:::

::: ba-container
![Move Method -
After](../images/refactoring/diagrams/Move%20Method%20-%20Afterfe74.png?id=df9892b4d0a367b12881e93e7d0ea0c0){width="134"}
:::
:::::
:::::::::
::::::::::

### Why Refactor

1.  You want to move a method to a class that contains most of the data
    used by the method. This makes **classes more internally coherent**.

2.  You want to move a method in order to reduce or eliminate the
    dependency of the class calling the method on the class in which
    it\'s located. This can be useful if the calling class is already
    dependent on the class to which you\'re planning to move the method.
    This **reduces dependency between classes**.

### How to Refactor

1.  Verify all features used by the old method in its class. It may be a
    good idea to move them as well. As a rule, if a feature is used only
    by the method under consideration, you should certainly move the
    feature to it. If the feature is used by other methods too, you
    should move these methods as well. Sometimes it\'s much easier to
    move a large number of methods than to set up relationships between
    them in different classes.

    Make sure that the method isn\'t declared in superclasses and
    subclasses. If this is the case, you will either have to refrain
    from moving or else implement a kind of polymorphism in the
    recipient class in order to ensure varying functionality of a method
    split up among donor classes.

2.  Declare the new method in the recipient class. You may want to give
    a new name for the method that\'s more appropriate for it in the new
    class.

3.  Decide how you will refer to the recipient class. You may already
    have a field or method that returns an appropriate object, but if
    not, you will need to write a new method or field to store the
    object of the recipient class.

    Now you have a way to refer to the recipient object and a new method
    in its class. With all this under your belt, you can turn the old
    method into a reference to the new method.

4.  Take a look: can you delete the old method entirely? If so, place a
    reference to the new method in all places that use the old one.

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

:::::::::::::::::::::::::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Similar refactorings

:::: dl
::: dt
[Extract Method](extract-method.html)
:::
::::

:::: dl
::: dt
[Move Field](move-field.html)
:::
::::
:::::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Helps other refactorings

:::: dl
::: dt
[Extract Class](extract-class.html)
:::
::::

:::: dl
::: dt
[Inline Class](inline-class.html)
:::
::::

:::: dl
::: dt
[Introduce Parameter Object](introduce-parameter-object.html)
:::
::::
:::::::::

::::::::::::::::: {.col-xs-12 .col-sm-6 .col-lg-4}
### Eliminates smell

:::: dl
::: dt
[Shotgun Surgery](smells/shotgun-surgery.html)
:::
::::

:::: dl
::: dt
[Feature Envy](smells/feature-envy.html)
:::
::::

:::: dl
::: dt
[Switch Statements](smells/switch-statements.html)
:::
::::

:::: dl
::: dt
[Parallel Inheritance
Hierarchies](smells/parallel-inheritance-hierarchies.html)
:::
::::

:::: dl
::: dt
[Message Chains](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Inappropriate Intimacy](smells/inappropriate-intimacy.html)
:::
::::

:::: dl
::: dt
[Data Class](smells/data-class.html)
:::
::::
:::::::::::::::::
::::::::::::::::::::::::::::::

::: next
#### 다음을 보세요

[Move Field []{.fa .fa-arrow-right}](move-field.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Moving Features between
Objects](refactoring/techniques/moving-features-between-objects.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::
