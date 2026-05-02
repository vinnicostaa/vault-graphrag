:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Посетитель {#посетитель .title}

::: aka
Также известен как: [Visitor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Посетитель** --- это поведенческий паттерн проектирования, который
позволяет добавлять в программу новые операции, не изменяя классы
объектов, над которыми эти операции могут выполняться.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Посетитель" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Ваша команда разрабатывает приложение, работающее с геоданными в виде
графа. Узлами графа являются городские локации: памятники, театры,
рестораны, важные предприятия и прочее. Каждый узел имеет ссылки на
другие, ближайшие к нему узлы. Каждому типу узлов соответствует свой
класс, а каждый узел представлен отдельным объектом.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Экспорт геоузлов в XML" />
<figcaption><p>Экспорт геоузлов в XML.</p></figcaption>
</figure>

Ваша задача --- сделать экспорт этого графа в XML. Дело было бы плёвым,
если бы вы могли редактировать классы узлов. Достаточно было бы добавить
метод экспорта в каждый тип узла, а затем, перебирая узлы графа,
вызывать этот метод для каждого узла. Благодаря полиморфизму, решение
получилось бы изящным, так как вам не пришлось бы привязываться к
конкретным классам узлов.

Но, к сожалению, классы узлов вам изменить не удалось. Системный
архитектор сослался на то, что код классов узлов сейчас очень стабилен,
и от него многое зависит, поэтому он не хочет рисковать и позволять
кому-либо его трогать.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-ru276d.png?id=db53b0a158e94893d6f54f2d34a7d1b2"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-ru.png?id=db53b0a158e94893d6f54f2d34a7d1b2 500w,/images/patterns/diagrams/visitor/problem2-ru-1.5x.png?id=1adf3ad02ce3e647f15c686dd5a133fb 750w,/images/patterns/diagrams/visitor/problem2-ru-2x.png?id=7fb08000dfe99d01cf3d1e173bac3341 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Код XML-экспорта придётся добавить во все классы узлов" />
<figcaption><p>Код XML-экспорта придётся добавить во все классы узлов, а
это слишком накладно.</p></figcaption>
</figure>

К тому же он сомневался в том, что экспорт в XML вообще уместен в рамках
этих классов. Их основная задача была связана с геоданными, а экспорт
выглядит в рамках этих классов чужеродно.

Была и ещё одна причина запрета. Если на следующей неделе вам бы
понадобился экспорт в какой-то другой формат данных, то эти классы снова
пришлось бы менять.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Посетитель предлагает разместить новое поведение в отдельном
классе, вместо того чтобы множить его сразу в нескольких классах.
Объекты, с которыми должно было быть связано поведение, не будут
выполнять его самостоятельно. Вместо этого вы будете передавать эти
объекты в методы посетителя.

Код поведения, скорее всего, должен отличаться для объектов разных
классов, поэтому и методов у посетителя должно быть несколько. Названия
и принцип действия этих методов будут схожи, но основное отличие будет в
типе принимаемого в параметрах объекта, например:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Здесь возникает вопрос: как подавать узлы в объект-посетитель? Так как
все методы имеют отличающуюся сигнатуру, использовать полиморфизм при
переборе узлов не получится. Придётся проверять тип узлов для того,
чтобы выбрать соответствующий метод посетителя.

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node in graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node)
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node)
    // ...</code></pre>
</figure>

Тут не поможет даже механизм перегрузки методов (доступный в Java и C#).
Если назвать все методы одинаково, то неопределённость реального типа
узла всё равно не даст вызвать правильный метод. Механизм перегрузки всё
время будет вызывать метод посетителя, соответствующий типу `Node`, а не
реального класса поданного узла.

Но паттерн Посетитель решает и эту проблему, используя механизм [двойной
диспетчеризации](visitor-double-dispatch.html). Вместо того, чтобы самим
искать нужный метод, мы можем поручить это объектам, которые передаём в
параметрах посетителю. А они уже вызовут правильный метод посетителя.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Client code
foreach (Node node in graph)
    node.accept(exportVisitor)

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this)
    // ...

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this)
    // ...</code></pre>
</figure>

Как видите, изменить классы узлов всё-таки придётся. Но это простое
изменение позволит применять к объектам узлов и другие поведения, ведь
классы узлов будут привязаны не к конкретному классу посетителей, а к их
общему интерфейсу. Поэтому если придётся добавить в программу новое
поведение, вы создадите новый класс посетителей и будете передавать его
в методы узлов.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Страховой агент" />
<figcaption><p>У страхового агента приготовлены полисы для разных
видов организаций.</p></figcaption>
</figure>

Представьте начинающего страхового агента, жаждущего получить новых
клиентов. Он беспорядочно посещает все дома в округе, предлагая свои
услуги. Но для каждого из посещаемых *типов* домов у него имеется особое
предложение.

-   Придя в дом к обычной семье, он предлагает оформить медицинскую
    страховку.
-   Придя в банк, он предлагает страховку от грабежа.
-   Придя на фабрику, он предлагает страховку предприятия от пожара и
    наводнения.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-ru9ad6.png?id=4b17ce6b6496af042526b5b5a78bd853"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-ru.png?id=4b17ce6b6496af042526b5b5a78bd853 520w,/images/patterns/diagrams/visitor/structure-ru-1.5x.png?id=2e2653d5ce566d585191d98ac7329466 780w,/images/patterns/diagrams/visitor/structure-ru-2x.png?id=06c2094bb5a5db9da862cf7e7f47c7b3 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура классов паттерна Посетитель" /><img
src="../../images/patterns/diagrams/visitor/structure-ru-indexedc0a1.png?id=15510b78b2ae334b14bbb414822879e2"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-ru-indexed.png?id=15510b78b2ae334b14bbb414822879e2 520w,/images/patterns/diagrams/visitor/structure-ru-indexed-1.5x.png?id=12f7ad4a5fe09cdd94f5e7bc84e035ee 780w,/images/patterns/diagrams/visitor/structure-ru-indexed-2x.png?id=38e4974dccded42c9ae6c0dcf6b2ce97 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура классов паттерна Посетитель" />
</figure>
:::

1.  **Посетитель** описывает общий интерфейс для всех типов посетителей.
    Он объявляет набор методов, отличающихся типом входящего параметра,
    которые нужны для запуска операции для всех типов конкретных
    элементов. В языках, поддерживающих перегрузку методов, эти методы
    могут иметь одинаковые имена, но типы их параметров должны
    отличаться.

2.  **Конкретные посетители** реализуют какое-то особенное поведение для
    всех типов элементов, которые можно подать через методы интерфейса
    посетителя.

3.  **Элемент** описывает метод *принятия* посетителя. Этот метод должен
    иметь единственный параметр, объявленный с типом интерфейса
    посетителя.

4.  **Конкретные элементы** реализуют методы *принятия* посетителя. Цель
    этого метода --- вызвать тот метод посещения, который соответствует
    типу этого элемента. Так посетитель узнает, с каким именно элементом
    он работает.

5.  **Клиентом** зачастую выступает коллекция или сложный составной
    объект, например, дерево [Компоновщика](composite.html). Зачастую
    клиент не привязан к конкретным классам элементов, работая с ними
    через общий интерфейс элементов.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Посетитель** добавляет в существующую иерархию классов
геометрических фигур возможность экспорта в XML.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура классов примера паттерна Посетитель" />
<figcaption><p>Пример организации экспорта объектов в XML через
отдельный класс-посетитель.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Сложная иерархия элементов.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Метод принятия посетителя должен быть реализован в каждом
// элементе, а не только в базовом классе. Это поможет программе
// определить, какой метод посетителя нужно вызвать, если вы не
// знаете тип элемента.
class Dot implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitDot(this)

class Circle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCircle(this)

class Rectangle implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitRectangle(this)

class CompoundShape implements Shape is
    // ...
    method accept(v: Visitor) is
        v.visitCompoundShape(this)


// Интерфейс посетителей должен содержать методы посещения
// каждого элемента. Важно, чтобы иерархия элементов менялась
// редко, так как при добавлении нового элемента придётся менять
// всех существующих посетителей.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Конкретный посетитель реализует одну операцию для всей
// иерархии элементов. Новая операция = новый посетитель.
// Посетитель выгодно применять, когда новые элементы
// добавляются очень редко, а новые операции — часто.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Экспорт id и координат центра точки.

    method visitCircle(c: Circle) is
        // Экспорт id, кординат центра и радиуса окружности.

    method visitRectangle(r: Rectangle) is
        // Экспорт id, кординат левого-верхнего угла, ширины и
        // высоты прямоугольника.

    method visitCompoundShape(cs: CompoundShape) is
        // Экспорт id составной фигуры, а также списка id
        // подфигур, из которых она состоит.


// Приложение может применять посетителя к любому набору
// объектов элементов, даже не уточняя их типы. Нужный метод
// посетителя будет выбран благодаря проходу через метод accept.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Вам не кажется, что вызов метода `accept` --- это лишнее звено? Если
так, то ещё раз рекомендую вам ознакомиться с проблемой раннего и
позднего связывания в статье [Посетитель и Double
Dispatch](visitor-double-dispatch.html).
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда вам нужно выполнить какую-то операцию над всеми элементами сложной
структуры объектов, например, деревом.
:::

::: applicability-solution
Посетитель позволяет применять одну и ту же операцию к объектам
различных классов.
:::

::: applicability-problem
Когда над объектами сложной структуры объектов надо выполнять некоторые
не связанные между собой операции, но вы не хотите «засорять» классы
такими операциями.
:::

::: applicability-solution
Посетитель позволяет извлечь родственные операции из классов,
составляющих структуру объектов, поместив их в один класс-посетитель.
Если структура объектов является общей для нескольких приложений, то
паттерн позволит в каждое приложение включить только нужные операции.
:::

::: applicability-problem
Когда новое поведение имеет смысл только для некоторых классов из
существующей иерархии.
:::

::: applicability-solution
Посетитель позволяет определить поведение только для этих классов,
оставив его пустым для всех остальных.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте интерфейс посетителя и объявите в нём методы «посещения»
    для каждого класса элемента, который существует в программе.

2.  Опишите интерфейс элементов. Если вы работаете с уже существующими
    классами, то объявите абстрактный метод принятия посетителей в
    базовом классе иерархии элементов.

3.  Реализуйте методы принятия во всех конкретных элементах. Они должны
    переадресовывать вызовы тому методу посетителя, в котором тип
    параметра совпадает с текущим классом элемента.

4.  Иерархия элементов должна знать только о базовом интерфейсе
    посетителей. С другой стороны, посетители будут знать обо всех
    классах элементов.

5.  Для каждого нового поведения создайте конкретный класс посетителя.
    Приспособьте это поведение для работы со всеми типами элементов,
    реализовав все методы интерфейса посетителей.

    Вы можете столкнуться с ситуацией, когда посетителю нужен будет
    доступ к приватным полям элементов. В этом случае вы можете либо
    раскрыть доступ к этим полям, нарушив инкапсуляцию элементов, либо
    сделать класс посетителя вложенным в класс элемента, если вам
    повезло писать на языке, который поддерживает вложенность классов.

6.  Клиент будет создавать объекты посетителей, а затем передавать их
    элементам, используя метод принятия.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Упрощает добавление операций, работающих со сложными структурами
    объектов.
-    Объединяет родственные операции в одном классе.
-    Посетитель может накапливать состояние при обходе структуры
    элементов.
:::

::: col-sm-6
-    Паттерн не оправдан, если иерархия элементов часто меняется.
-    Может привести к нарушению инкапсуляции элементов.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Посетитель](visitor.html) можно рассматривать как расширенный
    аналог [Команды](command.html), который способен работать сразу с
    несколькими видами получателей.

-   Вы можете выполнить какое-то действие над всем деревом
    [Компоновщика](composite.html) при помощи
    [Посетителя](visitor.html).

-   [Посетитель](visitor.html) можно использовать совместно с
    [Итератором](iterator.html). *Итератор* будет отвечать за обход
    структуры данных, а *Посетитель* --- за выполнение действий над
    каждым её компонентом.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Посетитель на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Посетитель на C#"){.prog-lang-link}
[![Посетитель на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Посетитель на C++"){.prog-lang-link}
[![Посетитель на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Посетитель на Go"){.prog-lang-link}
[![Посетитель на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Посетитель на Java"){.prog-lang-link}
[![Посетитель на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Посетитель на PHP"){.prog-lang-link}
[![Посетитель на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Посетитель на Python"){.prog-lang-link}
[![Посетитель на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Посетитель на Ruby"){.prog-lang-link}
[![Посетитель на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Посетитель на Rust"){.prog-lang-link}
[![Посетитель на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Посетитель на Swift"){.prog-lang-link}
[![Посетитель на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Посетитель на TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Дополнительные материалы {#extras}

-   Подробней о том, почему Посетитель нельзя заменить простой
    перегрузкой методов, читайте в статье [Посетитель и Double
    Dispatch](visitor-double-dispatch.html).
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-ru" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не втыкай в транспорте {#не-втыкай-в-транспорте .title}

Лучше почитай нашу книгу о паттернах проектирования.

Теперь это удобно делать даже во время поездок в общественном
транспорте.

[ Узнать больше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаем дальше

[Посетитель и Double Dispatch []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Шаблонный метод](template-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ruee5e.png?id=ea25feb28a6deb8aa4af6a633904a568){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ru-2x.png?id=95101b307f6dba409f28a417746ff3c1 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Эта статья является частью нашей электронной книги **Погружение в
Паттерны Проектирования**.

[ Узнать больше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
