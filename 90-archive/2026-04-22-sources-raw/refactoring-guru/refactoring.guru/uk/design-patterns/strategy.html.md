::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Стратегія {#стратегія .title}

::: aka
Також відомий як: [Strategy]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Стратегія** --- це поведінковий патерн проектування, який визначає
сімейство схожих алгоритмів і розміщує кожен з них у власному класі.
Після цього алгоритми можна заміняти один на інший прямо під час
виконання програми.

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy4ca6.png?id=379bfba335380500375881a3da6507e0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/strategy/strategy.png?id=379bfba335380500375881a3da6507e0 640w,/images/patterns/content/strategy/strategy-1.5x.png?id=33ecebc66a9761454f2786a9b3e9bbe5 960w,/images/patterns/content/strategy/strategy-2x.png?id=1cee47d05a76fddf07dce9c67b700748 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Стратегія" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Ви вирішили написати програму-навігатор для подорожуючих. Вона повинна
показувати гарну й зручну карту, яка дозволяла б з легкістю
орієнтуватися в незнайомому місті.

Однією з найбільш очікуваних функцій був пошук та прокладання маршрутів.
Перебуваючи в невідомому йому місті, користувач повинен мати можливість
вказати початкову точку та пункт призначення, а навігатор, в свою чергу,
прокладе оптимальний шлях.

Перша версія вашого навігатора могла прокладати маршрут лише
автомобільними шляхами, тому чудово підходила для подорожей автомобілем.
Але, вочевидь, не всі їздять у відпустку автомобілями. Тому наступним
кроком ви додали до навігатора можливість прокладання піших маршрутів.

Через деякий час з'ясувалося, що частина туристів під час пересування
містом віддають перевагу громадському транспорту. Тому ви додали ще й
таку опцію прокладання шляху.

Але й це ще не все. У найближчій перспективі ви хотіли б додати
прокладку маршрутів велодоріжками, а у віддаленому майбутньому ---
маршрути, пов'язані з відвідуванням цікавих та визначних місць.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/problem9b55.png?id=e5ca943e559a979bd31d20e232dc22b6"
style="aspect-ratio: 2.2;"
srcset="/images/patterns/diagrams/strategy/problem.png?id=e5ca943e559a979bd31d20e232dc22b6 330w,/images/patterns/diagrams/strategy/problem-1.5x.png?id=31d1042ffc28056845e45d5c616da2a9 495w,/images/patterns/diagrams/strategy/problem-2x.png?id=3974fb99c97aec525dd0ffcff2e48e78 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Код навігатора стає занадто роздутим" />
<figcaption><p>Код навігатора стає занадто роздутим.</p></figcaption>
</figure>

Якщо з популярністю навігатора не було жодних проблем, то технічна
частина викликала запитання й періодичний головний біль. З кожним новим
алгоритмом код основного класу навігатора збільшувався вдвічі. В такому
великому класі стало важкувато орієнтуватися.

Будь-яка зміна алгоритмів пошуку, чи то виправлення багів, чи додавання
нового алгоритму, зачіпала основний клас. Це підвищувало ризик створення
помилки шляхом випадкового внесення змін до робочого коду.

Крім того, ускладнювалася командна робота з іншими програмістами, яких
ви найняли після успішного релізу навігатора. Ваші зміни нерідко
торкалися одного і того самого коду, створюючи конфлікти, які вимагали
додаткового часу на їхнє вирішення.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Стратегія пропонує визначити сімейство схожих алгоритмів, які
часто змінюються або розширюються, й винести їх до власних класів, які
називають *стратегіями*.

Замість того, щоб початковий клас сам виконував той чи інший алгоритм,
він відіграватиме роль контексту, посилаючись на одну зі стратегій та
делегуючи їй виконання роботи. Щоб змінити алгоритм, вам буде достатньо
підставити в контекст інший об'єкт-стратегію.

Важливо, щоб всі стратегії мали єдиний інтерфейс. Використовуючи цей
інтерфейс, контекст буде незалежним від конкретних класів стратегій. З
іншого боку, ви зможете змінювати та додавати нові види алгоритмів, не
чіпаючи код контексту.

<figure class="image">
<img
src="../../images/patterns/diagrams/strategy/solution1435.png?id=0813a174b29a2ed5902d321aba28e5fc"
style="aspect-ratio: 2.04;"
srcset="/images/patterns/diagrams/strategy/solution.png?id=0813a174b29a2ed5902d321aba28e5fc 570w,/images/patterns/diagrams/strategy/solution-1.5x.png?id=ce3d4e57f4a2a06ebc96f2607b3d6691 855w,/images/patterns/diagrams/strategy/solution-2x.png?id=66b5ee048ea2ad25c4b20f180ebf94d7 1140w"
sizes="(max-width: 720px) 100vw, 570px" loading="lazy" width="570"
alt="Стратегії побудови шляху" />
<figcaption><p>Стратегії побудови шляху.</p></figcaption>
</figure>

У нашому прикладі кожен алгоритм пошуку шляху переїде до свого власного
класу. В цих класах буде визначено лише один метод, що приймає в
параметрах координати початку та кінця маршруту, а повертає масив всіх
точок маршруту.

Хоча кожен клас прокладатиме маршрут на свій розсуд, для навігатора це
не буде мати жодного значення, оскільки його робота полягає тільки у
зображенні маршруту. Навігатору достатньо подати до стратегії дані про
початок та кінець маршруту, щоб отримати масив точок маршруту в
обумовленому форматі.

Клас навігатора буде мати метод для встановлення стратегії, що дозволить
змінювати стратегію пошуку шляху «на льоту». Цей метод стане у нагоді
клієнтському коду навігатора, наприклад, кнопкам-перемикачам типів
маршрутів в інтерфейсі користувача.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/strategy/strategy-comic-1-en590c.png?id=f64d7d8e6f83a7792a769039a66007c1"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/strategy/strategy-comic-1-en.png?id=f64d7d8e6f83a7792a769039a66007c1 640w,/images/patterns/content/strategy/strategy-comic-1-en-1.5x.png?id=b952bc8d43f5e4e0db81b8e3c62dc205 960w,/images/patterns/content/strategy/strategy-comic-1-en-2x.png?id=7eb14bd7920ad630c1ecf448d40602df 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Способи пересування" />
<figcaption><p>Різні стратегії потрапляння
до аеропорту.</p></figcaption>
</figure>

Вам потрібно дістатися аеропорту. Можна доїхати автобусом, таксі або
велосипедом. Тут вид транспорту є стратегією. Ви вибираєте конкретну
стратегію в залежності від контексту --- наявності грошей або часу до
відльоту.
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
alt="Структура класів патерна Стратегія" /><img
src="../../images/patterns/diagrams/strategy/structure-indexed64c7.png?id=ff55c5a6273cf82a5667f3cab5b605c7"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.15;"
srcset="/images/patterns/diagrams/strategy/structure-indexed.png?id=ff55c5a6273cf82a5667f3cab5b605c7 450w,/images/patterns/diagrams/strategy/structure-indexed-1.5x.png?id=3d6ba05b8a74a9fb8e3f88eb9ca1e30f 675w,/images/patterns/diagrams/strategy/structure-indexed-2x.png?id=9f8e2f838f16705775411e2b4616820e 900w"
sizes="(max-width: 720px) 100vw, 450px" loading="lazy" width="450"
alt="Структура класів патерна Стратегія" />
</figure>
:::

1.  **Контекст** зберігає посилання на об'єкт конкретної стратегії,
    працюючи з ним через загальний інтерфейс стратегій.

2.  **Стратегія** визначає інтерфейс, спільний для всіх варіацій
    алгоритму. Контекст використовує цей інтерфейс для виклику
    алгоритму.

    Для контексту неважливо, яка саме варіація алгоритму буде обрана,
    оскільки всі вони мають однаковий інтерфейс.

3.  **Конкретні стратегії** реалізують різні варіації алгоритму.

4.  Під час виконання програми контекст отримує виклики від клієнта й
    делегує їх об'єкту конкретної стратегії.

5.  Клієнт повинен створити об'єкт конкретної стратегії та передати його
    до конструктора контексту. Крім того, клієнт повинен мати можливість
    замінити стратегію на льоту, використовуючи сетер поля стратегії.
    Завдяки цьому, контекст не знатиме про те, яку саме стратегію зараз
    обрано.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі контекст використовує **Стратегію** для виконання тієї
чи іншої арифметичної операції.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс стратегій.
interface Strategy is
    method execute(a, b)

// Кожна конкретна стратегія реалізує загальний інтерфейс у свій
// власний спосіб.
class ConcreteStrategyAdd implements Strategy is
    method execute(a, b) is
        return a + b

class ConcreteStrategySubtract implements Strategy is
    method execute(a, b) is
        return a - b

class ConcreteStrategyMultiply implements Strategy is
    method execute(a, b) is
        return a * b

// Контекст завжди працює зі стратегіями через загальний
// інтерфейс. Він не знає, яку саме стратегію йому подано.
class Context is
    private strategy: Strategy

    method setStrategy(Strategy strategy) is
        this.strategy = strategy

    method executeStrategy(int a, int b) is
        return strategy.execute(a, b)


// Конкретна стратегія вибирається на більш високому рівні,
// наприклад, конфігуратором всієї програми. Готовий об&#39;єкт-
// стратегія подається до клієнтського об&#39;єкта, а потім може
// бути замінений іншою стратегією, в будь-який момент, «на
// льоту».
class ExampleApplication is
    method main() is
        // 1. Створити об&#39;єкт контексту.
        // 2. Ввести перше число (n1).
        // 3. Ввести друге число (n2).
        // 4. Ввести бажану операцію.
        // 5. Потім, обрати стратегію:

        if (action == addition) then
            context.setStrategy(new ConcreteStrategyAdd())

        if (action == subtraction) then
            context.setStrategy(new ConcreteStrategySubtract())

        if (action == multiplication) then
            context.setStrategy(new ConcreteStrategyMultiply())

        // 6. Виконати операцію за допомогою стратегії:
        result = context.executeStrategy(n1, n2)

        // N. Вивести результат на екран.</code></pre>
</figure>
:::

:::::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::::: applicability
::: applicability-problem
Якщо вам потрібно використовувати різні варіації якого-небудь алгоритму
всередині одного об'єкта.
:::

::: applicability-solution
Стратегія дозволяє варіювати поведінку об'єкта під час виконання
програми, підставляючи до нього різні об'єкти-поведінки (наприклад, що
відрізняються балансом швидкості та споживання ресурсів).
:::

::: applicability-problem
Якщо у вас є безліч схожих класів, які відрізняються лише деякою
поведінкою.
:::

::: applicability-solution
Стратегія дозволяє відокремити поведінку, що відрізняється, у власну
ієрархію класів, а потім звести початкові класи до одного, налаштовуючи
його поведінку стратегіями.
:::

::: applicability-problem
Якщо ви не хочете оголювати деталі реалізації алгоритмів для інших
класів.
:::

::: applicability-solution
Стратегія дозволяє ізолювати код, дані й залежності алгоритмів від інших
об'єктів, приховавши ці деталі всередині класів-стратегій.
:::

::: applicability-problem
Якщо різні варіації алгоритмів реалізовано у вигляді розлогого умовного
оператора. Кожна гілка такого оператора є варіацією алгоритму.
:::

::: applicability-solution
Стратегія розміщує кожну лапу такого оператора до окремого
класу-стратегії. Потім контекст отримує певний об'єкт-стратегію від
клієнта й делегує йому роботу. Якщо раптом знадобиться змінити алгоритм,
до контексту можна подати іншу стратегію.
:::
:::::::::::
::::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначте алгоритм, що схильний до частих змін. Також підійде
    алгоритм, який має декілька варіацій, які обираються під час
    виконання програми.

2.  Створіть інтерфейс стратегій, що описує цей алгоритм. Він повинен
    бути спільним для всіх варіантів алгоритму.

3.  Помістіть варіації алгоритму до власних класів, які реалізують цей
    інтерфейс.

4.  У класі контексту створіть поле для зберігання посилання на поточний
    об'єкт-стратегію, а також метод для її зміни. Переконайтеся в тому,
    що контекст працює з цим об'єктом тільки через загальний інтерфейс
    стратегій.

5.  Клієнти контексту мають подавати до нього відповідний
    об'єкт-стратегію, коли хочуть, щоб контекст поводився певним чином.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Гаряча заміна алгоритмів на льоту.
-    Ізолює код і дані алгоритмів від інших класів.
-    Заміна спадкування делегуванням.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Ускладнює програму внаслідок додаткових класів.
-    Клієнт повинен знати, в чому полягає різниця між стратегіями, щоб
    вибрати потрібну.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Міст](bridge.html), [Стратегія](strategy.html) та
    [Стан](state.html) (а також трохи і [Адаптер](adapter.html)) мають
    схожі структури класів --- усі вони побудовані за принципом
    «композиції», тобто делегування роботи іншим об'єктам. Проте вони
    відрізняються тим, що вирішують різні проблеми. Пам'ятайте, що
    патерни --- це не тільки рецепт побудови коду певним чином, але й
    описування проблем, які призвели до такого рішення.

-   [Команда](command.html) та [Стратегія](strategy.html) схожі за
    принципом, але відрізняються масштабом та застосуванням:

    -   *Команду* використовують для перетворення будь-яких різнорідних
        дій на об'єкти. Параметри операції перетворюються на поля
        об'єкта. Цей об'єкт тепер можна логувати, зберігати в історії
        для скасування, передавати у зовнішні сервіси тощо.
    -   З іншого боку, *Стратегія* описує різні способи того, як зробити
        одну і ту саму дію, дозволяючи замінювати ці способи в якомусь
        об'єкті контексту прямо під час виконання програми.

-   [Стратегія](strategy.html) змінює поведінку об'єкта «зсередини», а
    [Декоратор](decorator.html) змінює його «ззовні».

-   [Шаблонний метод](template-method.html) використовує спадкування,
    щоб розширювати частини алгоритму. [Стратегія](strategy.html)
    використовує делегування, щоб змінювати «на льоту» алгоритми, що
    виконуються. *Шаблонний метод* працює на рівні класів. *Стратегія*
    дозволяє змінювати логіку окремих об'єктів.

-   [Стан](state.html) можна розглядати як надбудову над
    [Стратегією](strategy.html). Обидва патерни використовують
    композицію, щоб змінювати поведінку головного об'єкта, делегуючи
    роботу вкладеним об'єктам-помічникам. Проте в *Стратегії* ці об'єкти
    не знають один про одного і жодним чином не пов'язані. У *Стані*
    конкретні стани самостійно можуть перемикати контекст.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Стратегія на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](strategy/csharp/example.html "Стратегія на C#"){.prog-lang-link}
[![Стратегія на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](strategy/cpp/example.html "Стратегія на C++"){.prog-lang-link}
[![Стратегія на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](strategy/go/example.html "Стратегія на Go"){.prog-lang-link}
[![Стратегія на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](strategy/java/example.html "Стратегія на Java"){.prog-lang-link}
[![Стратегія на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](strategy/php/example.html "Стратегія на PHP"){.prog-lang-link}
[![Стратегія на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](strategy/python/example.html "Стратегія на Python"){.prog-lang-link}
[![Стратегія на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](strategy/ruby/example.html "Стратегія на Ruby"){.prog-lang-link}
[![Стратегія на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](strategy/rust/example.html "Стратегія на Rust"){.prog-lang-link}
[![Стратегія на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](strategy/swift/example.html "Стратегія на Swift"){.prog-lang-link}
[![Стратегія на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](strategy/typescript/example.html "Стратегія на TypeScript"){.prog-lang-link}
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

[Шаблонний метод []{.fa .fa-arrow-right}](template-method.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стан](state.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
