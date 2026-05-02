::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Шаблонный метод {#шаблонный-метод .title}

::: aka
Также известен как: [Template Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Шаблонный метод** --- это поведенческий паттерн проектирования,
который определяет скелет алгоритма, перекладывая ответственность за
некоторые его шаги на подклассы. Паттерн позволяет подклассам
переопределять шаги алгоритма, не меняя его общей структуры.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Шаблонный метод" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Вы пишете программу для дата-майнинга в офисных документах. Пользователи
будут загружать в неё документы в разных форматах (PDF, DOC, CSV), а
программа должна извлекать из них полезную информацию.

В первой версии вы ограничились только обработкой DOC-файлов. В
следующей версии добавили поддержку CSV. А через месяц прикрутили работу
с PDF-документами.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Классы дата-майнинга содержат много дублирования" />
<figcaption><p>Классы дата-майнинга содержат
много дублирования.</p></figcaption>
</figure>

В какой-то момент вы заметили, что код всех трёх классов обработки
документов хоть и отличается в части работы с файлами, но содержат
довольно много общего в части самого извлечения данных. Было бы здорово
избавится от повторной реализации алгоритма извлечения данных в каждом
из классов.

К тому же остальной код, работающий с объектами этих классов, наполнен
условиями, проверяющими тип обработчика перед началом работы. Весь этот
код можно упростить, если слить все три класса воедино либо свести их к
общему интерфейсу.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Шаблонный метод предлагает разбить алгоритм на
последовательность шагов, описать эти шаги в отдельных методах и
вызывать их в одном *шаблонном* методе друг за другом.

Это позволит подклассам переопределять некоторые шаги алгоритма,
оставляя без изменений его структуру и остальные шаги, которые для этого
подкласса не так важны.

В нашем примере с дата-майнингом мы можем создать общий базовый класс
для всех трёх алгоритмов. Этот класс будет состоять из шаблонного
метода, который последовательно вызывает шаги разбора документов.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-rucea3.png?id=2234a496eda263f4ba19d8e3fad302f3"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-ru.png?id=2234a496eda263f4ba19d8e3fad302f3 600w,/images/patterns/diagrams/template-method/solution-ru-1.5x.png?id=153681091f16918e97a33e86555f3706 900w,/images/patterns/diagrams/template-method/solution-ru-2x.png?id=2628665959256c8574e559883aa3fd2f 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Шаблонный метод содержит вызовы методов-шагов" />
<figcaption><p>Шаблонный метод разбивает алгоритм на шаги, позволяя
подклассам переопределить некоторые из них.</p></figcaption>
</figure>

Для начала шаги шаблонного метода можно сделать абстрактными. Из-за
этого все подклассы должны будут реализовать каждый из шагов по-своему.
В нашем случае все подклассы и так содержат реализацию каждого из шагов,
поэтому ничего дополнительно делать не нужно.

По-настоящему важным является следующий этап. Теперь мы можем определить
общее для всех классов поведение и вынести его в суперкласс. В нашем
примере шаги открытия, считывания и закрытия могут отличаться для разных
типов документов, поэтому останутся абстрактными. А вот одинаковый для
всех типов документов код обработки данных переедет в базовый класс.

Как видите, у нас получилось два вида шагов: *абстрактные*, которые
каждый подкласс обязательно должен реализовать, а также шаги *с
реализацией по умолчанию*, которые можно переопределять в подклассах, но
не обязательно.

Но есть и третий тип шагов --- *хуки*: их не обязательно переопределять,
но они не содержат никакого кода, выглядя как обычные методы. Шаблонный
метод останется рабочим, даже если ни один подкласс не переопределит
такой хук. Однако, хук даёт подклассам дополнительные точки
«вклинивания» в шаблонный метод.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Строительство типовых домов" />
<figcaption><p>Проект типового дома могут немного изменить по
желанию клиента.</p></figcaption>
</figure>

Строители используют подход, похожий на шаблонный метод при
строительстве типовых домов. У них есть основной архитектурный проект, в
котором расписаны шаги строительства: заливка фундамента, постройка
стен, перекрытие крыши, установка окон и так далее.

Но, несмотря на стандартизацию каждого этапа, строители могут вносить
небольшие изменения на любом из этапов, чтобы сделать дом чуточку
непохожим на другие.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Структура классов паттерна Шаблонный Метод" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Структура классов паттерна Шаблонный Метод" />
</figure>
:::

1.  **Абстрактный класс** определяет шаги алгоритма и содержит шаблонный
    метод, состоящий из вызовов этих шагов. Шаги могут быть как
    абстрактными, так и содержать реализацию по умолчанию.

2.  **Конкретный класс** переопределяет некоторые (или все) шаги
    алгоритма. Конкретные классы не переопределяют сам шаблонный метод.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Шаблонный метод** используется как заготовка для
стандартного искусственного интеллекта в простой игре-стратегии. Для
введения в игру новой расы достаточно создать подкласс и реализовать в
нём недостающие методы.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура классов примера паттерна Шаблонный метод" />
<figcaption><p>Пример классов искусственного интеллекта для
простой игры.</p></figcaption>
</figure>

Все расы игры будут содержать примерно такие же типы юнитов и строений,
поэтому структура ИИ будет одинаковой. Но разные расы могут по-разному
реализовать эти шаги. Так, например, орки будут агрессивней в атаке,
люди --- более активны в защите, а дикие монстры вообще не будут
заниматься строительством.

<figure class="code">
<pre class="code" lang="pseudocode"><code>class GameAI is
    // Шаблонный метод должен быть задан в базовом классе. Он
    // состоит из вызовов методов в определённом порядке. Чаще
    // всего эти методы являются шагами некоего алгоритма.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Некоторые из этих методов могут быть реализованы прямо в
    // базовом классе.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // А некоторые могут быть полностью абстрактными.
    abstract method buildStructures()
    abstract method buildUnits()

    // Кстати, шаблонных методов в классе может быть несколько.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Подклассы могут предоставлять свою реализацию шагов
// алгоритма, не изменяя сам шаблонный метод.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Строить фермы, затем бараки, а потом цитадель.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Построить раба и добавить в группу
                // разведчиков.
            else
                // Построить пехотинца и добавить в группу
                // воинов.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Отправить разведчиков на позицию.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Отправить воинов на позицию.

// Подклассы могут не только реализовывать абстрактные шаги, но
// и переопределять шаги, уже реализованные в базовом классе.
class MonstersAI extends GameAI is
    method collectResources() is
        // Ничего не делать.

    method buildStructures() is
        // Ничего не делать.

    method buildUnits() is
        // Ничего не делать.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда подклассы должны расширять базовый алгоритм, не меняя его
структуры.
:::

::: applicability-solution
Шаблонный метод позволяет подклассам расширять определённые шаги
алгоритма через наследование, не меняя при этом структуру алгоритмов,
объявленную в базовом классе.
:::

::: applicability-problem
Когда у вас есть несколько классов, делающих одно и то же с
незначительными отличиями. Если вы редактируете один класс, то
приходится вносить такие же правки и в остальные классы.
:::

::: applicability-solution
Паттерн шаблонный метод предлагает создать для похожих классов общий
суперкласс и оформить в нём главный алгоритм в виде шагов. Отличающиеся
шаги можно переопределить в подклассах.

Это позволит убрать дублирование кода в нескольких классах с похожим
поведением, но отличающихся в деталях.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Изучите алгоритм и подумайте, можно ли его разбить на шаги.
    Прикиньте, какие шаги будут стандартными для всех вариаций
    алгоритма, а какие --- изменяющимися.

2.  Создайте абстрактный базовый класс. Определите в нём шаблонный
    метод. Этот метод должен состоять из вызовов шагов алгоритма. Имеет
    смысл сделать шаблонный метод финальным, чтобы подклассы не могли
    переопределить его (если ваш язык программирования это позволяет).

3.  Добавьте в абстрактный класс методы для каждого из шагов алгоритма.
    Вы можете сделать эти методы абстрактными или добавить какую-то
    реализацию по умолчанию. В первом случае все подклассы *должны*
    будут реализовать эти методы, а во втором --- только если реализация
    шага в подклассе отличается от стандартной версии.

4.  Подумайте о введении в алгоритм хуков. Чаще всего, хуки располагают
    между основными шагами алгоритма, а также до и после всех шагов.

5.  Создайте конкретные классы, унаследовав их от абстрактного класса.
    Реализуйте в них все недостающие шаги и хуки.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Облегчает повторное использование кода.
:::

::: col-sm-6
-    Вы жёстко ограничены скелетом существующего алгоритма.
-    Вы можете нарушить *принцип подстановки Барбары Лисков*, изменяя
    базовое поведение одного из шагов алгоритма через подкласс.
-    С ростом количества шагов шаблонный метод становится слишком сложно
    поддерживать.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Фабричный метод](factory-method.html) можно рассматривать как
    частный случай [Шаблонного метода](template-method.html). Кроме
    того, *Фабричный метод* нередко бывает частью большого класса с
    *Шаблонными методами*.

-   [Шаблонный метод](template-method.html) использует наследование,
    чтобы расширять части алгоритма. [Стратегия](strategy.html)
    использует делегирование, чтобы изменять выполняемые алгоритмы на
    лету. *Шаблонный метод* работает на уровне классов. *Стратегия*
    позволяет менять логику отдельных объектов.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Шаблонный метод на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Шаблонный метод на C#"){.prog-lang-link}
[![Шаблонный метод на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Шаблонный метод на C++"){.prog-lang-link}
[![Шаблонный метод на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Шаблонный метод на Go"){.prog-lang-link}
[![Шаблонный метод на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Шаблонный метод на Java"){.prog-lang-link}
[![Шаблонный метод на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Шаблонный метод на PHP"){.prog-lang-link}
[![Шаблонный метод на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Шаблонный метод на Python"){.prog-lang-link}
[![Шаблонный метод на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Шаблонный метод на Ruby"){.prog-lang-link}
[![Шаблонный метод на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Шаблонный метод на Rust"){.prog-lang-link}
[![Шаблонный метод на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Шаблонный метод на Swift"){.prog-lang-link}
[![Шаблонный метод на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Шаблонный метод на TypeScript"){.prog-lang-link}
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

[Посетитель []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Стратегия](strategy.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
