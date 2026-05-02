::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Посередник {#посередник .title}

::: aka
Також відомий як:
[Intermediary, ]{style="display:inline-block"}[Controller, ]{style="display:inline-block"}[Mediator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Посередник** --- це поведінковий патерн проектування, що дає змогу
зменшити зв'язаність великої кількості класів між собою, завдяки
переміщенню цих зв'язків до одного класу-посередника.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Посередник" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Припустімо, що у вас є діалог створення профілю користувача. Він
складається з різноманітних елементів керування: текстових полів,
чекбоксів, кнопок.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-en03c1.png?id=86f99055b3e60bb8834dcc7222922bdf"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-en.png?id=86f99055b3e60bb8834dcc7222922bdf 600w,/images/patterns/diagrams/mediator/problem1-en-1.5x.png?id=947efe07eb2f81f770af58673f621623 900w,/images/patterns/diagrams/mediator/problem1-en-2x.png?id=e868165bd04a9438857d6ad41528024c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Безладні зв&#39;язки між елементами інтерфейсу користувача" />
<figcaption><p>Безладні зв’язки між елементами
інтерфейсу користувача.</p></figcaption>
</figure>

Окремі елементи діалогу повинні взаємодіяти одне з одним. Так,
наприклад, чекбокс «у мене є собака» відкриває приховане поле для
введення імені домашнього улюбленця, а клік по кнопці збереження
запускає перевірку значень усіх полів форми.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Код елементів роздутий умовами, які часто змінюються" />
<figcaption><p>Код елементів потрібно правити під час зміни
кожного діалогу.</p></figcaption>
</figure>

Прописавши цю логіку безпосередньо в коді елементів керування, ви
поставите хрест на їхньому повторному використанні в інших місцях
програми. Вони стануть занадто тісно пов'язаними з елементами діалогу
редагування профілю, які не потрібні в інших контекстах. Отже ви зможете
або використовувати всі елементи відразу, або не використовувати жоден.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Посередник змушує об'єкти спілкуватися через окремий
об'єкт-посередник, який знає, кому потрібно перенаправити той або інший
запит. Завдяки цьому компоненти системи залежатимуть тільки від
посередника, а не від десятків інших компонентів.

У нашому прикладі посередником міг би стати діалог. Імовірно, клас
діалогу вже знає, з яких елементів він складається. Тому жодних нових
зв'язків додавати до нього не доведеться.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-en2ee9.png?id=dd991a5b7830de8d43f82b084e021713"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-en.png?id=dd991a5b7830de8d43f82b084e021713 600w,/images/patterns/diagrams/mediator/solution1-en-1.5x.png?id=07dec16a26fdec8fb83dd4eee914cd70 900w,/images/patterns/diagrams/mediator/solution1-en-2x.png?id=0a1280789a5d7b1567c9d98e5fcd1125 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Елементи спілкуються через посередника" />
<figcaption><p>Елементи інтерфейсу спілкуються
через посередника.</p></figcaption>
</figure>

Основні зміни відбудуться всередині окремих елементів діалогу. Якщо
раніше при отриманні кліка від користувача об'єкт кнопки самостійно
перевіряв значення полів діалогу, то тепер його єдиний обов'язок ---
повідомити діалогу про те, що відбувся клік. Отримавши повідомлення,
діалог виконає всі необхідні перевірки полів. Таким чином, замість
кількох залежностей від інших елементів кнопка отримає лише одну --- від
самого діалогу.

Щоб зробити код ще гнучкішим, можна виділити єдиний інтерфейс для всіх
посередників, тобто діалогів програми. Наша кнопка стане залежною не від
конкретного діалогу створення користувача, а від абстрактного, що
дозволить використовувати її і в інших діалогах.

Таким чином, посередник приховує у собі всі складні зв'язки й залежності
між класами окремих компонентів програми. А чим менше зв'язків мають
класи, тим простіше їх змінювати, розширювати й повторно
використовувати.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Приклад з диспетчерською вежею." />
<figcaption><p>Пілоти літаків спілкуються не безпосередньо, а
через диспетчера.</p></figcaption>
</figure>

Пілоти літаків, що сідають або злітають, не спілкуються з іншими
пілотами безпосередньо. Замість цього вони зв'язуються з диспетчером,
який координує політ кількох літаків одночасно. Без диспетчера пілотам
доводилося б увесь час бути напоготові і стежити самостійно за всіма
літаками навколо. Це часто призводило б до катастроф у небі.

Важливо розуміти, що диспетчер не потрібен під час всього польоту. Він
задіяний тільки в зоні аеропорту, коли потрібно координувати взаємодію
багатьох літаків.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/structure71f0.png?id=1f2accc7820ecfe9665b6d30cbc0bc61"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure.png?id=1f2accc7820ecfe9665b6d30cbc0bc61 520w,/images/patterns/diagrams/mediator/structure-1.5x.png?id=958b373815bf6a56cd9b38763ed01dce 780w,/images/patterns/diagrams/mediator/structure-2x.png?id=5191daa1c9d4caa36e38af3c5b7d1522 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Посередник" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Посередник" />
</figure>
:::

1.  **Компоненти** --- це різнорідні об'єкти, що містять бізнес-логіку
    програми. Кожен компонент має посилання на об'єкт посередника, але
    працює з ним тільки через абстрактний інтерфейс посередників.
    Завдяки цьому компоненти можна повторно використовувати в інших
    програмах, зв'язавши їх з посередником іншого типу.

2.  **Посередник** визначає інтерфейс для обміну інформацією з
    компонентами. Зазвичай достатньо одного методу, щоби повідомляти
    посередника про події, що відбулися в компонентах. У параметрах
    цього методу можна передавати деталі події: посилання на компонент,
    в якому вона відбулася, та будь-які інші дані.

3.  **Конкретний посередник** містить код взаємодії кількох компонентів
    між собою. Найчастіше цей об'єкт не тільки зберігає посилання на всі
    свої компоненти, але й сам їх створює, керуючи подальшим життєвим
    циклом.

4.  Компоненти не повинні спілкуватися один з одним безпосередньо. Якщо
    в компоненті відбувається важлива подія, він повинен повідомити
    свого посередника, а той сам вирішить, чи стосується подія інших
    компонентів, і чи треба їх сповістити. При цьому
    компонент-відправник не знає, хто обробить його запит, а
    компонент-одержувач не знає, хто його надіслав.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Посередник** допомагає позбутися залежностей між
класами різних елементів користувацього інтерфейсу: кнопками, чекбоксами
й написами.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів прикладу патерна Посередник" />
<figcaption><p>Приклад структурування класів
UI діалогів.</p></figcaption>
</figure>

Реагуючи на дії користувачів, елементи не взаємодіють безпосередньо, а
лише повідомляють посередника про те, що вони змінилися.

Посередник у вигляді діалогу авторизації знає, як конкретні елементи
повинні взаємодіяти. Тому при отриманні повідомлень він може
перенаправити виклик тому чи іншому елементу.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс посередників.
interface Mediator is
    method notify(sender: Component, event: string)


// Конкретний посередник. Усі зв&#39;язки між конкретними
// компонентами переїхали до коду посередника. Він отримує
// повідомлення від своїх компонентів та знає, як на них
// реагувати.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Тут потрібно буде створити об&#39;єкти усіх компонентів,
        // подавши поточний об&#39;єкт-посередник до їхніх
        // конструкторів.

    // Коли щось трапляється з компонентом, він надсилає
    // посереднику повідомлення. Після отримання повідомлення
    // посередник може або зробити щось самостійно, або
    // перенаправити запит іншому компонентові.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Показати компоненти форми входу.
                // 2. Приховати компоненти форми реєстрації.
            else
                title = &quot;Register&quot;
                // 1. Показати компоненти форми реєстрації.
                // 2. Приховати компоненти форми входу.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Намагатись знайти користувача з даними із
                // форми логіна.
                if (!found)
                    // Показати помилку над формою логіна.
            else
                // 1. Створити аккаунт користувача з даними
                // форми реєстрації.
                // 2. Авторизувати цього користувача.
                // ...


// Класи компонентів спілкуються з посередниками через їх
// загальний інтерфейс. Завдяки цьому, одні й ті ж компоненти
// можна використовувати в різних посередниках.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Конкретні компоненти жодним чином не пов&#39;язані між собою. У
// них є тільки один канал спілкування — через надсилання
// повідомлень посереднику.
class Button extends Component is
    // ...

class Textbox extends Component is
    // ...

class Checkbox extends Component is
    method check() is
        dialog.notify(this, &quot;check&quot;)
    // ...</code></pre>
</figure>
:::

## Придатність

-   Коли вам складно змінювати деякі класи через те, що вони мають
    величезну кількість хаотичних зв'язків з іншими класами.

-   Посередник дозволяє розмістити усі ці зв'язки в одному класі. Після
    цього вам буде легше їх відрефакторити, зробити більш зрозумілими й
    гнучкими.

-   Коли ви не можете повторно використовувати клас, оскільки він
    залежить від безлічі інших класів.

-   Після застосування патерна компоненти втрачають колишні зв'язки з
    іншими компонентами, а все їхнє спілкування відбувається
    опосередковано, через об'єкт посередника.

-   Коли вам доводиться створювати багато підкласів компонентів, щоб
    використовувати одні й ті самі компоненти в різних контекстах.

-   Якщо раніше зміна відносин в одному компоненті могла призвести до
    лавини змін в усіх інших компонентах, то тепер вам достатньо
    створити підклас посередника та змінити в ньому зв'язки між
    компонентами.

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Знайдіть групу тісно сплетених класів, де можна отримати деяку
    користь, відв'язавши деякі один від одного. Наприклад, щоб повторно
    використовувати їхній код в іншій програмі.

2.  Створіть загальний інтерфейс посередників та опишіть в ньому методи
    для взаємодії з компонентами. У найпростішому випадку достатньо
    одного методу для отримання повідомлень від компонентів.

    Цей інтерфейс необхідний, якщо ви хочете повторно використовувати
    класи компонентів для інших завдань. У цьому випадку все, що
    потрібно зробити, --- це створити новий клас конкретного
    посередника.

3.  Реалізуйте цей інтерфейс у класі конкретного посередника. Помістіть
    до нього поля, які міститимуть посилання на всі об'єкти компонентів.

4.  Ви можете піти далі і перемістити код створення компонентів до класу
    конкретного посередника, перетворивши його на фабрику.

5.  Компоненти теж повинні мати посилання на об'єкт посередника. Зв'язок
    між ними зручніше всього встановити шляхом подання посередника до
    параметрів конструктора компонентів.

6.  Змініть код компонентів так, щоб вони викликали метод повідомлення
    посередника, замість методів інших компонентів. З протилежного боку,
    посередник має викликати методи потрібного компонента, коли отримує
    повідомлення від компонента.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Усуває залежності між компонентами, дозволяючи використовувати їх
    повторно.
-    Спрощує взаємодію між компонентами.
-    Централізує керування в одному місці.
:::

::: col-sm-6
-    Посередник може [сильно «роздутися»](../smells/large-class.html).
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

-   [Посередник](mediator.html) та [Фасад](facade.html) схожі тим, що
    намагаються організувати роботу багатьох існуючих класів.

    -   *Фасад* створює спрощений інтерфейс підсистеми, не вносячи в неї
        жодної додаткової функціональності. Сама підсистема не знає про
        існування *Фасаду*. Класи підсистеми спілкуються один з одним
        безпосередньо.
    -   *Посередник* централізує спілкування між компонентами системи.
        Компоненти системи знають тільки про існування *Посередника*, у
        них немає прямого доступу до інших компонентів.

-   Різниця між [Посередником](mediator.html) та
    [Спостерігачем](observer.html) не завжди очевидна. Найчастіше вони
    виступають як конкуренти, але іноді можуть працювати разом.

    Мета *Посередника* --- прибрати взаємні залежності між компонентами
    системи. Замість цього вони стають залежними від самого посередника.
    З іншого боку, мета *Спостерігача* --- забезпечити динамічний
    односторонній зв'язок, в якому одні об'єкти опосередковано залежать
    від інших.

    Досить популярною є реалізація *Посередника* за допомогою
    *Спостерігача*. При цьому об'єкт посередника буде виступати
    видавцем, а всі інші компоненти стануть передплатниками та зможуть
    динамічно стежити за подіями, що відбуваються у посереднику. У цьому
    випадку важко зрозуміти, чим саме відрізняються обидва патерни.

    Але *Посередник* має й інші реалізації, коли окремі компоненти
    жорстко прив'язані до об'єкта посередника. Такий код навряд чи буде
    нагадувати *Спостерігача*, але залишиться *Посередником*.

    Навпаки, у разі реалізації посередника з допомогою *Спостерігача*,
    представимо чи уявімо таку програму, в якій кожен компонент системи
    стає видавцем. Компоненти можуть підписуватися один на одного, не
    прив'язуючись до конкретних класів. Програма складатиметься з цілої
    мережі *Спостерігачів*, не маючи центрального об'єкта *Посередника*.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Посередник на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Посередник на C#"){.prog-lang-link}
[![Посередник на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Посередник на C++"){.prog-lang-link}
[![Посередник на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Посередник на Go"){.prog-lang-link}
[![Посередник на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Посередник на Java"){.prog-lang-link}
[![Посередник на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Посередник на PHP"){.prog-lang-link}
[![Посередник на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Посередник на Python"){.prog-lang-link}
[![Посередник на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Посередник на Ruby"){.prog-lang-link}
[![Посередник на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Посередник на Rust"){.prog-lang-link}
[![Посередник на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Посередник на Swift"){.prog-lang-link}
[![Посередник на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Посередник на TypeScript"){.prog-lang-link}
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

[Знімок []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Ітератор](iterator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::
