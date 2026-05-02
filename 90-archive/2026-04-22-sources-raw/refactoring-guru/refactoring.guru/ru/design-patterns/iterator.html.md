::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Итератор {#итератор .title}

::: aka
Также известен как: [Iterator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Итератор** --- это поведенческий паттерн проектирования, который даёт
возможность последовательно обходить элементы составных объектов, не
раскрывая их внутреннего представления.

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-enfc4d.png?id=d19123d71d355d01b0ede4be173eb695"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-en.png?id=d19123d71d355d01b0ede4be173eb695 640w,/images/patterns/content/iterator/iterator-en-1.5x.png?id=54aa477cafffe8f9b1b6e63c2e88c21b 960w,/images/patterns/content/iterator/iterator-en-2x.png?id=2a85705e8e5fab257802b2ce36d6d236 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Итератор" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Коллекции --- самая распространённая структура данных, которую вы можете
встретить в программировании. Это набор объектов, собранный в одну кучу
по каким-то критериям.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Разные типы коллекций" />
<figcaption><p>Разные типы коллекций.</p></figcaption>
</figure>

Большинство коллекций выглядят как обычный список элементов. Но есть и
экзотические коллекции, построенные на основе деревьев, графов и других
сложных структур данных.

Но как бы ни была структурирована коллекция, пользователь должен иметь
возможность последовательно обходить её элементы, чтобы проделывать с
ними какие-то действия.

Но каким способом следует перемещаться по сложной структуре данных?
Например, сегодня может быть достаточным обход дерева в глубину, но
завтра потребуется возможность перемещаться по дереву в ширину. А на
следующей неделе и того хуже --- понадобится обход коллекции в случайном
порядке.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Одну и ту же коллекцию можно обходить разными способами" />
<figcaption><p>Одну и ту же коллекцию можно обходить
разными способами.</p></figcaption>
</figure>

Добавляя всё новые алгоритмы в код коллекции, вы понемногу размываете её
основную задачу, которая заключается в эффективном хранении данных.
Некоторые алгоритмы могут быть и вовсе слишком «заточены» под
определённое приложение и смотреться дико в общем классе коллекции.
:::

::: {.section .solution}
##  Решение {#solution}

Идея паттерна Итератор состоит в том, чтобы вынести поведение обхода
коллекции из самой коллекции в отдельный класс.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Итераторы содержат код обхода коллекции" />
<figcaption><p>Итераторы содержат код обхода коллекции. Одну коллекцию
могут обходить сразу несколько итераторов.</p></figcaption>
</figure>

Объект-итератор будет отслеживать состояние обхода, текущую позицию в
коллекции и сколько элементов ещё осталось обойти. Одну и ту же
коллекцию смогут одновременно обходить различные итераторы, а сама
коллекция не будет даже знать об этом.

К тому же, если вам понадобится добавить новый способ обхода, вы сможете
создать отдельный класс итератора, не изменяя существующий код
коллекции.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-ru4411.png?id=54e6a360bdc2857c9b4a2823d19755b2"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-ru.png?id=54e6a360bdc2857c9b4a2823d19755b2 640w,/images/patterns/content/iterator/iterator-comic-1-ru-1.5x.png?id=a3a5dce019fcab361ea1c3558de1d020 960w,/images/patterns/content/iterator/iterator-comic-1-ru-2x.png?id=be84181899a2ef6d620b3dcaaa4aa2fe 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Варианты прогулок по Риму" />
<figcaption><p>Варианты прогулок по Риму.</p></figcaption>
</figure>

Вы планируете полететь в Рим и обойти все достопримечательности за пару
дней. Но приехав, вы можете долго петлять узкими улочками, пытаясь найти
Колизей.

Если у вас ограниченный бюджет --- не беда. Вы можете воспользоваться
виртуальным гидом, скачанным на телефон, который позволит отфильтровать
только интересные вам точки. А можете плюнуть и нанять локального гида,
который хоть и обойдётся в копеечку, но знает город как свои пять
пальцев, и сможет посвятить вас во все городские легенды.

Таким образом, Рим выступает коллекцией достопримечательностей, а ваш
мозг, навигатор или гид --- итератором по коллекции. Вы, как клиентский
код, можете выбрать один из итераторов, отталкиваясь от решаемой задачи
и доступных ресурсов.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/structure5247.png?id=35ea851f8f6bbe51d79eb91e6e6519d0"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.12;"
srcset="/images/patterns/diagrams/iterator/structure.png?id=35ea851f8f6bbe51d79eb91e6e6519d0 480w,/images/patterns/diagrams/iterator/structure-1.5x.png?id=19d4c2130ab2e3bd718f87e956fcd873 720w,/images/patterns/diagrams/iterator/structure-2x.png?id=b7b4708d3f279dd046eb1692f1cba710 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Структура классов паттерна Итератор" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура классов паттерна Итератор" />
</figure>
:::

1.  **Итератор** описывает интерфейс для доступа и обхода элементов
    коллекции.

2.  **Конкретный итератор** реализует алгоритм обхода какой-то
    конкретной коллекции. Объект итератора должен сам отслеживать
    текущую позицию при обходе коллекции, чтобы отдельные итераторы
    могли обходить одну и ту же коллекцию независимо.

3.  **Коллекция** описывает интерфейс получения итератора из коллекции.
    Как мы уже говорили, коллекции не всегда являются списком. Это может
    быть и база данных, и удалённое API, и даже дерево
    [Компоновщика](composite.html). Поэтому сама коллекция может
    создавать итераторы, так как она знает, какие именно итераторы
    способны с ней работать.

4.  **Конкретная коллекция** возвращает новый экземпляр определённого
    конкретного итератора, связав его с текущим объектом коллекции.
    Обратите внимание, что сигнатура метода возвращает интерфейс
    итератора. Это позволяет клиенту не зависеть от конкретных классов
    итераторов.

5.  **Клиент** работает со всеми объектами через интерфейсы коллекции и
    итератора. Так клиентский код не зависит от конкретных классов, что
    позволяет применять различные итераторы, не изменяя существующий код
    программы.

    В общем случае клиенты не создают объекты итераторов, а получают их
    из коллекций. Тем не менее, если клиенту требуется специальный
    итератор, он всегда может создать его самостоятельно.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере паттерн **Итератор** используется для реализации обхода
нестандартной коллекции, которая инкапсулирует доступ к социальному
графу Facebook. Коллекция предоставляет несколько итераторов, которые
могут по-разному обходить профили людей.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура классов примера паттерна Итератор" />
<figcaption><p>Пример обхода социальных профилей
через итератор.</p></figcaption>
</figure>

Так, итератор друзей перебирает всех друзей профиля, а итератор
коллег --- фильтрует друзей по принадлежности к компании профиля. Все
итераторы реализуют общий интерфейс, который позволяет клиентам работать
с профилями, не вникая в детали работы с социальной сетью (например, в
авторизацию, отправку REST-запросов и т. д.)

Кроме того, Итератор избавляет код от привязки к конкретным классам
коллекций. Это позволяет добавить поддержку другого вида коллекций
(например, LinkedIn), не меняя клиентский код, который работает с
итераторами и коллекциями.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс коллекций должен определить фабричный метод
// для производства итератора. Можно определить сразу несколько
// методов, чтобы дать пользователям различные варианты обхода
// одной и той же коллекции.
interface SocialNetwork is
    method createFriendsIterator(profileId):ProfileIterator
    method createCoworkersIterator(profileId):ProfileIterator


// Конкретная коллекция знает, объекты каких итераторов нужно
// создавать.
class Facebook implements SocialNetwork is
    // ...Основной код коллекции...

    // Код получения нужного итератора.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// Общий интерфейс итераторов.
interface ProfileIterator is
    method getNext():Profile
    method hasMore():bool


// Конкретный итератор.
class FacebookIterator implements ProfileIterator is
    // Итератору нужна ссылка на коллекцию, которую он обходит.
    private field facebook: Facebook
    private field profileId, type: string

    // Но каждый итератор обходит коллекцию, независимо от
    // остальных, поэтому он содержит информацию о текущей
    // позиции обхода.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Итератор реализует методы базового интерфейса по-своему.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Вот ещё полезная тактика: мы можем передавать объект
// итератора вместо коллекции в клиентские классы. При таком
// подходе клиентский код не будет иметь доступа к коллекциям, а
// значит, его не будут волновать подробности их реализаций. Ему
// будет доступен только общий интерфейс итераторов.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// Класс приложение конфигурирует классы, как захочет.
class Application is
    field network: SocialNetwork
    field spammer: SocialSpammer

    method config() is
        if working with Facebook
            this.network = new Facebook()
        if working with LinkedIn
            this.network = new LinkedIn()
        this.spammer = new SocialSpammer()

    method sendSpamToFriends(profile) is
        iterator = network.createFriendsIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)

    method sendSpamToCoworkers(profile) is
        iterator = network.createCoworkersIterator(profile.getId())
        spammer.send(iterator, &quot;Very important message&quot;)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда у вас есть сложная структура данных, и вы хотите скрыть от клиента
детали её реализации (из-за сложности или вопросов безопасности).
:::

::: applicability-solution
Итератор предоставляет клиенту всего несколько простых методов перебора
элементов коллекции. Это не только упрощает доступ к коллекции, но и
защищает её данные от неосторожных или злоумышленных действий.
:::

::: applicability-problem
Когда вам нужно иметь несколько вариантов обхода одной и той же
структуры данных.
:::

::: applicability-solution
Нетривиальные алгоритмы обхода структуры данных могут иметь довольно
объёмный код. Этот код будет захламлять всё вокруг --- будь то сам класс
коллекции или часть бизнес-логики программы. Применив итератор, вы
можете выделить код обхода структуры данных в собственный класс,
упростив поддержку остального кода.
:::

::: applicability-problem
Когда вам хочется иметь единый интерфейс обхода различных структур
данных.
:::

::: applicability-solution
Итератор позволяет вынести реализации различных вариантов обхода в
подклассы. Это позволит легко взаимозаменять объекты итераторов, в
зависимости от того, с какой структурой данных приходится работать.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Создайте общий интерфейс итераторов. Обязательный минимум --- это
    операция получения следующего элемента коллекции. Но для удобства
    можно предусмотреть и другое. Например, методы для получения
    предыдущего элемента, текущей позиции, проверки окончания обхода и
    прочие.

2.  Создайте интерфейс коллекции и опишите в нём метод получения
    итератора. Важно, чтобы сигнатура метода возвращала общий интерфейс
    итераторов, а не один из конкретных итераторов.

3.  Создайте классы конкретных итераторов для тех коллекций, которые
    нужно обходить с помощью паттерна. Итератор должен быть привязан
    только к одному объекту коллекции. Обычно эта связь устанавливается
    через конструктор.

4.  Реализуйте методы получения итератора в конкретных классах
    коллекций. Они должны создавать новый итератор того класса, который
    способен работать с данным типом коллекции. Коллекция должна
    передавать ссылку на собственный объект в конструктор итератора.

5.  В клиентском коде и в классах коллекций не должно остаться кода
    обхода элементов. Клиент должен получать новый итератор из объекта
    коллекции каждый раз, когда ему нужно перебрать её элементы.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Упрощает классы хранения данных.
-    Позволяет реализовать различные способы обхода структуры данных.
-    Позволяет одновременно перемещаться по структуре данных в разные
    стороны.
:::

::: col-sm-6
-    Не оправдан, если можно обойтись простым циклом.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   Вы можете обходить дерево [Компоновщика](composite.html), используя
    [Итератор](iterator.html).

-   [Фабричный метод](factory-method.html) можно использовать вместе с
    [Итератором](iterator.html), чтобы подклассы коллекций могли
    создавать подходящие им итераторы.

-   [Снимок](memento.html) можно использовать вместе с
    [Итератором](iterator.html), чтобы сохранить текущее состояние
    обхода структуры данных и вернуться к нему в будущем, если
    потребуется.

-   [Посетитель](visitor.html) можно использовать совместно с
    [Итератором](iterator.html). *Итератор* будет отвечать за обход
    структуры данных, а *Посетитель* --- за выполнение действий над
    каждым её компонентом.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Итератор на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Итератор на C#"){.prog-lang-link}
[![Итератор на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Итератор на C++"){.prog-lang-link}
[![Итератор на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Итератор на Go"){.prog-lang-link}
[![Итератор на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Итератор на Java"){.prog-lang-link}
[![Итератор на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Итератор на PHP"){.prog-lang-link}
[![Итератор на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Итератор на Python"){.prog-lang-link}
[![Итератор на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Итератор на Ruby"){.prog-lang-link}
[![Итератор на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Итератор на Rust"){.prog-lang-link}
[![Итератор на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Итератор на Swift"){.prog-lang-link}
[![Итератор на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Итератор на TypeScript"){.prog-lang-link}
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

[Посредник []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Команда](command.html){.btn .btn-default
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
