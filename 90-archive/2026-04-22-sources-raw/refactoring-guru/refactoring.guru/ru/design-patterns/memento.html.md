::::::::::::::::::::::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
:::::::::::::::::::::::::::::::::::::::::: {.main-content-container .center-content-container}
:::::::::::::::::::::::::::::::::::: {.pattern .page .text}
::: breadcrumb
[](../../index.html){.home} / [Паттерны
проектирования](../design-patterns.html){.type} / [Поведенческие
паттерны](behavioral-patterns.html){.type}
:::

# Снимок {#снимок .title}

::: aka
Также известен как:
[Хранитель, ]{style="display:inline-block"}[Memento]{style="display:inline-block"}
:::

::: {.section .intent}
##  Суть паттерна {#intent}

**Снимок** --- это поведенческий паттерн проектирования, который
позволяет сохранять и восстанавливать прошлые состояния объектов, не
раскрывая подробностей их реализации.

<figure class="image">
<img
src="../../images/patterns/content/memento/memento-rued7e.png?id=7d3bf6dbbe1e22bffa72df475ee064c8"
style="aspect-ratio: 1.6;"
srcset="/images/patterns/content/memento/memento-ru.png?id=7d3bf6dbbe1e22bffa72df475ee064c8 640w,/images/patterns/content/memento/memento-ru-1.5x.png?id=183cf8a530aa550bca1d0084014492dd 960w,/images/patterns/content/memento/memento-ru-2x.png?id=88fb7b0b6493e3ebfe441edb0bd2c2df 1280w"
sizes="(max-width: 720px) 100vw, 640px" data-fetchpriority="high"
width="640" alt="Паттерн Снимок" />
</figure>
:::

::: {.section .problem}
##  Проблема {#problem}

Предположим, что вы пишете программу текстового редактора. Помимо
обычного редактирования, ваш редактор позволяет менять форматирование
текста, вставлять картинки и прочее.

В какой-то момент вы решили сделать все эти действия отменяемыми. Для
этого вам нужно сохранять текущее состояние редактора перед тем, как
выполнить любое действие. Если потом пользователь решит отменить своё
действие, вы достанете копию состояния из истории и восстановите старое
состояние редактора.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem1-ru763a.png?id=abec92eadea625e191c25096e36497ae"
style="aspect-ratio: 2.61;"
srcset="/images/patterns/diagrams/memento/problem1-ru.png?id=abec92eadea625e191c25096e36497ae 470w,/images/patterns/diagrams/memento/problem1-ru-1.5x.png?id=d0d9092b5a340457053753ecbed61ec3 705w,/images/patterns/diagrams/memento/problem1-ru-2x.png?id=3a9b0638ae1f5d61021c7d7c17afcec2 940w"
sizes="(max-width: 720px) 100vw, 470px" loading="lazy" width="470"
alt="Схема отмены операций в редакторе" />
<figcaption><p>Перед выполнением команды вы можете сохранить копию
состояния редактора, чтобы потом иметь возможность
отменить операцию.</p></figcaption>
</figure>

Чтобы сделать копию состояния объекта, достаточно скопировать значение
его полей. Таким образом, если вы сделали класс редактора достаточно
открытым, то любой другой класс сможет заглянуть внутрь, чтобы
скопировать его состояние.

Казалось бы, что ещё нужно? Ведь теперь любая операция сможет сделать
резервную копию редактора перед своим действием. Но такой наивный подход
обеспечит вам уйму проблем в будущем. Ведь если вы решите провести
рефакторинг --- убрать или добавить парочку полей в класс редактора ---
то придётся менять код всех классов, которые могли копировать состояние
редактора.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/problem2-ru7b8b.png?id=09debb9c831f60b0d2f39bed85037006"
style="aspect-ratio: 1.57;"
srcset="/images/patterns/diagrams/memento/problem2-ru.png?id=09debb9c831f60b0d2f39bed85037006 330w,/images/patterns/diagrams/memento/problem2-ru-1.5x.png?id=337668f835310ddc39ca2f46fcfb4b0f 495w,/images/patterns/diagrams/memento/problem2-ru-2x.png?id=904b897f2635aacdd674dc127416d09b 660w"
sizes="(max-width: 720px) 100vw, 330px" loading="lazy" width="330"
alt="Как команде создать снимок состояния редактора, если все его поля приватные?" />
<figcaption><p>Как команде создать снимок состояния редактора, если все
его поля приватные?</p></figcaption>
</figure>

Но это ещё не все. Давайте теперь рассмотрим сами копии состояния
редактора. Из чего состоит состояние редактора? Даже самый примитивный
редактор должен иметь несколько полей для хранения текущего текста,
позиции курсора и прокрутки экрана. Чтобы сделать копию состояния, вам
нужно записать значения всех этих полей в некий «контейнер».

Скорее всего, вам понадобится хранить массу таких контейнеров в качестве
истории операций, поэтому удобнее всего сделать их объектами одного
класса. Этот класс должен иметь много полей, но практически никаких
методов. Чтобы другие объекты могли записывать и читать из него данные,
вам придётся сделать его поля публичными. Но это приведёт к той же
проблеме, что и с открытым классом редактора. Другие классы станут
зависимыми от любых изменений в классе контейнера, который подвержен тем
же изменениям, что и класс редактора.

Получается, нам придётся либо открыть классы для всех желающих,
испытывая массу хлопот с поддержкой кода, либо оставить классы
закрытыми, отказавшись от идеи отмены операций. Нет ли какого-то другого
пути?
:::

::: {.section .solution}
##  Решение {#solution}

Все проблемы, описанные выше, возникают из-за нарушения инкапсуляции.
Это когда одни объекты пытаются сделать работу за других, влезая в их
приватную зону, чтобы собрать необходимые для операции данные.

Паттерн Снимок поручает создание копии состояния объекта самому объекту,
который этим состоянием владеет. Вместо того, чтобы делать снимок
«извне», наш редактор сам сделает копию своих полей, ведь ему доступны
все поля, даже приватные.

Паттерн предлагает держать копию состояния в специальном
объекте-*снимке* с ограниченным интерфейсом, позволяющим, например,
узнать дату изготовления или название снимка. Но, с другой стороны,
снимок должен быть открыт для своего *создателя*, позволяя прочесть и
восстановить его внутреннее состояние.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/solution-rucf0f.png?id=8b8691b7c1ba88680d04b426d6ea88bf"
style="aspect-ratio: 1.36;"
srcset="/images/patterns/diagrams/memento/solution-ru.png?id=8b8691b7c1ba88680d04b426d6ea88bf 610w,/images/patterns/diagrams/memento/solution-ru-1.5x.png?id=eaf2887fc9e7127301269cac41ec8135 915w,/images/patterns/diagrams/memento/solution-ru-2x.png?id=d660345beb5593ba30870f1e97d78f61 1220w"
sizes="(max-width: 720px) 100vw, 610px" loading="lazy" width="610"
alt="Снимок полностью открыт для создателя, но лишь частично открыт для опекунов" />
<figcaption><p>Снимок полностью открыт для создателя, но лишь частично
открыт для опекунов.</p></figcaption>
</figure>

Такая схема позволяет создателям производить снимки и отдавать их для
хранения другим объектам, называемым *опекунами*. Опекунам будет
доступен только ограниченный интерфейс снимка, поэтому они никак не
смогут повлиять на «внутренности» самого снимка. В нужный момент опекун
может попросить создателя восстановить своё состояние, передав ему
соответствующий снимок.

В примере с редактором вы можете сделать опекуном отдельный класс,
который будет хранить список выполненных операций. Ограниченный
интерфейс снимков позволит демонстрировать пользователю красивый список
с названиями и датами выполненных операций. А когда пользователь решит
откатить операцию, класс истории возьмёт последний снимок из стека и
отправит его объекту редактор для восстановления.
:::

:::::::::: {.section .structure-container}
##  Структура {#structure}

::::::::: structure
#### Классическая реализация на вложенных классах

Классическая реализация паттерна полагается на механизм вложенных
классов, который доступен лишь в некоторых языках программирования (C++,
C#, Java).

:::: memento-classic
::: {.struct-image1 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure180c1.png?id=4b4a42363a005b617d4df06689787385"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1.png?id=4b4a42363a005b617d4df06689787385 580w,/images/patterns/diagrams/memento/structure1-1.5x.png?id=82cf757f153840c85d27bd63f3f3e133 870w,/images/patterns/diagrams/memento/structure1-2x.png?id=d4e77295e51c2417f22b7abb396d5977 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура классов паттерна Снимок (Хранитель)" /><img
src="../../images/patterns/diagrams/memento/structure1-indexeda7c3.png?id=f79a8356b087ae6b004aec42b787ae2e"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.87;"
srcset="/images/patterns/diagrams/memento/structure1-indexed.png?id=f79a8356b087ae6b004aec42b787ae2e 580w,/images/patterns/diagrams/memento/structure1-indexed-1.5x.png?id=0687dc84dd4c98b4591a70ebd05c4d8e 870w,/images/patterns/diagrams/memento/structure1-indexed-2x.png?id=62fea7bdbc96420568869ea3bd25f6ad 1160w"
sizes="(max-width: 720px) 100vw, 580px" loading="lazy" width="580"
alt="Структура классов паттерна Снимок (Хранитель)" />
</figure>
:::
::::

1.  **Создатель** может производить снимки своего состояния, а также
    воспроизводить прошлое состояние, если подать в него готовый снимок.

2.  **Снимок** --- это простой объект данных, содержащий состояние
    создателя. Надёжнее всего сделать объекты снимков неизменяемыми,
    передавая в них состояние только через конструктор.

3.  **Опекун** должен знать, когда делать снимок создателя и когда его
    нужно восстанавливать.

    Опекун может хранить историю прошлых состояний создателя в виде
    стека из снимков. Когда понадобится отменить выполненную операцию,
    он возьмёт «верхний» снимок из стека и передаст его создателю для
    восстановления.

4.  В данной реализации снимок --- это внутренний класс по отношению к
    классу создателя. Именно поэтому он имеет полный доступ к полям и
    методам создателя, даже приватным. С другой стороны, опекун не имеет
    доступа ни к состоянию, ни к методам снимков и может всего лишь
    хранить ссылки на эти объекты.

#### Реализация с пустым промежуточным интерфейсом

Подходит для языков, не имеющих механизма вложенных классов (например,
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
alt="Структура классов паттерна Снимок (Хранитель)" /><img
src="../../images/patterns/diagrams/memento/structure2-indexede045.png?id=2c98b4f64b03f2a30e159de31ca9f718"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.6;"
srcset="/images/patterns/diagrams/memento/structure2-indexed.png?id=2c98b4f64b03f2a30e159de31ca9f718 560w,/images/patterns/diagrams/memento/structure2-indexed-1.5x.png?id=1ba6e0f22bb613f3f1dcf86850c3b604 840w,/images/patterns/diagrams/memento/structure2-indexed-2x.png?id=2fb637daef1110dfa89f15b2d4627894 1120w"
sizes="(max-width: 720px) 100vw, 560px" loading="lazy" width="560"
alt="Структура классов паттерна Снимок (Хранитель)" />
</figure>
:::
::::

1.  В этой реализации создатель работает напрямую с конкретным классом
    снимка, а опекун --- только с его ограниченным интерфейсом.

2.  Благодаря этому достигается тот же эффект, что и в классической
    реализации. Создатель имеет полный доступ к снимку, а опекун ---
    нет.

#### Снимки с повышенной защитой

Когда нужно полностью исключить возможность доступа к состоянию
создателей и снимков.

:::: memento-safe
::: {.struct-image3 .struct-image}
<figure class="image">
<img
src="../../images/patterns/diagrams/memento/structure3f138.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f"
class="structure-img-non-indexed d-none d-xl-block"
style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3.png?id=6c3ef6a916be8c8ec6d6659d19a6a79f 590w,/images/patterns/diagrams/memento/structure3-1.5x.png?id=9bb6d9dd5567bc71d9e93efc9183ef3a 885w,/images/patterns/diagrams/memento/structure3-2x.png?id=988c37f92059457153d26ba3458d371e 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Снимок с повышенной защитой" /><img
src="../../images/patterns/diagrams/memento/structure3-indexedadf8.png?id=17e84b0ef89a41bb3fb844c8d7a445ad"
class="structure-img-indexed d-xl-none" style="aspect-ratio: 1.74;"
srcset="/images/patterns/diagrams/memento/structure3-indexed.png?id=17e84b0ef89a41bb3fb844c8d7a445ad 590w,/images/patterns/diagrams/memento/structure3-indexed-1.5x.png?id=c121c75333433b70b9a67b75e536214c 885w,/images/patterns/diagrams/memento/structure3-indexed-2x.png?id=fef9ae2a0151c105976075aafb8939dd 1180w"
sizes="(max-width: 720px) 100vw, 590px" loading="lazy" width="590"
alt="Снимок с повышенной защитой" />
</figure>
:::
::::

1.  Эта реализация разрешает иметь несколько видов создателей и снимков.
    Каждому классу создателей соответствует свой класс снимков. Ни
    создатели, ни снимки не позволяют другим объектам прочесть своё
    состояние.

2.  Здесь опекун ещё более жёстко ограничен в доступе к состоянию
    создателей и снимков. Но, с другой стороны, опекун становится
    независим от создателей, поскольку метод восстановления теперь
    находится в самих снимках.

3.  Снимки теперь связаны с теми создателями, из которых они сделаны.
    Они по-прежнему получают состояние через конструктор. Благодаря
    близкой связи между классами, снимки знают, как восстановить
    состояние своих создателей.
:::::::::
::::::::::

::: {.section .pseudocode}
##  Псевдокод {#pseudocode}

В этом примере паттерн **Снимок** используется совместно с паттерном
[Команда](command.html) и позволяет хранить резервные копии сложного
состояния текстового редактора и восстанавливать его, если потребуется.

<figure class="image">
<img
src="../../images/patterns/diagrams/memento/example4526.png?id=fb2196b065f374a1c2a64a0943463760"
style="aspect-ratio: 4.29;"
srcset="/images/patterns/diagrams/memento/example.png?id=fb2196b065f374a1c2a64a0943463760 600w,/images/patterns/diagrams/memento/example-1.5x.png?id=174b686f918a2c49e2545d5573c52d42 900w,/images/patterns/diagrams/memento/example-2x.png?id=41a73f3cc22bc3dd180f53e6968974d4 1200w"
sizes="(max-width: 720px) 100vw, 600px" loading="lazy" width="600"
alt="Структура классов примера паттерна Снимок" />
<figcaption><p>Пример сохранения снимков состояния
текстового редактора.</p></figcaption>
</figure>

Объекты команд выступают в роли опекунов и запрашивают снимки у
редактора перед тем, как выполнить своё действие. Если потребуется
отменить операцию, команда сможет восстановить состояние редактора,
используя сохранённый снимок.

При этом снимок не имеет публичных полей, поэтому другие объекты не
имеют доступа к его внутренним данным. Снимки связаны с определённым
редактором, который их создал. Они же и восстанавливают состояние своего
редактора. Это позволяет программе иметь одновременно несколько объектов
редакторов, например, разбитых по разным вкладкам программы.

<figure class="code">
<pre class="code" lang="pseudocode"><code>// Класс создателя должен иметь специальный метод, который
// сохраняет состояние создателя в новом объекте-снимке.
class Editor is
    private field text, curX, curY, selectionWidth

    method setText(text) is
        this.text = text

    method setCursor(x, y) is
        this.curX = x
        this.curY = y

    method setSelectionWidth(width) is
        this.selectionWidth = width

    method createSnapshot():Snapshot is
        // Снимок — неизменяемый объект, поэтому Создатель
        // передаёт все своё состояние через параметры
        // конструктора.
        return new Snapshot(this, text, curX, curY, selectionWidth)

// Снимок хранит прошлое состояние редактора.
class Snapshot is
    private field editor: Editor
    private field text, curX, curY, selectionWidth

    constructor Snapshot(editor, text, curX, curY, selectionWidth) is
        this.editor = editor
        this.text = text
        this.curX = x
        this.curY = y
        this.selectionWidth = selectionWidth

    // В нужный момент владелец снимка может восстановить
    // состояние редактора.
    method restore() is
        editor.setText(text)
        editor.setCursor(curX, curY)
        editor.setSelectionWidth(selectionWidth)

// Опекуном может выступать класс команд (см. паттерн Команда).
// В этом случае команда сохраняет снимок состояния объекта-
// получателя, перед тем как передать ему своё действие. А в
// случае отмены команда вернёт объект в прежнее состояние.
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
##  Применимость {#applicability}

::::::: applicability
::: applicability-problem
Когда вам нужно сохранять мгновенные снимки состояния объекта (или его
части), чтобы впоследствии объект можно было восстановить в том же
состоянии.
:::

::: applicability-solution
Паттерн Снимок позволяет создавать любое количество снимков объекта и
хранить их, независимо от объекта, с которого делают снимок. Снимки
часто используют не только для реализации операции отмены, но и для
транзакций, когда состояние объекта нужно «откатить», если операция не
удалась.
:::

::: applicability-problem
Когда прямое получение состояния объекта раскрывает приватные детали его
реализации, нарушая инкапсуляцию.
:::

::: applicability-solution
Паттерн предлагает изготовить снимок самому исходному объекту, поскольку
ему доступны все поля, даже приватные.
:::
:::::::
::::::::

::: {.section .checklist}
##  Шаги реализации {#checklist}

1.  Определите класс создателя, объекты которого должны создавать снимки
    своего состояния.

2.  Создайте класс снимка и опишите в нём все те же поля, которые
    имеются в оригинальном классе-создателе.

3.  Сделайте объекты снимков неизменяемыми. Они должны получать
    начальные значения только один раз, через свой конструктор.

4.  Если ваш язык программирования это позволяет, сделайте класс снимка
    вложенным в класс создателя. Если нет, извлеките из класса снимка
    пустой интерфейс, который будет доступен остальным объектам
    программы. Впоследствии вы можете добавить в этот интерфейс
    некоторые вспомогательные методы, дающие доступ к метаданным снимка,
    однако прямой доступ к данным создателя должен быть исключён.

5.  Добавьте в класс создателя метод получения снимков. Создатель должен
    создавать новые объекты снимков, передавая значения своих полей
    через конструктор.

    Сигнатура метода должна возвращать снимки через ограниченный
    интерфейс, если он у вас есть. Сам класс должен работать с
    конкретным классом снимка.

6.  Добавьте в класс создателя метод восстановления из снимка. Что
    касается привязки к типам, руководствуйтесь той же логикой, что и в
    пункте 4.

7.  Опекуны, будь то история операций, объекты команд или нечто иное,
    должны знать о том, когда запрашивать снимки у создателя, где их
    хранить и когда восстанавливать.

8.  Связь опекунов с создателями можно перенести внутрь снимков. В этом
    случае каждый снимок будет привязан к своему создателю и должен
    будет сам восстанавливать его состояние. Но это будет работать либо
    если классы снимков вложены в классы создателей, либо если создатели
    имеют соответствующие сеттеры для установки значений своих полей.
:::

:::::: {.section .pros-cons}
##  Преимущества и недостатки {#pros-cons}

::::: row
::: col-sm-6
-    Не нарушает инкапсуляции исходного объекта.
-    Упрощает структуру исходного объекта. Ему не нужно хранить историю
    версий своего состояния.
:::

::: col-sm-6
-    Требует много памяти, если клиенты слишком часто создают снимки.
-    Может повлечь дополнительные издержки памяти, если объекты,
    хранящие историю, не освобождают ресурсы, занятые устаревшими
    снимками.
-    В некоторых языках (например, PHP, Python, JavaScript) сложно
    гарантировать, чтобы только исходный объект имел доступ к состоянию
    снимка.
:::
:::::
::::::

::: {.section .relations}
##  Отношения с другими паттернами {#relations}

-   [Команду](command.html) и [Снимок](memento.html) можно использовать
    сообща для реализации отмены операций. В этом случае объекты команд
    будут отвечать за выполнение действия над объектом, а снимки будут
    хранить резервную копию состояния этого объекта, сделанную перед
    самым запуском команды.

-   [Снимок](memento.html) можно использовать вместе с
    [Итератором](iterator.html), чтобы сохранить текущее состояние
    обхода структуры данных и вернуться к нему в будущем, если
    потребуется.

-   [Снимок](memento.html) иногда можно заменить
    [Прототипом](prototype.html), если объект, состояние которого
    требуется сохранять в истории, довольно простой, не имеет активных
    ссылок на внешние ресурсы либо их можно легко восстановить.
:::

::: {.section .implementations}
##  Примеры реализации паттерна {#implementations}

[![Снимок на
C#](../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](memento/csharp/example.html "Снимок на C#"){.prog-lang-link}
[![Снимок на
C++](../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](memento/cpp/example.html "Снимок на C++"){.prog-lang-link}
[![Снимок на
Go](../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](memento/go/example.html "Снимок на Go"){.prog-lang-link}
[![Снимок на
Java](../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](memento/java/example.html "Снимок на Java"){.prog-lang-link}
[![Снимок на
PHP](../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](memento/php/example.html "Снимок на PHP"){.prog-lang-link}
[![Снимок на
Python](../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](memento/python/example.html "Снимок на Python"){.prog-lang-link}
[![Снимок на
Ruby](../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](memento/ruby/example.html "Снимок на Ruby"){.prog-lang-link}
[![Снимок на
Rust](../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](memento/rust/example.html "Снимок на Rust"){.prog-lang-link}
[![Снимок на
Swift](../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](memento/swift/example.html "Снимок на Swift"){.prog-lang-link}
[![Снимок на
TypeScript](../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](memento/typescript/example.html "Снимок на TypeScript"){.prog-lang-link}
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

[Наблюдатель []{.fa .fa-arrow-right}](observer.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Посредник](mediator.html){.btn .btn-default
rel="prev"}
:::
::::::::::::::::::::::::::::::::::::

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
::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::
