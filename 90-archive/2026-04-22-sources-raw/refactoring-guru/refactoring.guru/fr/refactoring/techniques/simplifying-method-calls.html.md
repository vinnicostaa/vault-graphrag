::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} /
[Techniques](../techniques.html){.type}
:::

# Simplifying Method Calls {#simplifying-method-calls .title}

These techniques make method calls simpler and easier to understand.
This, in turn, simplifies the interfaces for interaction between
classes.

::::::::::::::::::::::::::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Rename Method](../../rename-method.html)
:::

::: dd
**Problem:** The name of a method doesn't explain what the method does.

**Solution:** Rename the method.
:::
:::::

::::: dl
::: dt
[Add Parameter](../../add-parameter.html)
:::

::: dd
**Problem:** A method doesn't have enough data to perform certain
actions.

**Solution:** Create a new parameter to pass the necessary data.
:::
:::::

::::: dl
::: dt
[Remove Parameter](../../remove-parameter.html)
:::

::: dd
**Problem:** A parameter isn't used in the body of a method.

**Solution:** Remove the unused parameter.
:::
:::::

::::: dl
::: dt
[Separate Query from Modifier](../../separate-query-from-modifier.html)
:::

::: dd
**Problem:** Do you have a method that returns a value but also changes
something inside an object?

**Solution:** Split the method into two separate methods. As you would
expect, one of them should return the value and the other one modifies
the object.
:::
:::::

::::: dl
::: dt
[Parameterize Method](../../parameterize-method.html)
:::

::: dd
**Problem:** Multiple methods perform similar actions that are different
only in their internal values, numbers or operations.

**Solution:** Combine these methods by using a parameter that will pass
the necessary special value.
:::
:::::

::::: dl
::: dt
[Replace Parameter with Explicit
Methods](../../replace-parameter-with-explicit-methods.html)
:::

::: dd
**Problem:** A method is split into parts, each of which is run
depending on the value of a parameter.

**Solution:** Extract the individual parts of the method into their own
methods and call them instead of the original method.
:::
:::::

::::: dl
::: dt
[Preserve Whole Object](../../preserve-whole-object.html)
:::

::: dd
**Problem:** You get several values from an object and then pass them as
parameters to a method.

**Solution:** Instead, try passing the whole object.
:::
:::::

::::: dl
::: dt
[Replace Parameter with Method
Call](../../replace-parameter-with-method-call.html)
:::

::: dd
**Problem:** Calling a query method and passing its results as the
parameters of another method, while that method could call the query
directly.

**Solution:** Instead of passing the value through a parameter, try
placing a query call inside the method body.
:::
:::::

::::: dl
::: dt
[Introduce Parameter Object](../../introduce-parameter-object.html)
:::

::: dd
**Problem:** Your methods contain a repeating group of parameters.

**Solution:** Replace these parameters with an object.
:::
:::::

::::: dl
::: dt
[Remove Setting Method](../../remove-setting-method.html)
:::

::: dd
**Problem:** The value of a field should be set only when it's created,
and not change at any time after that.

**Solution:** So remove methods that set the field's value.
:::
:::::

::::: dl
::: dt
[Hide Method](../../hide-method.html)
:::

::: dd
**Problem:** A method isn't used by other classes or is used only inside
its own class hierarchy.

**Solution:** Make the method private or protected.
:::
:::::

::::: dl
::: dt
[Replace Constructor with Factory
Method](../../replace-constructor-with-factory-method.html)
:::

::: dd
**Problem:** You have a complex constructor that does something more
than just setting parameter values in object fields.

**Solution:** Create a factory method and use it to replace constructor
calls.
:::
:::::

::::: dl
::: dt
[Replace Error Code with
Exception](../../replace-error-code-with-exception.html)
:::

::: dd
**Problem:** A method returns a special value that indicates an error?

**Solution:** Throw an exception instead.
:::
:::::

::::: dl
::: dt
[Replace Exception with Test](../../replace-exception-with-test.html)
:::

::: dd
**Problem:** You throw an exception in a place where a simple test would
do the job?

**Solution:** Replace the exception with a condition test.
:::
:::::
:::::::::::::::::::::::::::::::::::::::::::::

::: next
#### Suivant

[Rename Method []{.fa .fa-arrow-right}](../../rename-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Introduce
Assertion](../../introduce-assertion.html){.btn .btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
