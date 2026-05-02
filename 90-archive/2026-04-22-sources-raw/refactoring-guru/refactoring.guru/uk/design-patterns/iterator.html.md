::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Ітератор {#ітератор .title}

::: aka
Також відомий як: [Iterator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Ітератор** --- це поведінковий патерн проектування, що дає змогу
послідовно обходити елементи складових об'єктів, не розкриваючи їхньої
внутрішньої організації.

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-enfc4d.png?id=d19123d71d355d01b0ede4be173eb695"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/iterator/iterator-en.png?id=d19123d71d355d01b0ede4be173eb695 640w,/images/patterns/content/iterator/iterator-en-1.5x.png?id=54aa477cafffe8f9b1b6e63c2e88c21b 960w,/images/patterns/content/iterator/iterator-en-2x.png?id=2a85705e8e5fab257802b2ce36d6d236 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Ітератор" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Колекції --- це найпоширеніша структура даних, яку ви можете зустріти в
програмуванні. Це набір об'єктів, зібраний в одну купу за якимись
критеріями.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem14cac.png?id=52ef2fe2d4920e3fed696c221fe757f2"
style="aspect-ratio: 4.9;"
srcset="/images/patterns/diagrams/iterator/problem1.png?id=52ef2fe2d4920e3fed696c221fe757f2 490w,/images/patterns/diagrams/iterator/problem1-1.5x.png?id=b46e39ade7de292fdcacc9790066c72f 735w,/images/patterns/diagrams/iterator/problem1-2x.png?id=1f4384c3c631be238bfc23d6eb84a0ef 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Різні типи колекцій" />
<figcaption><p>Різні типи колекцій.</p></figcaption>
</figure>

Більшість колекцій виглядають як звичайний список елементів. Але є й
екзотичні колекції, побудовані на основі дерев, графів та інших складних
структур даних.

Незважаючи на те, яким чином структуровано колекцію, користувач повинен
мати можливість послідовно обходити її елементи, щоб виконувати з ними
певні дії.

У який же спосіб слід переміщатися складною структурою даних? Наприклад,
сьогодні може бути достатнім обхід дерева в глибину, але завтра виникне
необхідність переміщуватися деревом по ширині. А на наступному тижні,
хай йому грець, знадобиться можливість обходу колекції у випадковому
порядку.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/problem29f71.png?id=f9c1a746c787320291c85fdc2a314192"
style="aspect-ratio: 3.75;"
srcset="/images/patterns/diagrams/iterator/problem2.png?id=f9c1a746c787320291c85fdc2a314192 600w,/images/patterns/diagrams/iterator/problem2-1.5x.png?id=7a003d020e789773e0c833d7b1df00e6 900w,/images/patterns/diagrams/iterator/problem2-2x.png?id=656b13c109d4d4960ea1f9fa993bae4c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Одну і ту саму колекцію можна обходити різними способами" />
<figcaption><p>Одну і ту саму колекцію можна обходити
різними способами.</p></figcaption>
</figure>

Додаючи все нові алгоритми до коду колекції, ви потроху розмиваєте її
основну задачу, що полягає в ефективному зберіганні даних. Деякі
алгоритми можуть бути аж занадто «заточені» під певну програму, а тому
виглядатимуть неприродно в загальному класі колекції.
:::

::: {.section .solution}
##  Рішення {#solution}

Ідея патерна Ітератор полягає в тому, щоб винести поведінку обходу
колекції з самої колекції в окремий об'єкт.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/solution19739.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059"
style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/iterator/solution1.png?id=2f5fbcce6099d8ea09b2fbb83e3e7059 400w,/images/patterns/diagrams/iterator/solution1-1.5x.png?id=68612372ede6e5029403d38b381fdc05 600w,/images/patterns/diagrams/iterator/solution1-2x.png?id=dcebcad4c197bfa5f25f680382d0e5ac 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Ітератори містять код обходу колекції" />
<figcaption><p>Ітератори містять код обходу колекції. Одну колекцію
можуть обходити відразу декілька ітераторів.</p></figcaption>
</figure>

Об'єкт-ітератор відстежуватиме стан обходу, поточну позицію в колекції
та кількість елементів, які ще залишилося обійти. Одну і ту саму
колекцію зможуть одночасно обходити різні ітератори, а сама колекція
навіть не знатиме про це.

До того ж, якщо вам потрібно буде додати новий спосіб обходу, ви зможете
створите окремий клас ітератора, не змінюючи існуючого коду колекції.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/content/iterator/iterator-comic-1-uk1010.png?id=bbb108ea2a4dde3bef3ba7f1746aa77a"
style="aspect-ratio: 2.13;"
srcset="/images/patterns/content/iterator/iterator-comic-1-uk.png?id=bbb108ea2a4dde3bef3ba7f1746aa77a 640w,/images/patterns/content/iterator/iterator-comic-1-uk-1.5x.png?id=4bde8ea7edf2fe0cab09cac6344528e2 960w,/images/patterns/content/iterator/iterator-comic-1-uk-2x.png?id=023bae668d334c94949c011a04eac65d 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Варіанти прогулянок Римом" />
<figcaption><p>Варіанти прогулянок Римом.</p></figcaption>
</figure>

Ви плануєте полетіти до Риму та обійти всі визначні пам'ятки за кілька
днів. Але по приїзді ви можете довго блукати вузькими вуличками,
намагаючись знайти один тільки Колізей.

Якщо у вас обмежений бюджет, ви можете скористатися віртуальним гідом,
встановленим у смартфоні, який дозволить відфільтрувати тільки цікаві
вам об'єкти. А можете плюнути на все та найняти місцевого гіда, який хоч
і обійдеться в копієчку, але знає все місто, як свої п'ять пальців, і
зможе «занурити» вас в усі міські легенди.

Таким чином, Рим виступає колекцією пам'яток, а ваш мозок, навігатор чи
гід --- ітератором колекції. Ви як клієнтський код можете вибрати одного
з ітераторів, відштовхуючись від вирішуваного завдання та доступних
ресурсів.
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
alt="Структура класів патерна Ітератор" /><img
src="../../images/patterns/diagrams/iterator/structure-indexed3472.png?id=7bc28907ff6b480db6635a93ebaa10ff"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/iterator/structure-indexed.png?id=7bc28907ff6b480db6635a93ebaa10ff 520w,/images/patterns/diagrams/iterator/structure-indexed-1.5x.png?id=32fde4c4c1cbfbe07aa6e716e5ac2346 780w,/images/patterns/diagrams/iterator/structure-indexed-2x.png?id=d809428b2262e2013972fe69d2434473 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура класів патерна Ітератор" />
</figure>
:::

1.  **Ітератор** описує інтерфейс для доступу та обходу елементів
    колекцій.

2.  **Конкретний ітератор** реалізує алгоритм обходу якоїсь конкретної
    колекції. Об'єкт ітератора повинен сам відстежувати поточну позицію
    при обході колекції, щоб окремі ітератори могли обходити одну і ту
    саму колекцію незалежно.

3.  **Колекція** описує інтерфейс отримання ітератора з колекції. Як ми
    вже говорили, колекції не завжди є списком. Це може бути і база
    даних, і віддалене API, і навіть дерево
    [Компонувальника](composite.html). Тому сама колекція може
    створювати ітератори, оскільки вона знає, які саме ітератори здатні
    з нею працювати.

4.  **Конкретна колекція** повертає новий екземпляр певного конкретного
    ітератора, зв'язавши його з поточним об'єктом колекції. Зверніть
    увагу на те, що сигнатура методу повертає інтерфейс ітератора. Це
    дозволяє клієнтові не залежати від конкретних класів ітераторів.

5.  **Клієнт** працює з усіма об'єктами через інтерфейси колекції та
    ітератора. Через це клієнтський код не залежить від конкретних
    класів, що дозволяє застосовувати різні ітератори, не змінюючи
    існуючого коду програми.

    В загальному випадку клієнти не створюють об'єкти ітераторів, а
    отримують їх з колекцій. Тим не менше, якщо клієнтові потрібний
    спеціальний ітератор, він завжди може створити його самостійно.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі патерн **Ітератор** використовується для реалізації
обходу нестандартної колекції, яка інкапсулює доступ до соціального
графа Facebook. Колекція надає декілька ітераторів, які можуть обходити
профілі людей різними способами.

<figure class="image">
<img
src="../../images/patterns/diagrams/iterator/examplee0a2.png?id=f2a24ef3787bf80ed450709240506ff2"
style="aspect-ratio: 0.95;"
srcset="/images/patterns/diagrams/iterator/example.png?id=f2a24ef3787bf80ed450709240506ff2 600w,/images/patterns/diagrams/iterator/example-1.5x.png?id=cab0e1459ffc3721579a3e7d6a4e9022 900w,/images/patterns/diagrams/iterator/example-2x.png?id=73c3e55f75ca0250bd84e8a1d8cc88d2 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура класів прикладу патерна Ітератор" />
<figcaption><p>Приклад обходу соціальних профілів
через ітератор.</p></figcaption>
</figure>

Зокрема, ітератор друзів перебирає всіх друзів профілю, а ітератор колег
фільтрує друзів згідно їхньої приналежності до компанії профілю. Всі
ітератори реалізують спільний інтерфейс, який дає змогу клієнтам
працювати з профілями, не заглиблюючись у деталі роботи з соціальною
мережею (наприклад, авторизацію, надсилання REST запитів та інше).

Крім того, Ітератор позбавляє код від прив'язки до конкретних класів
колекцій. Це дозволяє додати підтримку іншого виду колекцій (наприклад,
LinkedIn), не змінюючи клієнтський код, який працює з ітераторами та
колекціями.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс колекцій повинен визначити фабричний
// метод для виробництва ітератора. Можна визначити відразу
// кілька методів, щоб дати користувачам різні варіанти обходу
// однієї і тієї самої колекції.
interface SocialNetwork is
    method createFriendsIterator(profileId): ProfileIterator
    method createCoworkersIterator(profileId): ProfileIterator


// Конкретна колекція знає, об&#39;єкти яких ітераторів потрібно
// створювати.
class Facebook implements SocialNetwork is
    // ... Основний код колекції ...

    // Код отримання потрібного ітератора.
    method createFriendsIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;friends&quot;)
    method createCoworkersIterator(profileId) is
        return new FacebookIterator(this, profileId, &quot;coworkers&quot;)


// Загальний інтерфейс ітераторів.
interface ProfileIterator is
    method getNext(): Profile
    method hasMore(): bool


// Конкретний ітератор.
class FacebookIterator implements ProfileIterator is
    // Ітератору потрібне посилання на колекцію, яку він
    // обходить.
    private field facebook: Facebook
    private field profileId, type: string

    // Кожен ітератор обходить колекцію, незалежно від інших,
    // тому самостійно відслідковує поточну позицію обходу.
    private field currentPosition
    private field cache: array of Profile

    constructor FacebookIterator(facebook, profileId, type) is
        this.facebook = facebook
        this.profileId = profileId
        this.type = type

    private method lazyInit() is
        if (cache == null)
            cache = facebook.socialGraphRequest(profileId, type)

    // Всі конкретні ітератори реалізують методи загального
    // інтерфейсу по-своєму.
    method getNext() is
        if (hasMore())
            result = cache[currentPosition]
            currentPosition++
            return result

    method hasMore() is
        lazyInit()
        return currentPosition &lt; cache.length


// Ось іще корисна тактика: ми можемо передавати об&#39;єкт
// ітератора замість колекції до клієнтських класів. При такому
// підході клієнтський код не матиме доступу до колекцій, а
// значить, його не турбуватимуть подробиці їхньої реалізації.
// Йому буде доступний лише загальний інтерфейс ітераторів.
class SocialSpammer is
    method send(iterator: ProfileIterator, message: string) is
        while (iterator.hasMore())
            profile = iterator.getNext()
            System.sendEmail(profile.getEmail(), message)


// Головний клас програми конфігурує ітератори та колекції, як
// завгодно.
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
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо у вас є складна структура даних, і ви хочете приховати від клієнта
деталі її реалізації (з питань складності або безпеки).
:::

::: applicability-solution
Ітератор надає клієнтові лише кілька простих методів перебору елементів
колекції. Це не тільки спрощує доступ до колекції, але й захищає її від
необережних або злочинних дій.
:::

::: applicability-problem
Якщо вам потрібно мати кілька варіантів обходу однієї і тієї самої
структури даних.
:::

::: applicability-solution
Нетривіальні алгоритми обходу структури даних можуть мати досить
об'ємний код. Цей код буде захаращувати все навкруги --- чи то самий
клас колекції, чи частина бізнес-логіки програми. Застосувавши ітератор,
ви можете виділити код обходу структури даних в окремий клас, спростивши
підтримку решти коду.
:::

::: applicability-problem
Якщо вам хочеться мати єдиний інтерфейс обходу різних структур даних.
:::

::: applicability-solution
Ітератор дозволяє винести реалізації різних варіантів обходу в підкласи.
Це дозволить легко взаємозаміняти об'єкти ітераторів в залежності від
того, з якою структурою даних доводиться працювати.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Створіть загальний інтерфейс ітераторів. Обов'язковий мінімум --- це
    операція отримання наступного елемента. Але для зручності можна
    передбачити й інше. Наприклад, методи отримання попереднього
    елементу, поточної позиції, перевірки закінчення обходу тощо.

2.  Створіть інтерфейс колекції та опишіть у ньому метод отримання
    ітератора. Важливо, щоб сигнатура методу повертала загальний
    інтерфейс ітераторів, а не один з конкретних ітераторів.

3.  Створіть класи конкретних ітераторів для тих колекцій, які потрібно
    обходити за допомогою патерна. Ітератор повинен бути прив'язаний
    тільки до одного об'єкта колекції. Зазвичай цей зв'язок
    встановлюється через конструктор.

4.  Реалізуйте методи отримання ітератора в конкретних класах колекцій.
    Вони повинні створювати новий ітератор того класу, який здатен
    працювати з даним типом колекції. Колекція повинна передавати
    посилання на власний об'єкт до конструктора ітератора.

5.  У клієнтському коді та в класах колекцій не повинно залишитися коду
    обходу елементів. Клієнт повинен отримувати новий ітератор з об'єкта
    колекції кожного разу, коли йому потрібно перебрати її елементи.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Спрощує класи зберігання даних.
-    Дозволяє реалізувати різні способи обходу структури даних.
-    Дозволяє одночасно переміщуватися структурою даних у різних
    напрямках.
:::

::: col-sm-6
-    Невиправданий, якщо можна обійтися простим циклом.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   Ви можете обходити дерево [Компонувальника](composite.html),
    використовуючи [Ітератор](iterator.html).

-   [Фабричний метод](factory-method.html) можна використовувати разом з
    [Ітератором](iterator.html), щоб підкласи колекцій могли створювати
    необхідні їм ітератори.

-   [Знімок](memento.html) можна використовувати разом з
    [Ітератором](iterator.html), щоб зберегти поточний стан обходу
    структури даних та повернутися до нього в майбутньому, якщо буде
    потрібно.

-   [Відвідувач](visitor.html) можна використовувати спільно з
    [Ітератором](iterator.html). *Ітератор* відповідатиме за обхід
    структури даних, а *Відвідувач* --- за виконання дій над кожним її
    компонентом.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Ітератор на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](iterator/csharp/example.html "Ітератор на C#"){.prog-lang-link}
[![Ітератор на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](iterator/cpp/example.html "Ітератор на C++"){.prog-lang-link}
[![Ітератор на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](iterator/go/example.html "Ітератор на Go"){.prog-lang-link}
[![Ітератор на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](iterator/java/example.html "Ітератор на Java"){.prog-lang-link}
[![Ітератор на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](iterator/php/example.html "Ітератор на PHP"){.prog-lang-link}
[![Ітератор на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](iterator/python/example.html "Ітератор на Python"){.prog-lang-link}
[![Ітератор на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](iterator/ruby/example.html "Ітератор на Ruby"){.prog-lang-link}
[![Ітератор на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](iterator/rust/example.html "Ітератор на Rust"){.prog-lang-link}
[![Ітератор на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](iterator/swift/example.html "Ітератор на Swift"){.prog-lang-link}
[![Ітератор на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](iterator/typescript/example.html "Ітератор на TypeScript"){.prog-lang-link}
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

[Посередник []{.fa .fa-arrow-right}](mediator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Команда](command.html){.btn .btn-default
rel="prev"}
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
