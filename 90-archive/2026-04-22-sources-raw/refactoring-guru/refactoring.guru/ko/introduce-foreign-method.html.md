:::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Moving Features
between
Objects](refactoring/techniques/moving-features-between-objects.html){.type}
:::

# Introduce Foreign Method {#introduce-foreign-method .title}

::::: before-after-container
::: before
### Problem

A utility class doesn\'t contain the method that you need and you can\'t
add the method to the class.
:::

::: after
### Solution

Add the method to a client class and pass an object of the utility class
to it as an argument.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
class Report {
  // ...
  void sendReport() {
    Date nextDay = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
class Report {
  // ...
  void sendReport() {
    Date newStart = nextDay(previousEnd);
    // ...
  }
  private static Date nextDay(Date arg) {
    return new Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1);
  }
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
class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = previousEnd.AddDays(1);
    // ...
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
class Report 
{
  // ...
  void SendReport() 
  {
    DateTime nextDay = NextDay(previousEnd);
    // ...
  }
  private static DateTime NextDay(DateTime date) 
  {
    return date.AddDays(1);
  }
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
class Report {
  // ...
  public function sendReport() {
    $previousDate = clone $this->previousDate;
    $paymentDate = $previousDate->modify("+7 days");
    // ...
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
class Report {
  // ...
  public function sendReport() {
    $paymentDate = self::nextWeek($this->previousDate);
    // ...
  }
  /**
   * Foreign method. Should be in Date.
   */
  private static function nextWeek(DateTime $arg) {
    $previousDate = clone $arg;
    return $previousDate->modify("+7 days");
  }
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
class Report:
    # ...
    def sendReport(self):
        nextDay = Date(self.previousEnd.getYear(),
            self.previousEnd.getMonth(), self.previousEnd.getDate() + 1)
        # ...
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
class Report:
    # ...
    def sendReport(self):
        newStart = self._nextDay(self.previousEnd)
        # ...
        
    def _nextDay(self, arg):
        return Date(arg.getYear(), arg.getMonth(), arg.getDate() + 1)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
class Report {
  // ...
  sendReport(): void {
    let nextDay: Date = new Date(previousEnd.getYear(),
      previousEnd.getMonth(), previousEnd.getDate() + 1);
    // ...
  }
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
class Report {
  // ...
  sendReport() {
    let newStart: Date = nextDay(previousEnd);
    // ...
  }
  private static nextDay(arg: Date): Date {
    return new Date(arg.getFullYear(), arg.getMonth(), arg.getDate() + 1);
  }
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

You have code that uses the data and methods of a certain class. You
realize that the code will look and work much better inside a new method
in the class. But you can\'t add the method to the class because, for
example, the class is located in a third-party library.

This refactoring has a big payoff when the code that you want to move to
the method is repeated several times in different places in your
program.

Since you\'re passing an object of the utility class to the parameters
of the new method, you have access to all of its fields. Inside the
method, you can do practically everything that you want, as if the
method were part of the utility class.

### Benefits

-   Removes code duplication. If your code is repeated in several
    places, you can replace these code fragments with a method call.
    This is better than duplication even considering that the foreign
    method is located in a suboptimal place.

### Drawbacks

-   The reasons for having the method of a utility class in a client
    class won\'t always be clear to the person maintaining the code
    after you. If the method can be used in other classes, you could
    benefit by creating a wrapper for the utility class and placing the
    method there. This is also beneficial when there are several such
    utility methods. [Introduce Local
    Extension](introduce-local-extension.html) can help with this.

### How to Refactor

1.  Create a new method in the client class.

2.  In this method, create a parameter to which the object of the
    utility class will be passed. If this object can be obtained from
    the client class, you don\'t have to create such a parameter.

3.  Extract the relevant code fragments to this method and replace them
    with method calls.

4.  Be sure to leave the *Foreign method* tag in the comments for the
    method along with the advice to place this method in a utility class
    if such becomes possible later. This will make it easier to
    understand why this method is located in this particular class for
    those who\'ll be maintaining the software in the future.

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
#### 다음을 보세요

[Introduce Local Extension []{.fa
.fa-arrow-right}](introduce-local-extension.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### 돌아가기

[[]{.fa .fa-arrow-left} Remove Middle Man](remove-middle-man.html){.btn
.btn-default rel="prev"}
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
