::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Extract Method {#extract-method .title}

::::: before-after-container
::: before
### Problem

You have a code fragment that can be grouped together.
:::

::: after
### Solution

Move this code to a separate new method (or function) and replace the
old code with a call to the method.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
void printOwing() {
  printBanner();

  // Print details.
  System.out.println("name: " + name);
  System.out.println("amount: " + getOutstanding());
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
void printOwing() {
  printBanner();
  printDetails(getOutstanding());
}

void printDetails(double outstanding) {
  System.out.println("name: " + name);
  System.out.println("amount: " + outstanding);
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
void PrintOwing() 
{
  this.PrintBanner();

  // Print details.
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
void PrintOwing()
{
  this.PrintBanner();
  this.PrintDetails();
}

void PrintDetails()
{
  Console.WriteLine("name: " + this.name);
  Console.WriteLine("amount: " + this.GetOutstanding());
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
function printOwing() {
  $this->printBanner();

  // Print details.
  print("name:  " . $this->name);
  print("amount " . $this->getOutstanding());
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function printOwing() {
  $this->printBanner();
  $this->printDetails($this->getOutstanding());
}

function printDetails($outstanding) {
  print("name:  " . $this->name);
  print("amount " . $outstanding);
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
def printOwing(self):
    self.printBanner()

    # print details
    print("name:", self.name)
    print("amount:", self.getOutstanding())
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def printOwing(self):
    self.printBanner()
    self.printDetails(self.getOutstanding())

def printDetails(self, outstanding):
    print("name:", self.name)
    print("amount:", outstanding)
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
printOwing(): void {
  printBanner();

  // Print details.
  console.log("name: " + name);
  console.log("amount: " + getOutstanding());
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
printOwing(): void {
  printBanner();
  printDetails(getOutstanding());
}

printDetails(outstanding: number): void {
  console.log("name: " + name);
  console.log("amount: " + outstanding);
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

The more lines found in a method, the harder it's to figure out what the
method does. This is the main reason for this refactoring.

Besides eliminating rough edges in your code, extracting methods is also
a step in many other refactoring approaches.

### Benefits

-   More readable code! Be sure to give the new method a name that
    describes the method's purpose: `createOrder()`,
    `renderCustomerInfo()`, etc.

-   Less code duplication. Often the code that's found in a method can
    be reused in other places in your program. So you can replace
    duplicates with calls to your new method.

-   Isolates independent parts of code, meaning that errors are less
    likely (such as if the wrong variable is modified).

### How to Refactor

1.  Create a new method and name it in a way that makes its purpose
    self-evident.

2.  Copy the relevant code fragment to your new method. Delete the
    fragment from its old location and put a call for the new method
    there instead.

    Find all variables used in this code fragment. If they're declared
    inside the fragment and not used outside of it, simply leave them
    unchanged---they'll become local variables for the new method.

3.  If the variables are declared prior to the code that you're
    extracting, you will need to pass these variables to the parameters
    of your new method in order to use the values previously contained
    in them. Sometimes it's easier to get rid of these variables by
    resorting to [Replace Temp with
    Query](replace-temp-with-query.html).

4.  If you see that a local variable changes in your extracted code in
    some way, this may mean that this changed value will be needed later
    in your main method. Double-check! And if this is indeed the case,
    return the value of this variable to the main method to keep
    everything functioning.

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

::::::::::::::::::::::::::::::: row
::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Anti-refactoring

:::: dl
::: dt
[Inline Method](inline-method.html)
:::
::::
:::::

::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Similar refactorings

:::: dl
::: dt
[Move Method](move-method.html)
:::
::::
:::::

::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Helps other refactorings

:::: dl
::: dt
[Introduce Parameter Object](introduce-parameter-object.html)
:::
::::

:::: dl
::: dt
[Form Template Method](form-template-method.html)
:::
::::

:::: dl
::: dt
[Parameterize Method](parameterize-method.html)
:::
::::
:::::::::

::::::::::::::::: {.col-xs-12 .col-sm-6 .col-lg-3}
### Eliminates smell

:::: dl
::: dt
[Duplicate Code](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Long Method](smells/long-method.html)
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
[Message Chains](smells/message-chains.html)
:::
::::

:::: dl
::: dt
[Comments](smells/comments.html)
:::
::::

:::: dl
::: dt
[Data Class](smells/data-class.html)
:::
::::
:::::::::::::::::
:::::::::::::::::::::::::::::::

::: next
#### Suivant

[Inline Method []{.fa .fa-arrow-right}](inline-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Composing
Methods](refactoring/techniques/composing-methods.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
