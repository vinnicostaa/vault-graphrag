:::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Відвідувач {#відвідувач .title}

::: aka
Також відомий як: [Visitor]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Відвідувач** --- це поведінковий патерн проектування, що дає змогу
додавати до програми нові операції, не змінюючи класи об'єктів, над
якими ці операції можуть виконуватися.

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor4a4c.png?id=f36d100188340db7a18854ef7916f972"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/visitor/visitor.png?id=f36d100188340db7a18854ef7916f972 640w,/images/patterns/content/visitor/visitor-1.5x.png?id=751e251363b632b20df856d72d02ee72 960w,/images/patterns/content/visitor/visitor-2x.png?id=2c5d9ab3046d782c19809d3b80650d65 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Відвідувач" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Ваша команда розробляє програму, що працює з геоданими у вигляді графа.
Вузлами графа можуть бути як міста, так інші локації, такі, як пам'ятки,
великі підприємства тощо. Кожен вузол має посилання на найближчі до
нього вузли. Для кожного типу вузла існує свій власний клас, а кожен
вузол представлений окремим об'єктом.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem14338.png?id=e7076532da1e936f3519c63270da8454"
style="aspect-ratio: 2.43;"
srcset="/images/patterns/diagrams/visitor/problem1.png?id=e7076532da1e936f3519c63270da8454 560w,/images/patterns/diagrams/visitor/problem1-1.5x.png?id=250216d3458c38e9855eda96bf71251b 840w,/images/patterns/diagrams/visitor/problem1-2x.png?id=2e5d5143ac55af218754f28761bec17e 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Експорт гео-вузлів до XML" />
<figcaption><p>Експорт гео-вузлів до XML.</p></figcaption>
</figure>

Ваше завдання --- зробити експорт цього графа до XML. Справа була б
легкою, якщо б ви могли редагувати класи вузлів. У цьому випадку можна
було б додати метод експорту до кожного типу вузлів, а потім,
перебираючи всі вузли графа, викликати цей метод для кожного вузла.
Завдяки поліморфізму, рішення було б елегантним, оскільки ви могли б не
прив'язуватися до конкретних класів вузлів.

Але, на жаль, змінити класи вузлів у вас не вийшло. Системний архітектор
сказав, що код класів вузлів зараз дуже стабільний, і від нього багато
що залежить, а тому він не хоче ризикувати, дозволяючи будь-кому чіпати
цей код.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/problem2-ukdde3.png?id=631fcdaf6bf60ad851174173f61e771a"
style="aspect-ratio: 1.92;"
srcset="/images/patterns/diagrams/visitor/problem2-uk.png?id=631fcdaf6bf60ad851174173f61e771a 500w,/images/patterns/diagrams/visitor/problem2-uk-1.5x.png?id=6a7b28af2fc56a0e61c4e5c46fbc94c4 750w,/images/patterns/diagrams/visitor/problem2-uk-2x.png?id=1ddeb407060d9ffeb95b320bba1ac024 1000w"
sizes="(max-width: 720px) 100vw, 500px" loading="lazy" width="500"
alt="Код XML-експорту доведеться додати до всіх класів вузлів" />
<figcaption><p>Код XML-експорту доведеться додати до всіх класів вузлів,
а це дуже невигідно.</p></figcaption>
</figure>

До того ж він сумнівався в тому, що експорт до XML взагалі є доречним в
рамках цих класів. Їхнє основне завдання пов'язане з геоданими, а
експорт виглядає в межах цих класів, як біла ворона.

Була ще одна причина заборони. Наступного тижня вам міг знадобитися
експорт в який-небудь інший формат даних, а це призвело б до повторних
змін в класах.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Відвідувач пропонує розмістити нову поведінку в окремому класі,
замість того, щоб множити її відразу в декількох класах. Об'єкти, з
якими повинна бути пов'язана поведінка, не виконуватимуть її самостійно.
Замість цього ви будете передавати ці об'єкти до методів відвідувача.

Код поведінки, імовірно, повинен відрізнятися для об'єктів різних
класів, тому й методів у відвідувача повинно бути декілька. Назви та
принцип дії цих методів будуть подібними, а основна відмінність
торкатиметься типу, що приймається в параметрах об'єкта, наприклад:

<figure class="code">
<pre class="code" lang="pseudocode"><code>class ExportVisitor implements Visitor is
    method doForCity(City c) { ... }
    method doForIndustry(Industry f) { ... }
    method doForSightSeeing(SightSeeing ss) { ... }
    // ...</code></pre>
</figure>

Тут виникає запитання, яким чином ми будемо подавати вузли до об'єкта
відвідувача. Оскільки усі методи відрізняються сигнатурою, використати
поліморфізм при перебиранні вузлів не вийде. Доведеться перевіряти тип
вузлів для того, щоб вибрати відповідний метод відвідувача.

<figure class="code">
<pre class="code" lang="pseudocode"><code>foreach (Node node : graph)
    if (node instanceof City)
        exportVisitor.doForCity((City) node);
    if (node instanceof Industry)
        exportVisitor.doForIndustry((Industry) node);
    // ...</code></pre>
</figure>

Тут не допоможе навіть механізм перевантаження методів (доступний у Java
і C#). Якщо назвати всі методи однаково, то невизначеність реального
типу вузла все одно не дасть викликати правильний метод. Механізм
перевантаження весь час викликатиме метод відвідувача, відповідний типу
`Node`, а не реального класу поданого вузла.

Але патерн Відвідувач вирішує і цю проблему, використовуючи механізм
[подвійної диспетчеризації](visitor-double-dispatch.html). Замість того,
щоб самим шукати потрібний метод, ми можемо доручити це об'єктам, які
передаємо в параметрах відвідувачеві, а вони вже самостійно викличуть
правильний метод відвідувача.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Client code
foreach (Node node : graph)
    node.accept(exportVisitor);

// City
class City is
    method accept(Visitor v) is
        v.doForCity(this);
    // ...

// Industry
class Industry is
    method accept(Visitor v) is
        v.doForIndustry(this);
    // ...</code></pre>
</figure>

Як бачите, змінити класи вузлів все-таки доведеться. Проте ця проста
зміна дозволить застосувати до об'єктів вузлів й інші поведінки, адже
класи вузлів будуть прив'язані не до конкретного класу відвідувачів, а
до їхнього загального інтерфейсу. Тому, якщо доведеться додати до
програми нову поведінку, ви створите новий клас відвідувачів і будете
передавати його до методів вузлів.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/visitor/visitor-comic-17f02.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/visitor/visitor-comic-1.png?id=7ee4fa8800f7c4df4e1aa3b1aca2b7f1 600w,/images/patterns/content/visitor/visitor-comic-1-1.5x.png?id=c9cadd73d25cc63fce94312f3c82bfee 900w,/images/patterns/content/visitor/visitor-comic-1-2x.png?id=439032451eb49ebbcb5257f25ecee790 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Страховий агент" />
<figcaption><p>У страхового агента приготовані поліси для різних
видів організацій.</p></figcaption>
</figure>

Уявіть собі страхового агента-початківця, який прагне отримати нових
клієнтів. Він хаотично відвідує всі будинки навколо, пропонуючи свої
послуги. Але для кожного *типу* будинків, які він відвідує, у нього є
особлива пропозиція.

-   Прийшовши до будинку звичайної сім'ї, він пропонує оформити медичну
    страховку.
-   Прийшовши до банку, він пропонує страховку на випадок пограбування.
-   Прийшовши на фабрику, він пропонує страхування підприємства на
    випадок пожежі чи повені.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/structure-uk2e35.png?id=a8801912aaaa8724fe0031bd413ee293"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.96;"
srcset="/images/patterns/diagrams/visitor/structure-uk.png?id=a8801912aaaa8724fe0031bd413ee293 520w,/images/patterns/diagrams/visitor/structure-uk-1.5x.png?id=f85076f1ffe521e6ce730f268b920ba8 780w,/images/patterns/diagrams/visitor/structure-uk-2x.png?id=10f21b095d94d79d520f1d4d402b37f1 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Відвідувач" /><img
src="../../images/patterns/diagrams/visitor/structure-uk-indexed926f.png?id=c8b7d47bdfb08f57920f44234ec3e679"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/visitor/structure-uk-indexed.png?id=c8b7d47bdfb08f57920f44234ec3e679 520w,/images/patterns/diagrams/visitor/structure-uk-indexed-1.5x.png?id=3d25ec8367876019edf96caeeff9aeaa 780w,/images/patterns/diagrams/visitor/structure-uk-indexed-2x.png?id=9d40da5d10c2f8e01f17a39c098cc1c4 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Відвідувач" />
</figure>
:::

1.  **Відвідувач** описує спільний для всіх типів відвідувачів
    інтерфейс. Він оголошує набір методів, що відрізняються типом
    вхідного параметра. Кожному класу конкретних елементів повинен
    підходити свій метод. В мовах, які підтримують перевантаження
    методів, ці методи можуть мати однакові імена, але типи їхніх
    параметрів повинні відрізнятися.

2.  **Конкретні відвідувачі** реалізують якусь особливу поведінку для
    всіх типів елементів, які можна подати через методи інтерфейсу
    відвідувача.

3.  **Елемент** описує метод *прийому* відвідувача. Цей метод повинен
    мати лише один параметр, оголошений з типом загального інтерфейсу
    відвідувачів.

4.  **Конкретні елементи** реалізують методи *приймання* відвідувача.
    Мета цього методу --- викликати той метод відвідування, який
    відповідає типу цього елемента. Так відвідувач дізнається, з яким
    типом елементу він працює.

5.  **Клієнтом** зазвичай виступає колекція або складний складовий
    об'єкт, наприклад, дерево [Компонувальника](composite.html).
    Здебільшого, клієнт не прив'язаний до конкретних класів елементів,
    працюючи з ними через загальний інтерфейс елементів.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Відвідувач** додає до існуючої ієрархії класів
геометричних фігур можливість експорту до XML.

<figure class="image">
<img
src="../../images/patterns/diagrams/visitor/example9405.png?id=d66acd1b9096c47db17ab3bb82b54a59"
style="aspect-ratio: 1.14;"
srcset="/images/patterns/diagrams/visitor/example.png?id=d66acd1b9096c47db17ab3bb82b54a59 560w,/images/patterns/diagrams/visitor/example-1.5x.png?id=534425a20f1128cb81f418cbee30cba8 840w,/images/patterns/diagrams/visitor/example-2x.png?id=f44438b98f13fcb50898baefad67ffff 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів прикладу патерна Відвідувач" />
<figcaption><p>Приклад організації експорту об’єктів XML через
окремий клас-відвідувач.</p></figcaption>
</figure>

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Складна ієрархія елементів.
interface Shape is
    method move(x, y)
    method draw()
    method accept(v: Visitor)

// Метод прийняття відвідувача повинен бути реалізований у
// кожному елементі, а не тільки у базовому класі. Це допоможе
// програмі визначити, який метод відвідувача потрібно викликати
// у випадку, якщо ви не знаєте тип елемента.
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


// Інтерфейс відвідувачів повинен містити методи відвідування
// кожного елемента. Важливо, щоб ієрархія елементів змінювалася
// рідко, оскільки при додаванні нового елемента доведеться
// змінювати всіх існуючих відвідувачів.
interface Visitor is
    method visitDot(d: Dot)
    method visitCircle(c: Circle)
    method visitRectangle(r: Rectangle)
    method visitCompoundShape(cs: CompoundShape)

// Конкретний відвідувач реалізує одну операцію для всієї
// ієрархії елементів. Нова операція = новий відвідувач.
// Відвідувача вигідно застосовувати, коли нові елементи
// додаються дуже зрідка, а нові операції — часто.
class XMLExportVisitor implements Visitor is
    method visitDot(d: Dot) is
        // Експорт id та координат центру точки.

    method visitCircle(c: Circle) is
        // Експорт id, координат центру та радіусу кола.

    method visitRectangle(r: Rectangle) is
        // Експорт id, координат лівого-верхнього кута, висоти
        // та ширини прямокутника.

    method visitCompoundShape(cs: CompoundShape) is
        // Експорт id складової фігури, а також списку id
        // підфігур, з яких вона складається.


// Програма може застосовувати відвідувача до будь-якого набору
// об&#39;єктів елементів, навіть не уточнюючи їхні типи. Потрібний
// метод відвідувача буде обрано завдяки проходу через метод
// accept.
class Application is
    field allShapes: array of Shapes

    method export() is
        exportVisitor = new XMLExportVisitor()

        foreach (shape in allShapes) do
            shape.accept(exportVisitor)</code></pre>
</figure>

Вам не здається, що виклик методу `accept` --- це зайва ланка? Якщо так,
тоді ще раз рекомендую вам ознайомитися з проблемою раннього та пізнього
зв'язування в статті [Відвідувач і Double
Dispatch](visitor-double-dispatch.html).
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо вам потрібно виконати якусь операцію над усіма елементами складної
структури об'єктів, наприклад, деревом.
:::

::: applicability-solution
Відвідувач дозволяє застосовувати одну і ту саму операцію до об'єктів
різних класів.
:::

::: applicability-problem
Якщо над об'єктами складної структури об'єктів потрібно виконувати деякі
не пов'язані між собою операції, але ви не хочете «засмічувати» класи
такими операціями.
:::

::: applicability-solution
Відвідувач дозволяє витягти споріднені операції з класів, що складають
структуру об'єктів, помістивши їх до одного класу-відвідувача. Якщо
структура об'єктів використовується в декількох програмах, то патерн
дозволить кожній програмі мати тільки потрібні в ній операції.
:::

::: applicability-problem
Якщо нова поведінка має сенс тільки для деяких класів з існуючої
ієрархії.
:::

::: applicability-solution
Відвідувач дозволяє визначити поведінку тільки для цих класів, залишивши
її порожньою для всіх інших.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть інтерфейс відвідувача й оголосіть у ньому методи
    «відвідування» для кожного класу елемента, який існує в програмі.

2.  Опишіть інтерфейс елементів. Якщо ви працюєте з уже існуючими
    класами, оголосіть абстрактний метод прийняття відвідувачів у
    базовому класі ієрархії елементів.

3.  Реалізуйте методи прийняття в усіх конкретних елементах. Вони
    повинні переадресовувати виклики тому методу відвідувача, в якому
    тип параметра збігається з поточним класом елемента.

4.  Ієрархія елементів повинна знати тільки про загальний інтерфейс
    відвідувачів. З іншого боку, відвідувачі знатимуть про всі класи
    елементів.

5.  Для кожної нової поведінки створіть свій власний конкретний клас.
    Пристосуйте цю поведінку для роботи з усіма наявними типами
    елементів, реалізувавши всі методи інтерфейсу відвідувачів.

    Ви можете зіткнутися з ситуацією, коли відвідувачу потрібен доступ
    до приватних полів елементів. У цьому випадку ви можете або розкрити
    доступ до цих полів, порушивши інкапсуляцію елементів, або зробити
    клас відвідувача вкладеним в клас елемента, якщо вам пощастило
    писати мовою, яка підтримує механізм вкладених класів.

6.  Клієнт створюватиме об'єкти відвідувачів, а потім передаватиме їх
    елементам через метод прийняття.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Спрощує додавання операцій, працюючих зі складними структурами
    об'єктів.
-    Об'єднує споріднені операції в одному класі.
-    Відвідувач може накопичувати стан при обході структури елементів.
:::

::: col-sm-6
-    Патерн невиправданий, якщо ієрархія елементів часто змінюється.
-    Може призвести до порушення інкапсуляції елементів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Відвідувач](visitor.html) можна розглядати як розширений аналог
    [Команди](command.html), що здатен працювати відразу з декількома
    видами одержувачів.

-   Ви можете виконати якусь дію над усім деревом
    [Компонувальника](composite.html) за допомогою
    [Відвідувача](visitor.html).

-   [Відвідувач](visitor.html) можна використовувати спільно з
    [Ітератором](iterator.html). *Ітератор* відповідатиме за обхід
    структури даних, а *Відвідувач* --- за виконання дій над кожним її
    компонентом.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Відвідувач на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](visitor/csharp/example.html "Відвідувач на C#"){.prog-lang-link}
[![Відвідувач на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](visitor/cpp/example.html "Відвідувач на C++"){.prog-lang-link}
[![Відвідувач на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](visitor/go/example.html "Відвідувач на Go"){.prog-lang-link}
[![Відвідувач на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](visitor/java/example.html "Відвідувач на Java"){.prog-lang-link}
[![Відвідувач на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](visitor/php/example.html "Відвідувач на PHP"){.prog-lang-link}
[![Відвідувач на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](visitor/python/example.html "Відвідувач на Python"){.prog-lang-link}
[![Відвідувач на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](visitor/ruby/example.html "Відвідувач на Ruby"){.prog-lang-link}
[![Відвідувач на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](visitor/rust/example.html "Відвідувач на Rust"){.prog-lang-link}
[![Відвідувач на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](visitor/swift/example.html "Відвідувач на Swift"){.prog-lang-link}
[![Відвідувач на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](visitor/typescript/example.html "Відвідувач на TypeScript"){.prog-lang-link}
:::

::: {.section .extras}
##  Додаткові матеріали {#extras}

-   Детальніше про те, чому Відвідувача не можна замінити простим
    перевантаженням методів, читайте у статті [Відвідувач і Double
    Dispatch](visitor-double-dispatch.html).
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-uk" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не нудьгуй в транспорті {#не-нудьгуй-в-транспорті .title}

Краще почитай нашу книжку про патерни проектування.

Тепер це зручно робити навіть під час поїздок в громадському транспорті.

[ Дізнатися більше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаємо далі

[Відвідувач і Double Dispatch []{.fa
.fa-arrow-right}](visitor-double-dispatch.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Шаблонний метод](template-method.html){.btn
.btn-default rel="prev"}
:::
:::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-uk" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ukc0ed.png?id=03e01a1a94fb4b34fed20e260af002ec){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-uk-2x.png?id=beb38c652838bad2e9e7205bd0d7a293 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Ця стаття є частиною нашої електронної книги **Занурення в Патерни
Проектування**.

[ Дізнатися більше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::
