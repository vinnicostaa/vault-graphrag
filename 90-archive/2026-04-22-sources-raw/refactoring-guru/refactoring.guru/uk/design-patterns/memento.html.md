::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Знімок {#знімок .title}

::: aka
Також відомий як: [Memento]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Знімок** --- це поведінковий патерн проектування, що дає змогу
зберігати та відновлювати минулий стан об'єктів, не розкриваючи
подробиць їхньої реалізації.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-uk4058.png?id=ea9c25c9a837da160961fe20e54329a8"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-uk.png?id=ea9c25c9a837da160961fe20e54329a8 640w,/images/patterns/content/memento/memento-uk-1.5x.png?id=965a78c52d5b0e5a9c51e0cdf5df056b 960w,/images/patterns/content/memento/memento-uk-2x.png?id=a0054f760635fa4b05b065538a406800 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Знімок" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Припустімо, ви пишете програму текстового редактора. Крім звичайного
редагування, ваш редактор дозволяє змінювати форматування тексту,
вставляти малюнки та інше.

В певний момент ви вирішили надати можливість скасовувати усі ці дії.
Для цього вам потрібно зберігати поточний стан редактора перед тим, як
виконати будь-яку дію. Якщо користувач вирішить скасувати свою дію, ви
візьмете копію стану з історії та відновите попередній стан редактора.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-ukabf3.png?id=72f8291e3e07fc661d896789fcff492c"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-uk.png?id=72f8291e3e07fc661d896789fcff492c 470w,/images/patterns/diagrams/memento/problem1-uk-1.5x.png?id=4f6bb49eba684caef77bdc04d5fca26c 705w,/images/patterns/diagrams/memento/problem1-uk-2x.png?id=d2e112c01456aeabcf57fbd7cb951505 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Перед виконанням команди ви можете зберегти копію стану редактора, щоб потім мати можливість скасувати операцію" />
<figcaption><p>Перед виконанням команди ви можете зберегти копію стану
редактора, щоб потім мати можливість
скасувати операцію.</p></figcaption>
</figure>

Щоб зробити копію стану об'єкта, достатньо скопіювати значення полів.
Таким чином, якщо ви зробили клас редактора достатньо відкритим, то
будь-який інший клас зможе зазирнути всередину, щоб скопіювати його
стан.

Здавалося б, які проблеми? Тепер будь-яка операція зможе зробити
резервну копію редактора перед виконанням своєї дії. Але такий наївний
підхід забезпечить вам безліч проблем у майбутньому. Адже, якщо ви
вирішите провести рефакторинг --- прибрати або додати кілька полів до
класу редактора --- доведеться змінювати код усіх класів, які могли
копіювати стан редактора.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-ukb14b.png?id=da3cc54f95321b626b8e8683f86ddd20"
style="aspect-ratio: 1.57;"
srcset="/images/patterns/diagrams/memento/problem2-uk.png?id=da3cc54f95321b626b8e8683f86ddd20 330w,/images/patterns/diagrams/memento/problem2-uk-1.5x.png?id=238c59b12038d0c9c5040ac50fddaa86 495w,/images/patterns/diagrams/memento/problem2-uk-2x.png?id=ae42d9349b2c4cf2455395a2cd58e23d 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Як команді створити знімок стану редактора, якщо всі його поля приватні?" />
<figcaption><p>Як команді створити знімок стану редактора, якщо всі його
поля приватні?</p></figcaption>
</figure>

Але це ще не все. Давайте тепер поглянемо безпосередньо на копії стану,
які ми створювали. З чого складається стан редактора? Навіть
найпримітивніший редактор повинен мати декілька полів для зберігання
поточного тексту, позиції курсора та прокручування екрану. Щоб зробити
копію стану, вам потрібно додати значення всіх цих полів до деякого
«контейнера».

Імовірно, вам знадобиться зберігати масу таких контейнерів в якості
історії операцій, тому зручніше за все зробити їх об'єктами одного
класу. Цей клас повинен мати багато полів, але практично жодного методу.
Щоб інші об'єкти могли записувати та читати з нього дані, вам доведеться
зробити його поля публічними. Проте це призведе до тієї ж проблеми, що й
з відкритим класом редактора. Інші класи стануть залежними від будь-яких
змін класу контейнера, який схильний до таких самих змін, що і клас
редактора.

Виходить, що нам доведеться або відкрити класи для всіх бажаючих,
отримавши постійний клопіт з підтримкою коду, або залишити класи
закритими, відмовившись від ідеї скасування операцій. Чи немає тут
альтернативи?
:::

::: {.section .solution}
##  Рішення {#solution}

Усі проблеми, описані вище, виникають через порушення інкапсуляції, коли
одні об'єкти намагаються зробити роботу за інших, проникаючи до їхньої
приватної зони, щоб зібрати необхідні для операції дані.

Патерн Знімок доручає створення копії стану об'єкта самому об'єкту, який
цим станом володіє. Замість того, щоб робити знімок «ззовні», наш
редактор сам зробить копію своїх полів, адже йому доступні всі поля,
навіть приватні.

Патерн пропонує тримати копію стану в спеціальному об'єкті-*знімку* з
обмеженим інтерфейсом, що дозволяє, наприклад, дізнатися дату
виготовлення або назву знімка. Проте, знімок повинен бути відкритим для
свого *творця* і дозволяти прочитати та відновити його внутрішній стан.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-ukcae2.png?id=b6e50ab9a1f6f897a4fdce97b1513dd2"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-uk.png?id=b6e50ab9a1f6f897a4fdce97b1513dd2 610w,/images/patterns/diagrams/memento/solution-uk-1.5x.png?id=f33883b31abaddf6fb8317acee708903 915w,/images/patterns/diagrams/memento/solution-uk-2x.png?id=ee1b538c0d46b7b20c9d017f624554a0 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Знімок повністю відкритий для творця, але лише частково відкритий для опікунів" />
<figcaption><p>Знімок повністю відкритий для творця, але лише частково
відкритий для опікунів.</p></figcaption>
</figure>

Така схема дозволяє творцям робити знімки та віддавати їх на зберігання
іншим об'єктам, що називаються *опікунами*. Опікунам буде доступний
тільки обмежений інтерфейс знімка, тому вони ніяк не зможуть вплинути на
«нутрощі» самого знімку. У потрібний момент опікун може попросити творця
відновити свій стан, передавши йому відповідний знімок.

У нашому прикладі з редактором опікуном можна зробити окремий клас, який
зберігатиме список виконаних операцій. Обмежений інтерфейс знімків
дозволить демонструвати користувачеві гарний список з назвами й датами
виконаних операцій. Коли ж користувач вирішить скасувати операцію, клас
історії візьме останній знімок зі стека та надішле його об'єкту
редактора для відновлення.
:::

:::::::::: {.section .structure-container}
##  Структура {#structure}

::::::::: structure
#### Класична реалізація на вкладених класах

Класична реалізація патерна покладається на механізм вкладених класів,
який доступний тільки в деяких мовах програмування (C++, C#, Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура класів патерна Знімок" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура класів патерна Знімок" />
</figure>
:::
::::

1.  **Творець** може створювати знімки свого стану, а також відтворювати
    минулий стан, якщо до нього подати готовий знімок.

2.  **Знімок** --- це простий об'єкт даних, який містить стан творця.
    Надійніше за все зробити об'єкти знімків незмінними, встановлюючи в
    них стан тільки через конструктор.

3.  **Опікун** повинен знати, коли робити знімок творця та коли його
    потрібно відновлювати.

    Опікун може зберігати історію минулих станів творця у вигляді стека
    знімків. Коли треба буде скасувати останню операцію, він візьме
    «верхній» знімок зі стеку та передасть його творцеві для
    відновлення.

4.  У даній реалізації знімок --- це внутрішній клас по відношенню до
    класу творця. Саме тому він має повний доступ до всіх полів та
    методів творця, навіть приватних. З іншого боку, опікун не має
    доступу ані до стану, ані до методів знімків, а може лише зберігати
    посилання на ці об'єкти.

#### Реалізація з проміжним порожнім інтерфейсом

Підходить для мов, що не мають механізму вкладених класів (наприклад,
PHP).

:::: memento-empty-interface
::: {.struct-image2 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure21ec9.png?id=fcff71cb648389be2e302fbe55e2f847"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2.png?id=fcff71cb648389be2e302fbe55e2f847 560w,/images/patterns/diagrams/memento/structure2-1.5x.png?id=5c05495a7ca6545fcc57f70ea7ee904a 840w,/images/patterns/diagrams/memento/structure2-2x.png?id=aa7fb5d0f622d4344a2cb590f437f8c8 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів патерна Знімок" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура класів патерна Знімок" />
</figure>
:::
::::

1.  У цій реалізації творець працює безпосередньо з конкретним класом
    знімка, а опікун --- тільки з його обмеженим інтерфейсом.

2.  Завдяки цьому досягається той самий ефект, що і в класичній
    реалізації. Творець має повний доступ до знімка, а опікун --- ні.

#### Знімки з підвищеним захистом

Якщо потрібно повністю виключити можливість доступу до стану творців та
знімків.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Знімок з підвищеним захистом" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Знімок з підвищеним захистом" />
</figure>
:::
::::

1.  Ця реалізація дозволяє мати кілька видів творців та знімків. Кожному
    класу творців відповідає власний клас знімків. Ані творці, ані
    знімки не дозволяють іншим об'єктам читати свій стан.

2.  Тут опікун ще жорсткіше обмежений у доступі до стану творців та
    знімків, але, з іншого боку, опікун стає незалежним від творців,
    оскільки метод відновлення тепер знаходиться в самих знімках.

3.  Знімки тепер пов'язані з тими творцями, з яких вони зроблені. Вони,
    як і раніше, отримують стан через конструктор. Завдяки близькому
    зв'язку між класами, знімки знають, як відновити стан своїх творців.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі патерн **Знімок** використовується спільно з патерном
[Команда](command.html) та дозволяє зберігати резервні копії складного
стану текстового редактора й відновлювати його за потреби.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура класів прикладу патерна Знімок" />
<figcaption><p>Приклад збереження знімків стану
текстового редактора.</p></figcaption>
</figure>

Об'єкти команд виступають в ролі опікунів і запитують знімки в редактора
перед тим, як виконати свою дію. Якщо знадобиться скасувати операцію,
команда зможе відновити стан редактора, використовуючи збережений
знімок.

При цьому знімок не має публічних полів, тому інші об'єкти не мають
доступу до його внутрішніх даних. Знімки пов'язані з певним редактором,
який їх створив. Вони ж і відновлюють стан свого редактора. Це дозволяє
програмі мати одночасно кілька об'єктів редакторів, наприклад, розбитих
по різних вкладках програми.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Клас творця повинен мати спеціальний метод, який зберігає
// стан об&#39;єкта в новому об&#39;єкті-знімку.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    method createSnapshot(): Snapshot is
        // Знімок — це незмінний об&#39;єкт, тому творець передає до
        // нього свій стан через параметри конструктора.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// Знімок зберігає минулий стан редактора.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // У потрібний момент власник знімку може відновити стан
    // редактора.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Опікуном може виступати клас команд (див. патерн Команда). У
// цьому випадку команда зберігає знімок стану об&#39;єкта-
// одержувача перед тим, як передати йому дію. А в разі
// скасування дії, команда поверне об&#39;єкт до попереднього стану.
class Command is
    private field backup: Snapshot

    method makeBackup() is
        backup = editor.createSnapshot()

    method undo() is
        if (backup != null)
            backup.restore()
    // ...</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Коли вам потрібно зберігати миттєві знімки стану об'єкта (або його
частини) для того, щоб об'єкт можна було відновити в тому самому стані.
:::

::: applicability-solution
Патерн Знімок дозволяє створювати будь-яку кількість знімків об'єкта і
зберігати їх незалежно від об'єкта, з якого роблять знімок. Знімки часто
використовують не тільки для реалізації операції скасування, але й для
транзакцій, коли стан об'єкта потрібно «відкотити», якщо операція не
була вдалою.
:::

::: applicability-problem
Коли пряме отримання стану об'єкта розкриває приватні деталі його
реалізації, порушуючи інкапсуляцію.
:::

::: applicability-solution
Патерн пропонує виготовити знімок саме вихідному об'єкту, тому що йому
доступні всі поля, навіть приватні.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Визначте клас творця, об'єкти якого повинні створювати знімки свого
    стану.

2.  Створіть клас знімка та опишіть в ньому ті ж самі поля, які є в
    оригінальному класі-творці.

3.  Зробіть об'єкти знімків незмінними. Вони повинні одержувати
    початкові значення тільки один раз, через власний конструктор.

4.  Якщо ваша мова програмування це дозволяє, зробіть клас знімка
    вкладеним у клас творця.

    Якщо ні, вийміть з класу знімка порожній інтерфейс, який буде
    доступним іншим об'єктам програми. Згодом ви можете додати до цього
    інтерфейсу деякі допоміжні методи, що дають доступ до метаданих
    знімка, але прямий доступ до даних творця повинен бути виключеним.

5.  Додайте до класу творця метод одержання знімків. Творець повинен
    створювати нові об'єкти знімків, передаючи значення своїх полів
    через конструктор.

    Сигнатура методу повинна повертати знімки через обмежений інтерфейс,
    якщо він у вас є. Сам клас повинен працювати з конкретним класом
    знімка.

6.  Додайте до класу творця метод відновлення зі знімка. Щодо прив'язки
    до типів, керуйтеся тією ж логікою, що і в пункті 4.

7.  Опікуни, незалежно від того, чи це історія операцій, чи об'єкти
    команд, чи щось інше, повинні знати про те, коли запитувати знімки у
    творця, де їх зберігати та коли відновлювати.

8.  Зв'язок опікунів з творцями можна перенести всередину знімків. У
    цьому випадку кожен знімок буде прив'язаний до свого творця і
    повинен буде сам відновлювати його стан. Але це працюватиме або якщо
    класи знімків вкладені до класів творців, або якщо творці мають
    відповідні сетери для встановлення значень своїх полів.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Не порушує інкапсуляцію вихідного об'єкта.
-    Спрощує структуру вихідного об'єкта. Йому не потрібно зберігати
    історію версій свого стану.
:::

::: col-sm-6
-    Вимагає багато пам'яті, якщо клієнти дуже часто створюють знімки.
-    Може спричинити додаткові витрати пам'яті, якщо об'єкти, що
    зберігають історію, не звільняють ресурси, зайняті застарілими
    знімками.
-    В деяких мовах (наприклад, PHP, Python, JavaScript) складно
    гарантувати, щоб лише вихідний об'єкт мав доступ до стану знімка.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Команду](command.html) та [Знімок](memento.html) можна
    використовувати спільно для реалізації скасування операцій. У цьому
    випадку об'єкти команд відповідатимуть за виконання дії над
    об'єктом, а знімки зберігатимуть резервну копію стану цього об'єкта,
    зроблену перед запуском команди.

-   [Знімок](memento.html) можна використовувати разом з
    [Ітератором](iterator.html), щоб зберегти поточний стан обходу
    структури даних та повернутися до нього в майбутньому, якщо буде
    потрібно.

-   [Знімок](memento.html) іноді можна замінити
    [Прототипом](prototype.html), якщо об'єкт, чий стан потрібно
    зберігати в історії, досить простий, не має посилань на зовнішні
    ресурси або їх можна легко відновити.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Знімок на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Знімок на C#"){.prog-lang-link}
[![Знімок на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Знімок на C++"){.prog-lang-link}
[![Знімок на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Знімок на Go"){.prog-lang-link}
[![Знімок на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Знімок на Java"){.prog-lang-link}
[![Знімок на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Знімок на PHP"){.prog-lang-link}
[![Знімок на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Знімок на Python"){.prog-lang-link}
[![Знімок на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Знімок на Ruby"){.prog-lang-link}
[![Знімок на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Знімок на Rust"){.prog-lang-link}
[![Знімок на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Знімок на Swift"){.prog-lang-link}
[![Знімок на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Знімок на TypeScript"){.prog-lang-link}
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

[Спостерігач []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Посередник](mediator.html){.btn .btn-default
rel="prev"}
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
