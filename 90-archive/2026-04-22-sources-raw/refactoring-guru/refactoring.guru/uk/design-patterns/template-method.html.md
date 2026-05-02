::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Поведінкові
патерни](behavioral-patterns.html){.type}
:::

# Шаблонний метод {#шаблонний-метод .title}

::: aka
Також відомий як: [Template Method]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Шаблонний метод** --- це поведінковий патерн проектування, який
визначає кістяк алгоритму, перекладаючи відповідальність за деякі його
кроки на підкласи. Патерн дозволяє підкласам перевизначати кроки
алгоритму, не змінюючи його загальної структури.

<figure class="image">
<img
src="../../images/patterns/content/template-method/template-method4c1b.png?id=eee9461742f832814f19612ccf472819"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/template-method/template-method.png?id=eee9461742f832814f19612ccf472819 640w,/images/patterns/content/template-method/template-method-1.5x.png?id=2b4c179aaa02f5c45a59324b904cd130 960w,/images/patterns/content/template-method/template-method-2x.png?id=4e164dc41be4dcfa122628864c2be210 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Шаблонний метод" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Ви пишете програму для дата-майнінгу в офісних документах. Користувачі
завантажуватимуть до неї документи різних форматів (PDF, DOC, CSV), а
програма повинна видобути з них корисну інформацію.

У першій версії ви обмежилися обробкою тільки DOC файлів. У наступній
версії додали підтримку CSV. А через місяць «прикрутили» роботу з PDF
документами.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/problem6793.png?id=60fa4f735c467ac1c9438231a1782807"
style="aspect-ratio: 1.35;"
srcset="/images/patterns/diagrams/template-method/problem.png?id=60fa4f735c467ac1c9438231a1782807 620w,/images/patterns/diagrams/template-method/problem-1.5x.png?id=83fa009f7727bdcc69335b946919f0d8 930w,/images/patterns/diagrams/template-method/problem-2x.png?id=fc8b434afec7b6135043d0d2f48189f0 1240w"
sizes="(max-width: 720px) 100vw, 620px" loading="lazy" width="620"
alt="Класи дата-майнінгу містять багато дублювань" />
<figcaption><p>Класи дата-майнінгу містять
багато дублювань.</p></figcaption>
</figure>

В якийсь момент ви помітили, що код усіх трьох класів обробки документів
хоч і відрізняється в частині роботи з файлами, але містить досить
багато спільного в частині самого видобування даних. Було б добре
позбутися від повторної реалізації алгоритму видобування даних у кожному
з класів.

До того ж інший код, який працює з об'єктами цих класів, наповнений
умовами, що перевіряють тип обробника перед початком роботи. Весь цей
код можна спростити, якщо злити всі три класи в одне ціле або звести їх
до загального інтерфейсу.
:::

::: {.section .solution}
##  Рішення {#solution}

Патерн Шаблонний метод пропонує розбити алгоритм на послідовність
кроків, описати ці кроки в окремих методах і викликати їх в одному
*шаблонному* методі один за одним.

Це дозволить підкласам перевизначити деякі кроки алгоритму, залишаючи
без змін його структуру та інші кроки, які для цього підкласу не є
важливими.

У нашому прикладі з дата-майнінгом ми можемо створити загальний базовий
клас для всіх трьох алгоритмів. Цей клас складатиметься з шаблонного
методу, який послідовно викликає кроки розбору документів.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/solution-ukf72b.png?id=2276d19720a393e5cbe2935d1e599b08"
style="aspect-ratio: 1.43;"
srcset="/images/patterns/diagrams/template-method/solution-uk.png?id=2276d19720a393e5cbe2935d1e599b08 600w,/images/patterns/diagrams/template-method/solution-uk-1.5x.png?id=02a15a6eb00d3191afa57552d6c0afe0 900w,/images/patterns/diagrams/template-method/solution-uk-2x.png?id=10efaf60938ecf15fb1bb03142160789 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Шаблонний метод містить виклики методів-кроків" />
<figcaption><p>Шаблонний метод розбиває алгоритм на кроки, дозволяючи
підкласами перевизначити деякі з них.</p></figcaption>
</figure>

Для початку кроки шаблонного методу можна зробити абстрактними. З цієї
причини усі підкласи повинні будуть реалізувати кожен з кроків
по-своєму. В нашому випадку всі підкласи вже містять реалізацію кожного
з кроків, тому додатково нічого робити не потрібно.

Справді важливим є наступний етап. Тепер ми можемо визначити спільну
поведінку для всіх трьох класів і винести її до суперкласу. У нашому
прикладі кроки відкривання та закривання документів відрізнятимуться для
всіх підкласів, тому залишаться абстрактними. З іншого боку, код обробки
даних, однаковий для всіх типів документів, переїде до базового класу.

Як бачите, у нас з'явилося два типа кроків: *абстрактні*, що кожен
підклас обов'язково має реалізувати, а також кроки *з типовою
реалізацією*, які можна перевизначити в підкласах, але це не
обов'язково.

Але є ще й третій тип кроків --- *хуки*. Це опціональні кроки, які
виглядають як звичайні методи, але взагалі не містять коду. Шаблонний
метод залишиться робочим, навіть якщо жоден підклас не перевизначить
такий хук. Підсумовуючи сказане, хук дає підкласам додаткові точки
«вклинювання» в хід шаблонного методу.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/live-exampleea7a.png?id=2485d52852f87da06c9cc0e2fd257d6a"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/template-method/live-example.png?id=2485d52852f87da06c9cc0e2fd257d6a 590w,/images/patterns/diagrams/template-method/live-example-1.5x.png?id=92c5fd9345329da09e3233302158678c 885w,/images/patterns/diagrams/template-method/live-example-2x.png?id=89083a3dcd1fe2b627b9b6e6ff4986dc 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Будівництво типових будинків" />
<figcaption><p>Проект типового будинку можуть трохи змінити за
бажанням клієнта.</p></figcaption>
</figure>

Під час будівництва типових будинків будівельники використовують підхід,
схожий на шаблонний метод. У них є основний архітектурний проект, в
якому розписані кроки будівництва: заливка фундаменту, витягування стін,
покриття даху, встановлення вікон тощо.

Але, незважаючи на стандартизацію кожного етапу, будівельники можуть
робити невеликі зміни на кожному з етапів, щоб зробити будинок трішечки
не схожим на інші.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/structure1c3b.png?id=924692f994bff6578d8408d90f6fc459"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.89;"
srcset="/images/patterns/diagrams/template-method/structure.png?id=924692f994bff6578d8408d90f6fc459 340w,/images/patterns/diagrams/template-method/structure-1.5x.png?id=613d78ad8ec08536ec70f4e0976b5a1a 510w,/images/patterns/diagrams/template-method/structure-2x.png?id=25082d6d6a76f51c6b64d8aeeaffdbb5 680w"
sizes="(max-width: 720px) 100vw, 340px" loading="lazy" width="340"
alt="Структура класів патерна Шаблонний Метод" /><img
src="../../images/patterns/diagrams/template-method/structure-indexedb94b.png?id=4ced6107519bc66710d2f05c0f4097a1"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.92;"
srcset="/images/patterns/diagrams/template-method/structure-indexed.png?id=4ced6107519bc66710d2f05c0f4097a1 350w,/images/patterns/diagrams/template-method/structure-indexed-1.5x.png?id=58e3a9092786c824eb738a7771702093 525w,/images/patterns/diagrams/template-method/structure-indexed-2x.png?id=86f28789cdcc5a4c415d6a1100de56fc 700w"
sizes="(max-width: 720px) 100vw, 350px" loading="lazy" width="350"
alt="Структура класів патерна Шаблонний Метод" />
</figure>
:::

1.  **Абстрактний клас** визначає кроки алгоритму й містить шаблонний
    метод, що складається з викликів цих кроків. Кроки можуть бути як
    абстрактними, так і містити реалізацію за замовчуванням.

2.  **Конкретний клас** перевизначає деякі або всі кроки алгоритму.
    Конкретні класи не перевизначають сам шаблонний метод.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Шаблонний метод** використовується як заготовка для
стандартного штучного інтелекту в простій стратегічній грі. Для введення
в гру нової раси достатньо створити підклас і реалізувати в ньому
відсутні методи.

<figure class="image">
<img
src="../../images/patterns/diagrams/template-method/example32c6.png?id=c0ce5cc8070925a1cd345fac6afa16b6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/template-method/example.png?id=c0ce5cc8070925a1cd345fac6afa16b6 430w,/images/patterns/diagrams/template-method/example-1.5x.png?id=d85c6b81c900f46d55688406170600bc 645w,/images/patterns/diagrams/template-method/example-2x.png?id=d8b309659c4b722237ec97733da935bd 860w"
sizes="(max-width: 720px) 100vw, 430px" loading="lazy" width="430"
alt="Структура класів прикладу патерна Шаблонний метод" />
<figcaption><p>Приклад класів штучного інтелекту для
простої гри.</p></figcaption>
</figure>

Всі раси гри матимуть приблизно однакові типи юнітів та будівель, тому
структура штучного інтелекту буде однаковою. Але різні раси можуть
різним шляхом реалізувати ці кроки. Так, наприклад, орки будуть
агресивнішими в атаці, люди більш активними в захисті, а дикі монстри
взагалі не будуть займатися будівництвом.

<figure class="code">
<pre class="code" lang="pseudocode"><code>class GameAI is
    // Шаблонний метод повинен бути заданий у базовому класі.
    // Він складається з викликів методів у певному порядку.
    // Здебільшого, ці методи є кроками якогось алгоритму.
    method turn() is
        collectResources()
        buildStructures()
        buildUnits()
        attack()

    // Деякі з цих методів можуть бути реалізовані безпосередньо
    // у базовому класі.
    method collectResources() is
        foreach (s in this.builtStructures) do
            s.collect()

    // А деякі можуть бути повністю абстрактними.
    abstract method buildStructures()
    abstract method buildUnits()

    // До речі, клас може мати більше одного шаблонного методу.
    method attack() is
        enemy = closestEnemy()
        if (enemy == null)
            sendScouts(map.center)
        else
            sendWarriors(enemy.position)

    abstract method sendScouts(position)
    abstract method sendWarriors(position)

// Підкласи можуть надавати свою реалізацію кроків алгоритму, не
// змінюючи сам шаблонний метод.
class OrcsAI extends GameAI is
    method buildStructures() is
        if (there are some resources) then
            // Будувати ферми, потім бараки, а потім цитадель.

    method buildUnits() is
        if (there are plenty of resources) then
            if (there are no scouts)
                // Побудувати раба, додати до групи розвідників.
            else
                // Побудувати піхотинця, додати до групи воїнів.

    // ...

    method sendScouts(position) is
        if (scouts.length &gt; 0) then
            // Відправити розвідників на позицію.

    method sendWarriors(position) is
        if (warriors.length &gt; 5) then
            // Відправити воїнів на позицію.

// Підкласи можуть не тільки реалізовувати абстрактні кроки, але
// й перевизначати кроки, вже реалізовані в базовому класі.
class MonstersAI extends GameAI is
    method collectResources() is
        // Нічого не робити.

    method buildStructures() is
        // Нічого не робити.

    method buildUnits() is
        // Нічого не робити.</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо підкласи повинні розширювати базовий алгоритм, не змінюючи його
структури.
:::

::: applicability-solution
Шаблонний метод дозволяє підкласами розширювати певні кроки алгоритму
через спадкування, не змінюючи при цьому структуру алгоритмів, оголошену
в базовому класі.
:::

::: applicability-problem
Якщо у вас є кілька класів, які роблять одне й те саме з незначними
відмінностями. Якщо ви редагуєте один клас, тоді доводиться вносити такі
ж виправлення до інших класів.
:::

::: applicability-solution
Патерн шаблонний метод пропонує створити для схожих класів спільний
суперклас та оформити в ньому головний алгоритм у вигляді кроків. Кроки,
які відрізняються, можна перевизначити у підкласах.

Це дозволить прибрати дублювання коду в кількох класах, які
відрізняються деталями, але мають схожу поведінку.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Вивчіть алгоритм і подумайте, чи можна його розбити на кроки.
    Вирішіть, які кроки будуть стандартними для всіх варіацій алгоритму,
    а які можуть бути змінюваними.

2.  Створіть абстрактний базовий клас. Визначте в ньому шаблонний метод.
    Цей метод повинен складатися з викликів кроків алгоритму. Є сенс у
    тому, щоб зробити шаблонний метод фінальним, аби підкласи не могли
    перевизначити його (якщо ваша мова програмування це дозволяє).

3.  Додайте до абстрактного класу методи для кожного з кроків алгоритму.
    Ви можете зробити ці методи абстрактними або додати якусь типову
    реалізацію. У першому випадку всі підкласи *повинні* будуть
    реалізувати ці методи, а в другому --- тільки якщо реалізація кроку
    в підкласі відрізняється від стандартної версії.

4.  Подумайте про введення хуків в алгоритм. Найчастіше хуки
    розташовують між основними кроками алгоритму, а також до та після
    всіх кроків.

5.  Створіть конкретні класи, успадкувавши їх від абстрактного класу.
    Реалізуйте в них всі кроки та хуки, яких не вистачає.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Полегшує повторне використання коду.
:::

::: col-sm-6
-    Ви жорстко обмежені скелетом існуючого алгоритму.
-    Ви можете порушити *принцип підстановки Барбари Лісков*, змінюючи
    базову поведінку одного з кроків алгоритму через підклас.
-    У міру зростання кількості кроків шаблонний метод стає занадто
    складно підтримувати.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Фабричний метод](factory-method.html) можна розглядати як окремий
    випадок [Шаблонного методу](template-method.html). Крім того,
    *Фабричний метод* нерідко буває частиною великого класу з
    *Шаблонними методами*.

-   [Шаблонний метод](template-method.html) використовує спадкування,
    щоб розширювати частини алгоритму. [Стратегія](strategy.html)
    використовує делегування, щоб змінювати «на льоту» алгоритми, що
    виконуються. *Шаблонний метод* працює на рівні класів. *Стратегія*
    дозволяє змінювати логіку окремих об'єктів.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Шаблонний метод на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](template-method/csharp/example.html "Шаблонний метод на C#"){.prog-lang-link}
[![Шаблонний метод на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](template-method/cpp/example.html "Шаблонний метод на C++"){.prog-lang-link}
[![Шаблонний метод на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](template-method/go/example.html "Шаблонний метод на Go"){.prog-lang-link}
[![Шаблонний метод на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](template-method/java/example.html "Шаблонний метод на Java"){.prog-lang-link}
[![Шаблонний метод на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](template-method/php/example.html "Шаблонний метод на PHP"){.prog-lang-link}
[![Шаблонний метод на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](template-method/python/example.html "Шаблонний метод на Python"){.prog-lang-link}
[![Шаблонний метод на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](template-method/ruby/example.html "Шаблонний метод на Ruby"){.prog-lang-link}
[![Шаблонний метод на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](template-method/rust/example.html "Шаблонний метод на Rust"){.prog-lang-link}
[![Шаблонний метод на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](template-method/swift/example.html "Шаблонний метод на Swift"){.prog-lang-link}
[![Шаблонний метод на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](template-method/typescript/example.html "Шаблонний метод на TypeScript"){.prog-lang-link}
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

[Відвідувач []{.fa .fa-arrow-right}](visitor.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Стратегія](strategy.html){.btn .btn-default
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
