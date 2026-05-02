:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrones de
diseño](../../../design-patterns.html){.type} / [Chain of
Responsibility](../../chain-of-responsibility.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chain of
Responsibility](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chain of Responsibility** en Go {#chain-of-responsibility-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Chain of Responsibility** es un patrón de diseño de comportamiento que
permite pasar solicitudes a lo largo de la cadena de manejadores
potenciales hasta que uno de ellos gestiona la solicitud.

El patrón permite que varios objetos gestionen la solicitud sin acoplar
la clase emisora a las clases concretas de los receptores. La cadena
puede componerse dinámicamente durante el tiempo de ejecución con
cualquier manejador que siga una interfaz manejadora estándar.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Aprende más sobre el patrón Chain of Responsibility
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
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
 [department](#example-0--department-go)
:::

::: en-file
 [reception](#example-0--reception-go)
:::

::: en-file
 [doctor](#example-0--doctor-go)
:::

::: en-file
 [medical](#example-0--medical-go)
:::

::: en-file
 [cashier](#example-0--cashier-go)
:::

::: en-file
 [patient](#example-0--patient-go)
:::

::: en-file
 [main](#example-0--main-go)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::::::::
:::::::::::::::::

[]{#example-0}

## Ejemplo conceptual {#example-0-title}

Veamos el patrón Chain of Responsibility con el caso de una aplicación
de hospital. Un hospital puede tener varios departamentos, como:

-   Recepción
-   Consulta
-   Sala de curas
-   Caja

Cuando llega un paciente, primero pasa por recepción, después por la
consulta del médico y después por la sala de curas, y por último por la
caja (etcétera). El paciente pasa por una cadena de departamentos y cada
uno de ellos envía al paciente un poco más allá en la cadena, una vez
que se complete su función.

El patrón se aplica cuando hay varios candidatos para procesar la misma
solicitud. Cuando no quieres que el cliente elija el receptor ya que
varios objetos pueden gestionar la solicitud. Además, quieres desacoplar
el cliente de los receptores. El cliente solo necesita conocer el primer
elemento de la cadena.

Al igual que en el ejemplo del hospital, el paciente pasa primero por la
recepción. Después, en base al estado del paciente, desde recepción lo
envían al siguiente manejador de la cadena.

#### []{#example-0--department-go .anchor} **department.go:** Interfaz manejador

<figure class="code">
<pre class="code" lang="go"><code>package main

type Department interface {
    execute(*Patient)
    setNext(Department)
}</code></pre>
</figure>

#### []{#example-0--reception-go .anchor} **reception.go:** Manipulador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Reception struct {
    next Department
}

func (r *Reception) execute(p *Patient) {
    if p.registrationDone {
        fmt.Println(&quot;Patient registration already done&quot;)
        r.next.execute(p)
        return
    }
    fmt.Println(&quot;Reception registering patient&quot;)
    p.registrationDone = true
    r.next.execute(p)
}

func (r *Reception) setNext(next Department) {
    r.next = next
}</code></pre>
</figure>

#### []{#example-0--doctor-go .anchor} **doctor.go:** Manipulador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Doctor struct {
    next Department
}

func (d *Doctor) execute(p *Patient) {
    if p.doctorCheckUpDone {
        fmt.Println(&quot;Doctor checkup already done&quot;)
        d.next.execute(p)
        return
    }
    fmt.Println(&quot;Doctor checking patient&quot;)
    p.doctorCheckUpDone = true
    d.next.execute(p)
}

func (d *Doctor) setNext(next Department) {
    d.next = next
}</code></pre>
</figure>

#### []{#example-0--medical-go .anchor} **medical.go:** Manipulador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Medical struct {
    next Department
}

func (m *Medical) execute(p *Patient) {
    if p.medicineDone {
        fmt.Println(&quot;Medicine already given to patient&quot;)
        m.next.execute(p)
        return
    }
    fmt.Println(&quot;Medical giving medicine to patient&quot;)
    p.medicineDone = true
    m.next.execute(p)
}

func (m *Medical) setNext(next Department) {
    m.next = next
}</code></pre>
</figure>

#### []{#example-0--cashier-go .anchor} **cashier.go:** Manipulador concreto

<figure class="code">
<pre class="code" lang="go"><code>package main

import &quot;fmt&quot;

type Cashier struct {
    next Department
}

func (c *Cashier) execute(p *Patient) {
    if p.paymentDone {
        fmt.Println(&quot;Payment Done&quot;)
    }
    fmt.Println(&quot;Cashier getting money from patient patient&quot;)
}

func (c *Cashier) setNext(next Department) {
    c.next = next
}</code></pre>
</figure>

#### []{#example-0--patient-go .anchor} **patient.go**

<figure class="code">
<pre class="code" lang="go"><code>package main

type Patient struct {
    name              string
    registrationDone  bool
    doctorCheckUpDone bool
    medicineDone      bool
    paymentDone       bool
}</code></pre>
</figure>

#### []{#example-0--main-go .anchor} **main.go:** Código cliente

<figure class="code">
<pre class="code" lang="go"><code>package main

func main() {

    cashier := &amp;Cashier{}

    //Set next for medical department
    medical := &amp;Medical{}
    medical.setNext(cashier)

    //Set next for doctor department
    doctor := &amp;Doctor{}
    doctor.setNext(medical)

    //Set next for reception department
    reception := &amp;Reception{}
    reception.setNext(doctor)

    patient := &amp;Patient{name: &quot;abc&quot;}
    //Patient visiting
    reception.execute(patient)
}</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Resultado de la ejecución

<figure class="code">
<pre class="code" lang="output"><code>Reception registering patient
Doctor checking patient
Medical giving medicine to patient
Cashier getting money from patient patient</code></pre>
</figure>

::: next
#### Leer siguiente

[Command en Go []{.fa
.fa-arrow-right}](../../command/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Volver

[[]{.fa .fa-arrow-left} Proxy en Go](../../proxy/go/example.html){.btn
.btn-default rel="prev"}
:::

## **Chain of Responsibility** en otros lenguajes

[![Chain of Responsibility en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chain of Responsibility en C#"){.prog-lang-link}
[![Chain of Responsibility en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chain of Responsibility en C++"){.prog-lang-link}
[![Chain of Responsibility en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chain of Responsibility en Java"){.prog-lang-link}
[![Chain of Responsibility en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chain of Responsibility en PHP"){.prog-lang-link}
[![Chain of Responsibility en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chain of Responsibility en Python"){.prog-lang-link}
[![Chain of Responsibility en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chain of Responsibility en Ruby"){.prog-lang-link}
[![Chain of Responsibility en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chain of Responsibility en Rust"){.prog-lang-link}
[![Chain of Responsibility en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chain of Responsibility en Swift"){.prog-lang-link}
[![Chain of Responsibility en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chain of Responsibility en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

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
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
