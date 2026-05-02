:::::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Прийоми рефакторингу](refactoring/techniques.html){.type} /
[Організація даних](refactoring/techniques/organizing-data.html){.type}
:::

# Інкапсуляція поля {#інкапсуляція-поля .title}

::: aka
Також відомий як: [Encapsulate Field]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

У вас є публічне поле.
:::

::: after
### Рішення

Зробіть поле приватним і створіть для нього методи доступу.
:::
:::::

::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
class Person {
  public String name;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="java"}
class Person {
  private String name;

  public String getName() {
    return name;
  }
  public void setName(String arg) {
    name = arg;
  }
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
class Person 
{
  public string name;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="csharp"}
class Person 
{
  private string name;

  public string Name
  {
    get { return name; }
    set { name = value; }
  }
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
public $name;
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="php"}
private $name;

public getName() {
  return $this->name;
}

public setName($arg) {
  $this->name = $arg;
}
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
class Person {
  name: string;
}
```
::::

:::: after
::: ba
Після
:::

``` {.code lang="typescript"}
class Person {
  private _name: string;

  get name() {
    return this._name;
  }
  setName(arg: string): void {
    this._name = arg;
  }
}
```
::::
:::::::
:::::::::::::::::::::::

### Причини рефакторингу

Одним із стовпів об'єктного програмування є *Інкапсуляція* або
можливість приховання даних об'єкта. Інакше всі дані об'єктів були б
публічними, а інші об'єкти могли б отримувати і модифікувати дані вашого
об'єкта без його відома. При цьому розділяються дані та поведінка, яка
пов'язана з цими даними, погіршується модульність частин програми і
ускладнюється її підтримка.

### Переваги

-   Якщо дані і поведінка якогось компоненту тісно пов'язані між собою і
    знаходяться в одному місці коду, вам набагато простіше буде
    підтримувати і розвивати цей компонент.

-   Крім того, ви можете робити якісь складні операції, пов'язані з
    доступом до полів об'єкта.

### Коли не слід застосовувати

-   Зустрічаються випадки, коли інкапсуляція полів небажана з міркувань
    підвищення швидкодії. Ці випадки дуже рідкісні, але іноді цей момент
    буває дуже важливим.

    Наприклад, у вас є графічний редактор, в якому є об'єкти, що мають
    координати `x` і `y`. Ці поля навряд чи мінятимуться в майбутньому.
    До того ж, в програмі бере участь дуже багато різних об'єктів, в
    яких є присутніми ці поля. Тому звернення безпосередньо до полів
    координат економить значну частину процесорного часу, який інакше
    витрачався б на виклики методів доступу.

    Як ілюстрація цього виключення, існує клас
    [Point](http://docs.oracle.com/javase/7/docs/api/java/awt/Point.html)
    на Java, усі поля якого є публічними.

### Порядок рефакторингу

1.  Створіть геттер і сеттер для поля.

2.  Знайдіть усі звернення до поля. Замініть отримання значення з
    поля --- геттером, а установку нових значень в полі --- сеттером.

3.  Після того, як усі звернення до полів замінені, зробіть поле
    приватним.

#### Подальші кроки

«Інкапсуляція поля» є всього лише першим кроком до зближення даних і
поведінки над цими даними. Після того, як ви створили прості методи
доступу до полів, варто ще раз перевірити місця, де ці методи
викликаються. Цілком можливо, код з цих ділянок доречніше виглядав би в
самих методах доступу.

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

:::::::::: row
:::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Схожі рефакторинги

::::: dl
::: dt
[Самоінкапсуляція поля](self-encapsulate-field.html)
:::

::: dd
Замість прямого доступу до приватних полів всередині класу,
використовуємо методи доступу до цих полів.
:::
:::::
::::::

::::: {.col-xs-12 .col-sm-6 .col-lg-6}
### Бореться з запахом

:::: dl
::: dt
[Клас даних](smells/data-class.html)
:::
::::
:::::
::::::::::

::: next
#### Читаємо далі

[Інкапсуляція колекції []{.fa
.fa-arrow-right}](encapsulate-collection.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Заміна магічного числа символьною
константою](replace-magic-number-with-symbolic-constant.html){.btn
.btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::
