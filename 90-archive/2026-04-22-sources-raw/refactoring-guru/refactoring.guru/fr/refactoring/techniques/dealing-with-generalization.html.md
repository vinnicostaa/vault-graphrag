::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} /
[Techniques](../techniques.html){.type}
:::

# Dealing with Generalization {#dealing-with-generalization .title}

Abstraction has its own group of refactoring techniques, primarily
associated with moving functionality along the class inheritance
hierarchy, creating new classes and interfaces, and replacing
inheritance with delegation and vice versa.

::::::::::::::::::::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Pull Up Field](../../pull-up-field.html)
:::

::: dd
**Problem:** Two classes have the same field.

**Solution:** Remove the field from subclasses and move it to the
superclass.
:::
:::::

::::: dl
::: dt
[Pull Up Method](../../pull-up-method.html)
:::

::: dd
**Problem:** Your subclasses have methods that perform similar work.

**Solution:** Make the methods identical and then move them to the
relevant superclass.
:::
:::::

::::: dl
::: dt
[Pull Up Constructor Body](../../pull-up-constructor-body.html)
:::

::: dd
**Problem:** Your subclasses have constructors with code that's mostly
identical.

**Solution:** Create a superclass constructor and move the code that's
the same in the subclasses to it. Call the superclass constructor in the
subclass constructors.
:::
:::::

::::: dl
::: dt
[Push Down Method](../../push-down-method.html)
:::

::: dd
**Problem:** Is behavior implemented in a superclass used by only one
(or a few) subclasses?

**Solution:** Move this behavior to the subclasses.
:::
:::::

::::: dl
::: dt
[Push Down Field](../../push-down-field.html)
:::

::: dd
**Problem:** Is a field used only in a few subclasses?

**Solution:** Move the field to these subclasses.
:::
:::::

::::: dl
::: dt
[Extract Subclass](../../extract-subclass.html)
:::

::: dd
**Problem:** A class has features that are used only in certain cases.

**Solution:** Create a subclass and use it in these cases.
:::
:::::

::::: dl
::: dt
[Extract Superclass](../../extract-superclass.html)
:::

::: dd
**Problem:** You have two classes with common fields and methods.

**Solution:** Create a shared superclass for them and move all the
identical fields and methods to it.
:::
:::::

::::: dl
::: dt
[Extract Interface](../../extract-interface.html)
:::

::: dd
**Problem:** Multiple clients are using the same part of a class
interface. Another case: part of the interface in two classes is the
same.

**Solution:** Move this identical portion to its own interface.
:::
:::::

::::: dl
::: dt
[Collapse Hierarchy](../../collapse-hierarchy.html)
:::

::: dd
**Problem:** You have a class hierarchy in which a subclass is
practically the same as its superclass.

**Solution:** Merge the subclass and superclass.
:::
:::::

::::: dl
::: dt
[Form Template Method](../../form-template-method.html)
:::

::: dd
**Problem:** Your subclasses implement algorithms that contain similar
steps in the same order.

**Solution:** Move the algorithm structure and identical steps to a
superclass, and leave implementation of the different steps in the
subclasses.
:::
:::::

::::: dl
::: dt
[Replace Inheritance with
Delegation](../../replace-inheritance-with-delegation.html)
:::

::: dd
**Problem:** You have a subclass that uses only a portion of the methods
of its superclass (or it's not possible to inherit superclass data).

**Solution:** Create a field and put a superclass object in it, delegate
methods to the superclass object, and get rid of inheritance.
:::
:::::

::::: dl
::: dt
[Replace Delegation with
Inheritance](../../replace-delegation-with-inheritance.html)
:::

::: dd
**Problem:** A class contains many simple methods that delegate to all
methods of another class.

**Solution:** Make the class a delegate inheritor, which makes the
delegating methods unnecessary.
:::
:::::
:::::::::::::::::::::::::::::::::::::::

::: next
#### Suivant

[Pull Up Field []{.fa .fa-arrow-right}](../../pull-up-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Replace Exception with
Test](../../replace-exception-with-test.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
