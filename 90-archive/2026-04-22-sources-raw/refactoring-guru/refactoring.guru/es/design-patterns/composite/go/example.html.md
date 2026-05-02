::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Composite](../../composite.html){.type} / [Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Composite](../../../../images/patterns/cards/composite-minib543.png?id=a369d98d18b417f255d04568fd0131b8){srcset="/images/patterns/cards/composite-mini-2x.png?id=3f7f811fefeb0b64f6774746eb42af09 2x"}
:::

# **Composite** en Go {#composite-en-go .pattern-example-title-block-title}
::::

:::::::::::::: pattern-example-body
::: pattern-example-brief
**Composite** es un patrón de diseño estructural que permite componer
objetos en una estructura en forma de árbol y trabajar con ella como si
fuera un objeto único.

El patrón Composite se convirtió en una solución muy popular para la
mayoría de problemas que requieren la creación de una estructura de
árbol. La gran característica del Composite es la capacidad para
ejecutar métodos de forma recursiva por toda la estructura de árbol y
recapitular los resultados.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Composite ](../../composite.html){.btn
.btn-lg .btn-primary}
:::

::::::::::: {.sidebar-navigation .with-scroll}
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
 [component](#example-0--component-go)
:::

::: en-file
 [folder](#example-0--folder-go)
:::

::: en-file
 [file](#example-0--file-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
:::::::::::
::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Vamos a intentar imaginar el patrón Composite con un ejemplo de un
sistema de archivos del sistema operativo. En el sistema de archivos hay
dos tipos de objetos: archivos y carpetas. Hay casos en los que archivos
y carpetas deben tratarse de la misma manera. Aquí es donde el patrón
Composite resulta de utilidad.

Imagina que tienes que realizar una búsqueda de una palabra clave
particular en tu sistema de archivos. Esta operación de búsqueda se
aplica tanto a archivos como a carpetas. Para un archivo, buscará en los
contenidos del archivo; para una carpeta, recorrerá todos los archivos
de la carpeta para encontrar la palabra clave.

#### []{#example-0--component-go .anchor} **component.go:** Interfaz componente

<figure class="code">
<pre class="code" lang="go"><code>package main

type Component interface {
    search(string)
}</code></pre>
</figure>

#### []{#example-0--folder-go .anchor} **folder.go:** Compuesto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Folder struct {
    components []Component
    name       string
}

func (f *Folder) search(keyword string) {
    fmt.Printf(&quot;Serching recursively for keyword %s in folder %s\n&quot;, keyword, f.name)
    for _, composite := range f.components {
        composite.search(keyword)
    }
}

func (f *Folder) add(c Component) {
    f.components = append(f.components, c)
}</code></pre>
</figure>

#### []{#example-0--file-go .anchor} **file.go:** Hoja

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type File struct {
    name string
}

func (f *File) search(keyword string) {
    fmt.Printf(&quot;Searching for keyword %s in file %s\n&quot;, keyword, f.name)
}

func (f *File) getName() string {
    return f.name
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {
    file1 := &amp;File{name: &quot;File1&quot;}
    file2 := &amp;File{name: &quot;File2&quot;}
    file3 := &amp;File{name: &quot;File3&quot;}

    folder1 := &amp;Folder{
        name: &quot;Folder1&quot;,
    }

    folder1.add(file1)

    folder2 := &amp;Folder{
        name: &quot;Folder2&quot;,
    }
    folder2.add(file2)
    folder2.add(file3)
    folder2.add(folder1)

    folder2.search(&quot;rose&quot;)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Serching recursively for keyword rose in folder Folder2
Searching for keyword rose in file File2
Searching for keyword rose in file File3
Serching recursively for keyword rose in folder Folder1
Searching for keyword rose in file File1</code></pre>
</figure>

::: next
#### Leer siguiente

[Decorator en Go []{.fa
.fa-arrow-right}](../../decorator/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Bridge en Go](../../bridge/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Composite** en otros lenguajes

[![Composite en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Composite en C#"){.prog-lang-link}
[![Composite en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Composite en C++"){.prog-lang-link}
[![Composite en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Composite en Java"){.prog-lang-link}
[![Composite en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Composite en PHP"){.prog-lang-link}
[![Composite en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Composite en Python"){.prog-lang-link}
[![Composite en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Composite en Ruby"){.prog-lang-link}
[![Composite en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Composite en Rust"){.prog-lang-link}
[![Composite en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Composite en Swift"){.prog-lang-link}
[![Composite en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Composite en TypeScript"){.prog-lang-link}
::::::::::::::::::::

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
::::::::::::::::::::::::::
:::::::::::::::::::::::::::
