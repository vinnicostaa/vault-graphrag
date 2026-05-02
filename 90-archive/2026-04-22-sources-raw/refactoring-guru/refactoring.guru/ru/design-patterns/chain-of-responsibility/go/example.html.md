:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} / [Цепочка
обязанностей](../../chain-of-responsibility.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Цепочка
обязанностей](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Цепочка обязанностей** на Go {#цепочка-обязанностей-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Цепочка обязанностей** --- это поведенческий паттерн, позволяющий
передавать запрос по цепочке потенциальных обработчиков, пока один из
них не обработает запрос.

Избавляет от жёсткой привязки отправителя запроса к его получателю,
позволяя выстраивать цепь из различных обработчиков динамически.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Цепочка обязанностей
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
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

## Концептуальный пример {#example-0-title}

Давайте рассмотрим паттерн Цепочка обязанностей на примере приложения
больницы. Госпиталь может иметь разные помещения, например:

-   Приемное отделение
-   Доктор
-   Комната медикаментов
-   Кассир

Когда пациент прибывает в больницу, первым делом он попадает в Приемное
отделение, оттуда -- к Доктору, затем в Комнату медикаментов, после
этого -- к Кассиру, и так далее. Пациент проходит по цепочке помещений,
в которой каждое отправляет его по ней дальше сразу после выполнения
своей функции.

Этот паттерн можно применять в случаях, когда для выполнения одного
запроса есть несколько кандидатов, и когда вы не хотите, чтобы клиент
сам выбирал исполнителя. Важно знать, что клиента необходимо оградить от
исполнителей, ему необходимо знать лишь о существовании первого звена
цепи.

Используя пример больницы, пациент сперва попадает в Приемное отделение.
Затем, зависимо от его состояния, Приемное отделение отправляет его к
следующему исполнителю в цепи.

#### []{#example-0--department-go .anchor} **department.go:** Интерфейс обработчика

<figure class="code">
<pre class="code" lang="go"><code>package main

type Department interface {
    execute(*Patient)
    setNext(Department)
}</code></pre>
</figure>

#### []{#example-0--reception-go .anchor} **reception.go:** Конкретный обработчик

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

#### []{#example-0--doctor-go .anchor} **doctor.go:** Конкретный обработчик

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

#### []{#example-0--medical-go .anchor} **medical.go:** Конкретный обработчик

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

#### []{#example-0--cashier-go .anchor} **cashier.go:** Конкретный обработчик

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

#### []{#example-0--main-go .anchor} **main.go:** Клиентский код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Reception registering patient
Doctor checking patient
Medical giving medicine to patient
Cashier getting money from patient patient</code></pre>
</figure>

::: next
#### Читаем дальше

[Команда на Go []{.fa
.fa-arrow-right}](../../command/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Заместитель на
Go](../../proxy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Цепочка обязанностей** на других языках программирования

[![Цепочка обязанностей на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Цепочка обязанностей на C#"){.prog-lang-link}
[![Цепочка обязанностей на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Цепочка обязанностей на C++"){.prog-lang-link}
[![Цепочка обязанностей на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Цепочка обязанностей на Java"){.prog-lang-link}
[![Цепочка обязанностей на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Цепочка обязанностей на PHP"){.prog-lang-link}
[![Цепочка обязанностей на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Цепочка обязанностей на Python"){.prog-lang-link}
[![Цепочка обязанностей на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Цепочка обязанностей на Ruby"){.prog-lang-link}
[![Цепочка обязанностей на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Цепочка обязанностей на Rust"){.prog-lang-link}
[![Цепочка обязанностей на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Цепочка обязанностей на Swift"){.prog-lang-link}
[![Цепочка обязанностей на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Цепочка обязанностей на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
