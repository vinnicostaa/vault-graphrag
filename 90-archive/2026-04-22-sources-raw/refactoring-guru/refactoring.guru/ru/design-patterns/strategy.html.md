::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Стратегия {#стратегия .title}

::: aka
Также известен как: [Strategy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Стратегия** --- это поведенческий паттерн проектирования, который
определяет семейство схожих алгоритмов и помещает каждый из них в
собственный класс, после чего алгоритмы можно взаимозаменять прямо во
время исполнения программы.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Стратегия" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Вы решили написать приложение-навигатор для путешественников. Оно должно
показывать красивую и удобную карту, позволяющую с лёгкостью
ориентироваться в незнакомом городе.

Одной из самых востребованных функций являлся поиск и прокладывание
маршрутов. Пребывая в неизвестном ему городе, пользователь должен иметь
возможность указать начальную точку и пункт назначения, а навигатор ---
проложит оптимальный путь.

Первая версия вашего навигатора могла прокладывать маршрут лишь по
дорогам, поэтому отлично подходила для путешествий на автомобиле. Но,
очевидно, не все ездят в отпуск на машине. Поэтому следующим шагом вы
добавили в навигатор прокладывание пеших маршрутов.

Через некоторое время выяснилось, что некоторые люди предпочитают ездить
по городу на общественном транспорте. Поэтому вы добавили и такую опцию
прокладывания пути.

Но и это ещё не всё. В ближайшей перспективе вы хотели бы добавить
прокладывание маршрутов по велодорожкам. А в отдалённом будущем ---
интересные маршруты посещения достопримечательностей.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Код навигатора становится слишком раздутым" />
<figcaption><p>Код навигатора становится
слишком раздутым.</p></figcaption>
</figure>

Если с популярностью навигатора не было никаких проблем, то техническая
часть вызывала вопросы и периодическую головную боль. С каждым новым
алгоритмом код основного класса навигатора увеличивался вдвое. В таком
большом классе стало довольно трудно ориентироваться.

Любое изменение алгоритмов поиска, будь то исправление багов или
добавление нового алгоритма, затрагивало основной класс. Это повышало
риск сделать ошибку, случайно задев остальной работающий код.

Кроме того, осложнялась командная работа с другими программистами,
которых вы наняли после успешного релиза навигатора. Ваши изменения
нередко затрагивали один и тот же код, создавая конфликты, которые
требовали дополнительного времени на их разрешение.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Стратегия предлагает определить семейство схожих алгоритмов,
которые часто изменяются или расширяются, и вынести их в собственные
классы, называемые *стратегиями*.

Вместо того, чтобы изначальный класс сам выполнял тот или иной алгоритм,
он будет играть роль контекста, ссылаясь на одну из стратегий и
делегируя ей выполнение работы. Чтобы сменить алгоритм, вам будет
достаточно подставить в контекст другой объект-стратегию.

Важно, чтобы все стратегии имели общий интерфейс. Используя этот
интерфейс, контекст будет независимым от конкретных классов стратегий. С
другой стороны, вы сможете изменять и добавлять новые виды алгоритмов,
не трогая код контекста.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Стратегии построения пути" />
<figcaption><p>Стратегии построения пути.</p></figcaption>
</figure>

В нашем примере каждый алгоритм поиска пути переедет в свой собственный
класс. В этих классах будет определён лишь один метод, принимающий в
параметрах координаты начала и конца пути, а возвращающий массив точек
маршрута.

Хотя каждый класс будет прокладывать маршрут по-своему, для навигатора
это не будет иметь никакого значения, так как его работа заключается
только в отрисовке маршрута. Навигатору достаточно подать в стратегию
данные о начале и конце маршрута, чтобы получить массив точек маршрута в
оговорённом формате.

Класс навигатора будет иметь метод для установки стратегии, позволяя
изменять стратегию поиска пути на лету. Такой метод пригодится
клиентскому коду навигатора, например, переключателям типов маршрутов в
пользовательском интерфейсе.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-en590c.png?id=f64d7d8e6f83a7792a769039a66007c1"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-en.png?id=f64d7d8e6f83a7792a769039a66007c1 640w,/images/patterns/content/strategy/strategy-comic-1-en-1.5x.png?id=b952bc8d43f5e4e0db81b8e3c62dc205 960w,/images/patterns/content/strategy/strategy-comic-1-en-2x.png?id=7eb14bd7920ad630c1ecf448d40602df 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Способы передвижения" />
<figcaption><p>Различные стратегии попадания
в аэропорт.</p></figcaption>
</figure>

Вам нужно добраться до аэропорта. Можно доехать на автобусе, такси или
велосипеде. Здесь вид транспорта является стратегией. Вы выбираете
конкретную стратегию в зависимости от контекста --- наличия денег или
времени до отлёта.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/structuree491.png?id=c6aa910c94960f35d100bfca02810ea1"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/strategy/structure.png?id=c6aa910c94960f35d100bfca02810ea1 440w,/images/patterns/diagrams/strategy/structure-1.5x.png?id=5b1c6376b06374719dcae95455b068d8 660w,/images/patterns/diagrams/strategy/structure-2x.png?id=5bd791857c3bab419bcf4fa86877439d 880w"
sizes="(max-width: 720px) 100vw, 440px" loading="lazy" width="440"
alt="Структура классов паттерна Стратегия" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Структура классов паттерна Стратегия" />
</figure>
:::

1.  **Контекст** хранит ссылку на объект конкретной стратегии, работая с
    ним через общий интерфейс стратегий.

2.  **Стратегия** определяет интерфейс, общий для всех вариаций
    алгоритма. Контекст использует этот интерфейс для вызова алгоритма.

    Для контекста неважно, какая именно вариация алгоритма будет
    выбрана, так как все они имеют одинаковый интерфейс.

3.  **Конкретные стратегии** реализуют различные вариации алгоритма.

4.  Во время выполнения программы контекст получает вызовы от клиента и
    делегирует их объекту конкретной стратегии.

5.  Клиент должен создать объект конкретной стратегии и передать его в
    конструктор контекста. Кроме этого, клиент должен иметь возможность
    заменить стратегию на лету, используя сеттер. Благодаря этому,
    контекст не будет знать о том, какая именно стратегия сейчас
    выбрана.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере контекст использует **Стратегию** для выполнения той или
иной арифметической операции.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс всех стратегий.
interface Strategy is
    method execute(a, b)

// Каждая конкретная стратегия реализует общий интерфейс своим
// способом.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// Контекст всегда работает со стратегиями через общий
// интерфейс. Он не знает, какая именно стратегия ему подана.
class Context is
    private strategy: Strategy

    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// Конкретная стратегия выбирается на более высоком уровне,
// например, конфигуратором всего приложения. Готовый объект-
// стратегия подаётся в клиентский объект, а затем может быть
// заменён другой стратегией в любой момент на лету.
class ExampleApplication is
    method main() is
        // 1. Создать объект контекста.
        // 2. Получить первое число (n1).
        // 3. Получить второе число (n2).
        // 4. Получить желаемую операцию.
        // 5. Затем, выбрать стратегию:

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        // 6. Выполнить операцию с помощью стратегии:
        result = context.executeStrategy(n1, n2)

        // 7. Вывести результат на экран.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::::: applicability
::: applicability-problem
Когда вам нужно использовать разные вариации какого-то алгоритма внутри
одного объекта.
:::

::: applicability-solution
Стратегия позволяет варьировать поведение объекта во время выполнения
программы, подставляя в него различные объекты-поведения (например,
отличающиеся балансом скорости и потребления ресурсов).
:::

::: applicability-problem
Когда у вас есть множество похожих классов, отличающихся только
некоторым поведением.
:::

::: applicability-solution
Стратегия позволяет вынести отличающееся поведение в отдельную иерархию
классов, а затем свести первоначальные классы к одному, сделав поведение
этого класса настраиваемым.
:::

::: applicability-problem
Когда вы не хотите обнажать детали реализации алгоритмов для других
классов.
:::

::: applicability-solution
Стратегия позволяет изолировать код, данные и зависимости алгоритмов от
других объектов, скрыв эти детали внутри классов-стратегий.
:::

::: applicability-problem
Когда различные вариации алгоритмов реализованы в виде развесистого
условного оператора. Каждая ветка такого оператора представляет собой
вариацию алгоритма.
:::

::: applicability-solution
Стратегия помещает каждую лапу такого оператора в отдельный
класс-стратегию. Затем контекст получает определённый объект-стратегию
от клиента и делегирует ему работу. Если вдруг понадобится сменить
алгоритм, в контекст можно подать другую стратегию.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определите алгоритм, который подвержен частым изменениям. Также
    подойдёт алгоритм, имеющий несколько вариаций, которые выбираются во
    время выполнения программы.

2.  Создайте интерфейс стратегий, описывающий этот алгоритм. Он должен
    быть общим для всех вариантов алгоритма.

3.  Поместите вариации алгоритма в собственные классы, которые реализуют
    этот интерфейс.

4.  В классе контекста создайте поле для хранения ссылки на текущий
    объект-стратегию, а также метод для её изменения. Убедитесь в том,
    что контекст работает с этим объектом только через общий интерфейс
    стратегий.

5.  Клиенты контекста должны подавать в него соответствующий
    объект-стратегию, когда хотят, чтобы контекст вёл себя определённым
    образом.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Горячая замена алгоритмов на лету.
-    Изолирует код и данные алгоритмов от остальных классов.
-    Уход от наследования к делегированию.
-    Реализует *принцип открытости/закрытости*.
:::

::: col-sm-6
-    Усложняет программу за счёт дополнительных классов.
-    Клиент должен знать, в чём состоит разница между стратегиями, чтобы
    выбрать подходящую.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Мост](bridge.html), [Стратегия](strategy.html) и
    [Состояние](state.html) (а также слегка и [Адаптер](adapter.html))
    имеют схожие структуры классов --- все они построены на принципе
    «композиции», то есть делегирования работы другим объектам. Тем не
    менее, они отличаются тем, что решают разные проблемы. Помните, что
    паттерны --- это не только рецепт построения кода определённым
    образом, но и описание проблем, которые привели к данному решению.

-   [Команда](command.html) и [Стратегия](strategy.html) похожи по духу,
    но отличаются масштабом и применением:

    -   *Команду* используют, чтобы превратить любые разнородные
        действия в объекты. Параметры операции превращаются в поля
        объекта. Этот объект теперь можно логировать, хранить в истории
        для отмены, передавать во внешние сервисы и так далее.
    -   С другой стороны, *Стратегия* описывает разные способы
        произвести одно и то же действие, позволяя взаимозаменять эти
        способы в каком-то объекте контекста.

-   [Стратегия](strategy.html) меняет поведение объекта «изнутри», а
    [Декоратор](decorator.html) изменяет его «снаружи».

-   [Шаблонный метод](template-method.html) использует наследование,
    чтобы расширять части алгоритма. [Стратегия](strategy.html)
    использует делегирование, чтобы изменять выполняемые алгоритмы на
    лету. *Шаблонный метод* работает на уровне классов. *Стратегия*
    позволяет менять логику отдельных объектов.

-   [Состояние](state.html) можно рассматривать как надстройку над
    [Стратегией](strategy.html). Оба паттерна используют композицию,
    чтобы менять поведение основного объекта, делегируя работу вложенным
    объектам-помощникам. Однако в *Стратегии* эти объекты не знают друг
    о друге и никак не связаны. В *Состоянии* сами конкретные состояния
    могут переключать контекст.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Стратегия на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Стратегия на C#"){.prog-lang-link}
[![Стратегия на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Стратегия на C++"){.prog-lang-link}
[![Стратегия на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Стратегия на Go"){.prog-lang-link}
[![Стратегия на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Стратегия на Java"){.prog-lang-link}
[![Стратегия на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Стратегия на PHP"){.prog-lang-link}
[![Стратегия на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Стратегия на Python"){.prog-lang-link}
[![Стратегия на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Стратегия на Ruby"){.prog-lang-link}
[![Стратегия на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Стратегия на Rust"){.prog-lang-link}
[![Стратегия на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Стратегия на Swift"){.prog-lang-link}
[![Стратегия на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Стратегия на TypeScript"){.prog-lang-link}
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

[Шаблонный метод []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Состояние](state.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
