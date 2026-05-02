:::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Патерни
проектування](../../../design-patterns.html){.type} / [Ланцюжок
обов\'язків](../../chain-of-responsibility.html){.type} /
[Go](../../go.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Ланцюжок
обов\'язків](../../../../images/patterns/cards/chain-of-responsibility-mini6d42.png?id=36d85eba8d14986f053123de17aac7a7){srcset="/images/patterns/cards/chain-of-responsibility-mini-2x.png?id=8c81f7069e51259b2443801b91135f7f 2x"}
:::

# **Ланцюжок обов\'язків** на Go {#ланцюжок-обовязків-на-go .pattern-example-title-block-title}
::::

::::::::::::::::: pattern-example-body
::: pattern-example-brief
**Ланцюжок обов'язків** --- це поведінковий патерн, що дозволяє
передавати запит ланцюжком потенційних обробників до тих пір, поки один
з них не обробить його.

Позбавляє від жорсткої прив'язки відправника запиту до одержувача,
дозволяючи динамічно вибудовувати ланцюг з різних обробників.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Детальніше про Ланцюжок обов\'язків
](../../chain-of-responsibility.html){.btn .btn-lg .btn-primary}
:::

:::::::::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навігація
:::

::: en-intro
 [Інтро](#)
:::

::: en-intro
 [Концептуальний приклад](#example-0)
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

## Концептуальний приклад {#example-0-title}

Давайте розглянемо патерн Ланцюжок обов'язків на прикладі додатка
лікарні. Шпиталь може мати різні приміщення, наприклад:

-   Приймальне відділення
-   Доктор
-   Кімната медикаментів
-   Касир

Коли пацієнт приходить до лікарні, перш за все він потрапляє у
Приймальне відділення, звідти --- до Доктора, потім у Кімнату
медикаментів, після цього --- до Касира, і так далі. Пацієнт проходить
ланцюжком приміщень, в якому кожна ланка відправляє його далі відразу
після виконання своєї функції.

Цей патерн можна застосовувати у випадках, коли для виконання одного
запиту є кілька кандидатів, і коли ви не хочете, щоб клієнт сам вибирав
виконавця. Важливо знати, що клієнта необхідно відгородити від
виконавців, йому необхідно знати лише про існування першої ланки
ланцюга.

Використовуючи приклад лікарні, пацієнт спершу потрапляє в Приймальне
відділення. Потім, залежно від його стану, Приймальне відділення
відправляє його до наступного виконавця у ланцюзі.

#### []{#example-0--department-go .anchor} **department.go:** Інтерфейс обробника

<figure class="code">
<pre class="code" lang="go"><code>package main

type Department interface {
    execute(*Patient)
    setNext(Department)
}</code></pre>
</figure>

#### []{#example-0--reception-go .anchor} **reception.go:** Конкретний обробник

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

#### []{#example-0--doctor-go .anchor} **doctor.go:** Конкретний обробник

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

#### []{#example-0--medical-go .anchor} **medical.go:** Конкретний обробник

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

#### []{#example-0--cashier-go .anchor} **cashier.go:** Конкретний обробник

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

#### []{#example-0--main-go .anchor} **main.go:** Клієнтський код

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

#### []{#example-0--output-txt .anchor} **output.txt:** Результат виконання

<figure class="code">
<pre class="code" lang="output"><code>Reception registering patient
Doctor checking patient
Medical giving medicine to patient
Cashier getting money from patient patient</code></pre>
</figure>

::: next
#### Читаємо далі

[Команда на Go []{.fa
.fa-arrow-right}](../../command/go/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Замісник на
Go](../../proxy/go/example.html){.btn .btn-default rel="prev"}
:::

## **Ланцюжок обов\'язків** іншими мовами програмування

[![Ланцюжок обов\'язків на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Ланцюжок обов'язків на C#"){.prog-lang-link}
[![Ланцюжок обов\'язків на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Ланцюжок обов'язків на C++"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Ланцюжок обов'язків на Java"){.prog-lang-link}
[![Ланцюжок обов\'язків на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Ланцюжок обов'язків на PHP"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Ланцюжок обов'язків на Python"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Ланцюжок обов'язків на Ruby"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Ланцюжок обов'язків на Rust"){.prog-lang-link}
[![Ланцюжок обов\'язків на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Ланцюжок обов'язків на Swift"){.prog-lang-link}
[![Ланцюжок обов\'язків на
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Ланцюжок обов'язків на TypeScript"){.prog-lang-link}
:::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-uk" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архів з прикладами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Придбай книгу **Занурення в Патерни** та отримай архів з десятками
детальних прикладів, які можна відкривати прямо в IDE.

[ Дізнатися більше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::
