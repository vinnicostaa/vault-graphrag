::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Ланцюжок обов\'язків {#ланцюжок-обовязків .title}

::: aka
Також відомий як: [Ланцюг
відповідальностей, ]{style="display:inline-block"}[CoR, ]{style="display:inline-block"}[Chain
of Command, ]{style="display:inline-block"}[Chain of
Responsibility]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Ланцюжок обов'язків** --- це поведінковий патерн проектування, що дає
змогу передавати запити послідовно ланцюжком обробників. Кожен наступний
обробник вирішує, чи може він обробити запит сам і чи варто передавати
запит далі ланцюжком.

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibilityf2d1.png?id=56c10d0dc712546cc283cfb3fb463458"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility.png?id=56c10d0dc712546cc283cfb3fb463458 640w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-1.5x.png?id=770c03ad168fa797ec8ed4556ddf0b5c 960w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-2x.png?id=cc104b0a00a410f37fb39da80f392b88 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Ланцюжок обов&#39;язків" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви робите систему прийому онлайн-замовлень. Ви хочете
обмежити до неї доступ так, щоб тільки авторизовані користувачі могли
створювати замовлення. Крім того, певні користувачі, які володіють
правами адміністратора, повинні мати повний доступ до замовлень.

Ви швидко збагнули, що ці перевірки потрібно виконувати послідовно. Адже
користувача можна спробувати «залогувати» у систему, якщо його запит
містить логін і пароль. Але, якщо така спроба не вдалась, то перевіряти
розширені права доступу просто немає сенсу.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem1-uk479e.png?id=8a759c06592ef21784d867d5feb1ccde"
style="aspect-ratio: 2.5;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem1-uk.png?id=8a759c06592ef21784d867d5feb1ccde 600w,/images/patterns/diagrams/chain-of-responsibility/problem1-uk-1.5x.png?id=889c70834dad8db764c68490dadad416 900w,/images/patterns/diagrams/chain-of-responsibility/problem1-uk-2x.png?id=11cff8409191019fa94d701ae8bbb5b7 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Проблема, яку вирішує Ланцюжок обов&#39;язків" />
<figcaption><p>Запит проходить ряд перевірок перед доступом до
системи замовлень.</p></figcaption>
</figure>

Протягом наступних кількох місяців вам довелося додати ще декілька таких
послідовних перевірок.

-   Хтось слушно зауважив, що непогано було б перевіряти дані, що
    передаються в запиті, перед тим, як вносити їх до системи --- раптом
    запит містить дані про покупку неіснуючих продуктів.

-   Хтось запропонував блокувати масові надсилання форми з одним і тим
    самим логіном, щоб запобігти підбору паролів ботами.

-   Хтось зауважив, що непогано було б діставати форму замовлення з
    кешу, якщо вона вже була одного разу показана.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/problem2-uk7e0c.png?id=3b52d47c62fdb10b19b0373e26fccd28"
style="aspect-ratio: 1.65;"
srcset="/images/patterns/diagrams/chain-of-responsibility/problem2-uk.png?id=3b52d47c62fdb10b19b0373e26fccd28 610w,/images/patterns/diagrams/chain-of-responsibility/problem2-uk-1.5x.png?id=af9b82ab2f976d3cd7ae3c18880a23e8 915w,/images/patterns/diagrams/chain-of-responsibility/problem2-uk-2x.png?id=3d74d14af1517e4571b6dbfe5eefccbb 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="З часом код перевірок стає все більш заплутаним" />
<figcaption><p>З часом код перевірок стає все
більш заплутаним.</p></figcaption>
</figure>

З кожною новою «фічою» код перевірок, що виглядав як величезний клубок
умовних операторів, все більше і більше «розбухав». При зміні одного
правила доводилося змінювати код усіх інших перевірок. А щоб застосувати
перевірки до інших ресурсів, довелося також продублювати їхній код в
інших класах.

Підтримувати такий код стало не тільки вкрай незручно, але й витратно.
Аж ось одного прекрасного дня ви отримуєте завдання рефакторингу\...
:::

::: {.section .solution}
##  Рішення {#solution}

Як і багато інших поведінкових патернів, ланцюжок обов'язків базується
на тому, щоб перетворити окремі поведінки на об'єкти. У нашому випадку
кожна перевірка переїде до окремого класу з одним методом виконання.
Дані запиту, що перевіряється, передаватимуться до методу як аргументи.

А тепер справді важливий етап. Патерн пропонує зв'язати всі об'єкти
обробників в один ланцюжок. Кожен обробник міститиме посилання на
наступного обробника в ланцюзі. Таким чином, після отримання запиту
обробник зможе не тільки опрацювати його самостійно, але й передати
обробку наступному об'єкту в ланцюжку.

Передаючи запити до першого обробника ланцюжка, ви можете бути впевнені,
що всі об'єкти в ланцюзі зможуть його обробити. При цьому довжина
ланцюжка не має жодного значення.

І останній штрих. Обробник не обов'язково повинен передавати запит далі.
Причому ця особливість може бути використана різними шляхами.

У прикладі з фільтрацією доступу обробники переривають подальші
перевірки, якщо поточну перевірку не пройдено. Адже немає сенсу
витрачати даремно ресурси, якщо і так зрозуміло, що із запитом щось не
так.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution1-ukeecf.png?id=e4cd5065ac32aaa0405d4f1a0aa5463d"
style="aspect-ratio: 4;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution1-uk.png?id=e4cd5065ac32aaa0405d4f1a0aa5463d 640w,/images/patterns/diagrams/chain-of-responsibility/solution1-uk-1.5x.png?id=fc813e4037817ac16f8ca1e0d26e5671 960w,/images/patterns/diagrams/chain-of-responsibility/solution1-uk-2x.png?id=db75363d58dcdbb8317ff59bc1dc9598 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Обробники слідують в ланцюжку один за іншим" />
<figcaption><p>Обробники слідують в ланцюжку один
за іншим.</p></figcaption>
</figure>

Але є й інший підхід, коли обробники переривають ланцюг, тільки якщо
вони *можуть* обробити запит. У цьому випадку запит рухається ланцюгом,
поки не знайдеться обробник, який зможе його обробити. Дуже часто такий
підхід використовується для передачі подій, що генеруються у класах
графічного інтерфейсу внаслідок взаємодії з користувачем.

Наприклад, коли користувач клікає по кнопці, програма будує ланцюжок з
об'єкта цієї кнопки, всіх її батьківських елементів і загального вікна
програми на кінці. Подія кліку передається цим ланцюжком до тих пір,
поки не знайдеться об'єкт, здатний її обробити. Цей приклад примітний ще
й тим, що ланцюжок завжди можна виділити з деревоподібної структури
об'єктів, в яку зазвичай і згорнуті елементи користувацького інтерфейсу.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/solution2-ukd59a.png?id=40dbccf1aba5e82e862845247784032a"
style="aspect-ratio: 1.73;"
srcset="/images/patterns/diagrams/chain-of-responsibility/solution2-uk.png?id=40dbccf1aba5e82e862845247784032a 520w,/images/patterns/diagrams/chain-of-responsibility/solution2-uk-1.5x.png?id=aa75781516ae1a5cb39b4f8aa819ce1f 780w,/images/patterns/diagrams/chain-of-responsibility/solution2-uk-2x.png?id=da8f2e95f1a04046831a8a2c41865054 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Ланцюжок можна виділити навіть із дерева об&#39;єктів" />
<figcaption><p>Ланцюжок можна виділити навіть із
дерева об’єктів.</p></figcaption>
</figure>

Дуже важливо, щоб усі об'єкти ланцюжка мали спільний інтерфейс. Зазвичай
кожному конкретному обробникові достатньо знати тільки те, що наступний
об'єкт ланцюжка має метод `виконати`. Завдяки цьому зв'язки між
об'єктами ланцюжка будуть більш гнучкими. Крім того, ви зможете
формувати ланцюжки на льоту з різноманітних об'єктів, не прив'язуючись
до конкретних класів.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-ukcfb3.png?id=4e4d5b2a3baf08d7b92ae215049f7e88"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-uk.png?id=4e4d5b2a3baf08d7b92ae215049f7e88 600w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-uk-1.5x.png?id=7d28863ca184377268ff835390c46997 900w,/images/patterns/content/chain-of-responsibility/chain-of-responsibility-comic-1-uk-2x.png?id=169882480e75a86077d0e615196a6280 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Приклад спілкування з підтримкою" />
<figcaption><p>Приклад спілкування з підтримкою.</p></figcaption>
</figure>

Ви купили нову відеокарту. Вона автоматично визначилася й почала
працювати під Windows, але у вашій улюбленій Ubuntu «завести» її не
вдалося. Ви телефонуєте до служби підтримки виробника, але без особливих
сподівань на вирішення проблеми.

Спочатку ви чуєте голос автовідповідача, який пропонує вибір з десяти
стандартних рішень. Жоден з варіантів не підходить, і робот з'єднує вас
з живим оператором.

На жаль, звичайний оператор підтримки вміє спілкуватися тільки завченими
фразами і давати тільки шаблонні відповіді. Після чергової пропозиції
«вимкнути і ввімкнути комп'ютер» ви просите зв'язати вас зі справжніми
інженерами.

Оператор перекидає дзвінок черговому інженерові, який знемагає від
нудьги у своїй комірчині. От він вже точно знає, як вам допомогти!
Інженер розповідає вам, де завантажити драйвери та як налаштувати їх під
Ubuntu. Запит вирішено. Ви кладете слухавку.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/structure03c9.png?id=848f0fc8dca57a44974d63f8181f5406"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.93;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure.png?id=848f0fc8dca57a44974d63f8181f5406 380w,/images/patterns/diagrams/chain-of-responsibility/structure-1.5x.png?id=49dfbae38f146af7319f80dce9ea7e2a 570w,/images/patterns/diagrams/chain-of-responsibility/structure-2x.png?id=bb837faaac88e7f2a16f751d0beaa201 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Структура класів патерна Ланцюжок обов’язків" /><img
src="../../images/patterns/diagrams/chain-of-responsibility/structure-indexedfcb6.png?id=e13a5bf44f9ca47299223116af77cbef"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.88;"
srcset="/images/patterns/diagrams/chain-of-responsibility/structure-indexed.png?id=e13a5bf44f9ca47299223116af77cbef 380w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-1.5x.png?id=ca660e2cb697b512aadc07fdd3e109cd 570w,/images/patterns/diagrams/chain-of-responsibility/structure-indexed-2x.png?id=4f27e2c48e635f45a78472d707a8df3c 760w"
sizes="(max-width: 720px) 100vw, 380px" loading="lazy" width="380"
alt="Структура класів патерна Ланцюжок обов’язків" />
</figure>
:::

1.  **Обробник** визначає спільний для всіх конкретних обробників
    інтерфейс. Зазвичай достатньо описати один метод обробки запитів,
    але іноді тут може бути оголошений і метод встановлення наступного
    обробника.

2.  **Базовий обробник** --- опціональний клас, який дає змогу позбутися
    дублювання одного і того самого коду в усіх конкретних обробниках.

    Зазвичай цей клас має поле для зберігання посилання на наступного
    обробника у ланцюжку. Клієнт зв'язує обробників у ланцюг, подаючи
    посилання на наступного обробника через конструктор або сетер поля.
    Також в цьому класі можна реалізувати базовий метод обробки, який би
    просто перенаправляв запити наступному обробнику, перевіривши його
    наявність.

3.  **Конкретні обробники** містять код обробки запитів. При отриманні
    запиту кожен обробник вирішує, чи може він обробити запит, а також
    чи варто передати його наступному об'єкту.

    У більшості випадків обробники можуть працювати самостійно і бути
    незмінними, отримавши всі необхідні деталі через параметри
    конструктора.

4.  **Клієнт** може сформувати ланцюжок лише один раз і використовувати
    його протягом всього часу роботи програми, так і перебудовувати його
    динамічно, залежно від логіки програми. Клієнт може відправляти
    запити будь-якому об'єкту ланцюжка, не обов'язково першому з них.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Ланцюжок обов'язків** відповідає за показ контекстної
допомоги для активних елементів інтерфейсу користувача.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example-uk2f16.png?id=1cdeb29296d35318427d6a3f728b67f8"
style="aspect-ratio: 1.09;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example-uk.png?id=1cdeb29296d35318427d6a3f728b67f8 610w,/images/patterns/diagrams/chain-of-responsibility/example-uk-1.5x.png?id=c360ff8b732477d8c0910b0337b4ca4b 915w,/images/patterns/diagrams/chain-of-responsibility/example-uk-2x.png?id=dae2b542ea6dc5a060ddbbff141d2628 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Структура класів прикладу патерна Ланцюжок обов&#39;язків" />
<figcaption><p>Графічний інтерфейс побудований за допомогою
компонувальника, де кожен елемент має посилання на свій
елемент-контейнер. Ланцюжок можна вибудувати, пройшовши по всіх
контейнерах, у які вкладено елемент.</p></figcaption>
</figure>

Графічний інтерфейс програми зазвичай структурований у вигляді дерева.
Клас `Діалог`, який відображає все вікно програми, --- це корінь дерева.
Діалог містить `Панелі`, які, в свою чергу, можуть містити або інші
вкладені панелі, або прості елементи на зразок `Кнопок`.

Прості елементи можуть показувати невеликі підказки, якщо для них
вказано допоміжний текст. Але є й більш складні компоненти, для яких цей
спосіб демонстрації допомоги занадто простий. Вони визначають власний
спосіб відображення контекстної допомоги.

<figure class="image">
<img
src="../../images/patterns/diagrams/chain-of-responsibility/example2-uk3db6.png?id=3e9a54705aa4aba434c6a8f97306ca5f"
style="aspect-ratio: 0.81;"
srcset="/images/patterns/diagrams/chain-of-responsibility/example2-uk.png?id=3e9a54705aa4aba434c6a8f97306ca5f 250w,/images/patterns/diagrams/chain-of-responsibility/example2-uk-1.5x.png?id=4576f81c1c0f3fabc237a6da35881742 375w,/images/patterns/diagrams/chain-of-responsibility/example2-uk-2x.png?id=22643430190723d6e2cb116383a48b9f 500w"
sizes="(max-width: 720px) 100vw, 250px" loading="lazy" width="250"
alt="Структура класів прикладу патерна Ланцюжок обов&#39;язків" />
<figcaption><p>Приклад виклику контекстної допомоги у ланцюжку
об’єктів UI.</p></figcaption>
</figure>

Коли користувач наводить вказівник миші на елемент і тисне клавішу `F1`,
програма надсилає цьому елементу запит щодо показу допомоги. Якщо він не
містить жодної довідкової інформації, запит подорожує списком
контейнерів элемента, доки не знаходиться той, що може відобразити
допомогу.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Інтерфейс обробників.
interface ComponentWithContextualHelp is
    method showHelp()


// Базовий клас простих компонентів.
abstract class Component implements ComponentWithContextualHelp is
    field tooltipText: string

    // Контейнер, що містить компонент, служить в якості
    // наступної ланки ланцюга.
    protected field container: Container

    // Базова поведінка компонента заключається в тому, щоб
    // показати вспливаючу підказку, якщо для неї задано текст.
    // А якщо ні — перенаправити запит своєму контейнеру, якщо
    // той існує.
    method showHelp() is
        if (tooltipText != null)
            // Показати підказку.
        else
            container.showHelp()


// Контейнери можуть містити як прості компоненти, так й інші
// контейнери. Тут формуються зв&#39;язки ланцюжка. Клас успадкує
// метод showHelp від свого батька.
abstract class Container extends Component is
    protected field children: array of Component

    method add(child) is
        children.add(child)
        child.container = this


// Більшість конкретних компонентів влаштує базова поведінка
// допомоги із вспливаючою підказкою, що вони успадкують від
// класу Component.
class Button extends Component is
    // ...

// Але складні компоненти можуть перевизначати метод показу
// допомоги по-своєму. Але і в цьому випадку вони завжди можуть
// повернутися до базової реалізації, викликавши метод батька.
class Panel extends Container is
    field modalHelpText: string

    method showHelp() is
        if (modalHelpText != null)
            // Показати модальне вікно з допомогою.
        else
            super.showHelp()

// ...те саме, що й вище...
class Dialog extends Container is
    field wikiPageURL: string

    method showHelp() is
        if (wikiPageURL != null)
            // Відкрити сторінку Wiki в браузері.
        else
            super.showHelp()


// Клієнтський код.
class Application is
    // Кожна програма конфігурує ланцюжок по-своєму.
    method createUI() is
        dialog = new Dialog(&quot;Budget Reports&quot;)
        dialog.wikiPageURL = &quot;http://...&quot;
        panel = new Panel(0, 0, 400, 800)
        panel.modalHelpText = &quot;This panel does...&quot;
        ok = new Button(250, 760, 50, 20, &quot;OK&quot;)
        ok.tooltipText = &quot;This is an OK button that...&quot;
        cancel = new Button(320, 760, 50, 20, &quot;Cancel&quot;)
        // ...
        panel.add(ok)
        panel.add(cancel)
        dialog.add(panel)

    // Уявіть, що тут відбудеться.
    method onF1KeyPress() is
        component = this.getComponentAtMouseCoords()
        component.showHelp()</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо програма має обробляти різноманітні запити багатьма способами, але
заздалегідь невідомо, які конкретно запити надходитимуть і які обробники
для них знадобляться.
:::

::: applicability-solution
За допомогою Ланцюжка обов'язків ви можете зв'язати потенційних
обробників в один ланцюг і по отриманню запита по черзі питати кожного з
них, чи не хоче він обробити даний запит.
:::

::: applicability-problem
Якщо важливо, щоб обробники виконувалися один за іншим у суворому
порядку.
:::

::: applicability-solution
Ланцюжок обов'язків дозволяє запускати обробників один за одним у тій
послідовності, в якій вони стоять в ланцюзі.
:::

::: applicability-problem
Якщо набір об'єктів, здатних обробити запит, повинен задаватися
динамічно.
:::

::: applicability-solution
У будь-який момент ви можете втрутитися в існуючий ланцюжок і
перевизначити зв'язки так, щоби прибрати або додати нову ланку.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть інтерфейс обробника і опишіть в ньому основний метод
    обробки.

    Продумайте, в якому вигляді клієнт повинен передавати дані запиту до
    обробника. Найгнучкіший спосіб --- це перетворити дані запиту на
    об'єкт і повністю передавати його через параметри методу обробника.

2.  Є сенс у тому, щоб створити абстрактний базовий клас обробників, аби
    не дублювати реалізацію методу отримання наступного обробника в усіх
    конкретних обробниках.

    Додайте до базового обробника поле для збереження посилання на
    наступний елемент ланцюжка. Встановлюйте початкове значення цього
    поля через конструктор. Це зробить об'єкти обробників незмінюваними.
    Але якщо програма передбачає динамічну перебудову ланцюжків, можете
    додати ще й сетер для поля.

    Реалізуйте базовий метод обробки так, щоб він перенаправляв запит
    наступному об'єкту, перевіривши його наявність. Це дозволить
    повністю приховати поле-посилання від підкласів, давши їм можливість
    передавати запити далі ланцюгом, звертаючись до батьківської
    реалізації методу.

3.  Один за іншим створіть класи конкретних обробників та реалізуйте в
    них методи обробки запитів. При отриманні запиту кожен обробник
    повинен вирішити:

    -   Чи може він обробити запит, чи ні?
    -   Чи потрібно передавати запит наступному обробникові, чи ні?

4.  Клієнт може збирати ланцюжок обробників самостійно, спираючись на
    свою бізнес-логіку, або отримувати вже готові ланцюжки ззовні. В
    останньому випадку ланцюжки збираються фабричними об'єктами,
    спираючись на конфігурацію програми або параметри оточення.

5.  Клієнт може надсилати запити будь-якому обробникові ланцюга, а не
    лише першому. Запит передаватиметься ланцюжком, допоки який-небудь
    обробник не відмовиться передавати його далі або коли буде досягнуто
    кінець ланцюга.

6.  Клієнт повинен знати про динамічну природу ланцюжка і бути готовим
    до таких випадків:

    -   Ланцюжок може складатися з одного об'єкта.
    -   Запити можуть не досягати кінця ланцюга.
    -   Запити можуть досягати кінця, залишаючись необробленими.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Зменшує залежність між клієнтом та обробниками.
-    Реалізує *принцип єдиного обов'язку*.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Запит може залишитися ніким не опрацьованим.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Ланцюжок обов'язків](chain-of-responsibility.html),
    [Команда](command.html) [Посередник](mediator.html) та
    [Спостерігач](observer.html) показують різні способи роботи тих, хто
    надсилає запити, та тих, хто їх отримує:

    -   *Ланцюжок обов'язків* передає запит послідовно через ланцюжок
        потенційних отримувачів, очікуючи, що один з них обробить запит.
    -   *Команда* встановлює непрямий односторонній зв'язок від
        відправників до одержувачів.
    -   *Посередник* прибирає прямий зв'язок між відправниками та
        одержувачами, змушуючи їх спілкуватися опосередковано, через
        себе.
    -   *Спостерігач* передає запит одночасно всім зацікавленим
        одержувачам, але дозволяє їм динамічно підписуватися або
        відписуватися від таких повідомлень.

-   [Ланцюжок обов'язків](chain-of-responsibility.html) часто
    використовують разом з [Компонувальником](composite.html). У цьому
    випадку запит передається від дочірніх компонентів до їхніх батьків.

-   Обробники в [Ланцюжкові обов'язків](chain-of-responsibility.html)
    можуть бути виконані у вигляді [Команд](command.html). В цьому
    випадку роль запиту відіграє контекст команд, який послідовно
    подається до кожної команди у ланцюгу.

    Але є й інший підхід, в якому сам запит є *Командою*, надісланою
    ланцюжком об'єктів. У цьому випадку одна і та сама операція може
    бути застосована до багатьох різних контекстів, представлених у
    вигляді ланцюжка.

-   [Ланцюжок обов'язків](chain-of-responsibility.html) та
    [Декоратор](decorator.html) мають дуже схожі структури. Обидва
    патерни базуються на принципі рекурсивного виконання операції через
    серію пов'язаних об'єктів. Але є декілька важливих відмінностей.

    Обробники в *Ланцюжку обов'язків* можуть виконувати довільні дії,
    незалежні одна від одної, а також у будь-який момент переривати
    подальшу передачу ланцюжком. З іншого боку, *Декоратори* розширюють
    певну дію, не ламаючи інтерфейс базової операції і не перериваючи
    виконання інших декораторів.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Ланцюжок обов'язків на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/csharp/example.html "Ланцюжок обов’язків на C#"){.prog-lang-link}
[![Ланцюжок обов'язків на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/cpp/example.html "Ланцюжок обов’язків на C++"){.prog-lang-link}
[![Ланцюжок обов'язків на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/go/example.html "Ланцюжок обов’язків на Go"){.prog-lang-link}
[![Ланцюжок обов'язків на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/java/example.html "Ланцюжок обов’язків на Java"){.prog-lang-link}
[![Ланцюжок обов'язків на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/php/example.html "Ланцюжок обов’язків на PHP"){.prog-lang-link}
[![Ланцюжок обов'язків на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/python/example.html "Ланцюжок обов’язків на Python"){.prog-lang-link}
[![Ланцюжок обов'язків на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/ruby/example.html "Ланцюжок обов’язків на Ruby"){.prog-lang-link}
[![Ланцюжок обов'язків на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/rust/example.html "Ланцюжок обов’язків на Rust"){.prog-lang-link}
[![Ланцюжок обов'язків на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/swift/example.html "Ланцюжок обов’язків на Swift"){.prog-lang-link}
[![Ланцюжок обов'язків на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](chain-of-responsibility/typescript/example.html "Ланцюжок обов’язків на TypeScript"){.prog-lang-link}
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

[Команда []{.fa .fa-arrow-right}](command.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Поведінкові
патерни](behavioral-patterns.html){.btn .btn-default rel="prev"}
:::
::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
