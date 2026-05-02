:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Simplifying Method
Calls](refactoring/techniques/simplifying-method-calls.html){.type}
:::

# Replace Error Code with Exception {#replace-error-code-with-exception .title}

::::: before-after-container
::: before
### Problem

A method returns a special value that indicates an error?
:::

::: after
### Solution

Throw an exception instead.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
int withdraw(int amount) {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
void withdraw(int amount) throws BalanceException {
  if (amount > _balance) {
    throw new BalanceException();
  }
  balance -= amount;
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
int Withdraw(int amount) 
{
  if (amount > _balance) 
  {
    return -1;
  }
  else 
  {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
///<exception cref="BalanceException">Thrown when amount > _balance</exception>
void Withdraw(int amount)
{
  if (amount > _balance) 
  {
    throw new BalanceException();
  }
  balance -= amount;
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
function withdraw($amount) {
  if ($amount > $this->balance) {
    return -1;
  } else {
    $this->balance -= $amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function withdraw($amount) {
  if ($amount > $this->balance) {
    throw new BalanceException;
  }
  $this->balance -= $amount;
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
def withdraw(self, amount):
    if amount > self.balance:
        return -1
    else:
        self.balance -= amount
    return 0
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def withdraw(self, amount):
    if amount > self.balance:
        raise BalanceException()
    self.balance -= amount
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
withdraw(amount: number): number {
  if (amount > _balance) {
    return -1;
  }
  else {
    balance -= amount;
    return 0;
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
withdraw(amount: number): void {
  if (amount > _balance) {
    throw new Error();
  }
  balance -= amount;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

Returning error codes is an obsolete holdover from procedural
programming. In modern programming, error handling is performed by
special classes, which are named exceptions. If a problem occurs, you
"throw" an error, which is then "caught" by one of the exception
handlers. Special error-handling code, which is ignored in normal
conditions, is activated to respond.

### Benefits

-   Frees code from a large number of conditionals for checking various
    error codes. Exception handlers are a much more succinct way to
    differentiate normal execution paths from abnormal ones.

-   Exception classes can implement their own methods, thus containing
    part of the error handling functionality (such as for sending error
    messages).

-   Unlike exceptions, error codes can't be used in a constructor, since
    a constructor must return only a new object.

### Drawbacks

-   An exception handler can turn into a goto-like crutch. Avoid this!
    Don't use exceptions to manage code execution. Exceptions should be
    thrown only to inform of an error or critical situation.

### How to Refactor

Try to perform these refactoring steps for only one error code at a
time. This will make it easier to keep all the important information in
your head and avoid errors.

1.  Find all calls to a method that returns error codes and, instead of
    checking for an error code, wrap it in `try`/`catch` blocks.

2.  Inside the method, instead of returning an error code, throw an
    exception.

3.  Change the method signature so that it contains information about
    the exception being thrown (`@throws` section).

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

::: next
#### Leia a seguir

[Replace Exception with Test []{.fa
.fa-arrow-right}](replace-exception-with-test.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Replace Constructor with Factory
Method](replace-constructor-with-factory-method.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::
