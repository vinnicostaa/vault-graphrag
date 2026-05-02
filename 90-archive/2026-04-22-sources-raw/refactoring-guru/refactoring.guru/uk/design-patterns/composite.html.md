::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Патерни
проектування](../design-patterns.html){.type} / [Структурні
патерни](structural-patterns.html){.type}
:::

# Компонувальник {#компонувальник .title}

::: aka
Також відомий як:
[Дерево, ]{style="display:inline-block"}[Composite]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть патерна {#intent}

**Компонувальник** --- це структурний патерн проектування, що дає змогу
згрупувати декілька об'єктів у деревоподібну структуру, а потім
працювати з нею так, ніби це одиничний об'єкт.

<figure class="image">
<img
src="../../images/patterns/content/composite/compositec299.png?id=73bcf0d94db360b636cd745f710d19db"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/composite/composite.png?id=73bcf0d94db360b636cd745f710d19db 640w,/images/patterns/content/composite/composite-1.5x.png?id=098acddafae70b1ec1cb1b05e833c3ae 960w,/images/patterns/content/composite/composite-2x.png?id=8847e6f8e2cb892ed2229faba83bd1b7 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Патерн Компонувальник" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Патерн Компонувальник має сенс тільки в тих випадках, коли основна
модель вашої програми може бути структурована у вигляді дерева.

Наприклад, є два об'єкти --- `Продукт` і `Коробка`. `Коробка` може
містити кілька `Продуктів` та інших `Коробок` меншого розміру. Останні,
в свою чергу, також містять або `Продукти`, або `Коробки` і так далі.

Тепер, припустімо, що ваші `Продукти` й `Коробки` можуть бути частиною
замовлень. При цьому замовлення може містити як звичайні `Продукт` без
пакування, так і наповнені змістом `Коробки`. Ваше завдання полягає в
тому, щоб дізнатися вартість всього замовлення.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/problem-ukebdf.png?id=ed2fdec46a815aaf76bc6521b22825d6"
style="aspect-ratio: 1;"
srcset="/images/patterns/diagrams/composite/problem-uk.png?id=ed2fdec46a815aaf76bc6521b22825d6 370w,/images/patterns/diagrams/composite/problem-uk-1.5x.png?id=2379c9b8f8fc1c32b0b8c3903de5d6fe 555w,/images/patterns/diagrams/composite/problem-uk-2x.png?id=af2b4026d7493bb2c3b24c77625318da 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура складного замовлення" />
<figcaption><p>Замовлення може складатися з різних продуктів,
запакованих у власні коробки.</p></figcaption>
</figure>

Якщо спробувати вирішити завдання напролом, тоді потрібно відкрити усі
коробки замовлення, перебрати продукти й порахувати їхню загальну
вартість. Але це занадто велика морока, оскільки типи коробок і їхній
вміст можуть бути вам невідомі заздалегідь. Крім того, наперед невідомою
є і кількість рівнів вкладеності коробок, тому перебрати коробки простим
циклом не вийде.
:::

::: {.section .solution}
##  Рішення {#solution}

Компонувальник пропонує розглядати `Продукт` і `Коробку` через єдиний
інтерфейс зі спільним методом отримання ціни.

`Продукт` просто поверне свою вартість, а `Коробка` запитає про вартість
кожного предмета всередині себе і поверне суму результатів. Якщо одним
із внутрішніх предметів виявиться трохи менша коробка, вона теж буде
перебирати власний вміст, і так далі, допоки не порахується вміст усіх
складових частин.

<figure class="image">
<img
src="../../images/patterns/content/composite/composite-comic-1-ukd5e5.png?id=dc086e215eff5464425c3653fa6f4202"
style="aspect-ratio: 2;"
srcset="/images/patterns/content/composite/composite-comic-1-uk.png?id=dc086e215eff5464425c3653fa6f4202 600w,/images/patterns/content/composite/composite-comic-1-uk-1.5x.png?id=362276a952fef0a215f64f9bd5732912 900w,/images/patterns/content/composite/composite-comic-1-uk-2x.png?id=6a70070ecdd09cdea56cb458ea2cfa4a 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Рішення з Компонувальником" />
<figcaption><p>Компонувальник рекурсивно запускає дію по всіх
компонентах дерева — від коріння до листя.</p></figcaption>
</figure>

Для вас як клієнта важливим є те, що вже не потрібно нічого знати про
структуру замовлень. Ви викликаєте метод отримання ціни, він повертає
цифру, і ви не «тонете» в горах картону та скотчу.
:::

::: {.section .analogy}
##  Аналогія з життя {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/live-examplee724.png?id=548a7cec45b493af66e8bfe524a137d3"
style="aspect-ratio: 1.22;"
srcset="/images/patterns/diagrams/composite/live-example.png?id=548a7cec45b493af66e8bfe524a137d3 280w,/images/patterns/diagrams/composite/live-example-1.5x.png?id=2129195f4103a5a4d3440a086ce3dc87 420w,/images/patterns/diagrams/composite/live-example-2x.png?id=b555458f20fc30425ae6dada5da492af 560w"
sizes="(max-width: 720px) 100vw, 280px" loading="lazy" width="280"
alt="Приклад армійської структури" />
<figcaption><p>Приклад армійської структури.</p></figcaption>
</figure>

Армії більшості країн можуть бути представлені у вигляді перевернутих
дерев. На нижньому рівні у вас солдати, далі взводи, далі полки, а далі
цілі армії. Накази віддаються зверху вниз структурою командування до тих
пір, поки вони не доходять до конкретного солдата.
:::

::::: {.section .structure-container}
##  Структура {#structure}

:::: structure
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/composite/structure-ukdeff.png?id=55dbecfa0415deba017d9cfc5191a839"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 0.8;"
srcset="/images/patterns/diagrams/composite/structure-uk.png?id=55dbecfa0415deba017d9cfc5191a839 360w,/images/patterns/diagrams/composite/structure-uk-1.5x.png?id=6862a855ec0e25a0e4eaa90daec98355 540w,/images/patterns/diagrams/composite/structure-uk-2x.png?id=40c06f7b73d0fe014fbdfc704b244caf 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Структура класів патерна Компонувальник" /><img
src="../../images/patterns/diagrams/composite/structure-uk-indexedd934.png?id=6faad0069287e602c21094a79c60011f"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 0.85;"
srcset="/images/patterns/diagrams/composite/structure-uk-indexed.png?id=6faad0069287e602c21094a79c60011f 400w,/images/patterns/diagrams/composite/structure-uk-indexed-1.5x.png?id=7f87e78e3628c3a9b4eeeca797f94a69 600w,/images/patterns/diagrams/composite/structure-uk-indexed-2x.png?id=3642e8a37075cf23d779c2b6338d9d98 800w"
sizes="(max-width: 720px) 100vw, 400px" loading="lazy" width="400"
alt="Структура класів патерна Компонувальник" />
</figure>
:::

1.  **Компонент** описує загальний інтерфейс для простих і складових
    компонентів дерева.

2.  **Лист** --- це простий компонент дерева, який не має відгалужень.
    Класи листя міститимуть більшу частину корисного коду, тому що їм
    нікому передавати його виконання.

3.  **Контейнер** (або *композит*) --- це складовий компонент дерева.
    Він містить набір дочірніх компонентів, але нічого не знає про їхні
    типи. Це можуть бути як прості компоненти-листя, так і інші
    компоненти-контейнери. Проте, це не проблема, якщо усі дочірні
    компоненти дотримуються єдиного інтерфейсу.

    Методи контейнера переадресовують основну роботу своїм дочірнім
    компонентам, хоча можуть додавати щось своє до результату.

4.  **Клієнт** працює з деревом через загальний інтерфейс компонентів.

    Завдяки цьому, клієнту не важливо, що перед ним знаходиться ---
    простий чи складовий компонент дерева.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

У цьому прикладі **Компонувальник** допомагає реалізувати вкладені
геометричні фігури.

<figure class="image">
<img
src="../../images/patterns/diagrams/composite/example7ea7.png?id=98ba81d07c979038dd2ebf3c83a2e19f"
style="aspect-ratio: 0.84;"
srcset="/images/patterns/diagrams/composite/example.png?id=98ba81d07c979038dd2ebf3c83a2e19f 370w,/images/patterns/diagrams/composite/example-1.5x.png?id=ae008efd4146628cd7303925d4ce90be 555w,/images/patterns/diagrams/composite/example-2x.png?id=d21edef39d3792e8a4c6736727ac7305 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Структура класів прикладу патерна Компонувальник" />
<figcaption><p>Приклад редактора геометричних фігур.</p></figcaption>
</figure>

Клас `CompoundGraphic` може містити будь-яку кількість підфігур, включно
з такими самими контейнерами, як і він сам. Контейнер реалізує ті ж самі
методи, що і прості фігури. Але замість безпосередньої дії він передає
виклики всім вкладеним компонентам, використовуючи рекурсію. Потім він
як би «підсумовує» результати всіх вкладених фігур.

Клієнтський код працює з усіма фігурами через загальний інтерфейс фігур
і не знає що перед ним --- проста фігура чи складова. Це дозволяє
клієнтському коду працювати з деревами об'єктів будь-якої складності, не
прив'язуючись до конкретних класів об'єктів, що формують дерево.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Загальний інтерфейс компонентів.
interface Graphic is
    method move(x, y)
    method draw()

// Простий компонент.
class Dot implements Graphic is
    field x, y

    constructor Dot(x, y) { ... }

    method move(x, y) is
        this.x += x, this.y += y

    method draw() is
        // Намалювати крапку у координатах X, Y.

// Компоненти можуть розширювати інші компоненти.
class Circle extends Dot is
    field radius

    constructor Circle(x, y, radius) { ... }

    method draw() is
        // Намалювати коло в координатах X, Y з радіусом R.

// Контейнер містить операції додавання/видалення дочірніх
// компонентів. Усі стандартні операції інтерфейсу компонентів
// він делегує кожному з дочірніх компонентів.
class CompoundGraphic implements Graphic is
    field children: array of Graphic

    method add(child: Graphic) is
        // Додати компонент до списка дочірніх.

    method remove(child: Graphic) is
        // Прибрати компонент зі списку дочірніх.

    method move(x, y) is
        foreach (child in children) do
            child.move(x, y)

    method draw() is
        // 1. Для кожного дочірнього компонента:
        //     - Відобразити компонент.
        //     - Визначити координати максимальної межі.
        // 2. Намалювати пунктирну межу навколо всієї області.


// Програма працює одноманітно, як з одиничними компонентами,
// так і з цілими групами компонентів.
class ImageEditor is
    field all: CompoundGraphic

    method load() is
        all = new CompoundGraphic()
        all.add(new Dot(1, 2))
        all.add(new Circle(5, 3, 10))
        // ...

    // Групування обраних компонентів в один складний компонент.
    method groupSelected(components: array of Graphic) is
        group = new CompoundGraphic()
        foreach (component in components) do
            group.add(component)
            all.remove(component)
        all.add(group)
        // Усі компоненти будуть промальованими.
        all.draw()</code></pre>
</figure>
:::

:::::::: {.section .applicability-container}
##  Застосування {#applicability}

::::::: applicability
::: applicability-problem
Якщо вам потрібно представити деревоподібну структуру об'єктів.
:::

::: applicability-solution
Патерн Компонувальник пропонує зберігати в складових об'єктах посилання
на інші прості або складові об'єкти. Вони, у свою чергу, теж можуть
зберігати свої вкладені об'єкти і так далі. У підсумку, ви можете
будувати складну деревоподібну структуру даних, використовуючи всього
два основних різновида об'єктів.
:::

::: applicability-problem
Якщо клієнти повинні однаково трактувати прості та складові об'єкти.
:::

::: applicability-solution
Завдяки тому, що прості та складові об'єкти реалізують спільний
інтерфейс, клієнту байдуже, з яким саме об'єктом він працюватиме.
:::
:::::::
::::::::

::: {.section .checklist}
##  Кроки реалізації {#checklist}

1.  Переконайтеся, що вашу бізнес-логіку можна представити як
    деревоподібну структуру. Спробуйте розбити її на прості компоненти й
    контейнери. Пам'ятайте, що контейнери можуть містити як прості
    компоненти, так і інші вкладені контейнери.

2.  Створіть загальний інтерфейс компонентів, який об'єднає операції
    контейнерів та простих компонентів дерева. Інтерфейс буде вдалим,
    якщо ви зможете використовувати його, щоб взаємозаміняти прості й
    складові компоненти без втрати сенсу.

3.  Створіть клас компонентів-листя, які не мають подальших відгалужень.
    Майте на увазі, що програма може містити декілька таких класів.

4.  Створіть клас компонентів-контейнерів і додайте до нього масив для
    зберігання посилань на вкладені компоненти. Цей масив повинен бути
    здатен містити як прості, так і складові компоненти, тому
    переконайтеся, що його оголошено з типом інтерфейсу компонентів.

    Реалізуйте в контейнері методи інтерфейсу компонентів, пам'ятаючи
    про те, що контейнери повинні делегувати основну роботу своїм
    дочірнім компонентам.

5.  Додайте операції додавання й видалення дочірніх компонентів до класу
    контейнерів.

    Майте на увазі, що методи додавання/видалення дочірніх компонентів
    можна оголосити також і в інтерфейсі компонентів. Так, це порушить
    *принцип розділення інтерфейсу*, тому що реалізації методів будуть
    порожніми в компонентах-листях. Проте усі компоненти дерева стануть
    дійсно однаковими для клієнта.
:::

:::::: {.section .pros-cons}
##  Переваги та недоліки {#pros-cons}

::::: row
::: col-sm-6
-    Спрощує архітектуру клієнта при роботі зі складним деревом
    компонентів.
-    Полегшує додавання нових видів компонентів.
:::

::: col-sm-6
-    Створює занадто загальний дизайн класів.
:::
:::::
::::::

::: {.section .relations}
##  Відносини з іншими патернами {#relations}

-   [Будівельник](builder.html) дозволяє покроково конструювати дерево
    [Компонувальника](composite.html).

-   [Ланцюжок обов'язків](chain-of-responsibility.html) часто
    використовують разом з [Компонувальником](composite.html). У цьому
    випадку запит передається від дочірніх компонентів до їхніх батьків.

-   Ви можете обходити дерево [Компонувальника](composite.html),
    використовуючи [Ітератор](iterator.html).

-   Ви можете виконати якусь дію над усім деревом
    [Компонувальника](composite.html) за допомогою
    [Відвідувача](visitor.html).

-   [Компонувальник](composite.html) часто поєднують з
    [Легковаговиком](flyweight.html), щоб реалізувати спільні гілки
    дерева та заощадити при цьому пам'ять.

-   [Компонувальник](composite.html) та [Декоратор](decorator.html)
    мають схожі структури класів, бо обидва побудовані на рекурсивній
    вкладеності. Вона дозволяє зв'язати в одну структуру нескінченну
    кількість об'єктів.

    *Декоратор* обгортає тільки один об'єкт, а вузол *Компонувальника*
    може мати багато дітей. *Декоратор* додає вкладеному об'єкту нової
    функціональності, а *Компонувальник* не додає нічого нового, але
    «підсумовує» результати всіх своїх дітей.

    Але вони можуть і співпрацювати: *Компонувальник* може
    використовувати *Декоратор*, щоб перевизначити функції окремих
    частин дерева компонентів.

-   Архітектура, побудована на [Компонувальниках](composite.html) та
    [Декораторах](decorator.html), часто може поліпшуватися за рахунок
    впровадження [Прототипу](prototype.html). Він дозволяє клонувати
    складні структури об'єктів, а не збирати їх заново.
:::

::: {.section .implementations}
##  Приклади реалізації патерна {#implementations}

[![Компонувальник на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](composite/csharp/example.html "Компонувальник на C#"){.prog-lang-link}
[![Компонувальник на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](composite/cpp/example.html "Компонувальник на C++"){.prog-lang-link}
[![Компонувальник на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](composite/go/example.html "Компонувальник на Go"){.prog-lang-link}
[![Компонувальник на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](composite/java/example.html "Компонувальник на Java"){.prog-lang-link}
[![Компонувальник на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](composite/php/example.html "Компонувальник на PHP"){.prog-lang-link}
[![Компонувальник на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](composite/python/example.html "Компонувальник на Python"){.prog-lang-link}
[![Компонувальник на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](composite/ruby/example.html "Компонувальник на Ruby"){.prog-lang-link}
[![Компонувальник на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](composite/rust/example.html "Компонувальник на Rust"){.prog-lang-link}
[![Компонувальник на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](composite/swift/example.html "Компонувальник на Swift"){.prog-lang-link}
[![Компонувальник на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](composite/typescript/example.html "Компонувальник на TypeScript"){.prog-lang-link}
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

[Декоратор []{.fa .fa-arrow-right}](decorator.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Повернутись назад

[[]{.fa .fa-arrow-left} Міст](bridge.html){.btn .btn-default rel="prev"}
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
