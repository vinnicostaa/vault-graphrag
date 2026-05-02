::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::: {.page .text}
::: breadcrumb
[](../../../index.html){.home} /
[Refactoring](../../refactoring.html){.type} /
[Techniques](../techniques.html){.type}
:::

# Simplifying Conditional Expressions {#simplifying-conditional-expressions .title}

Conditionals tend to get more and more complicated in their logic over
time, and there are yet more techniques to combat this as well.

::::::::::::::::::::::::::: {.relations .link-list}
::::: dl
::: dt
[Decompose Conditional](../../decompose-conditional.html)
:::

::: dd
**Problem:** You have a complex conditional (`if-then`/`else` or
`switch`).

**Solution:** Decompose the complicated parts of the conditional into
separate methods: the condition, `then` and `else`.
:::
:::::

::::: dl
::: dt
[Consolidate Conditional
Expression](../../consolidate-conditional-expression.html)
:::

::: dd
**Problem:** You have multiple conditionals that lead to the same
result or action.

**Solution:** Consolidate all these conditionals in a single expression.
:::
:::::

::::: dl
::: dt
[Consolidate Duplicate Conditional
Fragments](../../consolidate-duplicate-conditional-fragments.html)
:::

::: dd
**Problem:** Identical code can be found in all branches of a
conditional.

**Solution:** Move the code outside of the conditional.
:::
:::::

::::: dl
::: dt
[Remove Control Flag](../../remove-control-flag.html)
:::

::: dd
**Problem:** You have a boolean variable that acts as a control flag for
multiple boolean expressions.

**Solution:** Instead of the variable, use `break`, `continue` and
`return`.
:::
:::::

::::: dl
::: dt
[Replace Nested Conditional with Guard
Clauses](../../replace-nested-conditional-with-guard-clauses.html)
:::

::: dd
**Problem:** You have a group of nested conditionals and it's hard to
determine the normal flow of code execution.

**Solution:** Isolate all special checks and edge cases into separate
clauses and place them before the main checks. Ideally, you should
have a "flat" list of conditionals, one after the other.
:::
:::::

::::: dl
::: dt
[Replace Conditional with
Polymorphism](../../replace-conditional-with-polymorphism.html)
:::

::: dd
**Problem:** You have a conditional that performs various actions
depending on object type or properties.

**Solution:** Create subclasses matching the branches of the
conditional. In them, create a shared method and move code from the
corresponding branch of the conditional to it. Then replace the
conditional with the relevant method call. The result is that the proper
implementation will be attained via polymorphism depending on the object
class.
:::
:::::

::::: dl
::: dt
[Introduce Null Object](../../introduce-null-object.html)
:::

::: dd
**Problem:** Since some methods return `null` instead of real objects,
you have many checks for `null` in your code.

**Solution:** Instead of `null`, return a null object that exhibits the
default behavior.
:::
:::::

::::: dl
::: dt
[Introduce Assertion](../../introduce-assertion.html)
:::

::: dd
**Problem:** For a portion of code to work correctly, certain
conditions or values must be true.

**Solution:** Replace these assumptions with specific assertion checks.
:::
:::::
:::::::::::::::::::::::::::

::: next
#### Czytaj dalej

[Decompose Conditional []{.fa
.fa-arrow-right}](../../decompose-conditional.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Replace Subclass with
Fields](../../replace-subclass-with-fields.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
