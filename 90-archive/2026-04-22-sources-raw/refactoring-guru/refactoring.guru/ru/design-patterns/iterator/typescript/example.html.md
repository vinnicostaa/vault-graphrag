:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Итератор](../../iterator.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Итератор](../../../../images/patterns/cards/iterator-minie2c2.png?id=76c28bb48f997b36965983dd2b41f02e){srcset="/images/patterns/cards/iterator-mini-2x.png?id=7d28f66a487066774a743cfceaa40ea4 2x"}
:::

# **Итератор** на TypeScript {#итератор-на-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Итератор** --- это поведенческий паттерн, позволяющий последовательно
обходить сложную коллекцию, без раскрытия деталей её реализации.

Благодаря Итератору, клиент может обходить разные коллекции одним и тем
же способом, используя единый интерфейс итераторов.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Итератор ](../../iterator.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Навигация
:::

::: en-intro
 [Интро](#)
:::

::: en-intro
 [Концептуальный пример](#example-0)
:::

::: en-file
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Сложность:** []{.fa-stars}

**Популярность:** []{.fa-stars}

**Применимость:** Паттерн можно часто встретить в TypeScript-коде,
особенно в программах, работающих с разными типами коллекций, и где
требуется обход разных сущностей.

**Признаки применения паттерна:** Итератор легко определить по методам
навигации (например, получения следующего/предыдущего элемента и т. д.).
Код использующий итератор зачастую вообще не имеет ссылок на коллекцию,
с которой работает итератор. Итератор либо принимает коллекцию в
параметрах конструктора при создании, либо возвращается самой
коллекцией.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Итератор**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--index-ts .anchor} **index.ts:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * Паттерн Итератор
 *
 * Назначение: Даёт возможность последовательно обходить элементы составных
 * объектов, не раскрывая их внутреннего представления.
 */

interface Iterator&lt;T&gt; {
    // Возврат текущего элемента.
    current(): T;

    // Возврат текущего элемента и переход к следующему элементу.
    next(): T;

    // Возврат ключа текущего элемента.
    key(): number;

    // Проверяет корректность текущей позиции.
    valid(): boolean;

    // Перемотка Итератора к первому элементу.
    rewind(): void;
}

interface Aggregator {
    // Получить внешний итератор.
    getIterator(): Iterator&lt;string&gt;;
}

/**
 * Конкретные Итераторы реализуют различные алгоритмы обхода. Эти классы
 * постоянно хранят текущее положение обхода.
 */

class AlphabeticalOrderIterator implements Iterator&lt;string&gt; {
    private collection: WordsCollection;

    /**
     * Хранит текущее положение обхода. У итератора может быть множество других
     * полей для хранения состояния итерации, особенно когда он должен работать
     * с определённым типом коллекции.
     */
    private position: number = 0;

    /**
     * Эта переменная указывает направление обхода.
     */
    private reverse: boolean = false;

    constructor(collection: WordsCollection, reverse: boolean = false) {
        this.collection = collection;
        this.reverse = reverse;

        if (reverse) {
            this.position = collection.getCount() - 1;
        }
    }

    public rewind() {
        this.position = this.reverse ?
            this.collection.getCount() - 1 :
            0;
    }

    public current(): string {
        return this.collection.getItems()[this.position];
    }

    public key(): number {
        return this.position;
    }

    public next(): string {
        const item = this.collection.getItems()[this.position];
        this.position += this.reverse ? -1 : 1;
        return item;
    }

    public valid(): boolean {
        if (this.reverse) {
            return this.position &gt;= 0;
        }

        return this.position &lt; this.collection.getCount();
    }
}

/**
 * Конкретные Коллекции предоставляют один или несколько методов для получения
 * новых экземпляров итератора, совместимых с классом коллекции.
 */
class WordsCollection implements Aggregator {
    private items: string[] = [];

    public getItems(): string[] {
        return this.items;
    }

    public getCount(): number {
        return this.items.length;
    }

    public addItem(item: string): void {
        this.items.push(item);
    }

    public getIterator(): Iterator&lt;string&gt; {
        return new AlphabeticalOrderIterator(this);
    }

    public getReverseIterator(): Iterator&lt;string&gt; {
        return new AlphabeticalOrderIterator(this, true);
    }
}

/**
 * Клиентский код может знать или не знать о Конкретном Итераторе или классах
 * Коллекций, в зависимости от уровня косвенности, который вы хотите сохранить в
 * своей программе.
 */
const collection = new WordsCollection();
collection.addItem(&#39;First&#39;);
collection.addItem(&#39;Second&#39;);
collection.addItem(&#39;Third&#39;);

const iterator = collection.getIterator();

console.log(&#39;Straight traversal:&#39;);
while (iterator.valid()) {
    console.log(iterator.next());
}

console.log(&#39;&#39;);
console.log(&#39;Reverse traversal:&#39;);
const reverseIterator = collection.getReverseIterator();
while (reverseIterator.valid()) {
    console.log(reverseIterator.next());
}</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Straight traversal:
First
Second
Third

Reverse traversal:
Third
Second
First</code></pre>
</figure>

::: next
#### Читаем дальше

[Посредник на TypeScript []{.fa
.fa-arrow-right}](../../mediator/typescript/example.html){.btn
.btn-primary rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Команда на
TypeScript](../../command/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Итератор** на других языках программирования

[![Итератор на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Итератор на C#"){.prog-lang-link}
[![Итератор на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Итератор на C++"){.prog-lang-link}
[![Итератор на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Итератор на Go"){.prog-lang-link}
[![Итератор на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Итератор на Java"){.prog-lang-link}
[![Итератор на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Итератор на PHP"){.prog-lang-link}
[![Итератор на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Итератор на Python"){.prog-lang-link}
[![Итератор на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Итератор на Ruby"){.prog-lang-link}
[![Итератор на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Итератор на Rust"){.prog-lang-link}
[![Итератор на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Итератор на Swift"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-ru" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Архив с примерами](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Купи книгу **Погружение в Паттерны** и получи архив с десятками
детальных примеров, которые можно открывать прямо в IDE.

[ Узнать больше...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
