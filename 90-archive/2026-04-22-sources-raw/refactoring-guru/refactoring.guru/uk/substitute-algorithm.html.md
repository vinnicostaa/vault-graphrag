::::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} / [Складання
методів](refactoring/techniques/composing-methods.html){.type}
:::

# Заміна алгоритму {#заміна-алгоритму .title}

::: aka
Також відомий як: [Substitute Algorithm]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

Ви хочете замінити існуючий алгоритм іншим?
:::

::: after
### Рішення

Замініть тіло методу, що реалізує старий алгоритм, новим алгоритмом.
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
Після
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
Після
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
Після
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
Після
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
Після
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

### Причини рефакторингу

1.  Поетапний рефакторинг --- не єдиний варіант поліпшити програму.
    Іноді ви зустрічаєтеся з таким нагромадженням проблем в методі, що
    його набагато простіше переписати наново. З іншого боку, ви могли
    знайти алгоритм, який куди простіше та ефективніше за поточний. В
    цьому випадку потрібно просто замінити старий алгоритм новим.

2.  З часом ваш алгоритм може бути включений в набір відомої бібліотеки
    або фреймворка, і ви можете захотіти позбутися власної реалізації,
    щоби полегшити собі підтримку програми.

3.  Вимоги до роботи програми можуть змінитися настільки сильно, що
    старий алгоритм неможливо просто «допиляти» до відповідності їм.

### Порядок рефакторингу

1.  Переконайтеся, що ви по максимуму спростили поточний алгоритм.
    Перенесіть несуттєвий код в інші методи за допомогою [відокремлення
    методу](extract-method.html). Чим менше «рухомих частин» залишиться
    в початковому алгоритмі, тим простіше буде його замінити.

2.  Створіть ваш новий алгоритм в новому методі. Замініть старий
    алгоритм новим і запустіть тести програми.

3.  Якщо результати виявилися різними, поверніть стару реалізацію і
    порівняйте ще раз результати. З'ясуйте, чому результати не
    співпадають. Іноді буває, що причиною стають помилки в старому
    алгоритмі, але здебільшого --- не працює щось у новому.

4.  Коли всі тести почнуть проходити, остаточно видаліть старий
    алгоритм.

::::: {.prom .banner-content .banner .banner-striped .banner-with-image data-id="Ref: Part of the ebook" creative-id="animated-uk" position="content_bottom"}
::: banner-image
Your browser does not support HTML video.
:::

::: banner-text
### Замучились читати? {#замучились-читати .title}

Збігайте за подушкою, в нас тут контенту на [7 годин]{.blue} читання.

Або спробуйте наш новий інтерактивний курс. Він набагато цікавіший за
банальний тест.

[ Дізнатися більше...](refactoring/course.html){.btn .btn-secondary}
:::
:::::

:::::::: row
::::::: {.col-xs-12 .col-sm-6 .col-lg-12}
### Бореться з запахом

:::: dl
::: dt
[Дублювання коду](smells/duplicate-code.html)
:::
::::

:::: dl
::: dt
[Довгий метод](smells/long-method.html)
:::
::::
:::::::
::::::::

::: next
#### Читаємо далі

[Переміщення функцій між об\'єктами []{.fa
.fa-arrow-right}](refactoring/techniques/moving-features-between-objects.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна методу об\'єктом
методів](replace-method-with-method-object.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::::::::

:::::: {.prom .banner-sidebar .banner-removable .banner-removable-refactoring data-id="Ref: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
::::: ban
::: {.image .product-image}
[Знижки!]{.banner-discount}
[![](../images/refactoring/course/course-cover-uk4341.jpg?id=e53397ef67078e742fb132da37ca7cc8){width="300"
height="300" loading="lazy"}](refactoring/course.html)
:::

::: {.banner-text .banner-text-uk}
Цей рефакторинг є частиною нашого інтерактивного **онлайн курсу з
рефакторингу**.

[ Подробиці...](refactoring/course.html){.btn .btn-secondary .btn-block}
:::
:::::
::::::
::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::
