::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Спостерігач {#спостерігач .title}

::: aka
Також відомий як:
[Видавець-Підписник, ]{style="display:inline-block"}[Слухач, ]{style="display:inline-block"}[Observer]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Спостерігач** --- це поведінковий патерн проектування, який створює
механізм підписки, що дає змогу одним об'єктам стежити й реагувати на
події, які відбуваються в інших об'єктах.

<figure class="image">
<img
src="../../images/patterns/content/observer/observerd508.png?id=6088e31e1b0d4a417506a66614dcf065"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/observer/observer.png?id=6088e31e1b0d4a417506a66614dcf065 640w,/images/patterns/content/observer/observer-1.5x.png?id=aa64f9f336354462b57bbff5c09d8269 960w,/images/patterns/content/observer/observer-2x.png?id=d5a83e115528e9fd633f04ad2650f1db 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Спостерігач" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Уявіть, що ви маєте два об'єкти: `Покупець` і `Магазин`. До магазину
мають ось-ось завезти новий товар, який цікавить покупця.

Покупець може щодня ходити до магазину, щоб перевіряти наявність товару.
Але через це він буде дратуватися, даремно витрачаючи свій дорогоцінний
час.

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-1-uk77ba.png?id=891b0246f4ee598b178850c78299696a"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-1-uk.png?id=891b0246f4ee598b178850c78299696a 600w,/images/patterns/content/observer/observer-comic-1-uk-1.5x.png?id=950cbca54d0d20abb2139615560c1cac 900w,/images/patterns/content/observer/observer-comic-1-uk-2x.png?id=7b1bee6c24de36005200740d4dc6c2d9 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Постійне відвідування магазину чи спам?" />
<figcaption><p>Постійне відвідування магазину чи спам?</p></figcaption>
</figure>

З іншого боку, магазин може розсилати спам кожному своєму покупцеві.
Багатьох покупців це засмутить, оскільки товар специфічний і потрібний
не всім.

Виходить конфлікт: або покупець гає час на періодичні перевірки, або
магазин розтрачує ресурси на непотрібні сповіщення.
:::

::: {.section .solution}
##  Рішення {#solution}

Давайте називати `Видавцями` ті об'єкти, які містять важливий або
цікавий для інших стан. Решту об'єктів, які хотіли б відстежувати зміни
цього стану, назвемо `Підписниками`.

Патерн Спостерігач пропонує зберігати всередині об'єкта видавця список
посилань на об'єкти підписників. Причому видавець не повинен вести
список підписки самостійно. Він повинен надати методи, за допомогою яких
підписники могли б додавати або прибирати себе зі списку.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution1-uked5b.png?id=1fa3c0feed5e4c0731f7c915146663cb"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/observer/solution1-uk.png?id=1fa3c0feed5e4c0731f7c915146663cb 470w,/images/patterns/diagrams/observer/solution1-uk-1.5x.png?id=9d93cc2ae2928855093150a1c388bcb4 705w,/images/patterns/diagrams/observer/solution1-uk-2x.png?id=b084d66747755abcf6ae24ad0cc2112f 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Підписка на події" />
<figcaption><p>Підписка на події.</p></figcaption>
</figure>

Тепер найцікавіше. Коли у видавця відбуватиметься важлива подія, він
буде проходитися за списком передплатників та сповіщувати їх про подію,
викликаючи певний метод об'єктів-передплатників.

Видавцю байдуже, якого класу буде той чи інший підписник, бо всі вони
повинні слідувати загальному інтерфейсу й мати єдиний метод оповіщення.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/solution2-uk3c6c.png?id=479d34a06c51a45969ee45136ac11692"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/observer/solution2-uk.png?id=479d34a06c51a45969ee45136ac11692 460w,/images/patterns/diagrams/observer/solution2-uk-1.5x.png?id=2a53cf5cfd3d073118e746c403f3c1f4 690w,/images/patterns/diagrams/observer/solution2-uk-2x.png?id=91a46b973d4bf03ff845d83318bb3911 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Сповіщення про події" />
<figcaption><p>Сповіщення про події.</p></figcaption>
</figure>

Побачивши, як добре все працює, ви можете виділити загальний інтерфейс і
для всіх видавців, який буде складатися з методів підписки та відписки.
Після цього підписники зможуть працювати з різними типами видавців, і
отримувати від них сповіщення через єдиний метод.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/observer/observer-comic-2-uk3efa.png?id=c51aa2e89536b3b83a5a9ddfe3359d55"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/observer/observer-comic-2-uk.png?id=c51aa2e89536b3b83a5a9ddfe3359d55 600w,/images/patterns/content/observer/observer-comic-2-uk-1.5x.png?id=132361556fc1d217cf9a2dd84a3ab83c 900w,/images/patterns/content/observer/observer-comic-2-uk-2x.png?id=579596dbb6990a3c5fa9248fad4974ed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Передплата та доставка газет." />
<figcaption><p>Передплата та доставка газет.</p></figcaption>
</figure>

Після того, як ви оформили підписку на журнал, вам більше не потрібно
їздити до супермаркета та дізнаватись, чи вже вийшов черговий номер.
Натомість видавництво надсилатиме нові номери поштою прямо до вас
додому, відразу після їхнього виходу.

Видавництво веде список підписників і знає, кому який журнал слати. Ви
можете в будь-який момент відмовитися від підписки, й журнал перестане
до вас надходити.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/observer/structurea121.png?id=365b7e2b8fbecc8948f34b9f8f16f33c"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.97;"
srcset="/images/patterns/diagrams/observer/structure.png?id=365b7e2b8fbecc8948f34b9f8f16f33c 610w,/images/patterns/diagrams/observer/structure-1.5x.png?id=7f23a48c9a2d789844f912d3a0f289e6 915w,/images/patterns/diagrams/observer/structure-2x.png?id=228af9bded4d6ee6daf43a0e23cca9ff 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Структура класів патерна Спостерігач" /><img
src="../../images/patterns/diagrams/observer/structure-indexed8afd.png?id=2ca2c123503ede860740af2a22bc4b4d"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.88;"
srcset="/images/patterns/diagrams/observer/structure-indexed.png?id=2ca2c123503ede860740af2a22bc4b4d 620w,/images/patterns/diagrams/observer/structure-indexed-1.5x.png?id=20b9e7765e83ca75514c202968d9e9fa 930w,/images/patterns/diagrams/observer/structure-indexed-2x.png?id=910eec855bc41f05199e510494078926 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Структура класів патерна Спостерігач" />
</figure>
:::

1.  **Видавець** володіє внутрішнім станом, зміни якого цікаво
    відслідковувати підписникам. Видавець містить механізм підписки:
    список підписників та методи підписки/відписки.

2.  Коли внутрішній стан видавця змінюється, він сповіщає своїх
    підписників. Для цього видавець проходиться за списком підписників і
    викликає їхній метод сповіщення, який описаний в загальному
    інтерфейсі підписників.

3.  **Підписник** визначає інтерфейс, яким користується видавець для
    надсилання сповіщень. Здебільшого для цього досить одного методу.

4.  **Конкретні підписники** виконують щось у відповідь на сповіщення,
    яке надійшло від видавця. Ці класи мають дотримуватися загального
    інтерфейсу, щоб видавець не залежав від конкретних класів
    підписників.

5.  Після отримання сповіщення підписнику необхідно отримати оновлений
    стан видавця. Видавець може передати цей стан через параметри методу
    сповіщення. Більш гнучкий варіант --- передавати через параметри
    весь об'єкт видавця, щоб підписник міг сам отримати необхідні дані.
    Як варіант, підписник може постійно зберігати посилання на об'єкт
    видавця, переданий йому через конструктор.

6.  **Клієнт** створює об'єкти видавців і підписників, а потім реєструє
    підписників на оновлення у видавцях.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Спостерігач** дає змогу об'єкту текстового редактора
сповіщати інші об'єкти про зміни свого стану.

<figure class="image">
<img
src="../../images/patterns/diagrams/observer/example04a2.png?id=6d0603ab5a00e4463b81d9639cd746a2"
style="aspect-ratio: 1.19;"
srcset="/images/patterns/diagrams/observer/example.png?id=6d0603ab5a00e4463b81d9639cd746a2 510w,/images/patterns/diagrams/observer/example-1.5x.png?id=67adccd1030a38dd263bfa09867f8dbe 765w,/images/patterns/diagrams/observer/example-2x.png?id=e2838e1562325e485fc7c2828a8ca445 1020w"
sizes="(max-width: 720px) 100vw, 510px" loading="lazy" width="510"
alt="Структура класів прикладу патерна Спостерігач" />
<figcaption><p>Приклад сповіщення об’єктів про події в
інших об’єктах.</p></figcaption>
</figure>

Список підписників складається динамічно, об'єкти можуть як
підписуватися на певні події, так і відписуватися від них прямо під час
виконання програми.

У цій реалізації редактор не веде список підписників самостійно, а
делегує це вкладеному об'єкту. Це дає змогу використовувати механізм
підписки не лише в класі редактора, а і в інших класах програми.

Для додавання до програми нових підписників не потрібно змінювати класи
видавців, допоки вони працюють із підписниками через загальний
інтерфейс.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Базовий клас-видавець. Містить код керування підписниками та
// надсилання їм сповіщень.
class EventManager is
    private field listeners: hash map of event types and listeners

    method subscribe(eventType, listener) is
        listeners.add(eventType, listener)

    method unsubscribe(eventType, listener) is
        listeners.remove(eventType, listener)

    method notify(eventType, data) is
        foreach (listener in listeners.of(eventType)) do
            listener.update(data)

// Конкретний клас-видавець, що містить цікаву для інших
// компонентів бізнес-логіку. Ми могли б зробити його прямим
// нащадком EventManager, але в реальному житті це не завжди є
// можливим (наприклад, якщо в класу вже є предок). Тому тут ми
// підключаємо механізм підписки за допомогою композиції.
class Editor is
    public field events: EventManager
    private field file: File

    constructor Editor() is
        events = new EventManager()

    // Методи бізнес-логіки, які сповіщають підписників про
    // зміни.
    method openFile(path) is
        this.file = new File(path)
        events.notify(&quot;open&quot;, file.name)

    method saveFile() is
        file.write()
        events.notify(&quot;save&quot;, file.name)
    // ...


// Загальний інтерфейс підписників. У багатьох мовах, що мають
// функціональні типи, можна обійтися без цього інтерфейсу та
// конкретних класів, замінивши об&#39;єкти підписників функціями.
interface EventListener is
    method update(filename)

// Набір конкретних підписників. Кожен з них виконує якусь
// поведінку, реагуючи на сповіщення від видавця.
class LoggingListener implements EventListener is
    private field log: File
    private field message: string

    constructor LoggingListener(log_filename, message) is
        this.log = new File(log_filename)
        this.message = message

    method update(filename) is
        log.write(replace(&#39;%s&#39;,filename,message))

class EmailAlertsListener implements EventListener is
    private field email: string
    private field message: string

    constructor EmailAlertsListener(email, message) is
        this.email = email
        this.message = message

    method update(filename) is
        system.email(email, replace(&#39;%s&#39;,filename,message))


// Програма може сконфігурувати видавців та підписників, як
// завгодно, залежно від цілей та оточення.
class Application is
    method config() is
        editor = new Editor()

        logger = new LoggingListener(
            &quot;/path/to/log.txt&quot;,
            &quot;Someone has opened file: %s&quot;);
        editor.events.subscribe(&quot;open&quot;, logger)

        emailAlerts = new EmailAlertsListener(
            &quot;admin@example.com&quot;,
            &quot;Someone has changed the file: %s&quot;)
        editor.events.subscribe(&quot;save&quot;, emailAlerts)</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо після зміни стану одного об'єкта потрібно щось зробити в інших, але
ви не знаєте наперед, які саме об'єкти мають відреагувати.
:::

::: applicability-solution
Описана проблема може виникнути при розробленні бібліотек користувацього
інтерфейсу, якщо вам необхідно надати можливість стороннім класам
реагувати на кліки по кнопках.

Патерн Спостерігач надає змогу будь-якому об'єкту з інтерфейсом
підписника зареєструватися для отримання сповіщень про події, що
трапляються в об'єктах-видавцях.
:::

::: applicability-problem
Якщо одні об'єкти мають спостерігати за іншими, але тільки у визначених
випадках.
:::

::: applicability-solution
Видавці ведуть динамічні списки. Усі спостерігачі можуть підписуватися
або відписуватися від отримання сповіщень безпосередньо під час
виконання програми.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Розбийте вашу функціональність на дві частини: незалежне ядро та
    опціональні залежні частини. Незалежне ядро стане видавцем. Залежні
    частини стануть підписниками.

2.  Створіть інтерфейс підписників. Зазвичай достатньо визначити в ньому
    лише один метод сповіщення.

3.  Створіть інтерфейс видавців та опишіть у ньому операції керування
    підпискою. Пам'ятайте, що видавці повинні працювати з підписниками
    тільки через їхній загальний інтерфейс.

4.  Вам потрібно вирішити, куди помістити код ведення підписки, адже він
    зазвичай буває однаковим для всіх типів видавців. Найочевидніший
    спосіб --- це винесення коду до проміжного абстрактного класу, від
    якого будуть успадковуватися всі видавці.

    Якщо ж ви інтегруєте патерн до існуючих класів, то створити новий
    базовий клас може бути важко. У цьому випадку ви можете помістити
    логіку підписки в допоміжний об'єкт та делегувати йому роботу з
    видавцями.

5.  Створіть класи конкретних видавців. Реалізуйте їх таким чином, щоб
    після кожної зміні стану вони слали сповіщення всім своїм
    підписникам.

6.  Реалізуйте метод сповіщення в конкретних підписниках. Не забудьте
    передбачити параметри, через які видавець міг би відправляти якісь
    дані, пов'язані з подією, що відбулась.

    Можливий і інший варіант, коли підписник, отримавши сповіщення, сам
    візьме потрібні дані з об'єкта видавця. Але в цьому разі ви будете
    змушені прив'язати клас підписника до конкретного класу видавця.

7.  Клієнт повинен створювати необхідну кількість об'єктів підписників
    та підписувати їх у видавців.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Видавці не залежать від конкретних класів підписників і навпаки.
-    Ви можете підписувати і відписувати одержувачів «на льоту».
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Підписники сповіщуються у випадковій послідовності.
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

[![Спостерігач на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](observer/csharp/example.html "Спостерігач на C#"){.prog-lang-link}
[![Спостерігач на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](observer/cpp/example.html "Спостерігач на C++"){.prog-lang-link}
[![Спостерігач на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](observer/go/example.html "Спостерігач на Go"){.prog-lang-link}
[![Спостерігач на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](observer/java/example.html "Спостерігач на Java"){.prog-lang-link}
[![Спостерігач на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](observer/php/example.html "Спостерігач на PHP"){.prog-lang-link}
[![Спостерігач на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](observer/python/example.html "Спостерігач на Python"){.prog-lang-link}
[![Спостерігач на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](observer/ruby/example.html "Спостерігач на Ruby"){.prog-lang-link}
[![Спостерігач на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](observer/rust/example.html "Спостерігач на Rust"){.prog-lang-link}
[![Спостерігач на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](observer/swift/example.html "Спостерігач на Swift"){.prog-lang-link}
[![Спостерігач на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](observer/typescript/example.html "Спостерігач на TypeScript"){.prog-lang-link}
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

[Стан []{.fa .fa-arrow-right}](state.html){.btn .btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Знімок](memento.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::
