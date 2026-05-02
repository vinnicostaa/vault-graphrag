::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Стан {#стан .title}

::: aka
Також відомий як: [State]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Стан** --- це поведінковий патерн проектування, що дає змогу об'єктам
змінювати поведінку в залежності від їхнього стану. Ззовні створюється
враження, ніби змінився клас об'єкта.

<figure class="image">
<img
src="../../images/patterns/content/state/state-ukd9df.png?id=b5edf6d1affd9ade2b1a8b0038fc45fd"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/state/state-uk.png?id=b5edf6d1affd9ade2b1a8b0038fc45fd 640w,/images/patterns/content/state/state-uk-1.5x.png?id=6a6b8180a61e287e1d0a8c7b69c6a9e6 960w,/images/patterns/content/state/state-uk-2x.png?id=d17bff3e31a5c259453a74f4cdd85fb1 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Стан" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Патерн Стан неможливо розглядати у відриві від концепції *машини
станів*, також відомої як *стейт-машина* або *скінченний
автомат* []{.pop-i}[Скінченний автомат:
[https://refactoring.guru/uk/fsm](https://uk.wikipedia.org/wiki/%D0%A1%D0%BA%D1%96%D0%BD%D1%87%D0%B5%D0%BD%D0%BD%D0%B8%D0%B9_%D0%B0%D0%B2%D1%82%D0%BE%D0%BC%D0%B0%D1%82)]{.pop-content}.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem10244.png?id=503968745461a0970d1fbb4362dc186f"
style="aspect-ratio: 1.45;"
srcset="/images/patterns/diagrams/state/problem1.png?id=503968745461a0970d1fbb4362dc186f 320w,/images/patterns/diagrams/state/problem1-1.5x.png?id=47c7ca068eceaa9896d320690e6f3368 480w,/images/patterns/diagrams/state/problem1-2x.png?id=ae03c2233939eace11d44925ddeb912d 640w"
sizes="(max-width: 720px) 100vw, 320px" loading="lazy" width="320"
alt="Cкінченний автомат" />
<figcaption><p>Cкінченний автомат.</p></figcaption>
</figure>

Основна ідея в тому, що програма може знаходитися в одному з кількох
станів, які увесь час змінюють один одного. Набір цих станів, а також
переходів між ними, визначений наперед та *скінченний*. Перебуваючи в
різних станах, програма може по-різному реагувати на одні і ті самі
події, що відбуваються з нею.

Такий підхід можна застосувати і до окремих об'єктів. Наприклад, об'єкт
`Документ` може приймати три стани: `Чернетка`, `Модерація` або
`Опублікований`. У кожному з цих станів метод `опублікувати` працюватиме
по-різному:

-   З *чернетки* він надішле документ на *модерацію*.
-   З *модерації* --- в публікацію, але за умови, що це зробив
    адміністратор.
-   В *опублікованому* стані метод не буде робити нічого.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/problem2-uk768a.png?id=780a08b5483e940691b7ce450dc2667e"
style="aspect-ratio: 1.27;"
srcset="/images/patterns/diagrams/state/problem2-uk.png?id=780a08b5483e940691b7ce450dc2667e 560w,/images/patterns/diagrams/state/problem2-uk-1.5x.png?id=07167bb393d187d0c087ece295038613 840w,/images/patterns/diagrams/state/problem2-uk-2x.png?id=9e8491719777bd968523201490342ae3 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Можливі стани документу та переходи між ними" />
<figcaption><p>Можливі стани документу та переходи
між ними.</p></figcaption>
</figure>

Машину станів найчастіше реалізують за допомогою множини умовних
операторів, `if` або `switch`, які перевіряють поточний стан об'єкта та
виконують відповідну поведінку. Ймовірніше за все, ви вже реалізували у
своєму житті хоча б одну машину станів, навіть не знаючи про це. Не
вірите? Як щодо такого коду, виглядає знайомо?

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

Побудована таким чином машина станів має критичну ваду, яка покаже себе,
якщо до `Документа` додати ще з десяток станів. Кожен метод буде
складатися з об'ємного умовного оператора, який перебирає доступні
стани.

Такий код дуже складно підтримувати. Навіть найменша зміна логіки
переходів змусить вас перевіряти роботу всіх методів, які містять умовні
оператори машини станів.

Плутанина та нагромадження умов особливо сильно проявляється в старих
проектах. Набір можливих станів буває важко визначити заздалегідь, тому
вони увесь час додаються в процесі еволюції програми. Через це рішення,
що здавалося простим і ефективним на початку розробки проекту, може
згодом стати проекцією величезного макаронного монстра.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Стан пропонує створити окремі класи для кожного стану, в якому
може перебувати контекстний об'єкт, а потім винести туди поведінки, що
відповідають цим станам.

Замість того, щоб зберігати код всіх станів, початковий об'єкт, який
зветься *контекстом*, міститиме посилання на один з об'єктів-станів і
делегуватиме йому роботу в залежності від стану.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/solution-uk544c.png?id=4b6564e9309d3455d538a08aa9c707c8"
style="aspect-ratio: 1.53;"
srcset="/images/patterns/diagrams/state/solution-uk.png?id=4b6564e9309d3455d538a08aa9c707c8 490w,/images/patterns/diagrams/state/solution-uk-1.5x.png?id=658ce9468b7ef96d3c4ee7c8c3d3d041 735w,/images/patterns/diagrams/state/solution-uk-2x.png?id=11cafd8b1cda86fc58eebc599480edca 980w"
sizes="(max-width: 720px) 100vw, 490px" loading="lazy" width="490"
alt="Сторінка делегує виконання своєму активному стану" />
<figcaption><p>Сторінка делегує виконання своєму
активному стану.</p></figcaption>
</figure>

Завдяки тому, що об'єкти станів матимуть спільний інтерфейс, контекст
зможе делегувати роботу стану, не прив'язуючись до його класу. Поведінку
контексту можна буде змінити в будь-який момент, підключивши до нього
інший об'єкт-стан.

Дуже важливим нюансом, який відрізняє цей патерн від
[Стратегії](strategy.html), є те, що і контекст, і конкретні стани
можуть знати один про одного та ініціювати переходи від одного стану до
іншого.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

Ваш смартфон поводиться по-різному в залежності від поточного стану:

-   Якщо телефон розблоковано, натискання кнопок телефону призведе до
    якихось дій.
-   Якщо телефон заблоковано, натискання кнопок призведе до появи екрану
    розблокування.
-   Якщо телефон розряджено, натискання кнопок призведе до появи екрану
    зарядки.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/state/structure-ukf6d3.png?id=33c1da057dfc28b9de2b31b504c2d8f7"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.32;"
srcset="/images/patterns/diagrams/state/structure-uk.png?id=33c1da057dfc28b9de2b31b504c2d8f7 540w,/images/patterns/diagrams/state/structure-uk-1.5x.png?id=54bbb061bfb27bb8fd75d4d5ba3eebb4 810w,/images/patterns/diagrams/state/structure-uk-2x.png?id=452708535f5cedd3c91f5cdb54c5e726 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура класів патерна Стан" /><img
src="../../images/patterns/diagrams/state/structure-uk-indexedf641.png?id=c24d4a98b75abf7a2ba90781d1f93b76"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.26;"
srcset="/images/patterns/diagrams/state/structure-uk-indexed.png?id=c24d4a98b75abf7a2ba90781d1f93b76 540w,/images/patterns/diagrams/state/structure-uk-indexed-1.5x.png?id=afddba014ded2a1bd8145504a2d1edfc 810w,/images/patterns/diagrams/state/structure-uk-indexed-2x.png?id=76562f2b15aba1b16f60df9ad66825d9 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура класів патерна Стан" />
</figure>
:::

1.  **Контекст** зберігає посилання на об'єкт стану та делегує йому
    частину роботи, яка залежить від станів. Контекст працює з цим
    об'єктом через загальний інтерфейс станів. Контекст повинен мати
    метод для присвоєння йому нового об'єкта-стану.

2.  **Стан** описує спільний для всіх конкретних станів інтерфейс.

3.  **Конкретні стани** реалізують поведінки, пов'язані з певним станом
    контексту. Іноді доводиться створювати цілі ієрархії класів станів,
    щоб узагальнити дублюючий код.

    Стан може мати зворотнє посилання на об'єкт контексту. Через нього
    не тільки зручно отримувати з контексту потрібну інформацію, але й
    здійснювати зміну стану.

4.  І контекст, і об'єкти конкретних станів можуть вирішувати, коли і
    який стан буде обрано наступним. Щоб перемкнути стан, потрібно
    подати інший об'єкт-стан до контексту.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі патерн **Стан** змінює функціональність одних і тих
самих елементів керування музичним програвачем, залежно від стану, в
якому зараз знаходиться програвач.

<figure class="image">
<img
src="../../images/patterns/diagrams/state/example1dc9.png?id=1ffdb109b3ebb85d223b9d1651d2034c"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/state/example.png?id=1ffdb109b3ebb85d223b9d1651d2034c 590w,/images/patterns/diagrams/state/example-1.5x.png?id=a9ff56d0a139530fa103d496513dfa06 885w,/images/patterns/diagrams/state/example-2x.png?id=cd81e3ffb8aba5883983a59c111b805f 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Структура класів прикладу патерна Стан" />
<figcaption><p>Приклад зміни поведінки програвача за
допомогою станів.</p></figcaption>
</figure>

Об'єкт програвача містить об'єкт-стан, якому й делегує головну роботу.
Змінюючи стан, можна впливати на те, як поводяться елементи керування
програвача.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс усіх станів.
abstract class State is
    protected field player: AudioPlayer

    // Контекст передає себе до конструктора стану, щоб стан міг
    // звертатися до його даних та методів у майбутньому, якщо
    // буде потрібно.
    constructor State(player) is
        this.player = player

    abstract method clickLock()
    abstract method clickPlay()
    abstract method clickNext()
    abstract method clickPrevious()


// Конкретні стани реалізують методи загального стану по-своєму.
class LockedState extends State is

    // При розблокуванні програвача із заблокованими клавішами,
    // він може прийняти один з двох станів.
    method clickLock() is
        if (player.playing)
            player.changeState(new PlayingState(player))
        else
            player.changeState(new ReadyState(player))

    method clickPlay() is
        // Нічого не робити.

    method clickNext() is
        // Нічого не робити.

    method clickPrevious() is
        // Нічого не робити.


// Конкретні стани самі можуть переводити контекст в інші стани.
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


// Програвач виступає в ролі контексту.
class AudioPlayer is
    field state: State
    field UI, volume, playlist, currentSong

    constructor AudioPlayer() is
        this.state = new ReadyState(this)

        // Контекст змушує стан реагувати на користувацький ввід
        // замість себе. Реакція може бути різною, залежно від
        // того, який стан зараз активний.
        UI = new UserInterface()
        UI.lockButton.onClick(this.clickLock)
        UI.playButton.onClick(this.clickPlay)
        UI.nextButton.onClick(this.clickNext)
        UI.prevButton.onClick(this.clickPrevious)

    // Інші об&#39;єкти теж повинні мати можливість замінити стан
    // програвача.
    method changeState(state: State) is
        this.state = state

    // Методи UI делегуватимуть роботу активному стану.
    method clickLock() is
        state.clickLock()
    method clickPlay() is
        state.clickPlay()
    method clickNext() is
        state.clickNext()
    method clickPrevious() is
        state.clickPrevious()

    // Сервісні методи контексту, що викликаються станами.
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
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо у вас є об'єкт, поведінка якого кардинально змінюється в залежності
від внутрішнього стану, причому типів станів багато, а їхній код часто
змінюється.
:::

::: applicability-solution
Патерн пропонує виділити в окремі класи всі поля й методи, пов'язані з
визначеним станом. Початковий об'єкт буде постійно посилатися на один з
об'єктів-станів, делегуючи йому частину своєї роботи. Для зміни стану до
контексту достатньо буде підставляти інший об'єкт-стан.
:::

::: applicability-problem
Якщо код класу містить безліч великих, схожих один на одного умовних
операторів, які вибирають поведінки в залежності від поточних значень
полів класу.
:::

::: applicability-solution
Патерн пропонує перемістити кожну гілку такого умовного оператора до
власного класу. Сюди ж можна поселити й усі поля, пов'язані з цим
станом.
:::

::: applicability-problem
Якщо ви свідомо використовуєте табличну машину станів, побудовану на
умовних операторах, але змушені миритися з дублюванням коду для схожих
станів та переходів.
:::

::: applicability-solution
Патерн Стан дозволяє реалізувати ієрархічну машину станів, що базується
на наслідуванні. Ви можете успадкувати схожі стани від одного
батьківського класу та винести туди весь дублюючий код.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначтеся з класом, який відіграватиме роль контексту. Це може бути
    як існуючий клас, який вже має залежність від стану, так і новий
    клас, якщо код станів «розмазаний» по кількох класах.

2.  Створіть загальний інтерфейс станів. Він повинен описувати методи,
    спільні для всіх станів, виявлених у контексті. Зверніть увагу, що
    не всю поведінку контексту потрібно переносити до стану, а тільки
    ту, яка залежить від станів.

3.  Для кожного фактичного стану створіть клас, який реалізує інтерфейс
    стану. Перемістіть код, пов'язаний з конкретними станами, до
    потрібних класів. Зрештою, всі методи інтерфейсу стану повинні бути
    реалізовані в усіх класах станів.

    При перенесенні поведінки з контексту ви можете зіткнутися з тим, що
    ця поведінка залежить від приватних полів або методів контексту, до
    яких немає доступу з об'єкта стану. Є кілька способів, щоб обійти цю
    проблему.

    Найпростіший --- залишити поведінку всередині контексту, викликаючи
    його з об'єкта стану. З іншого боку, ви можете зробити класи станів
    вкладеними до класу контексту, і тоді вони отримають доступ до всіх
    приватних частин контексту. Останній спосіб, щоправда, доступний
    лише в деяких мовах програмування (наприклад, Java, C#).

4.  Створіть в контексті поле для зберігання об'єктів-станів, а також
    публічний метод для зміни значення цього поля.

5.  Старі методи контексту, в яких перебував залежний від стану код,
    замініть на виклики відповідних методів об'єкта-стану.

6.  В залежності від бізнес-логіки, розмістіть код, який перемикає стан
    контексту, або всередині контексту, або всередині класів конкретних
    станів.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Позбавляє від безлічі великих умовних операторів машини станів.
-    Концентрує в одному місці код, пов'язаний з певним станом.
-    Спрощує код контексту.
:::

::: col-sm-6
-    Може невиправдано ускладнити код, якщо станів мало, і вони рідко
    змінюються.
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

-   [Стан](state.html) можна розглядати як надбудову над
    [Стратегією](strategy.html). Обидва патерни використовують
    композицію, щоб змінювати поведінку головного об'єкта, делегуючи
    роботу вкладеним об'єктам-помічникам. Проте в *Стратегії* ці об'єкти
    не знають один про одного і жодним чином не пов'язані. У *Стані*
    конкретні стани самостійно можуть перемикати контекст.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Стан на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](state/csharp/example.html "Стан на C#"){.prog-lang-link}
[![Стан на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](state/cpp/example.html "Стан на C++"){.prog-lang-link}
[![Стан на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](state/go/example.html "Стан на Go"){.prog-lang-link}
[![Стан на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](state/java/example.html "Стан на Java"){.prog-lang-link}
[![Стан на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](state/php/example.html "Стан на PHP"){.prog-lang-link}
[![Стан на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](state/python/example.html "Стан на Python"){.prog-lang-link}
[![Стан на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](state/ruby/example.html "Стан на Ruby"){.prog-lang-link}
[![Стан на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](state/rust/example.html "Стан на Rust"){.prog-lang-link}
[![Стан на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](state/swift/example.html "Стан на Swift"){.prog-lang-link}
[![Стан на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](state/typescript/example.html "Стан на TypeScript"){.prog-lang-link}
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

[Стратегія []{.fa .fa-arrow-right}](strategy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Спостерігач](observer.html){.btn .btn-default
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
