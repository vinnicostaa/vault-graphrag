::::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::::::: {.refactoring .page .text}
::: breadcrumb
[](../index.html){.home} / [Рефакторинг](refactoring.html){.type} /
[Приёмы](refactoring/techniques.html){.type} / [Организация
данных](refactoring/techniques/organizing-data.html){.type}
:::

# Замена магического числа символьной константой {#замена-магического-числа-символьной-константой .title}

::: aka
Также известен как: [Replace Magic Number with Symbolic
Constant]{style="display:inline-block"}
:::

::::: before-after-container
::: before
### Проблема

В коде используется число, которое несёт какой-то определённый смысл.
:::

::: after
### Решение

Замените это число константой с человеко-читаемым названием, объясняющим
смысл этого числа.
:::
:::::

:::::::::::::::::::::::::::: examples
::::::: {.before-after .before-after-container .java data-lang="java"}
:::: before
::: ba
До
:::

``` {.code lang="java"}
double potentialEnergy(double mass, double height) {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="java"}
static final double GRAVITATIONAL_CONSTANT = 9.81;

double potentialEnergy(double mass, double height) {
  return mass * height * GRAVITATIONAL_CONSTANT;
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
double PotentialEnergy(double mass, double height) 
{
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="csharp"}
const double GRAVITATIONAL_CONSTANT = 9.81;

double PotentialEnergy(double mass, double height) 
{
  return mass * height * GRAVITATIONAL_CONSTANT;
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
function potentialEnergy($mass, $height) {
  return $mass * $height * 9.81;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="php"}
define("GRAVITATIONAL_CONSTANT", 9.81);

function potentialEnergy($mass, $height) {
  return $mass * $height * GRAVITATIONAL_CONSTANT;
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
def potentialEnergy(mass, height):
    return mass * height * 9.81
```
::::

:::: after
::: ba
После
:::

``` {.code lang="python"}
GRAVITATIONAL_CONSTANT = 9.81

def potentialEnergy(mass, height):
    return mass * height * GRAVITATIONAL_CONSTANT
```
::::
:::::::

::::::: {.before-after .before-after-container .typescript data-lang="typescript"}
:::: before
::: ba
До
:::

``` {.code lang="typescript"}
potentialEnergy(mass: number, height: number): number {
  return mass * height * 9.81;
}
```
::::

:::: after
::: ba
После
:::

``` {.code lang="typescript"}
static const GRAVITATIONAL_CONSTANT = 9.81;

potentialEnergy(mass: number, height: number): number {
  return mass * height * GRAVITATIONAL_CONSTANT;
}
```
::::
:::::::
::::::::::::::::::::::::::::

### Причины рефакторинга

Магические числа --- это числовые значения, встречающиеся в коде, но при
этом неочевидно, что они означают. Данный антипаттерн затрудняет
понимание программы и усложняет её рефакторинг.

Дополнительные сложности возникают, когда нужно поменять определённое
магическое число. Это нельзя сделать автозаменой, так как одно и то же
число может использоваться для разных целей, а значит, вам нужно будет
проверять каждый участок кода, где используется это число.

### Достоинства

-   Символьная константа может служить живой документацией смысла
    значения, которое в ней хранится.

-   Значение константы намного проще заменить, чем искать нужное число
    по всему коду, при этом рискуя заменить такое же число, которое в
    данном конкретном случае использовалось для других целей.

-   Убирает дублирование использования числа или строки по всему коду.
    Это особенно актуально, если значение является сложным и длинным
    (например, `-14159`, `0xCAFEBABE`).

### Полезные факты

#### Не все числа являются магическими.

Если предназначения чисел очевидны, их не надо заменять константами,
классический пример:

<figure class="code">
<pre class="code"><code>for (i = 0; i &lt; сount; i++) { ... }</code></pre>
</figure>

#### Альтернативы

1.  Иногда, магическое число можно заменить вызовом метода. Например,
    если у вас есть магическое число, обозначающее количество элементов
    коллекции, вам не обязательно использовать его для проверок
    последнего элемента коллекции. Вместо этого можно использовать
    встроенный метод получения длины коллекции.

2.  Магические числа могут быть использованы для реализации кодирования
    типа. Например, у вас есть два типа пользователей, и чтобы
    обозначить их, у вас есть числовое поле в классе, в котором для
    администраторов хранится число `1`, а для простых пользователей ---
    число `2`.

    В этом случае имеет смысл использовать один из рефакторингов
    избавления от кодирования типа:

    -   [замена кодирования типа
        классом](replace-type-code-with-class.html)

    -   [замена кодирования типа
        подклассами](replace-type-code-with-subclasses.html)

    -   [замена кодирования типа
        состоянием/стратегией](replace-type-code-with-state-strategy.html)

### Порядок рефакторинга

1.  Объявите константу и присвойте ей значение магического числа.

2.  Найдите все упоминания магического числа.

3.  Для всех найденных чисел проверьте, согласуется ли это магическое
    число с предназначением константы. Если да, замените его вашей
    константой. Эта проверка важна, так как одно и тоже число может
    означать совершенно разные вещи (в этом случае, они должны быть
    заменены разными константами).

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

::: next
#### Читаем дальше

[Инкапсуляция поля []{.fa .fa-arrow-right}](encapsulate-field.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Замена двунаправленной связи
однонаправленной](change-bidirectional-association-to-unidirectional.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::
