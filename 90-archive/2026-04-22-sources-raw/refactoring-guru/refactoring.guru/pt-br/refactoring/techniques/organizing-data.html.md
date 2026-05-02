:::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} /
[Techniques](../techniques.html){.type}
:::

# Organizing Data {#organizing-data .title}

These refactoring techniques help with data handling, replacing
primitives with rich class functionality.

Another important result is untangling of class associations, which
makes classes more portable and reusable.

:::::::::::::::::::::::::::::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Self Encapsulate Field](../../self-encapsulate-field.html)
:::

::: dd
**Problem:** You use direct access to private fields inside a class.

**Solution:** Create a getter and setter for the field, and use only
them for accessing the field.
:::
:::::

::::: dl
::: dt
[Replace Data Value with
Object](../../replace-data-value-with-object.html)
:::

::: dd
**Problem:** A class (or group of classes) contains a data field. The
field has its own behavior and associated data.

**Solution:** Create a new class, place the old field and its behavior
in the class, and store the object of the class in the original class.
:::
:::::

::::: dl
::: dt
[Change Value to Reference](../../change-value-to-reference.html)
:::

::: dd
**Problem:** So you have many identical instances of a single class that
you need to replace with a single object.

**Solution:** Convert the identical objects to a single reference
object.
:::
:::::

::::: dl
::: dt
[Change Reference to Value](../../change-reference-to-value.html)
:::

::: dd
**Problem:** You have a reference object that's too small and
infrequently changed to justify managing its life cycle.

**Solution:** Turn it into a value object.
:::
:::::

::::: dl
::: dt
[Replace Array with Object](../../replace-array-with-object.html)
:::

::: dd
**Problem:** You have an array that contains various types of data.

**Solution:** Replace the array with an object that will have separate
fields for each element.
:::
:::::

::::: dl
::: dt
[Duplicate Observed Data](../../duplicate-observed-data.html)
:::

::: dd
**Problem:** Is domain data stored in classes responsible for the GUI?

**Solution:** Then it's a good idea to separate the data into separate
classes, ensuring connection and synchronization between the domain
class and the GUI.
:::
:::::

::::: dl
::: dt
[Change Unidirectional Association to
Bidirectional](../../change-unidirectional-association-to-bidirectional.html)
:::

::: dd
**Problem:** You have two classes that each need to use the features of
the other, but the association between them is only unidirectional.

**Solution:** Add the missing association to the class that needs it.
:::
:::::

::::: dl
::: dt
[Change Bidirectional Association to
Unidirectional](../../change-bidirectional-association-to-unidirectional.html)
:::

::: dd
**Problem:** You have a bidirectional association between classes, but
one of the classes doesn't use the other's features.

**Solution:** Remove the unused association.
:::
:::::

::::: dl
::: dt
[Replace Magic Number with Symbolic
Constant](../../replace-magic-number-with-symbolic-constant.html)
:::

::: dd
**Problem:** Your code uses a number that has a certain meaning to it.

**Solution:** Replace this number with a constant that has a
human-readable name explaining the meaning of the number.
:::
:::::

::::: dl
::: dt
[Encapsulate Field](../../encapsulate-field.html)
:::

::: dd
**Problem:** You have a public field.

**Solution:** Make the field private and create access methods for it.
:::
:::::

::::: dl
::: dt
[Encapsulate Collection](../../encapsulate-collection.html)
:::

::: dd
**Problem:** A class contains a collection field and a simple getter and
setter for working with the collection.

**Solution:** Make the getter-returned value read-only and create
methods for adding/deleting elements of the collection.
:::
:::::

::::: dl
::: dt
[Replace Type Code with Class](../../replace-type-code-with-class.html)
:::

::: dd
**Problem:** A class has a field that contains type code. The values of
this type aren't used in operator conditions and don't affect the
behavior of the program.

**Solution:** Create a new class and use its objects instead of the type
code values.
:::
:::::

::::: dl
::: dt
[Replace Type Code with
Subclasses](../../replace-type-code-with-subclasses.html)
:::

::: dd
**Problem:** You have a coded type that directly affects program
behavior (values of this field trigger various code in conditionals).

**Solution:** Create subclasses for each value of the coded type. Then
extract the relevant behaviors from the original class to these
subclasses. Replace the control flow code with polymorphism.
:::
:::::

::::: dl
::: dt
[Replace Type Code with
State/Strategy](../../replace-type-code-with-state-strategy.html)
:::

::: dd
**Problem:** You have a coded type that affects behavior but you can't
use subclasses to get rid of it.

**Solution:** Replace type code with a state object. If it's necessary
to replace a field value with type code, another state object is
"plugged in".
:::
:::::

::::: dl
::: dt
[Replace Subclass with Fields](../../replace-subclass-with-fields.html)
:::

::: dd
**Problem:** You have subclasses differing only in their
(constant-returning) methods.

**Solution:** Replace the methods with fields in the parent class and
delete the subclasses.
:::
:::::
::::::::::::::::::::::::::::::::::::::::::::::::

::: next
#### Leia a seguir

[Self Encapsulate Field []{.fa
.fa-arrow-right}](../../self-encapsulate-field.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Introduce Local
Extension](../../introduce-local-extension.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::
