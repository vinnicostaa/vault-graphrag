:::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Легковаговик {#легковаговик .title}

::: aka
Також відомий як:
[Пристосуванець, ]{style="display:inline-block"}[Кеш, ]{style="display:inline-block"}[Flyweight]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Легковаговик** --- це структурний патерн проектування, що дає змогу
вмістити більшу кількість об'єктів у відведеній оперативній пам'яті.
Легковаговик заощаджує пам'ять, розподіляючи спільний стан об'єктів між
собою, замість зберігання однакових даних у кожному об'єкті.

<figure class="image">
<img
src="../../images/patterns/content/flyweight/flyweight7be2.png?id=e34fbacb847dd609b5e68aaf252c4db0"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/flyweight/flyweight.png?id=e34fbacb847dd609b5e68aaf252c4db0 640w,/images/patterns/content/flyweight/flyweight-1.5x.png?id=b32df2297473b8a7577e1900e57325ac 960w,/images/patterns/content/flyweight/flyweight-2x.png?id=6a8f17d9550c75c3d648a605c4d31b45 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Легковаговик" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

На дозвіллі ви вирішили написати невелику гру, в якій гравці
переміщуються по карті та стріляють один в одного. Фішкою гри повинна
була стати реалістична система частинок. Кулі, снаряди, уламки від
вибухів --- все це повинно реалістично літати та гарно виглядати.

Гра відмінно працювала на вашому потужному комп'ютері, проте ваш друг
повідомив, що гра починає гальмувати й вилітає через кілька хвилин після
запуску. Передивившись логи, ви виявили, що гра вилітає через нестачу
оперативної пам'яті. У вашого друга комп'ютер значно менше «прокачаний»,
тому проблема в нього й проявляється так швидко.

Дійсно, кожна частинка у грі представлена власним об'єктом, що має
безліч даних. У певний момент, коли побоїще на екрані досягає
кульмінації, оперативна пам'ять комп'ютера вже не може вмістити нові
об'єкти частинок, і програма «вилітає».

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/problem-uk571c.png?id=8ed58e43c5969ddcf9ee2f5c20d788aa"
style="aspect-ratio: 2.54;"
srcset="/images/patterns/diagrams/flyweight/problem-uk.png?id=8ed58e43c5969ddcf9ee2f5c20d788aa 660w,/images/patterns/diagrams/flyweight/problem-uk-1.5x.png?id=cca5e423a26a90f313138dd096ced118 990w,/images/patterns/diagrams/flyweight/problem-uk-2x.png?id=52e5824abf22a1c20b6b8be81adb9192 1320w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="660"
alt="Проблема патерна Легковаговик" />
</figure>
:::

::: {.section .solution}
##  Рішення {#solution}

Якщо уважно подивитися на клас частинок, то можна помітити, що колір і
спрайт займають найбільше пам'яті. Більше того, ці поля зберігаються в
кожному об'єкті, хоча фактично їхні значення є однаковими для більшості
частинок.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution1-ukc558.png?id=cd2be8684617f2c736a6b02b23548bf6"
style="aspect-ratio: 2.06;"
srcset="/images/patterns/diagrams/flyweight/solution1-uk.png?id=cd2be8684617f2c736a6b02b23548bf6 640w,/images/patterns/diagrams/flyweight/solution1-uk-1.5x.png?id=d7a24e74d014d0cd2c0ef1ce628e9736 960w,/images/patterns/diagrams/flyweight/solution1-uk-2x.png?id=3a45cc4aa89303526a56eee0d06c9e69 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Рішення патерна Легковаговик" />
</figure>

Інший стан об'єктів --- координати, вектор руху й швидкість
відрізняються для всіх частинок. Таким чином, ці поля можна розглядати
як контекст, у якому використовується частинка, а колір і спрайт --- це
дані, що не змінюються в часі.

Незмінні дані об'єкта прийнято називати «внутрішнім станом». Всі інші
дані --- це «зовнішній стан».

Патерн Легковаговик пропонує не зберігати зовнішній стан у класі, а
передавати його до тих чи інших методів через параметри. Таким чином,
одні і ті самі об'єкти можна буде повторно використовувати в різних
контекстах. Головна ж перевага в тому, що тепер знадобиться набагато
менше об'єктів, адже вони тепер відрізнятимуться тільки внутрішнім
станом, а він не має так багато варіацій.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution3-uk30e9.png?id=8a180f7ec65ef23a537fbcc0dfef0b55"
style="aspect-ratio: 0.76;"
srcset="/images/patterns/diagrams/flyweight/solution3-uk.png?id=8a180f7ec65ef23a537fbcc0dfef0b55 424w,/images/patterns/diagrams/flyweight/solution3-uk-1.5x.png?id=9a9d2ca333c9fcfa3d19b08d089b03d8 636w,/images/patterns/diagrams/flyweight/solution3-uk-2x.png?id=f5a0b0115128696b684876d74c9b90ac 848w"
sizes="(max-width: 720px) 100vw, 424px" loading="lazy" width="424"
alt="Рішення патерна Легковаговик" />
</figure>

У нашому прикладі з частинками достатньо буде залишити лише три об'єкти,
що відрізняються спрайтами і кольором --- для куль, снарядів та уламків.
Нескладно здогадатися, що такі полегшені об'єкти називають
*легковаговиками* []{.pop-i}[Назва прийшла з боксу і означає вагову
категорію до 50 кг.]{.pop-content}.

#### Сховище зовнішнього стану

Але куди переїде зовнішній стан? Адже хтось повинен його зберігати.
Найчастіше його переміщують до контейнера, який керував об'єктами до
застосування патерна.

В нашому випадку це був головний клас гри. Ви могли б додати до його
класу поля-масиви для зберігання координат, векторів і швидкостей
частинок. Крім цього, потрібен буде ще один масив для зберігання
посилань на об'єкти-легковаговики, що відповідають тій чи іншій
частинці.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/solution2-ukd565.png?id=b78f85d98c710e1e6b84a588f8c6facc"
style="aspect-ratio: 1.94;"
srcset="/images/patterns/diagrams/flyweight/solution2-uk.png?id=b78f85d98c710e1e6b84a588f8c6facc 640w,/images/patterns/diagrams/flyweight/solution2-uk-1.5x.png?id=42faf6566029a61cc4c3a9dc51e66617 960w,/images/patterns/diagrams/flyweight/solution2-uk-2x.png?id=501c99a8ee598a09a229eb46e9a14718 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Рішення патерна Легковаговик" />
</figure>

Більш елегантним рішенням було б створити додатковий клас-контекст, який
пов'язував би зовнішній стан з тим чи іншим легковаговиком. Це дозволить
обійтися тільки одним полем-масивом у класі контейнера.

«Але стривайте, нам буде потрібно стільки ж цих об'єктів, скільки було
на самому початку!» --- скажете ви і будете праві! Але річ у тім, що
об'єкти-контексти займають набагато менше місця, ніж початкові. Адже
найважчі поля залишилися всередині легковаговика (вибачте за каламбур),
і зараз ми будемо посилатися на ці об'єкти з контекстів, замість того,
щоб повторно зберігати стан, що дублюється.

#### Незмінність Легковаговиків

Оскільки об'єкти легковаговиків будуть використані в різних контекстах,
ви повинні бути впевненими в тому, що їхній стан неможливо змінити після
створення. Весь внутрішній стан легковаговик повинен отримувати через
параметри конструктора. Він не повинен мати сеттерів і публічних полів.

#### Фабрика Легковаговиків

Для зручності роботи з легковаговиками і контекстами можна створити
фабричний метод, що приймає в параметрах увесь внутрішній (іноді й
зовнішній) стан бажаного об'єкта.

Найбільша користь цього методу в тому, щоб знаходити вже створених
легковаговиків з таким самим внутрішнім станом, як потрібно. Якщо
легковаговик знаходиться, його можна повторно використовувати. Якщо
немає --- просто створюємо новий.

Зазвичай цей метод додають до контейнера легковаговиків або створюють
окремий клас-фабрику. Його навіть можна зробити статичним і розмістити в
класі легковаговиків.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/structure7b00.png?id=c1e7e1748f957a4792822f902bc1d420"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.64;"
srcset="/images/patterns/diagrams/flyweight/structure.png?id=c1e7e1748f957a4792822f902bc1d420 640w,/images/patterns/diagrams/flyweight/structure-1.5x.png?id=34cf08f6c2b09dcd1c521c7512cc52b6 960w,/images/patterns/diagrams/flyweight/structure-2x.png?id=a7c8347421bde16435fc90a706f5dd34 1280w"
sizes="(max-width: 720px) 100vw, 640px" loading="lazy" width="640"
alt="Патерн Легковаговик (Пристосуванець)" /><img
src="../../images/patterns/diagrams/flyweight/structure-indexed086a.png?id=aa490792baa26b04464dacbc1f4a19cd"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/structure-indexed.png?id=aa490792baa26b04464dacbc1f4a19cd 630w,/images/patterns/diagrams/flyweight/structure-indexed-1.5x.png?id=b22a5eab6aacbbd03c66c34802b240ca 945w,/images/patterns/diagrams/flyweight/structure-indexed-2x.png?id=205e2f7d08b4ac0695f445a9db8989c4 1260w"
sizes="(max-width: 720px) 100vw, 630px" loading="lazy" width="630"
alt="Патерн Легковаговик (Пристосуванець)" />
</figure>
:::

1.  Ви завжди повинні пам'ятати про те, що легковаговик застосовується в
    програмі, яка має величезну кількість однакових об'єктів. Цих
    об'єктів повинно бути так багато, щоб вони не вміщалися в доступній
    оперативній пам'яті без додаткових хитрощів. Патерн розділяє дані
    цих об'єктів на дві частини --- легковаговики та контексти.

2.  **Легковаговик** містить стан, який повторювався в багатьох
    первинних об'єктах. Один і той самий легковаговик може
    використовуватись у зв'язці з безліччю контекстів. Стан, що
    зберігається тут, називається *внутрішнім*, а той, який він отримує
    ззовні, --- *зовнішнім*.

3.  **Контекст** містить «зовнішню» частину стану, унікальну для кожного
    об'єкта. Контекст пов'язаний з одним з об'єктів-легковаговиків, що
    зберігають стан, який залишився.

4.  Поведінку оригінального об'єкта найчастіше залишають у
    легковаговику, передаючи значення контексту через параметри методів.
    Тим не менше, поведінку можна розмістити й в контексті,
    використовуючи легковаговик як об'єкт даних.

5.  **Клієнт** обчислює або зберігає контекст, тобто зовнішній стан
    легковаговиків. Для клієнта легковаговики виглядають як шаблонні
    об'єкти, які можна налаштувати під час використання, передавши
    контекст через параметри.

6.  **Фабрика легковаговиків** керує створенням і повторним
    використанням легковаговиків. Фабрика отримує запити, в яких
    зазначено бажаний стан легковаговика. Якщо легковаговик з таким
    станом вже створений, фабрика відразу його повертає, а якщо ні ---
    створює новий об'єкт.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Легковаговик** допомагає заощадити оперативну пам'ять
при відображенні на екрані мільйонів об'єктів-дерев.

<figure class="image">
<img
src="../../images/patterns/diagrams/flyweight/example5053.png?id=0818d078c1a79f373e96397f37b7ee06"
style="aspect-ratio: 1.54;"
srcset="/images/patterns/diagrams/flyweight/example.png?id=0818d078c1a79f373e96397f37b7ee06 540w,/images/patterns/diagrams/flyweight/example-1.5x.png?id=449fa5906e277c134870c07e33dd4b62 810w,/images/patterns/diagrams/flyweight/example-2x.png?id=9423640fe3688a64201389b6e7aa1f48 1080w"
sizes="(max-width: 720px) 100vw, 540px" loading="lazy" width="540"
alt="Структура класів прикладу патерна Легковаговик" />
</figure>

Легковаговик виділяє повторювану частину стану з основного класу `Tree`
і розміщує його в додатковому класі `TreeType`.

Тепер, замість зберігання повторюваних даних в усіх об'єктах, окремі
дерева будуть посилатися на кілька спільних об'єктів, що зберігають ці
дані. Клієнт працює з деревами через фабрику дерев, яка приховує від
нього складність кешування спільних даних дерев.

Таким чином, програма буде використовувати набагато менше оперативної
пам'яті, що дозволить намалювати на екрані більше дерев, використовуючи
те ж саме «залізо».

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Цей клас-легковаговик містить лише частину полів, які
// описують дерева. На відміну, наприклад, від координат, ці
// поля не є унікальними для кожного дерева, оскільки декілька
// дерев можуть мати такий самий колір чи текстуру. Тому ми
// переносимо повторювані дані до одного єдиного об&#39;єкта й
// посилаємося на нього з множини окремих дерев.
class TreeType is
    field name
    field color
    field texture
    constructor TreeType(name, color, texture) { ... }
    method draw(canvas, x, y) is
        // 1. Створити зображення даного типу, кольору й
        // текстури.
        // 2. Відобразити його на полотні в позиції X, Y.

// Фабрика легковаговиків вирішує, коли потрібно створити нового
// легковаговика, а коли можна обійтися існуючим.
class TreeFactory is
    static field treeTypes: collection of tree types
    static method getTreeType(name, color, texture) is
        type = treeTypes.find(name, color, texture)
        if (type == null)
            type = new TreeType(name, color, texture)
            treeTypes.add(type)
        return type

// Контекстний об&#39;єкт, з якого ми виділили легковаговик
// TreeType. У програмі можуть бути тисячі об&#39;єктів Tree,
// оскільки накладні витрати на їхнє зберігання зовсім
// невеликі — в пам&#39;яті треба зберігати лише три цілих числа
// (дві координати й посилання).
class Tree is
    field x,y
    field type: TreeType
    constructor Tree(x, y, type) { ... }
    method draw(canvas) is
        type.draw(canvas, this.x, this.y)

// Класи Tree і Forest є клієнтами Легковаговика. За умови, що
// надалі вам не потрібно розширювати клас дерев, їх можна злити
// докупи.
class Forest is
    field trees: collection of Trees

    method plantTree(x, y, name, color, texture) is
        type = TreeFactory.getTreeType(name, color, texture)
        tree = new Tree(x, y, type)
        trees.add(tree)

    method draw(canvas) is
        foreach (tree in trees) do
            tree.draw(canvas)</code></pre>
</figure>
:::

:::::: {.section .applicability-container}
##  Застосування {#applicability}

::::: applicability
::: applicability-problem
Якщо не вистачає оперативної пам'яті для підтримки всіх потрібних
об'єктів.
:::

::: applicability-solution
Ефективність патерна **Легковаговик** багато в чому залежить від того,
як і де він використовується. Застосовуйте цей патерн у випадках, коли
виконано всі перераховані умови:

-   у програмі використовується велика кількість об'єктів;
-   через це високі витрати оперативної пам'яті;
-   більшу частину стану об'єктів можна винести за межі їхніх класів;
-   великі групи об'єктів можна замінити невеликою кількістю об'єктів,
    що розділяються, оскільки зовнішній стан винесено.
:::
:::::
::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Розділіть поля класу, який стане легковаговиком, на дві частини:

    -   внутрішній стан: значення цих полів однакові для великої
        кількості об'єктів.
    -   зовнішній стан (контекст): значення полів унікальні для кожного
        об'єкта.

2.  Залишіть поля внутрішнього стану в класі, але переконайтеся, що їхні
    значення неможливо змінити. Ці поля повинні ініціалізуватись тільки
    через конструктор.

3.  Перетворіть поля зовнішнього стану на параметри методів, у яких ці
    поля використовувалися. Потім видаліть поля з класу.

4.  Створіть фабрику, яка буде кешувати та повторно віддавати вже
    створені об'єкти. Клієнт повинен отримувати легковаговика з певним
    внутрішнім станом саме з цієї фабрики, а не створювати його
    безпосередньо.

5.  Клієнт повинен зберігати або обчислювати значення зовнішнього стану
    (контекст) і передавати його до методів об'єкта легковаговика.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Заощаджує оперативну пам'ять.
:::

::: col-sm-6
-    Витрачає процесорний час на пошук/обчислення контексту.
-    Ускладнює код програми внаслідок введення безлічі додаткових
    класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Компонувальник](composite.html) часто поєднують з
    [Легковаговиком](flyweight.html), щоб реалізувати спільні гілки
    дерева та заощадити при цьому пам'ять.

-   [Легковаговик](flyweight.html) показує, як створювати багато дрібних
    об'єктів, а [Фасад](facade.html) показує, як створити один об'єкт,
    який відображає цілу підсистему.

-   Патерн [Легковаговик](flyweight.html) може нагадувати
    [Одинака](singleton.html), якщо для конкретного завдання ви змогли
    зменшити кількість об'єктів до одного. Але пам'ятайте, що між
    патернами є дві суттєві відмінності:

    1.  На відміну від *Одинака*, ви можете мати безліч
        об'єктів-легковаговиків.
    2.  Об'єкти-легковаговики повинні бути незмінними, тоді як
        об'єкт-одинак допускає зміну свого стану.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Легковаговик на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](flyweight/csharp/example.html "Легковаговик на C#"){.prog-lang-link}
[![Легковаговик на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](flyweight/cpp/example.html "Легковаговик на C++"){.prog-lang-link}
[![Легковаговик на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](flyweight/go/example.html "Легковаговик на Go"){.prog-lang-link}
[![Легковаговик на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](flyweight/java/example.html "Легковаговик на Java"){.prog-lang-link}
[![Легковаговик на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](flyweight/php/example.html "Легковаговик на PHP"){.prog-lang-link}
[![Легковаговик на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](flyweight/python/example.html "Легковаговик на Python"){.prog-lang-link}
[![Легковаговик на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](flyweight/ruby/example.html "Легковаговик на Ruby"){.prog-lang-link}
[![Легковаговик на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](flyweight/rust/example.html "Легковаговик на Rust"){.prog-lang-link}
[![Легковаговик на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](flyweight/swift/example.html "Легковаговик на Swift"){.prog-lang-link}
[![Легковаговик на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](flyweight/typescript/example.html "Легковаговик на TypeScript"){.prog-lang-link}
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

[Замісник []{.fa .fa-arrow-right}](proxy.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Фасад](facade.html){.btn .btn-default
rel="prev"}
:::
:::::::::::::::::::::::::::::

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
:::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::
