:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Prototype](../../prototype.html){.type} /
[C#](../../csharp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Prototype](../../../../images/patterns/cards/prototype-mini6540.png?id=bc3046bb39ff36574c08d49839fd1c8e){srcset="/images/patterns/cards/prototype-mini-2x.png?id=b871f789a736e7efbb1cd082d2de6398 2x"}
:::

# **Prototype** en C# {#prototype-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Prototype** es un patrón de diseño creacional que permite la clonación
de objetos, incluso los complejos, sin acoplarse a sus clases
específicas.

Todas las clases prototipo deben tener una interfaz común que haga
posible copiar objetos incluso si sus clases concretas son desconocidas.
Los objetos prototipo pueden producir copias completas, ya que los
objetos de la misma clase pueden acceder a los campos privados de los
demás.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Prototype ](../../prototype.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegación
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Ejemplo conceptual](#example-0)
:::

::: en-file
 [Program](#example-0--Program-cs)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón Prototype está disponible en C# listo
para usarse con una interfaz `ICloneable`.

**Identificación:** El prototipo puede reconocerse fácilmente por un
método `clone` o `copy`, etc.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Prototype**.
Se centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--Program-cs .anchor} **Program.cs:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="csharp"><code>using System;

namespace RefactoringGuru.DesignPatterns.Prototype.Conceptual
{
    public class Person
    {
        public int Age;
        public DateTime BirthDate;
        public string Name;
        public IdInfo IdInfo;

        public Person ShallowCopy()
        {
            return (Person) this.MemberwiseClone();
        }

        public Person DeepCopy()
        {
            Person clone = (Person) this.MemberwiseClone();
            clone.IdInfo = new IdInfo(IdInfo.IdNumber);
            clone.Name = String.Copy(Name);
            return clone;
        }
    }

    public class IdInfo
    {
        public int IdNumber;

        public IdInfo(int idNumber)
        {
            this.IdNumber = idNumber;
        }
    }

    class Program
    {
        static void Main(string[] args)
        {
            Person p1 = new Person();
            p1.Age = 42;
            p1.BirthDate = Convert.ToDateTime(&quot;1977-01-01&quot;);
            p1.Name = &quot;Jack Daniels&quot;;
            p1.IdInfo = new IdInfo(666);

            // Perform a shallow copy of p1 and assign it to p2.
            Person p2 = p1.ShallowCopy();
            // Make a deep copy of p1 and assign it to p3.
            Person p3 = p1.DeepCopy();

            // Display values of p1, p2 and p3.
            Console.WriteLine(&quot;Original values of p1, p2, p3:&quot;);
            Console.WriteLine(&quot;   p1 instance values: &quot;);
            DisplayValues(p1);
            Console.WriteLine(&quot;   p2 instance values:&quot;);
            DisplayValues(p2);
            Console.WriteLine(&quot;   p3 instance values:&quot;);
            DisplayValues(p3);

            // Change the value of p1 properties and display the values of p1,
            // p2 and p3.
            p1.Age = 32;
            p1.BirthDate = Convert.ToDateTime(&quot;1900-01-01&quot;);
            p1.Name = &quot;Frank&quot;;
            p1.IdInfo.IdNumber = 7878;
            Console.WriteLine(&quot;\nValues of p1, p2 and p3 after changes to p1:&quot;);
            Console.WriteLine(&quot;   p1 instance values: &quot;);
            DisplayValues(p1);
            Console.WriteLine(&quot;   p2 instance values (reference values have changed):&quot;);
            DisplayValues(p2);
            Console.WriteLine(&quot;   p3 instance values (everything was kept the same):&quot;);
            DisplayValues(p3);
        }

        public static void DisplayValues(Person p)
        {
            Console.WriteLine(&quot;      Name: {0:s}, Age: {1:d}, BirthDate: {2:MM/dd/yy}&quot;,
                p.Name, p.Age, p.BirthDate);
            Console.WriteLine(&quot;      ID#: {0:d}&quot;, p.IdInfo.IdNumber);
        }
    }
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Original values of p1, p2, p3:
   p1 instance values: 
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666
   p2 instance values:
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666
   p3 instance values:
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666

Values of p1, p2 and p3 after changes to p1:
   p1 instance values: 
      Name: Frank, Age: 32, BirthDate: 01/01/00
      ID#: 7878
   p2 instance values (reference values have changed):
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 7878
   p3 instance values (everything was kept the same):
      Name: Jack Daniels, Age: 42, BirthDate: 01/01/77
      ID#: 666</code></pre>
</figure>

::: next
#### Leer siguiente

[Singleton en C# []{.fa
.fa-arrow-right}](../../singleton/csharp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Factory Method en
C#](../../factory-method/csharp/example.html){.btn .btn-default
rel="prev"}
:::

## **Prototype** en otros lenguajes

[![Prototype en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Prototype en C++"){.prog-lang-link}
[![Prototype en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Prototype en Go"){.prog-lang-link}
[![Prototype en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Prototype en Java"){.prog-lang-link}
[![Prototype en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Prototype en PHP"){.prog-lang-link}
[![Prototype en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Prototype en Python"){.prog-lang-link}
[![Prototype en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Prototype en Ruby"){.prog-lang-link}
[![Prototype en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Prototype en Rust"){.prog-lang-link}
[![Prototype en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Prototype en Swift"){.prog-lang-link}
[![Prototype en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Prototype en TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-es" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archivo con ejemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compra el libro electrónico **Sumérgete en los patrones de diseño** para
tener acceso al archivo con decenas de ejemplos detallados que puedes
abrir en tu IDE.

[ Saber más...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
