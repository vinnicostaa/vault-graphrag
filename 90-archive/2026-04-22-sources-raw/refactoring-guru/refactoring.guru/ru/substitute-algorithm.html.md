::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Составление
методов](refactoring/techniques/composing-methods.html){.type}
:::

# Замена алгоритма {#замена-алгоритма .title}

::: aka
Также известен как: [Substitute Algorithm]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Вы хотите заменить существующий алгоритм другим?
:::

::: after
### Решение

Замените тело метода, реализующего старый алгоритм, новым алгоритмом.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
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
После
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
До
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
После
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
До
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
После
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
До
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
После
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
До
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
После
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

### Причины рефакторинга

1.  Поэтапный рефакторинг --- не единственный способ улучшить программу.
    Иногда вы сталкиваетесь с таким нагромождением проблем в методе, что
    его гораздо проще переписать заново. С другой стороны, вы могли
    найти алгоритм, который куда проще и эффективнее текущего. В этом
    случае надо просто заменить старый алгоритм новым.

2.  С течением времени ваш алгоритм может оказаться включен в набор
    известной библиотеки или фреймворка, и вы можете пожелать избавиться
    от собственной реализации, чтобы облегчить себе поддержку программы.

3.  Требования к работе программы могут измениться настолько сильно, что
    старый алгоритм невозможно просто «допилить» до соответствия им.

### Порядок рефакторинга

1.  Убедитесь, что вы по максимуму упростили текущий алгоритм.
    Перенесите несущественный код в другие методы с помощью [извлечения
    метода](extract-method.html). Чем меньше «движущихся частей»
    останется в исходном алгоритме, тем проще будет его заменить.

2.  Создайте ваш новый алгоритм в новом методе. Замените старый алгоритм
    новым и запустите тесты программы.

3.  Если результаты не сходятся, верните старую реализацию и сравните
    результаты. Выясните, почему результаты не совпадают. Нередко,
    причиной могут быть ошибки в старом алгоритме, но с большей
    вероятностью не работает что-то в новом.

4.  Когда все тесты начнут проходить, окончательно удалите старый
    алгоритм.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-ru" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Устали читать? {#устали-читать .title}

Сбегайте за подушкой, у нас тут контента на [7 часов]{.blue} чтения.

Или попробуйте наш интерактивный курс. Он гораздо более интересный, чем
банальный текст.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Борется с запахом

:::: dl
::: dt
[Дублирование кода](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Длинный метод](smells/long-method.html)
:::
::::
:::::::
::::::::

::: next
#### Читаем дальше

[Перемещение функций между объектами []{.fa
.fa-arrow-right}](refactoring/techniques/moving-features-between-objects.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена метода объектом
методов](replace-method-with-method-object.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
::::: ban
::: {.image .product-image}
[Скидки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-rua60d.jpg?id=e9d6b4015f4c6c48cf06f7479874d8d7){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-ru}
Этот рефакторинг --- малая часть интерактивного **онлайн курса по
рефакторингу**.

[ Узнать больше...](refactoring/course.html){.btn .btn-secondary
.btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
