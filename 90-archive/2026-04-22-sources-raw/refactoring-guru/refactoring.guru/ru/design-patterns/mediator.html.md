::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Посредник {#посредник .title}

::: aka
Также известен как:
[Intermediary, ]{style="display:inline-block"}[Controller, ]{style="display:inline-block"}[Mediator]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Посредник** --- это поведенческий паттерн проектирования, который
позволяет уменьшить связанность множества классов между собой, благодаря
перемещению этих связей в один класс-посредник.

<figure class="image">
<img
src="../../images/patterns/content/mediator/mediator6761.png?id=0264bd857a231b6ea2d0c537c092e698"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/mediator/mediator.png?id=0264bd857a231b6ea2d0c537c092e698 640w,/images/patterns/content/mediator/mediator-1.5x.png?id=b3d5df41892274b5c84282bae8737441 960w,/images/patterns/content/mediator/mediator-2x.png?id=250c2bf72ca1fdee2e6d97ed5a4765f2 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Посредник" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Предположим, что у вас есть диалог создания профиля пользователя. Он
состоит из всевозможных элементов управления --- текстовых полей,
чекбоксов, кнопок.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem1-en03c1.png?id=86f99055b3e60bb8834dcc7222922bdf"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/problem1-en.png?id=86f99055b3e60bb8834dcc7222922bdf 600w,/images/patterns/diagrams/mediator/problem1-en-1.5x.png?id=947efe07eb2f81f770af58673f621623 900w,/images/patterns/diagrams/mediator/problem1-en-2x.png?id=e868165bd04a9438857d6ad41528024c 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Беспорядочные связи между элементами пользовательского интерфейса" />
<figcaption><p>Беспорядочные связи между элементами
пользовательского интерфейса.</p></figcaption>
</figure>

Отдельные элементы диалога должны взаимодействовать друг с другом. Так,
например, чекбокс «у меня есть собака» открывает скрытое поле для ввода
имени домашнего любимца, а клик по кнопке отправки запускает проверку
значений всех полей формы.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/problem2a7f7.png?id=072c51eee4dd90c0972866440c87c1c3"
style="aspect-ratio: 3.27;"
srcset="/images/patterns/diagrams/mediator/problem2.png?id=072c51eee4dd90c0972866440c87c1c3 360w,/images/patterns/diagrams/mediator/problem2-1.5x.png?id=b805abc211e60fa567fb114192e24608 540w,/images/patterns/diagrams/mediator/problem2-2x.png?id=d298ec82a47fa2938ed6cf64b7da78c1 720w"
sizes="(max-width: 720px) 100vw, 360px" loading="lazy" width="360"
alt="Код элементов раздут условиями, которые часто меняются" />
<figcaption><p>Код элементов нужно трогать при изменении
каждого диалога.</p></figcaption>
</figure>

Прописав эту логику прямо в коде элементов управления, вы поставите
крест на их повторном использовании в других местах приложения. Они
станут слишком тесно связанными с элементами диалога редактирования
профиля, которые не нужны в других контекстах. Поэтому вы сможете
использовать либо все элементы сразу, либо ни одного.
:::

::: {.section .solution}
##  Решение {#solution}

Паттерн Посредник заставляет объекты общаться не напрямую друг с другом,
а через отдельный объект-посредник, который знает, кому нужно
перенаправить тот или иной запрос. Благодаря этому, компоненты системы
будут зависеть только от посредника, а не от десятков других
компонентов.

В нашем примере посредником мог бы стать диалог. Скорее всего, класс
диалога и так знает, из каких элементов состоит, поэтому никаких новых
связей добавлять в него не придётся.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/solution1-en2ee9.png?id=dd991a5b7830de8d43f82b084e021713"
style="aspect-ratio: 2.22;"
srcset="/images/patterns/diagrams/mediator/solution1-en.png?id=dd991a5b7830de8d43f82b084e021713 600w,/images/patterns/diagrams/mediator/solution1-en-1.5x.png?id=07dec16a26fdec8fb83dd4eee914cd70 900w,/images/patterns/diagrams/mediator/solution1-en-2x.png?id=0a1280789a5d7b1567c9d98e5fcd1125 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Элементы общаются через посредника" />
<figcaption><p>Элементы интерфейса общаются
через посредника.</p></figcaption>
</figure>

Основные изменения произойдут внутри отдельных элементов диалога. Если
раньше при получении клика от пользователя объект кнопки сам проверял
значения полей диалога, то теперь его единственной обязанностью будет
сообщить диалогу о том, что произошёл клик. Получив извещение, диалог
выполнит все необходимые проверки полей. Таким образом, вместо
нескольких зависимостей от остальных элементов кнопка получит только
одну --- от самого диалога.

Чтобы сделать код ещё более гибким, можно выделить общий интерфейс для
всех посредников, то есть диалогов программы. Наша кнопка станет
зависимой не от конкретного диалога создания пользователя, а от
абстрактного, что позволит использовать её и в других диалогах.

Таким образом, посредник скрывает в себе все сложные связи и зависимости
между классами отдельных компонентов программы. А чем меньше связей
имеют классы, тем проще их изменять, расширять и повторно использовать.
:::

::: {.section .analogy}
##  Аналогия из жизни {#analogy}

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/live-example3f2a.png?id=aa1de3cb7b63aa623e63578a1e43399a"
style="aspect-ratio: 1.85;"
srcset="/images/patterns/diagrams/mediator/live-example.png?id=aa1de3cb7b63aa623e63578a1e43399a 370w,/images/patterns/diagrams/mediator/live-example-1.5x.png?id=315109aa1099cc6e7c06057fa139881c 555w,/images/patterns/diagrams/mediator/live-example-2x.png?id=fd55a9f9d8cf49effa223555c7369504 740w"
sizes="(max-width: 720px) 100vw, 370px" loading="lazy" width="370"
alt="Пример с диспетчерской башней." />
<figcaption><p>Пилоты самолётов общаются не напрямую, а
через диспетчера.</p></figcaption>
</figure>

Пилоты садящихся или улетающих самолётов не общаются напрямую с другими
пилотами. Вместо этого они связываются с диспетчером, который
координирует действия нескольких самолётов одновременно. Без диспетчера
пилотам приходилось бы все время быть начеку и следить за всеми
окружающими самолётами самостоятельно, а это приводило бы к частым
катастрофам в небе.

Важно понимать, что диспетчер не нужен во время всего полёта. Он
задействован только в зоне аэропорта, когда нужно координировать
взаимодействие многих самолётов.
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
alt="Структура классов паттерна Посредник" /><img
src="../../images/patterns/diagrams/mediator/structure-indexed5302.png?id=a82d4cf1b92a4f72af32f231ffd21131"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.21;"
srcset="/images/patterns/diagrams/mediator/structure-indexed.png?id=a82d4cf1b92a4f72af32f231ffd21131 520w,/images/patterns/diagrams/mediator/structure-indexed-1.5x.png?id=81d4f842ae5157a3cb09f8f3c05159dd 780w,/images/patterns/diagrams/mediator/structure-indexed-2x.png?id=88722da01a5c82b0452078c9339ca497 1040w"
sizes="(max-width: 720px) 100vw, 520px" loading="lazy" width="520"
alt="Структура классов паттерна Посредник" />
</figure>
:::

1.  **Компоненты** --- это разнородные объекты, содержащие бизнес-логику
    программы. Каждый компонент хранит ссылку на объект посредника, но
    работает с ним только через абстрактный интерфейс посредников.
    Благодаря этому, компоненты можно повторно использовать в другой
    программе, связав их с посредником другого типа.

2.  **Посредник** определяет интерфейс для обмена информацией с
    компонентами. Обычно хватает одного метода, чтобы оповещать
    посредника о событиях, произошедших в компонентах. В параметрах
    этого метода можно передавать детали события: ссылку на компонент, в
    котором оно произошло, и любые другие данные.

3.  **Конкретный посредник** содержит код взаимодействия нескольких
    компонентов между собой. Зачастую этот объект не только хранит
    ссылки на все свои компоненты, но и сам их создаёт, управляя
    дальнейшим жизненным циклом.

4.  Компоненты не должны общаться друг с другом напрямую. Если в
    компоненте происходит важное событие, он должен оповестить своего
    посредника, а тот сам решит --- касается ли событие других
    компонентов, и стоит ли их оповещать. При этом компонент-отправитель
    не знает кто обработает его запрос, а компонент-получатель не знает
    кто его прислал.
::::
:::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере **Посредник** помогает избавиться от зависимостей между
классами различных элементов пользовательского интерфейса: кнопками,
чекбоксами и надписями.

<figure class="image">
<img
src="../../images/patterns/diagrams/mediator/example576a.png?id=3151c153533e816e226be0ef977211e8"
style="aspect-ratio: 1.24;"
srcset="/images/patterns/diagrams/mediator/example.png?id=3151c153533e816e226be0ef977211e8 560w,/images/patterns/diagrams/mediator/example-1.5x.png?id=cf441780d96f3306ac54d6809f85b87d 840w,/images/patterns/diagrams/mediator/example-2x.png?id=02064e5a7c4f065f806747e1b04ac1b0 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура классов примера паттерна Посредник" />
<figcaption><p>Пример структурирования
классов UI-диалогов.</p></figcaption>
</figure>

По реакции на действия пользователей элементы не взаимодействуют
напрямую, а всего лишь уведомляют посредника о том, что они изменились.

Посредник в виде диалога авторизации знает, как конкретные элементы
должны взаимодействовать. Поэтому при получении уведомлений он может
перенаправить вызов тому или иному элементу.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Общий интерфейс посредников.
interface Mediator is
    method notify(sender: Component, event: string)


// Конкретный посредник. Все связи между конкретными
// компонентами переехали в код посредника. Он получает
// извещения от своих компонентов и знает, как на них
// реагировать.
class AuthenticationDialog implements Mediator is
    private field title: string
    private field loginOrRegisterChkBx: Checkbox
    private field loginUsername, loginPassword: Textbox
    private field registrationUsername, registrationPassword,
                  registrationEmail: Textbox
    private field okBtn, cancelBtn: Button

    constructor AuthenticationDialog() is
        // Здесь нужно создать объекты всех компонентов, подав
        // текущий объект-посредник в их конструктор.

    // Когда что-то случается с компонентом, он шлёт посреднику
    // оповещение. После получения извещения посредник может
    // либо сделать что-то самостоятельно, либо перенаправить
    // запрос другому компоненту.
    method notify(sender, event) is
        if (sender == loginOrRegisterChkBx and event == &quot;check&quot;)
            if (loginOrRegisterChkBx.checked)
                title = &quot;Log in&quot;
                // 1. Показать компоненты формы входа.
                // 2. Скрыть компоненты формы регистрации.
            else
                title = &quot;Register&quot;
                // 1. Показать компоненты формы регистрации.
                // 2. Скрыть компоненты формы входа.

        if (sender == okBtn &amp;&amp; event == &quot;click&quot;)
            if (loginOrRegister.checked)
                // Попробовать найти пользователя с данными из
                // формы логина.
                if (!found)
                    // Показать ошибку над формой логина.
            else
                // 1. Создать пользовательский аккаунт с данными
                // из формы регистрации.
                // 2. Авторизировать этого пользователя.
                // ...


// Классы компонентов общаются с посредниками через их общий
// интерфейс. Благодаря этому одни и те же компоненты можно
// использовать в разных посредниках.
class Component is
    field dialog: Mediator

    constructor Component(dialog) is
        this.dialog = dialog

    method click() is
        dialog.notify(this, &quot;click&quot;)

    method keypress() is
        dialog.notify(this, &quot;keypress&quot;)

// Конкретные компоненты не связаны между собой напрямую. У них
// есть только один канал общения — через отправку уведомлений
// посреднику.
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

:::::::::: {.section .applicability-container}
##  Применимость {#applicability}

::::::::: applicability
::: applicability-problem
Когда вам сложно менять некоторые классы из-за того, что они имеют
множество хаотичных связей с другими классами.
:::

::: applicability-solution
Посредник позволяет поместить все эти связи в один класс, после чего вам
будет легче их отрефакторить, сделать более понятными и гибкими.
:::

::: applicability-problem
Когда вы не можете повторно использовать класс, поскольку он зависит от
уймы других классов.
:::

::: applicability-solution
После применения паттерна компоненты теряют прежние связи с другими
компонентами, а всё их общение происходит косвенно, через
объект-посредник.
:::

::: applicability-problem
Когда вам приходится создавать множество подклассов компонентов, чтобы
использовать одни и те же компоненты в разных контекстах.
:::

::: applicability-solution
Если раньше изменение отношений в одном компоненте могли повлечь за
собой лавину изменений во всех остальных компонентах, то теперь вам
достаточно создать подкласс посредника и поменять в нём связи между
компонентами.
:::
:::::::::
::::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Найдите группу тесно переплетённых классов, отвязав которые друг от
    друга, можно получить некоторую пользу. Например, чтобы повторно
    использовать их код в другой программе.

2.  Создайте общий интерфейс посредников и опишите в нём методы для
    взаимодействия с компонентами. В простейшем случае достаточно одного
    метода для получения оповещений от компонентов.

    Этот интерфейс необходим, если вы хотите повторно использовать
    классы компонентов для других задач. В этом случае всё, что нужно
    сделать --- это создать новый класс конкретного посредника.

3.  Реализуйте этот интерфейс в классе конкретного посредника. Поместите
    в него поля, которые будут содержать ссылки на все объекты
    компонентов.

4.  Вы можете пойти дальше и переместить код создания компонентов в
    класс посредника, после чего он может напоминать
    [фабрику](abstract-factory.html) или [фасад](facade.html).

5.  Компоненты тоже должны иметь ссылку на объект посредника. Связь
    между ними удобнее всего установить, подавая посредника в параметры
    конструктора компонентов.

6.  Измените код компонентов так, чтобы они вызывали метод оповещения
    посредника, вместо методов других компонентов. С противоположной
    стороны, посредник должен вызывать методы нужного компонента, когда
    получает оповещение от компонента.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Устраняет зависимости между компонентами, позволяя повторно их
    использовать.
-    Упрощает взаимодействие между компонентами.
-    Централизует управление в одном месте.
:::

::: col-sm-6
-    Посредник может [сильно раздуться](../smells/large-class.html).
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Цепочка обязанностей](chain-of-responsibility.html),
    [Команда](command.html), [Посредник](mediator.html) и
    [Наблюдатель](observer.html) показывают различные способы работы
    отправителей запросов с их получателями:

    -   *Цепочка обязанностей* передаёт запрос последовательно через
        цепочку потенциальных получателей, ожидая, что какой-то из них
        обработает запрос.
    -   *Команда* устанавливает косвенную одностороннюю связь от
        отправителей к получателям.
    -   *Посредник* убирает прямую связь между отправителями и
        получателями, заставляя их общаться опосредованно, через себя.
    -   *Наблюдатель* передаёт запрос одновременно всем заинтересованным
        получателям, но позволяет им динамически подписываться или
        отписываться от таких оповещений.

-   [Посредник](mediator.html) и [Фасад](facade.html) похожи тем, что
    пытаются организовать работу множества существующих классов.

    -   *Фасад* создаёт упрощённый интерфейс к подсистеме, не внося в
        неё никакой добавочной функциональности. Сама подсистема не
        знает о существовании *Фасада*. Классы подсистемы общаются друг
        с другом напрямую.
    -   *Посредник* централизует общение между компонентами системы.
        Компоненты системы знают только о существовании *Посредника*, у
        них нет прямого доступа к другим компонентам.

-   Разница между [Посредником](mediator.html) и
    [Наблюдателем](observer.html) не всегда очевидна. Чаще всего они
    выступают как конкуренты, но иногда могут работать вместе.

    Цель *Посредника* --- убрать обоюдные зависимости между компонентами
    системы. Вместо этого они становятся зависимыми от самого
    посредника. С другой стороны, цель *Наблюдателя* --- обеспечить
    динамическую одностороннюю связь, в которой одни объекты косвенно
    зависят от других.

    Довольно популярна реализация *Посредника* при помощи *Наблюдателя*.
    При этом объект посредника будет выступать издателем, а все
    остальные компоненты станут подписчиками и смогут динамически
    следить за событиями, происходящими в посреднике. В этом случае
    трудно понять, чем же отличаются оба паттерна.

    Но *Посредник* имеет и другие реализации, когда отдельные компоненты
    жёстко привязаны к объекту посредника. Такой код вряд ли будет
    напоминать *Наблюдателя*, но всё же останется *Посредником*.

    Напротив, в случае реализации посредника с помощью *Наблюдателя*
    представим такую программу, в которой каждый компонент системы
    становится издателем. Компоненты могут подписываться друг на друга,
    в то же время не привязываясь к конкретным классам. Программа будет
    состоять из целой сети *Наблюдателей*, не имея центрального
    объекта-*Посредника*.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Посредник на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](mediator/csharp/example.html "Посредник на C#"){.prog-lang-link}
[![Посредник на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](mediator/cpp/example.html "Посредник на C++"){.prog-lang-link}
[![Посредник на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](mediator/go/example.html "Посредник на Go"){.prog-lang-link}
[![Посредник на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](mediator/java/example.html "Посредник на Java"){.prog-lang-link}
[![Посредник на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](mediator/php/example.html "Посредник на PHP"){.prog-lang-link}
[![Посредник на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](mediator/python/example.html "Посредник на Python"){.prog-lang-link}
[![Посредник на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](mediator/ruby/example.html "Посредник на Ruby"){.prog-lang-link}
[![Посредник на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](mediator/rust/example.html "Посредник на Rust"){.prog-lang-link}
[![Посредник на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](mediator/swift/example.html "Посредник на Swift"){.prog-lang-link}
[![Посредник на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](mediator/typescript/example.html "Посредник на TypeScript"){.prog-lang-link}
:::

:::::: {#book-promo .banner-set}
::::: {.prom .banner-content .banner-bg .banner-striped .banner-with-image data-id="DP: 1: Ne vtykay" creative-id="standard-ru" position="content_bottom"}
::: banner-image
[![](../../images/patterns/banners/patterns-book-banner-1-b158a.png?id=0877cab2f0102d98cd07b50af0e5beea){width="200"
height="200" loading="lazy"
srcset="/images/patterns/banners/patterns-book-banner-1-b-2x.png?id=5572aa55e5b09e59780aca9e0ea8e44b 2x"}](book.html)
:::

::: banner-text
### Не втыкай в транспорте {#не-втыкай-в-транспорте .title}

Лучше почитай нашу книгу о паттернах проектирования.

Теперь это удобно делать даже во время поездок в общественном
транспорте.

[ Узнать больше...](book.html){.btn .btn-secondary}
:::
:::::
::::::

::: next
#### Читаем дальше

[Снимок []{.fa .fa-arrow-right}](memento.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Итератор](iterator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::

::::::: {.prom .banner-sidebar .banner-removable .banner-removable-patterns data-id="DP: Sidebar" creative-id="standard-sidebar-ru" position="sidebar"}
:::::: banner-inner
:::: image3d-book-right
::: {.image3d-cover style="background: #0b3752;"}
[![](../../images/patterns/book/web-cover-ruee5e.png?id=ea25feb28a6deb8aa4af6a633904a568){width="250"
height="375" loading="lazy"
srcset="/images/patterns/book/web-cover-ru-2x.png?id=95101b307f6dba409f28a417746ff3c1 2x"}](book.html)
:::
::::

::: {style="margin-top: 1rem"}
Эта статья является частью нашей электронной книги **Погружение в
Паттерны Проектирования**.

[ Узнать больше...](book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::
