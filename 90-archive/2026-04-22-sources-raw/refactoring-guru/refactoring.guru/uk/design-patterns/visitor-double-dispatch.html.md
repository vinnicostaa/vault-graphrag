:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type} /
[Відвідувач](visitor.html){.type}
:::

# Відвідувач і Double Dispatch {#відвідувач-і-double-dispatch .title}

Розглянемо приклад, у якому ми маємо невелику ієрархію класів
геометричних фігур (обережно, псевдокод):

<figure class="code">
<pre class="code" lang="pseudocode"><code>interface Graphic is
    method draw()

class Shape implements Graphic is
    field id
    method draw()
    // ...

class Dot extends Shape is
    field x, y
    method draw()
    // ...

class Circle extends Dot is
    field radius
    method draw()
    // ...

class Rectangle extends Shape is
    field width, height
    method draw()
    // ...

class CompoundGraphic implements Graphic is
    field children: array of Graphic
    method draw()
    // ...</code></pre>
</figure>

Нам потрібно додати зовнішню оперцію над усіма цими компонентами,
наприклад, експорт. У нашій мові (Java, C#, \...) є перевантаження
методів, тому ми створюємо такий клас:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Exporting shape&quot;)
    method export(d: Dot)
        print(&quot;Exporting dot&quot;)
    method export(c: Circle)
        print(&quot;Exporting circle&quot;)
    method export(r: Rectangle)
        print(&quot;Exporting rectangle&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Exporting compound&quot;)</code></pre>
</figure>

Здається, що все добре. Але давайте спробуємо такий клас на ділі:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// На жаль, виведе &quot;Exporting shape&quot;.</code></pre>
</figure>

Як? Але чому?!

## Побувати в шкурі компілятора

Примітка: все, що тут описано --- правдиве для більшості сучасних
об'єктних мов програмування (Java, C#, PHP та інші).

### Пізнє/динамічне зв'язування

Давайте уявімо себе компілятором. Вам потрібно зрозуміти, як
скомпілювати такий код:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Отже, виклик метода `draw` у класі `Shape`. Але нам також відомо про
чотири класи, що перевизначають цей метод. Чи можливо вже зараз
зрозуміти, яку реалізацію потрібно вибрати? Схоже, що ні, адже для цього
доведеться запустити програму й дізнатися, який саме об'єкт буде поданий
у параметр. Але одне ви знаєте напевне --- який би об'єкт не був
переданий, він обов'язково матиме реалізацію `draw`.

В кінцевому рахунку машинний код, який ви створите, під час переходу
через цю ділянку буде щоразу перевіряти, що за об'єкт цей `shape`, і
вибирати реалізацію метода `draw` із відповідного класа.

Така динамічна перевірка типу називається пізнім або динамічним
зв'язуванням:

-   **Пізнім**, тому що ми пов'язуємо об'єкт та реалізацію вже після
    компіляції.
-   **Динамічним**, тому що ми робимо це під час кожного проходження
    через цю ділянку.

### Раннє/статичне зв'язування

Тепер давайте «скомпілюємо» такий код:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

Зі створенням об'єкту все зрозуміло. Як щодо виклику методу `export`? У
класі `Exporter` у нас є п'ять версій методу з таким ім'ям, які
відрізняються лише типом параметру. Схоже, що тут також доведеться
динамічно відстежувати тип параметра, що передається, і за ним
визначати, який з методів вибрати.

Але тут на нас чекає прикра несподіванка. Що як хто-небудь подасть у
метод `exportShape` такий об'єкт, для якого відсутній метод `export` у
класі `Exporter`? Наприклад, об'єкт `Ellipse`, для якого у нас немає
експорту. Дійсно, ми не маємо гарантії, що необхідний метод буде
присутній, як це було з перевизначеними методами. А отже, виникне
неоднозначна ситуація.

Саме через це всі розробники компіляторів обирають безпечний шлях і
використовують раннє або статичне зв'язування для перевантажених
методів:

-   **Раннє**, тому що воно відбувається ще на етапі компіляції
    програми.
-   **Статичне**, тому що його вже не можна змінити під час виконання.

Повернемося до нашого прикладу. Ми впевнені в тому, що маємо параметр з
типом `Shape`. Ми знаємо, що в `Exporter` є реалізація, яка підходить
для цього: `export(s: Shape)`. Відповідно, цю ділянку коду ми жорстко
зв'язуємо з відомою реалізацією методу.

І тому навіть якщо ми подамо у параметрах один з підкласів `Shape`,
однаково буде викликана реалізація `export(s: Shape)`.

## Double dispatch

Подвійна диспетчеризація (або double dispatch) --- це трюк, який дає
змогу оминути обмеженість раннього зв'язування в перевантажених методах.
Ось як це робиться:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Visited shape&quot;)
    method visit(d: Dot)
        print(&quot;Visited dot&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // Компілятор знає, що тут `this` це `Shape`.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Компілятор знає, що тут `this` це `Dot`.
        // Це означає, що можна статично пов’язати цей виклик
        // з реалізацією visit(d: Dot).
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// Метод accept() —перевизначений, але не перевантажений.
// Відповідно, пов’язаний динамічно. Тому реалізація `accept` буде 
// вибрана під час виконання вже з того класу, об’єкт якого його
// викликав (клас Dot).
g.accept(v);

// Виведе &quot;Visited dot&quot;.</code></pre>
</figure>

### Післяслово

Хоча патерн [Відвідувач](visitor.html) і побудований на механіці
подвійної диспетчеризації, це не основна його ідея. Відвідувач дає змогу
додавати операції до цілої ієрархії класів, без потреби міняти код цих
класів.

::: next
#### Читаємо далі

[Патерни проектування на C# []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Відвідувач](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
