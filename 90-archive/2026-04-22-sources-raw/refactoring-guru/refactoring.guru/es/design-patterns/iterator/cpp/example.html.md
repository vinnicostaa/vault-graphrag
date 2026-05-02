:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} /
[Iterator](../../iterator.html){.type} / [C++](../../cpp.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Iterator](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Iterator** en C++ {#iterator-en-c .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Iterator** es un patrón de diseño de comportamiento que permite el
recorrido secuencial por una estructura de datos compleja sin exponer
sus detalles internos.

Gracias al patrón Iterator, los clientes pueden recorrer elementos de
colecciones diferentes de un modo similar, utilizando una única interfaz
iteradora.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Iterator ](../../iterator.html){.btn
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
 [main](#example-0--main-cc)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complejidad:** []{.fa-stars}

**Popularidad:** []{.fa-stars}

**Ejemplos de uso:** El patrón es muy común en el código C++. Muchos
*frameworks* y bibliotecas lo utilizan para proporcionar una forma
estandarizada de recorrer sus colecciones.

**Identificación:** El patrón Iterator es fácil de reconocer por sus
métodos de navegación (como `next`, `previous` y otros). El código
cliente que utiliza iteradores puede no tener acceso directo a la
colección recorrida.
:::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Este ejemplo ilustra la estructura del patrón de diseño **Iterator**. Se
centra en responder las siguientes preguntas:

-   ¿De qué clases se compone?
-   ¿Qué papeles juegan esas clases?
-   ¿De qué forma se relacionan los elementos del patrón?

#### []{#example-0--main-cc .anchor} **main.cc:** Ejemplo conceptual

<figure class="code">
<pre class="code" lang="cpp"><code>/**
 * Iterator Design Pattern
 *
 * Intent: Lets you traverse elements of a collection without exposing its
 * underlying representation (list, stack, tree, etc.).
 */

#include &lt;iostream&gt;
#include &lt;string&gt;
#include &lt;vector&gt;

/**
 * C++ has its own implementation of iterator that works with a different
 * generics containers defined by the standard library.
 */

template &lt;typename T, typename U&gt;
class Iterator {
 public:
  typedef typename std::vector&lt;T&gt;::iterator iter_type;
  Iterator(U *p_data, bool reverse = false) : m_p_data_(p_data) {
    m_it_ = m_p_data_-&gt;m_data_.begin();
  }

  void First() {
    m_it_ = m_p_data_-&gt;m_data_.begin();
  }

  void Next() {
    m_it_++;
  }

  bool IsDone() {
    return (m_it_ == m_p_data_-&gt;m_data_.end());
  }

  iter_type Current() {
    return m_it_;
  }

 private:
  U *m_p_data_;
  iter_type m_it_;
};

/**
 * Generic Collections/Containers provides one or several methods for retrieving
 * fresh iterator instances, compatible with the collection class.
 */

template &lt;class T&gt;
class Container {
  friend class Iterator&lt;T, Container&gt;;

 public:
  void Add(T a) {
    m_data_.push_back(a);
  }

  Iterator&lt;T, Container&gt; *CreateIterator() {
    return new Iterator&lt;T, Container&gt;(this);
  }

 private:
  std::vector&lt;T&gt; m_data_;
};

class Data {
 public:
  Data(int a = 0) : m_data_(a) {}

  void set_data(int a) {
    m_data_ = a;
  }

  int data() {
    return m_data_;
  }

 private:
  int m_data_;
};

/**
 * The client code may or may not know about the Concrete Iterator or Collection
 * classes, for this implementation the container is generic so you can used
 * with an int or with a custom class.
 */
void ClientCode() {
  std::cout &lt;&lt; &quot;________________Iterator with int______________________________________&quot; &lt;&lt; std::endl;
  Container&lt;int&gt; cont;

  for (int i = 0; i &lt; 10; i++) {
    cont.Add(i);
  }

  Iterator&lt;int, Container&lt;int&gt;&gt; *it = cont.CreateIterator();
  for (it-&gt;First(); !it-&gt;IsDone(); it-&gt;Next()) {
    std::cout &lt;&lt; *it-&gt;Current() &lt;&lt; std::endl;
  }

  Container&lt;Data&gt; cont2;
  Data a(100), b(1000), c(10000);
  cont2.Add(a);
  cont2.Add(b);
  cont2.Add(c);

  std::cout &lt;&lt; &quot;________________Iterator with custom Class______________________________&quot; &lt;&lt; std::endl;
  Iterator&lt;Data, Container&lt;Data&gt;&gt; *it2 = cont2.CreateIterator();
  for (it2-&gt;First(); !it2-&gt;IsDone(); it2-&gt;Next()) {
    std::cout &lt;&lt; it2-&gt;Current()-&gt;data() &lt;&lt; std::endl;
  }
  delete it;
  delete it2;
}

int main() {
  ClientCode();
  return 0;
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>________________Iterator with int______________________________________
0
1
2
3
4
5
6
7
8
9
________________Iterator with custom Class______________________________
100
1000
10000</code></pre>
</figure>

::: next
#### Leer siguiente

[Mediator en C++ []{.fa
.fa-arrow-right}](../../mediator/cpp/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Command en
C++](../../command/cpp/example.html){.btn .btn-default rel="prev"}
:::

## **Iterator** en otros lenguajes

[![Iterator en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Iterator en C#"){.prog-lang-link}
[![Iterator en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Iterator en Go"){.prog-lang-link}
[![Iterator en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Iterator en Java"){.prog-lang-link}
[![Iterator en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Iterator en PHP"){.prog-lang-link}
[![Iterator en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Iterator en Python"){.prog-lang-link}
[![Iterator en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Iterator en Ruby"){.prog-lang-link}
[![Iterator en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Iterator en Rust"){.prog-lang-link}
[![Iterator en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Iterator en Swift"){.prog-lang-link}
[![Iterator en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Iterator en TypeScript"){.prog-lang-link}
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
