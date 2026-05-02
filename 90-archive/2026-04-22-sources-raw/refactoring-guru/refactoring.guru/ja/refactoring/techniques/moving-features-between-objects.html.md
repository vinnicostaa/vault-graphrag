::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} /
[Techniques](../techniques.html){.type}
:::

# Moving Features between Objects {#moving-features-between-objects .title}

Even if you have distributed functionality among different classes in a
less-than-perfect way, there\'s still hope.

These refactoring techniques show how to safely move functionality
between classes, create new classes, and hide implementation details
from public access.

::::::::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Move Method](../../move-method.html)
:::

::: dd
**Problem:** A method is used more in another class than in its own
class.

**Solution:** Create a new method in the class that uses the method the
most, then move code from the old method to there. Turn the code of the
original method into a reference to the new method in the other class or
else remove it entirely.
:::
:::::

::::: dl
::: dt
[Move Field](../../move-field.html)
:::

::: dd
**Problem:** A field is used more in another class than in its own
class.

**Solution:** Create a field in a new class and redirect all users of
the old field to it.
:::
:::::

::::: dl
::: dt
[Extract Class](../../extract-class.html)
:::

::: dd
**Problem:** When one class does the work of two, awkwardness results.

**Solution:** Instead, create a new class and place the fields and
methods responsible for the relevant functionality in it.
:::
:::::

::::: dl
::: dt
[Inline Class](../../inline-class.html)
:::

::: dd
**Problem:** A class does almost nothing and isn\'t responsible for
anything, and no additional responsibilities are planned for it.

**Solution:** Move all features from the class to another one.
:::
:::::

::::: dl
::: dt
[Hide Delegate](../../hide-delegate.html)
:::

::: dd
**Problem:** The client gets object B from a field or method of object
А. Then the client calls a method of object B.

**Solution:** Create a new method in class A that delegates the call to
object B. Now the client doesn\'t know about, or depend on, class B.
:::
:::::

::::: dl
::: dt
[Remove Middle Man](../../remove-middle-man.html)
:::

::: dd
**Problem:** A class has too many methods that simply delegate to other
objects.

**Solution:** Delete these methods and force the client to call the end
methods directly.
:::
:::::

::::: dl
::: dt
[Introduce Foreign Method](../../introduce-foreign-method.html)
:::

::: dd
**Problem:** A utility class doesn\'t contain the method that you need
and you can\'t add the method to the class.

**Solution:** Add the method to a client class and pass an object of the
utility class to it as an argument.
:::
:::::

::::: dl
::: dt
[Introduce Local Extension](../../introduce-local-extension.html)
:::

::: dd
**Problem:** A utility class doesn\'t contain some methods that you
need. But you can\'t add these methods to the class.

**Solution:** Create a new class containing the methods and make it
either the child or wrapper of the utility class.
:::
:::::
:::::::::::::::::::::::::::

::: next
#### 次を読む

[Move Method []{.fa .fa-arrow-right}](../../move-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### 戻る

[[]{.fa .fa-arrow-left} Substitute
Algorithm](../../substitute-algorithm.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
