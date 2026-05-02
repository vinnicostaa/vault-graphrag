:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Refactoring](refactoring.html){.type} /
[Techniques](refactoring/techniques.html){.type} / [Composing
Methods](refactoring/techniques/composing-methods.html){.type}
:::

# Substitute Algorithm {#substitute-algorithm .title}

::::: before-after-container
::: before
### Problem

So you want to replace an existing algorithm with a new one?
:::

::: after
### Solution

Replace the body of the method that implements the algorithm with a new
algorithm.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
Before
:::

``` {.code lang="java"}
String foundPerson(String[] people){
  for (int i = 0; i < people.length; i++) {
    if (people[i].equals("Don")){
      return "Don";
    }
    if (people[i].equals("John")){
      return "John";
    }
    if (people[i].equals("Kent")){
      return "Kent";
    }
  }
  return "";
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="java"}
String foundPerson(String[] people){
  List candidates =
    Arrays.asList(new String[] {"Don", "John", "Kent"});
  for (int i=0; i < people.length; i++) {
    if (candidates.contains(people[i])) {
      return people[i];
    }
  }
  return "";
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
string FoundPerson(string[] people)
{
  for (int i = 0; i < people.Length; i++) 
  {
    if (people[i].Equals("Don"))
    {
      return "Don";
    }
    if (people[i].Equals("John"))
    {
      return "John";
    }
    if (people[i].Equals("Kent"))
    {
      return "Kent";
    }
  }
  return String.Empty;
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="csharp"}
string FoundPerson(string[] people)
{
  List<string> candidates = new List<string>() {"Don", "John", "Kent"};
  
  for (int i = 0; i < people.Length; i++) 
  {
    if (candidates.Contains(people[i])) 
    {
      return people[i];
    }
  }
  
  return String.Empty;
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
function foundPerson(array $people){
  for ($i = 0; $i < count($people); $i++) {
    if ($people[$i] === "Don") {
      return "Don";
    }
    if ($people[$i] === "John") {
      return "John";
    }
    if ($people[$i] === "Kent") {
      return "Kent";
    }
  }
  return "";
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="php"}
function foundPerson(array $people){
  foreach (["Don", "John", "Kent"] as $needle) {
    $id = array_search($needle, $people, true);
    if ($id !== false) {
      return $people[$id];
    }
  }
  return "";
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
def foundPerson(people):
    for i in range(len(people)):
        if people[i] == "Don":
            return "Don"
        if people[i] == "John":
            return "John"
        if people[i] == "Kent":
            return "Kent"
    return ""
```
::::

:::: after
::: ba
After
:::

``` {.code lang="python"}
def foundPerson(people):
    candidates = ["Don", "John", "Kent"]
    return people if people in candidates else ""
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
Before
:::

``` {.code lang="typescript"}
foundPerson(people: string[]): string{
  for (let person of people) {
    if (person.equals("Don")){
      return "Don";
    }
    if (person.equals("John")){
      return "John";
    }
    if (person.equals("Kent")){
      return "Kent";
    }
  }
  return "";
}
```
::::

:::: after
::: ba
After
:::

``` {.code lang="typescript"}
foundPerson(people: string[]): string{
  let candidates = ["Don", "John", "Kent"];
  for (let person of people) {
    if (candidates.includes(person)) {
      return person;
    }
  }
  return "";
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Why Refactor

1.  Gradual refactoring isn't the only method for improving a program.
    Sometimes a method is so cluttered with issues that it's easier to
    tear down the method and start fresh. And perhaps you have found an
    algorithm that's much simpler and more efficient. If this is the
    case, you should simply replace the old algorithm with the new one.

2.  As time goes on, your algorithm may be incorporated into a
    well-known library or framework and you want to get rid of your
    independent implementation, in order to simplify maintenance.

3.  The requirements for your program may change so heavily that your
    existing algorithm can't be salvaged for the task.

### How to Refactor

1.  Make sure that you have simplified the existing algorithm as much as
    possible. Move unimportant code to other methods using [Extract
    Method](extract-method.html). The fewer moving parts in your
    algorithm, the easier it's to replace.

2.  Create your new algorithm in a new method. Replace the old algorithm
    with the new one and start testing the program.

3.  If the results don't match, return to the old implementation and
    compare the results. Identify the causes of the discrepancy. While
    the cause is often an error in the old algorithm, it's more likely
    due to something not working in the new one.

4.  When all tests are successfully completed, delete the old algorithm
    for good!

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

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
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
:::::::
::::::::

::: next
#### Leia a seguir

[Moving Features between Objects []{.fa
.fa-arrow-right}](refactoring/techniques/moving-features-between-objects.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Replace Method with Method
Object](replace-method-with-method-object.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
