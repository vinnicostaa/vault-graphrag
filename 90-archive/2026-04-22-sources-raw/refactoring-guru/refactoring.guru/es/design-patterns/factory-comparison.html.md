:::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::: {.main-content-container .center-content-container}
:::::: {.page .text}
::: breadcrumb
[](../../index.html){.home} / [Patrones de
diseño](../design-patterns.html){.type} / [Patrones
creacionales](creational-patterns.html){.type}
:::

# Comparación de fábricas {#comparación-de-fábricas .title}

Este artículo muestra la diferencia entre:

1.  Fábrica ("factory", en inglés)
2.  Método de creación
3.  Método de creación estática
4.  Patrón *Simple Factory*
5.  Patrón [Factory Method](factory-method.html)
6.  Patrón [Abstract Factory](abstract-factory.html)

Puedes encontrar referencias a estos términos por toda la web y, aunque
pueden parecer similares, todos tienen significados diferentes. Mucha
gente no se da cuenta de ello, lo que provoca malos entendidos y
confusión.

Vamos a intentar establecer las diferencias y resolver este problema de
una vez por todas.

## 1. Fábrica o Factory

**Fábrica** es un término ambiguo que se refiere a una función, método o
clase que debe producir algo. Normalmente las fábricas producen objetos,
pero también pueden producir archivos, registros en bases de datos, etc.

Por ejemplo, alguien podría referirse a cualquiera de estas cosas como
una "fábrica":

-   una función o un método que crea la GUI de un programa;
-   una clase que crea usuarios;
-   un método estático que invoca una clase constructora de cierta
    manera;
-   uno de los patrones de diseño creacionales.

Normalmente, cuando alguien emplea la palabra "fábrica", el significado
exacto debe quedar claro por el contexto. Pero si tienes dudas,
pregunta. Es posible que el autor lo ignore.

## 2. Método de creación

El método de creación se define en el libro [Refactoring To
Patterns](https://www.amazon.com/-/dp/0321213351) como "un método que
crea objetos". Esto significa que todo resultado de un patrón Factory
Method es un "método de creación" pero no necesariamente a la inversa.
También significa que puedes sustituir el término "método de creación"
allá donde Martin Fowler utiliza el término "método fábrica" en
[Refactoring](https://www.amazon.com/-/dp/0201485672) y allá donde
Joshua Bloch utiliza el término "método fábrica estático" en [Effective
Java](https://www.amazon.com/-/dp/0134685997).

En realidad, el método de creación simplemente es un envoltorio
alrededor de una llamada al constructor. Puede tener un nombre que
exprese mejor tus intenciones. Por otro lado, puede ayudar a aislar tu
código de cambios en el constructor. Puede incluso contener una lógica
particular que devuelva objetos existentes en lugar de crear unos
nuevos.

Muchas personas llamarían a tales métodos "métodos fábrica"
sencillamente porque producen nuevos objetos. La lógica es sencilla: el
método crea objetos y, como todas las *fábricas* crean objetos, este
método claramente debe ser un *método fábrica*. Naturalmente, hay mucha
confusión en lo que se refiere al verdadero patrón [Factory
Method](factory-method.html).

En el siguiente ejemplo, `next` es un método de creación:

<figure class="code">
<pre class="code" lang="php"><code>class Number {
    private $value;

    public function __construct($value) {
        $this-&gt;value = $value;
    }

    public function next() {
        return new Number ($this-&gt;value + 1);
    }
}</code></pre>
</figure>

## 3. Método de creación estático

El **método de creación estático** es un método de creación declarado
como `estático`. En otras palabras, puede invocarse en una clase y no
necesita un objeto para ser creado.

No te confundas cuando alguien llame a un método como éste un "método
fábrica estático". Es una mala costumbre. El [Factory
Method](factory-method.html) es un patrón de diseño que se basa en la
herencia. Si lo haces `estático`, ya no podrás extenderlo en subclases,
lo cual contradice el propósito del patrón.

Cuando un método de creación estático devuelve nuevos objetos se
convierte en un constructor alternativo.

Puede ser de utilidad cuando:

-   Necesitas varios constructores diferentes con distintos propósitos
    pero cuyas firmas coinciden. Por ejemplo, tener `Random(int max)` y
    `Random(int min)` es imposible en Java, C++, C# y muchos otros
    lenguajes. La solución más popular es crear varios métodos estáticos
    que invoquen el constructor por defecto y establezcan después los
    valores adecuados.

-   Quieras reutilizar objetos existentes, en lugar de instanciar unos
    nuevos (véase el patrón [Singleton](singleton.html)). En la mayoría
    de los lenguajes de programación, los constructores deben devolver
    nuevas instancias de clase. El método de creación estático es una
    solución a esta limitación. Dentro de un método estático, tu código
    puede decidir si crear una nueva instancia invocando al constructor,
    o devolver un objeto existente a partir de una memoria caché.

En el siguiente ejemplo, el método `load` es un método de creación
estático. Ofrece un modo conveniente de recuperar usuarios de una base
de datos.

<figure class="code">
<pre class="code" lang="php"><code>class User {
    private $id, $name, $email, $phone;

    public function __construct($id, $name, $email, $phone) {
        $this-&gt;id = $id;
        $this-&gt;name = $name;
        $this-&gt;email = $email;
        $this-&gt;phone = $phone;
    }

    public static function load($id) {
        list($id, $name, $email, $phone) = DB::load_data(&#39;users&#39;, &#39;id&#39;, &#39;name&#39;, &#39;email&#39;, &#39;phone&#39;);
        $user = new User($id, $name, $email, $phone);
        return $user;
    }
}</code></pre>
</figure>

## 4. Patrón *Simple Factory*

El patrón **Simple Factory** (fábrica simple) []{.pop-i}[Definido en el
libro [Head First Design
Patterns](http://shop.oreilly.com/product/9780596007126.do).]{.pop-content}
describe una clase que tiene un método de creación con un gran
condicional que, basándose en los parámetros del método, elige la clase
de producto que instanciar y devolver.

La gente suele confundir las *fábricas simples* con *fábricas* en
general, o con uno de los patrones de diseño creacionales. En la mayoría
de los casos, una fábrica simple es un paso intermedio para introducir
los patrones [Factory method](factory-method.html) o [Abstract
factory](abstract-factory.html).

Las fábricas simples no suelen tener subclases. Pero, después de extraer
subclases de una fábrica simple, empiezan a parecerse a un patrón
*factory method* clásico.

Por cierto, si declaras una fábrica simple como `abstracta`, no se
convierte en el patrón *abstract factory* por arte de magia.

Aquí tienes un ejemplo de *fábrica simple*:

<figure class="code">
<pre class="code" lang="php"><code>class UserFactory {
    public static function create($type) {
        switch ($type) {
            case &#39;user&#39;: return new User();
            case &#39;customer&#39;: return new Customer();
            case &#39;admin&#39;: return new Admin();
            default:
                throw new Exception(&#39;Wrong user type passed.&#39;);
        }
    }
}</code></pre>
</figure>

## 5. Patrón *Factory Method*

El patrón **Factory Method** []{.pop-i}[Definido en el libro de la GoF
[Patrones de diseño](https://www.amazon.com/-/dp/8478290591)
.]{.pop-content} es un patrón de diseño creacional que proporciona una
interfaz para crear objetos pero permite a las subclases alterar el tipo
de objetos que se crearán.

Si tienes un método de creación en la clase base y subclases que lo
extienden, puede que se trate del método fábrica.

<figure class="code">
<pre class="code" lang="php"><code>abstract class Department {
    public abstract function createEmployee($id);

    public function fire($id) {
        $employee = $this-&gt;createEmployee($id);
        $employee-&gt;paySalary();
        $employee-&gt;dismiss();
    }
}

class ITDepartment extends Department {
    public function createEmployee($id) {
        return new Programmer($id);
    }
}

class AccountingDepartment extends Department {
    public function createEmployee($id) {
        return new Accountant($id);
    }
}</code></pre>
</figure>

## 6. Patrón *Abstract Factory*

El patrón **Abstract Factory** []{.pop-i}[También explicado en el [libro
de la GoF](https://www.amazon.com/-/dp/8478290591).]{.pop-content} es un
patrón de diseño creacional que permite la producción de familias de
objetos relacionados o dependientes, sin especificar sus clases
concretas.

¿Qué son las \"familias de objetos\"? Por ejemplo, tomemos este grupo de
clases: `Transporte` + `Motor` + `Controles`. Puede haber diversas
variantes de ellas:

1.  `Coche` + `MotordeCombustión` + `Volante`
2.  `Avión` + `Reactor` + `Horquilla`

Si tu programa no funciona con familias de productos, entonces no
necesitas una fábrica abstracta.

Y, de nuevo, mucha gente confunde el patrón *abstract factory* con una
simple clase fábrica declarada como `abstracta`. ¡No hagas tú lo mismo!

### Epílogo

Ahora que conoces las diferencias, repasa los patrones de diseño:

-   [Factory Method](factory-method.html)
-   [Abstract Factory](abstract-factory.html)

::: next
#### Leer siguiente

[Builder []{.fa .fa-arrow-right}](builder.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Abstract Factory](abstract-factory.html){.btn
.btn-default rel="prev"}
:::
::::::
:::::::
::::::::
