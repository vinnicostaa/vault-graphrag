:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying
Conditional
Expressions](refactoring/techniques/simplifying-conditional-expressions.html){.type}
:::

# Introduce Assertion {#introduce-assertion .title}

::::: before-after-container
::: before
### Problem

For a portion of code to work correctly, certain conditions or values
must be true.
:::

::: after
### Solution

Replace these assumptions with specific assertion checks.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
double getExpenseLimit() {
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
double getExpenseLimit() {
  Assert.isTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
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
double GetExpenseLimit() 
{
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit :
    primaryProject.GetMemberExpenseLimit();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
double GetExpenseLimit() 
{
  Assert.IsTrue(expenseLimit != NULL_EXPENSE || primaryProject != null);

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.GetMemberExpenseLimit();
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
function getExpenseLimit() {
  // Should have either expense limit or
  // a primary project.
  return ($this->expenseLimit !== NULL_EXPENSE) ?
    $this->expenseLimit:
    $this->primaryProject->getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function getExpenseLimit() {
  assert($this->expenseLimit !== NULL_EXPENSE || isset($this->primaryProject));

  return ($this->expenseLimit !== NULL_EXPENSE) ?
    $this->expenseLimit:
    $this->primaryProject->getMemberExpenseLimit();
}
```
::::
:::::::

::::::: {.before-after .before-after-container .python data-lang="python"}
:::: before
::: ba
Before
:::

``` {.code lang="python"}
def getExpenseLimit(self):
    # Should have either expense limit or
    # a primary project.
    return self.expenseLimit if self.expenseLimit != NULL_EXPENSE else \
        self.primaryProject.getMemberExpenseLimit()
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def getExpenseLimit(self):
    assert (self.expenseLimit != NULL_EXPENSE) or (self.primaryProject != None)

    return self.expenseLimit if (self.expenseLimit != NULL_EXPENSE) else \
        self.primaryProject.getMemberExpenseLimit()
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
getExpenseLimit(): number {
  // Should have either expense limit or
  // a primary project.
  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
getExpenseLimit(): number {
  // TypeScript and JS doesn't have built-in assertions, so we'll use
  // good-old console.error(). You can always extract this into a
  // designated assertion function.
  if (!(expenseLimit != NULL_EXPENSE ||
       (typeof primaryProject !== 'undefined' && primaryProject))) {
      console.error("Assertion failed: getExpenseLimit()");
  }

  return (expenseLimit != NULL_EXPENSE) ?
    expenseLimit:
    primaryProject.getMemberExpenseLimit();
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Say that a portion of code assumes something about, for example, the
current condition of an object or value of a parameter or local
variable. Usually this assumption will always hold true except in the
event of an error.

Make these assumptions obvious by adding corresponding assertions. As
with type hinting in method parameters, these assertions can act as live
documentation for your code.

As a guideline to see where your code needs assertions, check for
comments that describe the conditions under which a particular method
will work.

### Benefits

-   If an assumption isn't true and the code therefore gives the wrong
    result, it's better to stop execution before this causes fatal
    consequences and data corruption. This also means that you neglected
    to write a necessary test when devising ways to perform testing of
    the program.

### Drawbacks

-   Sometimes an exception is more appropriate than a simple assertion.
    You can select the necessary class of the exception and let the
    remaining code handle it correctly.

-   When is an exception better than a simple assertion? If the
    exception can be caused by actions of the user or system and you can
    handle the exception. On the other hand, ordinary unnamed and
    unhandled exceptions are basically equivalent to simple
    assertions---you don't handle them and they're caused exclusively as
    the result of a program bug that never should have occurred.

### How to Refactor

When you see that a condition is assumed, add an assertion for this
condition in order to make sure.

Adding the assertion shouldn't change the program's behavior.

Don't overdo it with use of assertions for **everything** in your code.
Check for only the conditions that are necessary for correct functioning
of the code. If your code is working normally even when a particular
assertion is false, you can safely remove the assertion.

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
[Comments](smells/comments.html)
:::
::::
:::::
::::::

::: next
#### Leia a seguir

[Simplifying Method Calls []{.fa
.fa-arrow-right}](refactoring/techniques/simplifying-method-calls.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Introduce Null
Object](introduce-null-object.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
