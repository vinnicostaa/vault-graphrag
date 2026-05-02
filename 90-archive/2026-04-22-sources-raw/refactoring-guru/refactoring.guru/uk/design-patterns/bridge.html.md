:::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Міст {#міст .title}

::: aka
Також відомий як: [Bridge]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Міст** --- це структурний патерн проектування, який розділяє один або
кілька класів на дві окремі ієрархії --- абстракцію та реалізацію,
дозволяючи змінювати код в одній гілці класів, незалежно від іншої.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge3e59.png?id=bd543d4fb32e11647767301581a5ad54"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge.png?id=bd543d4fb32e11647767301581a5ad54 640w,/images/patterns/content/bridge/bridge-1.5x.png?id=8ef07b78d61cc3830b7e2b783c76c775 960w,/images/patterns/content/bridge/bridge-2x.png?id=1e905ae5742e5cd10a7eb0e3175ef00d 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Міст" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

*Абстракція?* *Реалізація?!* Звучить страхітливо! Розгляньмо простенький
приклад, щоб зрозуміти про що йде мова.

У вас є клас геометричних `Фігур`, який має підкласи `Круг` та
`Квадрат`. Ви хочете розширити ієрархію фігур за кольором, тобто мати
`Червоні` та `Сині` фігури. Але для того, щоб все це об'єднати,
доведеться створити 4 комбінації підкласів на зразок `СиніКруги` та
`ЧервоніКвадрати`.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/problem-uk6897.png?id=e31cf4bc7e7d1518c360edf94c227828"
style="aspect-ratio: 1.41;"
srcset="/images/patterns/diagrams/bridge/problem-uk.png?id=e31cf4bc7e7d1518c360edf94c227828 480w,/images/patterns/diagrams/bridge/problem-uk-1.5x.png?id=1ff2554653a18321ad49ce043fe46c29 720w,/images/patterns/diagrams/bridge/problem-uk-2x.png?id=889c5b2ac48e841a9a6c808ac255d089 960w"
sizes="(max-width: 720px) 100vw, 480px" loading="lazy" width="480"
alt="Проблема патерна Міст" />
<figcaption><p>Кількість підкласів зростає в
геометричній прогресії.</p></figcaption>
</figure>

При додаванні нових видів фігур і кольорів кількість комбінацій
зростатиме в геометричній прогресії. Наприклад, щоб ввести в програму
фігури трикутників, доведеться створити відразу два нових класи
трикутників, по одному для кожного кольору. Після цього введення нового
кольору вимагатиме створення вже трьох класів, по одному для кожного
виду фігур. Чим далі, тим гірше.
:::

::: {.section .solution}
##  Рішення {#solution}

Корінь проблеми полягає в тому, що ми намагаємося розширити класи фігур
одразу в двох незалежних площинах --- за видом та кольором. Саме це
призводить до розростання дерева класів.

Патерн Міст пропонує замінити спадкування на делегування. Для цього
потрібно виділити одну з таких «площин» в окрему ієрархію і посилатися
на об'єкт цієї ієрархії, замість зберігання його стану та поведінки
всередині одного класу.

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/solution-uk09eb.png?id=4bd0de9f389eb66cbcc3553e2f18d61f"
style="aspect-ratio: 2.3;"
srcset="/images/patterns/diagrams/bridge/solution-uk.png?id=4bd0de9f389eb66cbcc3553e2f18d61f 460w,/images/patterns/diagrams/bridge/solution-uk-1.5x.png?id=183a38dabc79f5fd6422008880129910 690w,/images/patterns/diagrams/bridge/solution-uk-2x.png?id=2c8bc596bcf03ee2807a92498bd76708 920w"
sizes="(max-width: 720px) 100vw, 460px" loading="lazy" width="460"
alt="Рішення патерна Міст" />
<figcaption><p>Розмноження підкласів можна зупинити, розбивши класи на
кілька ієрархій.</p></figcaption>
</figure>

Таким чином, ми можемо зробити `Колір` окремим класом з підкласами
`Червоний` та `Синій`. Клас `Фігур` отримає посилання на об'єкт
`Кольору` і зможе делегувати йому роботу, якщо виникне така
необхідність. Такий зв'язок і стане мостом між `Фігурами` та `Кольором`.
При додаванні нових класів кольорів не потрібно буде звертатись до
класів фігур і навпаки.

#### Абстракція і Реалізація

Ці терміни було введено в книзі GoF []{.pop-i}[Gang of Four / «Банда
чотирьох». Автори книги *Design Patterns: Elements of Reusable
Object-Oriented Software*
[https://refactoring.guru/uk/gof-book](https://rozetka.com.ua/ua/6475864/p6475864/).]{.pop-content}
при описі Мосту. На мій погляд, вони виглядають занадто академічними та
показують патерн складнішим, ніж він є насправді. Пам'ятаючи про приклад
з фігурами й кольорами, давайте все ж таки розберемося, що мали на увазі
автори патерна.

Отже, *абстракція* (або *інтерфейс*) --- це уявний рівень керування
чим-небудь, що не виконує роботу самостійно, а делегує її рівню
*реалізації* (який зветься *платформою*).

> Тільки не плутайте ці терміни з *інтерфейсами* або *абстрактними
> класами* вашої мови програмування --- це не одне і те ж саме.

Якщо говорити про реальні програми, то абстракцією може виступати
графічний інтерфейс програми (GUI), а реалізацією --- низькорівневий код
операційної системи (API), до якого графічний інтерфейс звертається,
реагуючи на дії користувача.

Ви можете розвивати програму у двох різних напрямках:

-   мати кілька різних GUI (наприклад, для звичайних користувачів та
    адміністраторів).
-   підтримувати багато видів API (наприклад, працювати під Windows,
    Linux і macOS).

Така програма може виглядати як один великий клубок коду, в якому
змішано умовні оператори рівнів GUI та API.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-3-uk165f.png?id=8b20c0f2100fc24fafbd0b12bb894866"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/bridge/bridge-3-uk.png?id=8b20c0f2100fc24fafbd0b12bb894866 600w,/images/patterns/content/bridge/bridge-3-uk-1.5x.png?id=af74ec5ca122f8ef7e565bb2369a5442 900w,/images/patterns/content/bridge/bridge-3-uk-2x.png?id=68160642232950f6356eeccf601cf7ed 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Захист від змін" />
<figcaption><p>Коли зміни беруть проект в «осаду», вам легше
відбиватися, якщо розділити монолітний код на частини.</p></figcaption>
</figure>

Ви можете спробувати структурувати цей хаос, створивши для кожної з
варіацій інтерфейсу-платформи свої підкласи. Але такий підхід призведе
до зростання класів комбінацій, і з кожною новою платформою їх буде все
більше й більше.

Ми можемо вирішити цю проблему, застосувавши Міст. Патерн пропонує
розплутати цей код, розділивши його на дві частини:

-   Абстракцію: рівень графічного інтерфейсу програми.
-   Реалізацію: рівень взаємодії з операційною системою.

<figure class="image">
<img
src="../../images/patterns/content/bridge/bridge-2-uk5082.png?id=615f29f7746b0f9fb0260cef62787960"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/bridge/bridge-2-uk.png?id=615f29f7746b0f9fb0260cef62787960 640w,/images/patterns/content/bridge/bridge-2-uk-1.5x.png?id=fe0f91ee837c3a1e433d63614bbc4680 960w,/images/patterns/content/bridge/bridge-2-uk-2x.png?id=77694eee4cae9188d9828ed2f623c5ad 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Варіант крос-платформової архітектури" />
<figcaption><p>Один з варіантів
крос-платформової архітектури.</p></figcaption>
</figure>

Абстракція делегуватиме роботу одному з об'єктів реалізації. Причому,
реалізації можна буде взаємозаміняти, але тільки за умови, що всі вони
слідуватимуть єдиному інтерфейсу.

Таким чином, ви зможете змінювати графічний інтерфейс програми, не
чіпаючи низькорівневий код роботи з операційною системою. І навпаки, ви
зможете додавати підтримку нових операційних систем, створюючи нові
підкласи реалізації, без необхідності правити код у класах графічного
інтерфейсу.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/structure-uk6ca5.png?id=f7e593ce132822c3c90d7f3c5fb8772d"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.47;"
srcset="/images/patterns/diagrams/bridge/structure-uk.png?id=f7e593ce132822c3c90d7f3c5fb8772d 560w,/images/patterns/diagrams/bridge/structure-uk-1.5x.png?id=1bec5ffeb311b8e1bc6a8c5425c935f0 840w,/images/patterns/diagrams/bridge/structure-uk-2x.png?id=f893df875d884d8c52c320ee2d02f6ca 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів патерна Міст" /><img
src="../../images/patterns/diagrams/bridge/structure-uk-indexed66c9.png?id=2fa8e5455b355140339cd75b17943994"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.44;"
srcset="/images/patterns/diagrams/bridge/structure-uk-indexed.png?id=2fa8e5455b355140339cd75b17943994 560w,/images/patterns/diagrams/bridge/structure-uk-indexed-1.5x.png?id=4497395a4880a0a73a787fa4659e8fde 840w,/images/patterns/diagrams/bridge/structure-uk-indexed-2x.png?id=1706b9ebfddb89023919222e568d771a 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів патерна Міст" />
</figure>
:::

1.  **Абстракція** містить керуючу логіку. Код абстракції делегує
    реальну роботу пов'язаному об'єктові реалізації.

2.  **Реалізація** описує загальний інтерфейс для всіх реалізацій. Всі
    методи, які тут описані, будуть доступні з класу абстракції та його
    підкласів.

    Інтерфейси абстракції та реалізації можуть або збігатися, або бути
    абсолютно різними. Проте, зазвичай в реалізації живуть базові
    операції, на яких будуються складні операції абстракції.

3.  **Конкретні реалізації** містять платформо-залежний код.

4.  **Розширені абстракції** містять різні варіації керуючої логіки. Як
    і батьківский клас, працює з реалізаціями тільки через загальний
    інтерфейс реалізацій.

5.  **Клієнт** працює тільки з об'єктами абстракції. Не рахуючи
    початкового зв'язування абстракції з однією із реалізацій,
    клієнтський код не має прямого доступу до об'єктів реалізації.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Міст** ділить монолітний код приладів та пультів на
дві частини: прилади (виступають реалізацією) і пульти керування ними
(виступають абстракцією).

<figure class="image">
<img
src="../../images/patterns/diagrams/bridge/example-uk3be2.png?id=cc59fa73f2b5640e0cf28bfe850b802e"
style="aspect-ratio: 1.33;"
srcset="/images/patterns/diagrams/bridge/example-uk.png?id=cc59fa73f2b5640e0cf28bfe850b802e 640w,/images/patterns/diagrams/bridge/example-uk-1.5x.png?id=5a3f13322d9fe3fac91f934d99eb7d09 960w,/images/patterns/diagrams/bridge/example-uk-2x.png?id=6ae1d0674cd9403dee2bb2005aade4a5 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Структура класів прикладу патерна Міст" />
<figcaption><p>Приклад поділу двох ієрархій класів — приладів та
пультів керування.</p></figcaption>
</figure>

Клас пульта має посилання на об'єкт приладу, яким він керує. Пульти
працюють з приладами через загальний інтерфейс. Це дає можливість
зв'язати пульти з різними приладами.

Пульти можна розвивати незалежно від приладів. Для цього достатньо
створити новий підклас абстракції. Ви можете створити як простий пульт з
двома кнопками, так і більш складний пульт з тач-інтерфейсом.

Клієнтському коду залишається вибрати версію абстракції та реалізації, з
якими він хоче працювати, та зв'язати їх між собою.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Клас пультів має посилання на пристрій, яким керує. Методи
// цього класу делегують роботу методам пов&#39;язаного пристрою.
class Remote is
    protected field device: Device
    constructor Remote(device: Device) is
        this.device = device
    method togglePower() is
        if (device.isEnabled()) then
            device.disable()
        else
            device.enable()
    method volumeDown() is
        device.setVolume(device.getVolume() - 10)
    method volumeUp() is
        device.setVolume(device.getVolume() + 10)
    method channelDown() is
        device.setChannel(device.getChannel() - 1)
    method channelUp() is
        device.setChannel(device.getChannel() + 1)


// Ви можете розширювати клас пультів, не чіпаючи код пристроїв.
class AdvancedRemote extends Remote is
    method mute() is
        device.setVolume(0)


// Всі пристрої мають спільний інтерфейс, тому з ними може
// працювати будь-який пульт.
interface Device is
    method isEnabled()
    method enable()
    method disable()
    method getVolume()
    method setVolume(percent)
    method getChannel()
    method setChannel(channel)


// Разом з цим, кожен пристрій має особливу реалізацію.
class Tv implements Device is
    // ...

class Radio implements Device is
    // ...


// Десь у клієнтському програмному коді.
tv = new Tv()
remote = new Remote(tv)
remote.togglePower()

radio = new Radio()
remote = new AdvancedRemote(radio)</code></pre>
</figure>
:::

:::::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::::: applicability
::: applicability-problem
Якщо ви хочете розділити монолітний клас, який містить кілька різних
реалізацій якої-небудь функціональності (наприклад, якщо клас може
працювати з різними системами баз даних).
:::

::: applicability-solution
Чим більший клас, тим важче розібратись у його коді, і тим більше це
розтягує час розробки. Крім того, зміни, що вносяться в одну з
реалізацій, призводять до редагування всього класу, що може викликати
появу несподіваних помилок у коді.

Міст дозволяє розділити монолітний клас на кілька окремих ієрархій.
Після цього ви можете змінювати код в одній гілці класів незалежно від
іншої. Це спрощує роботу над кодом і зменшує ймовірність внесення
помилок.
:::

::: applicability-problem
Якщо клас потрібно розширювати в двох незалежних площинах.
:::

::: applicability-solution
Міст пропонує виділити одну з таких площин в окрему ієрархію класів,
зберігаючи посилання на один з її об'єктів у початковому класі.
:::

::: applicability-problem
Якщо ви хочете мати можливість змінювати реалізацію під час виконання
програми.
:::

::: applicability-solution
Міст дозволяє замінювати реалізацію навіть під час виконання програми,
оскільки конкретна реалізація не «зашита» в клас абстракції.

*До речі, через цей пункт Міст часто плутають із
[Стратегією](strategy.html). Зверніть увагу, що у Моста цей пункт займає
останнє місце за значущістю, оскільки його головна задача ---
структурна.*
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначте, чи існують у ваших класах два непересічних виміри. Це може
    бути функціональність/платформа, предметна область/інфраструктура,
    фронт-енд/бек-енд або інтерфейс/реалізація.

2.  Продумайте, які операції будуть потрібні клієнтам, і опишіть їх у
    базовому класі *абстракції*.

3.  Визначте поведінки, які доступні на всіх платформах, та виберіть з
    них ту частину, яка буде потрібна для абстракції. На підставі цього
    опишіть загальний інтерфейс *реалізації*.

4.  Для кожної платформи створіть власний клас конкретної реалізації.
    Всі вони повинні дотримуватися загального інтерфейсу, який ми
    виділили перед цим.

5.  Додайте до класу абстракції посилання на об'єкт реалізації.
    Реалізуйте методи абстракції, делегуючи основну роботу пов'язаному
    об'єкту реалізації.

6.  Якщо у вас є кілька варіацій абстракції, створіть для кожної з них
    власний підклас.

7.  Клієнт повинен подати об'єкт реалізації до конструктора абстракції,
    щоб зв'язати їх разом. Після цього він може вільно використовувати
    об'єкт абстракції, забувши про реалізацію.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Дозволяє будувати платформо-незалежні програми.
-    Приховує зайві або небезпечні деталі реалізації від клієнтського
    коду.
-    Реалізує *принцип відкритості/закритості*.
:::

::: col-sm-6
-    Ускладнює код програми внаслідок введення додаткових класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Міст](bridge.html) проектують заздалегідь, щоб розвивати великі
    частини програми окремо одну від одної. [Адаптер](adapter.html)
    застосовується постфактум, щоб змусити несумісні класи працювати
    разом.

-   [Міст](bridge.html), [Стратегія](strategy.html) та
    [Стан](state.html) (а також трохи і [Адаптер](adapter.html)) мають
    схожі структури класів --- усі вони побудовані за принципом
    «композиції», тобто делегування роботи іншим об'єктам. Проте вони
    відрізняються тим, що вирішують різні проблеми. Пам'ятайте, що
    патерни --- це не тільки рецепт побудови коду певним чином, але й
    описування проблем, які призвели до такого рішення.

-   [Абстрактна фабрика](abstract-factory.html) може працювати спільно з
    [Мостом](bridge.html). Це особливо корисно, якщо у вас є абстракції,
    які можуть працювати тільки з деякими реалізаціями. В цьому випадку
    фабрика визначатиме типи створюваних абстракцій та реалізацій.

-   Патерн [Будівельник](builder.html) може бути побудований у вигляді
    [Мосту](bridge.html): *директор* гратиме роль абстракції, а
    *будівельники* --- реалізації.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Міст на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](bridge/csharp/example.html "Міст на C#"){.prog-lang-link}
[![Міст на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](bridge/cpp/example.html "Міст на C++"){.prog-lang-link}
[![Міст на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](bridge/go/example.html "Міст на Go"){.prog-lang-link}
[![Міст на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](bridge/java/example.html "Міст на Java"){.prog-lang-link}
[![Міст на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](bridge/php/example.html "Міст на PHP"){.prog-lang-link}
[![Міст на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](bridge/python/example.html "Міст на Python"){.prog-lang-link}
[![Міст на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](bridge/ruby/example.html "Міст на Ruby"){.prog-lang-link}
[![Міст на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](bridge/rust/example.html "Міст на Rust"){.prog-lang-link}
[![Міст на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](bridge/swift/example.html "Міст на Swift"){.prog-lang-link}
[![Міст на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](bridge/typescript/example.html "Міст на TypeScript"){.prog-lang-link}
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

[Компонувальник []{.fa .fa-arrow-right}](composite.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Адаптер](adapter.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::
