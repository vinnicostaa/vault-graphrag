::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Состояние {#состояние .title}

::: aka
Также известен как: [State]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Состояние** --- это поведенческий паттерн проектирования, который
позволяет объектам менять поведение в зависимости от своего состояния.
Извне создаётся впечатление, что изменился класс объекта.

<figure class="image">
<img
src="../../images/patterns/content/state/state-ru6be0.png?id=ffd3a3d9e82ff0f947cb2321b3a43375"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-ru.png?id=ffd3a3d9e82ff0f947cb2321b3a43375 640w,/images/patterns/content/state/state-ru-1.5x.png?id=faf870826cde6a7518e92b1d3170d7e5 960w,/images/patterns/content/state/state-ru-2x.png?id=dc0c78340356c82d013200759f16c822 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Состояние" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Паттерн Состояние невозможно рассматривать в отрыве от концепции *машины
состояний*, также известной как *стейт-машина* или *конечный
автомат* []{.pop-i}[Конечный автомат:
[https://refactoring.guru/ru/fsm](https://ru.wikipedia.org/wiki/%D0%9A%D0%BE%D0%BD%D0%B5%D1%87%D0%BD%D1%8B%D0%B9_%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Конечный автомат" />
<figcaption><p>Конечный автомат.</p></figcaption>
</figure>

Основная идея в том, что программа может находиться в одном из
нескольких состояний, которые всё время сменяют друг друга. Набор этих
состояний, а также переходов между ними, предопределён и *конечен*.
Находясь в разных состояниях, программа может по-разному реагировать на
одни и те же события, которые происходят с ней.

Такой подход можно применить и к отдельным объектам. Например, объект
`Документ` может принимать три состояния: `Черновик`, `Модерация` или
`Опубликован`. В каждом из этих состоянии метод `опубликовать` будет
работать по-разному:

-   Из черновика он отправит документ на модерацию.
-   Из модерации --- в публикацию, но при условии, что это сделал
    администратор.
-   В опубликованном состоянии метод не будет делать ничего.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-ru748b.png?id=41e38a78c644750e19603a0d867e5ab6"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-ru.png?id=41e38a78c644750e19603a0d867e5ab6 560w,/images/patterns/diagrams/state/problem2-ru-1.5x.png?id=dc34dca01909fc9703966ff1735a1f06 840w,/images/patterns/diagrams/state/problem2-ru-2x.png?id=f89d1a206d2b551efdc0f81541c16b09 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Возможные состояния документа и переходы между ними" />
<figcaption><p>Возможные состояния документа и переходы
между ними.</p></figcaption>
</figure>

Машину состояний чаще всего реализуют с помощью множества условных
операторов, `if` либо `switch`, которые проверяют текущее состояние
объекта и выполняют соответствующее поведение. Наверняка вы уже
реализовали хотя бы одну машину состояний в своей жизни, даже не зная об
этом. Как насчёт вот такого кода, выглядит знакомо?

<figure class="code">
<pre class="code" lang="pseudocode"><code>class Document is
    field state: string
    // ...
    method publish() is
        switch (state)
            &quot;draft&quot;:
                state = &quot;moderation&quot;
                break
            &quot;moderation&quot;:
                if (currentUser.role == &quot;admin&quot;)
                    state = &quot;published&quot;
                break
            &quot;published&quot;:
                // Do nothing.
                break
    // ...</code></pre>
</figure>

Основная проблема такой машины состояний проявится в том случае, если в
`Документ` добавить ещё десяток состояний. Каждый метод будет состоять
из увесистого условного оператора, перебирающего доступные состояния.
Такой код крайне сложно поддерживать. Малейшее изменение логики
переходов заставит вас перепроверять работу всех методов, которые
содержат условные операторы машины состояний.

Путаница и нагромождение условий особенно сильно проявляется в старых
проектах. Набор возможных состояний бывает трудно предопределить
заранее, поэтому они всё время добавляются в процессе эволюции
программы. Из-за этого решение, которое выглядело простым и эффективным
в самом начале разработки, может впоследствии стать проекцией большого
макаронного монстра.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Состояние предлагает создать отдельные классы для каждого
состояния, в котором может пребывать объект, а затем вынести туда
поведения, соответствующие этим состояниям.

Вместо того, чтобы хранить код всех состояний, первоначальный объект,
называемый *контекстом*, будет содержать ссылку на один из
объектов-состояний и делегировать ему работу, зависящую от состояния.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-ru9832.png?id=add4fcfcf144a29cbe27526108012377"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-ru.png?id=add4fcfcf144a29cbe27526108012377 490w,/images/patterns/diagrams/state/solution-ru-1.5x.png?id=46a71c18323c17b4328e35f9a13a13e4 735w,/images/patterns/diagrams/state/solution-ru-2x.png?id=455d71443da13fc9cc3d4bc84da6e276 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Документ делегирует работу своему активному объекту-состоянию" />
<figcaption><p>Документ делегирует работу своему
активному объекту-состоянию.</p></figcaption>
</figure>

Благодаря тому, что объекты состояний будут иметь общий интерфейс,
контекст сможет делегировать работу состоянию, не привязываясь к его
классу. Поведение контекста можно будет изменить в любой момент,
подключив к нему другой объект-состояние.

Очень важным нюансом, отличающим этот паттерн от
[Стратегии](strategy.html), является то, что и контекст, и сами
конкретные состояния могут знать друг о друге и инициировать переходы от
одного состояния к другому.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

Ваш смартфон ведёт себя по-разному, в зависимости от текущего состояния:

-   Когда телефон разблокирован, нажатие кнопок телефона приводит к
    каким-то действиям.
-   Когда телефон заблокирован, нажатие кнопок приводит к экрану
    разблокировки.
-   Когда телефон разряжен, нажатие кнопок приводит к экрану зарядки.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-ru8a4e.png?id=e31e364041f13593df1e49a6270de8b7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-ru.png?id=e31e364041f13593df1e49a6270de8b7 540w,/images/patterns/diagrams/state/structure-ru-1.5x.png?id=2c4e76f2c2ab2aeaa237d5fe7266a009 810w,/images/patterns/diagrams/state/structure-ru-2x.png?id=af0dedbd5a7b509161ea4151c6f8fe3a 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура классов паттерна Состояние" /><img
src="../../images/patterns/diagrams/state/structure-ru-indexed955b.png?id=c5c8050bca281a7ecac39676d440f524"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-ru-indexed.png?id=c5c8050bca281a7ecac39676d440f524 540w,/images/patterns/diagrams/state/structure-ru-indexed-1.5x.png?id=01a5b005fdb2605f677abca4da5914d4 810w,/images/patterns/diagrams/state/structure-ru-indexed-2x.png?id=f97a92d19f87903e6649ef51c87530a3 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура классов паттерна Состояние" />
</figure>
:::

1.  **Контекст** хранит ссылку на объект состояния и делегирует ему
    часть работы, зависящей от состояний. Контекст работает с этим
    объектом через общий интерфейс состояний. Контекст должен иметь
    метод для присваивания ему нового объекта-состояния.

2.  **Состояние** описывает общий интерфейс для всех конкретных
    состояний.

3.  **Конкретные состояния** реализуют поведения, связанные с
    определённым состоянием контекста. Иногда приходится создавать целые
    иерархии классов состояний, чтобы обобщить дублирующий код.

    Состояние может иметь обратную ссылку на объект контекста. Через неё
    не только удобно получать из контекста нужную информацию, но и
    осуществлять смену его состояния.

4.  И контекст, и объекты конкретных состояний могут решать, когда и
    какое следующее состояние будет выбрано. Чтобы переключить
    состояние, нужно подать другой объект-состояние в контекст.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере паттерн **Состояние** изменяет функциональность одних и
тех же элементов управления музыкальным проигрывателем, в зависимости от
того, в каком состоянии находится сейчас проигрыватель.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Структура классов примера паттерна Состояние" />
<figcaption><p>Пример изменение поведения проигрывателя с
помощью состояний.</p></figcaption>
</figure>

Объект проигрывателя содержит объект-состояние, которому и делегирует
основную работу. Изменяя состояния, можно менять то, как ведут себя
элементы управления проигрывателя.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс всех состояний.
abstract class State is
    protected field player: AudioPlayer

    // Контекст передаёт себя в конструктор состояния, чтобы
    // состояние могло обращаться к его данным и методам в
    // будущем, если потребуется.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Конкретные состояния реализуют методы абстрактного состояния
// по-своему.
class LockedState extends State is

    // При разблокировке проигрователя с заблокированными
    // клавишами он может принять одно из двух состояний.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Ничего не делать.

    method clickNext() is
        // Ничего не делать.

    method clickPrevious() is
        // Ничего не делать.


// Конкретные состояния сами могут переводить контекст в другое
// состояние.
class ReadyState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.startPlayback()
        player.changeState(new PlayingState(player))

    method clickNext() is
        player.nextSong()

    method clickPrevious() is
        player.previousSong()


class PlayingState extends State is
    method clickLock() is
        player.changeState(new LockedState(player))

    method clickPlay() is
        player.stopPlayback()
        player.changeState(new ReadyState(player))

    method clickNext() is
        if (event.doubleclick)
            player.nextSong()
        else
            player.fastForward(5)

    method clickPrevious() is
        if (event.doubleclick)
            player.previous()
        else
            player.rewind(5)


// Проигрыватель выступает в роли контекста.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Контекст заставляет состояние реагировать на
        // пользовательский ввод вместо себя. Реакция может быть
        // разной, в зависимости от того, какое состояние сейчас
        // активно.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Другие объекты тоже должны иметь возможность заменять
    // состояние проигрывателя.
    method changeState(state: State) is
        this.state = state

    // Методы UI будут делегировать работу активному состоянию.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Сервисные методы контекста, вызываемые состояниями.
    method startPlayback() is
        // ...
    method stopPlayback() is
        // ...
    method nextSong() is
        // ...
    method previousSong() is
        // ...
    method fastForward(time) is
        // ...
    method rewind(time) is
        // ...</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда у вас есть объект, поведение которого кардинально меняется в
зависимости от внутреннего состояния, причём типов состояний много, и их
код часто меняется.
:::

::: applicability-solution
Паттерн предлагает выделить в собственные классы все поля и методы,
связанные с определёнными состояниями. Первоначальный объект будет
постоянно ссылаться на один из объектов-состояний, делегируя ему часть
своей работы. Для изменения состояния в контекст достаточно будет
подставить другой объект-состояние.
:::

::: applicability-problem
Когда код класса содержит множество больших, похожих друг на друга,
условных операторов, которые выбирают поведения в зависимости от текущих
значений полей класса.
:::

::: applicability-solution
Паттерн предлагает переместить каждую ветку такого условного оператора в
собственный класс. Тут же можно поселить и все поля, связанные с данным
состоянием.
:::

::: applicability-problem
Когда вы сознательно используете табличную машину состояний, построенную
на условных операторах, но вынуждены мириться с дублированием кода для
похожих состояний и переходов.
:::

::: applicability-solution
Паттерн Состояние позволяет реализовать иерархическую машину состояний,
базирующуюся на наследовании. Вы можете отнаследовать похожие состояния
от одного родительского класса и вынести туда весь дублирующий код.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определитесь с классом, который будет играть роль контекста. Это
    может быть как существующий класс, в котором уже есть зависимость от
    состояния, так и новый класс, если код состояний размазан по
    нескольким классам.

2.  Создайте общий интерфейс состояний. Он должен описывать методы,
    общие для всех состояний, обнаруженных в контексте. Заметьте, что не
    всё поведение контекста нужно переносить в состояние, а только то,
    которое зависит от состояний.

3.  Для каждого фактического состояния создайте класс, реализующий
    интерфейс состояния. Переместите код, связанный с конкретными
    состояниями в нужные классы. В конце концов, все методы интерфейса
    состояния должны быть реализованы во всех классах состояний.

    При переносе поведения из контекста вы можете столкнуться с тем, что
    это поведение зависит от приватных полей или методов контекста, к
    которым нет доступа из объекта состояния. Существует парочка
    способов обойти эту проблему.

    Самый простой --- оставить поведение внутри контекста, вызывая его
    из объекта состояния. С другой стороны, вы можете сделать классы
    состояний вложенными в класс контекста, и тогда они получат доступ
    ко всем приватным частям контекста. Но последний способ доступен
    только в некоторых языках программирования (например, Java, C#).

4.  Создайте в контексте поле для хранения объектов-состояний, а также
    публичный метод для изменения значения этого поля.

5.  Старые методы контекста, в которых находился зависимый от состояния
    код, замените на вызовы соответствующих методов объекта-состояния.

6.  В зависимости от бизнес-логики, разместите код, который переключает
    состояние контекста либо внутри контекста, либо внутри классов
    конкретных состояний.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Избавляет от множества больших условных операторов машины
    состояний.
-    Концентрирует в одном месте код, связанный с определённым
    состоянием.
-    Упрощает код контекста.
:::

::: col-sm-6
-    Может неоправданно усложнить код, если состояний мало и они редко
    меняются.
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

-   [Состояние](state.html) можно рассматривать как надстройку над
    [Стратегией](strategy.html). Оба паттерна используют композицию,
    чтобы менять поведение основного объекта, делегируя работу вложенным
    объектам-помощникам. Однако в *Стратегии* эти объекты не знают друг
    о друге и никак не связаны. В *Состоянии* сами конкретные состояния
    могут переключать контекст.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Состояние на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "Состояние на C#"){.prog-lang-link}
[![Состояние на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "Состояние на C++"){.prog-lang-link}
[![Состояние на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "Состояние на Go"){.prog-lang-link}
[![Состояние на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "Состояние на Java"){.prog-lang-link}
[![Состояние на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "Состояние на PHP"){.prog-lang-link}
[![Состояние на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "Состояние на Python"){.prog-lang-link}
[![Состояние на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "Состояние на Ruby"){.prog-lang-link}
[![Состояние на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "Состояние на Rust"){.prog-lang-link}
[![Состояние на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "Состояние на Swift"){.prog-lang-link}
[![Состояние на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "Состояние на TypeScript"){.prog-lang-link}
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

[Стратегия []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Наблюдатель](observer.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
