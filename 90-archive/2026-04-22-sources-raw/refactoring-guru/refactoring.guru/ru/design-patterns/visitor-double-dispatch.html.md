:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type} /
[Посетитель](visitor.html){.type}
:::

# Посетитель и Double Dispatch {#посетитель-и-double-dispatch .title}

Рассмотрим пример, в котором у нас есть небольшая иерархия классов
геометрических фигур (осторожно, псевдокод):

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

Нам нужно добавить внешнюю операцию над всеми этими компонентами,
например, экспорт. В нашем языке (Java, C#, \...) есть перегрузка
методов, поэтому мы создаём такой класс:

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

Кажется, что всё хорошо. Но давайте испробуем такой класс в деле:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// К сожалению, выведет &quot;Exporting shape&quot;.</code></pre>
</figure>

Как? Но почему?!

## Побывать в шкуре компилятора

Примечание: всё что здесь описано --- правда для большинства современных
объектных языков программирования (Java, C#, PHP и другие).

### Позднее/динамическое связывание

Давайте представим себя компилятором. Вам нужно понять как
скомпилировать такой код:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Итак, вызов метода `draw` в классе `Shape`. Но нам известно ещё и о
четырёх классах переопределяющих этот метод. Можно ли уже сейчас понять
какую реализацию нужно выбрать? Похоже, что нет, ведь для этого придётся
запустить программу и узнать какой же объект будет подан в параметр. Но
одно вы знаете точно --- какой бы объект ни был передан, он точно будет
иметь реализацию `draw`.

В результате машинный код, который вы создадите, будет каждый раз при
проходе через этот участок проверять что за объект этот `shape`, и
выбирать реализацию метода `draw` из соответствующего класса.

Такая динамическая проверка типа называется поздним или динамическим
связыванием:

-   **Поздним**, потому что мы связываем объект и реализацию уже после
    компиляции.
-   **Динамическим**, потому что мы делаем это при каждом прохождении
    через этот участок.

### Раннее/статическое связывание

Теперь давайте «скомпилируем» такой код:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

С созданием объекта всё ясно. Как насчёт вызова метода `export`? В
классе `Exporter` у нас есть пять версий метода с таким именем, которые
отличаются только типом параметра. Похоже, здесь тоже придётся
динамически отслеживать тип передаваемого параметра и по нему определять
какой из методов выбрать.

Но здесь нас ждёт засада. Что если кто-то подаст в метод `exportShape`
такой объект, для которого не существует метода `export` в классе
`Exporter`? Например, объект `Ellipse`, для которого у нас нет экспорта.
Действительно, у нас нет гарантии что необходимый метод будет
существовать, как это было с переопределенными методами. А значит,
возникнет неоднозначная ситуация.

Именно поэтому все разработчики компиляторов выбирают безопасную
тропинку и применяют раннее или статическое связывание для перегруженных
методов:

-   **Раннее**, потому что оно происходит ещё на этапе компиляции
    программы.
-   **Статическое**, потому что его уже не изменить во время выполнения.

Вернемся к нашему примеру. Мы уверены в том, что имеем параметр с типом
`Shape`. Мы знаем что в `Exporter` существует подходящая реализация:
`export(s: Shape)`. Значит, этот участок кода мы жёстко связываем с
известной реализацией метода.

И поэтому даже если мы подадим в параметрах один из подклассов `Shape`,
всё равно будет вызвана реализация `export(s: Shape)`.

## Double dispatch

Двойная диспетчеризация или double dispatch --- это трюк, позволяющий
обойти ограниченность раннего связывания в перегруженных методах. Вот
как это делается:

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
        // Компилятор знает, что здесь `this` это `Shape`.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // Компилятор знает, что здесь `this` это `Dot`.
        // А значит можно статически связать этот вызов
        // с реализацией visit(d: Dot).
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// Метод accept() — переопределен, но не перегружен. А значит, связан
// динамически. Поэтому реализация `accept` будет выбрана во время выполнения
// уже из того класса, объект которого его вызвал (класс Dot).
g.accept(v);

// Выведет &quot;Visited dot&quot;.</code></pre>
</figure>

### Послесловие

Хотя паттерн [Посетитель](visitor.html) и построен на механизме двойной
диспетчеризации, это не основная его идея. Посетитель позволяет
добавлять операции к целой иерархии классов, без надобности менять код
этих классов.

::: next
#### Читаем дальше

[Паттерны проектирования на C# []{.fa
.fa-arrow-right}](csharp.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Посетитель](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
