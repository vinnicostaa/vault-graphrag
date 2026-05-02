:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} / [Chaîne de
responsabilité](../../chain-of-responsibility.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Chaîne de
responsabilité](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Chaîne de responsabilité** en Go {#chaîne-de-responsabilité-en-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
La **Chaîne de responsabilité** est un patron de conception
comportemental qui permet de faire circuler une demande tout au long
d'une chaîne de handlers, jusqu'à ce que l'un d'entre eux la traite.

Ce patron permet à plusieurs objets de traiter une demande sans coupler
la classe du demandeur aux classes concrètes des récepteurs. La chaîne
peut être assemblée dynamiquement à l'exécution à l'aide de tout handler
implémentant l'interface standard des handlers.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Chaîne de responsabilité
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
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

## Exemple conceptuel {#example-0-title}

Regardons un peu la chaîne de responsabilité utilisée dans l'application
d'un hôpital. Un hôpital peut être composé de plusieurs départements
comme :

-   La réception
-   Les docteurs
-   La salle de soins
-   La caisse

Lorsqu'un patient arrive, il se rend d'abord à la réception, consulte
ensuite un docteur, passe du temps dans la salle de soins et finit par
aller à la caisse pour payer, etc. Le patient est envoyé à travers une
chaîne de départements, et chacun d'entre eux expédie le patient plus
loin dans la chaîne une fois qu'il a rempli son rôle.

Ce patron est utilisable lorsque plusieurs candidats vont effectuer la
même demande. Il se montre très utile lorsque vous voulez empêcher le
client de choisir son récepteur, car plusieurs objets peuvent gérer la
demande. Il est également pratique si vous voulez découpler le client
des récepteurs : le client n'aura besoin que de connaître le premier
élément de la chaîne.

Si l'on reprend l'exemple de l'hôpital, le patient arrive d'abord à la
réception. Ensuite, en fonction de son état, la réception l'envoie au
prochain handler de la chaîne.

#### []{#example-0--department-go .anchor} **department.go:** Interface handler

<figure class="code">
<pre class="code" lang="go"><code>package main

type Department interface {
    execute(*Patient)
    setNext(Department)
}</code></pre>
</figure>

#### []{#example-0--reception-go .anchor} **reception.go:** Handler concret

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

#### []{#example-0--doctor-go .anchor} **doctor.go:** Handler concret

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

#### []{#example-0--medical-go .anchor} **medical.go:** Handler concret

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

#### []{#example-0--cashier-go .anchor} **cashier.go:** Handler concret

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

#### []{#example-0--main-go .anchor} **main.go:** Code client

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

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Reception registering patient
Doctor checking patient
Medical giving medicine to patient
Cashier getting money from patient patient</code></pre>
</figure>

::: next
#### Suivant

[Commande en Go []{.fa
.fa-arrow-right}](../../command/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Retour

[[]{.fa .fa-arrow-left} Procuration en
Go](../../proxy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Chaîne de responsabilité** dans les autres langues

[![Chaîne de responsabilité en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Chaîne de responsabilité en C#"){.prog-lang-link}
[![Chaîne de responsabilité en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Chaîne de responsabilité en C++"){.prog-lang-link}
[![Chaîne de responsabilité en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Chaîne de responsabilité en Java"){.prog-lang-link}
[![Chaîne de responsabilité en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Chaîne de responsabilité en PHP"){.prog-lang-link}
[![Chaîne de responsabilité en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Chaîne de responsabilité en Python"){.prog-lang-link}
[![Chaîne de responsabilité en
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Chaîne de responsabilité en Ruby"){.prog-lang-link}
[![Chaîne de responsabilité en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Chaîne de responsabilité en Rust"){.prog-lang-link}
[![Chaîne de responsabilité en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Chaîne de responsabilité en Swift"){.prog-lang-link}
[![Chaîne de responsabilité en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Chaîne de responsabilité en TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
