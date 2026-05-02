:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones de
comportamiento](behavioral-patterns.html){.type} /
[Visitor](visitor.html){.type}
:::

# Visitor y Double Dispatch {#visitor-y-double-dispatch .title}

Veamos la siguiente jerarquía de clases de formas geométricas (ing.
"shape"), atención al pseudocódigo:

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

El código funciona bien y la aplicación está produciendo. Pero un día
decides crear una función de exportación. Resultaría extraño colocar el
código de exportación en estas clases. De modo que, en lugar de añadir
la exportación a todas las clases de esta jerarquía, decides crear una
nueva clase, externa a la jerarquía, y colocar toda la lógica de
exportación dentro de ella. La clase obtendrá métodos para exportar el
estado público de cada objeto a cadenas XML:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Exporter is
    method export(s: Shape) is
        print(&quot;Forma exportada&quot;)
    method export(d: Dot)
        print(&quot;Punto exportado&quot;)
    method export(c: Circle)
        print(&quot;Círculo exportado&quot;)
    method export(r: Rectangle)
        print(&quot;Rectángulo exportado&quot;)
    method export(cs: CompoundGraphic)
        print(&quot;Forma compuesta exportado&quot;)</code></pre>
</figure>

El código tiene buen aspecto, pero vamos a probarlo:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class App() is
    method export(shape: Shape) is
        Exporter exporter = new Exporter()
        exporter.export(shape);

app.export(new Circle());
// Lamentablemente, esto imprimirá &quot;Figura exportada&quot;.</code></pre>
</figure>

¡Espera! ¿Por qué?

## Piensa como un compilador

Nota: la siguiente información es válida para la mayoría de lenguajes
modernos de programación orientada a objetos (Java, C#, PHP y otros).

### Vinculación tardía/dinámica

Imagina que eres un compilador. Tienes que decidir cómo compilar el
siguiente código:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method drawShape(shape: Shape) is
    shape.draw();</code></pre>
</figure>

Veamos\... el método `draw` definido en la clase `Shape`. Espera un
minuto, pero también hay cuatro subclases que sobrescriben este método.
¿Podemos estar seguros de las implementaciones que debemos invocar aquí?
No lo parece. La única forma de saberlo con seguridad es ejecutando el
programa y comprobando la clase de un objeto pasado al método. Lo único
que sabemos con certeza es que el objeto **tendrá** la implementación
del método `draw`.

Por lo tanto, el código máquina resultante comprobará la clase del
parámetro `s` y tomará la implementación `draw` de la clase adecuada.

Tal comprobación de un tipo dinámico se denomina vinculación tardía (o
dinámica):

-   **Tardía** porque vinculamos el objeto y su implementación después
    de la compilación, durante el tiempo de ejecución.
-   **Dinámica**, porque puede que haya que vincular cada nuevo objeto a
    una implementación diferente.

### Vinculación temprana/estática

Ahora vamos a "compilar" el siguiente código:

<figure class="code">
<pre class="code" lang="pseudocode"><code>method exportShape(shape: Shape) is
    Exporter exporter = new Exporter()
    exporter.export(shape);</code></pre>
</figure>

Todo queda claro con la segunda línea: la clase `Exporter` no tiene un
constructor, por lo que nos limitamos a instanciar un objeto. ¿Qué pasa
con la invocación a `export`? La clase `Exporter` tiene cinco métodos
con el mismo nombre, que se diferencian en los tipos de parámetros.
¿Cuál invocar? Parece que aquí también vamos a necesitar una vinculación
dinámica.

Pero hay otro problema. ¿Qué sucede si hay una clase de forma que no
tiene el método `export` adecuado en la clase `Exporter`? Por ejemplo,
un objeto `Elipse`. El compilador no puede garantizar que exista el
método sobrecargado adecuado, en contraste con métodos sobrescritos.
Surge una situación ambigua que un compilador no puede permitir.

Por lo tanto, los desarrolladores de compiladores utilizan una ruta
segura y utilizan la vinculación temprana (o estática) para métodos
sobrecargados:

-   **Temprana** porque sucede durante el tiempo de compilación, antes
    de que se lance el programa.
-   **Estática** porque no se puede alterar durante el tiempo de
    ejecución.

Regresemos a nuestro ejemplo. Sabemos con seguridad que el argumento
entrante será de la jerarquía `Shape`, ya sea de la clase `Shape` o bien
una de sus subclases. También sabemos que la clase `Exporter` tiene una
implementación básica de la exportación que soporta la clase `Shape`:
`export(s: Shape)`.

Esa es la única implementación que puede vincularse de forma segura con
un código dado sin provocar ambigüedad. Ese es el motivo por el que, si
pasamos un objeto `Rectángulo` a `exportShape`, el exportador aún
invocará un método `export(s: Shape)`.

## Double dispatch (envío doble)

El **Double dispatch (envío doble)** es un truco que permite el uso de
la vinculación dinámica junto a métodos sobrecargados. Se hace así:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Visitor is
    method visit(s: Shape) is
        print(&quot;Forma visitado&quot;)
    method visit(d: Dot)
        print(&quot;Punto visitado&quot;)

interface Graphic is
    method accept(v: Visitor)

class Shape implements Graphic is
    method accept(v: Visitor)
        // El compilador sabe con seguridad que `this` es una `Shape`.
        // Lo cual significa que puede invocarse `visit(s: Shape)`
        // con seguridad.
        v.visit(this)

class Dot extends Shape is
    method accept(v: Visitor)
        // El compilador sabe con seguridad que `this` es un `Dot`.
        // Lo cual significa que puede invocarse `visit(s: Dot)`
        // con seguridad.
        v.visit(this)


Visitor v = new Visitor();
Graphic g = new Dot();

// El método `accept` es sobrescrito, no sobrecargado. El compilador lo
// vincula dinámicamente. Por lo tanto, `accept` se ejecutará en una
// clase que corresponda a un objeto que invoque un método (en nuestro
// caso, la clase `Dot`).
g.accept(v);

// Resultado: &quot;Punto visitado&quot;.</code></pre>
</figure>

### Epílogo

Aunque el patrón [Visitor](visitor.html) se basa en el principio del
*double dispatch*, éste no es su principal propósito. Visitor te permite
añadir operaciones "externas" a toda una jerarquía de clase sin cambiar
el código existente de esas clases.

::: next
#### Leer siguiente

[Patrones de diseño en C# []{.fa .fa-arrow-right}](csharp.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Visitor](visitor.html){.btn .btn-default
rel="prev"}
:::
::::::
:::::::
::::::::
