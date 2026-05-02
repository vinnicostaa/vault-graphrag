:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Легковес](../../flyweight.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Легковес](../../../../images/patterns/cards/flyweight-minie3d8.png?id=422ca8d2f90614dce810a8812c626698){srcset="/images/patterns/cards/flyweight-mini-2x.png?id=857ca676fff84317bb0dab76abfce08e 2x"}
:::

# **Легковес** на TypeScript {#легковес-на-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Легковес** --- это структурный паттерн, который экономит память,
благодаря разделению общего состояния, вынесенного в один объект, между
множеством объектов.

Легковес позволяет экономить память, кешируя одинаковые данные,
используемые в разных объектах.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Легковес ](../../flyweight.html){.btn .btn-lg
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

**Применимость:** Весь смысл использования Легковеса --- в экономии
памяти. Поэтому, если в приложении нет такой проблемы, то вы вряд ли
найдёте там примеры Легковеса.

**Признаки применения паттерна:** Легковес можно определить по создающим
методам класса, которые возвращают закешированные объекты, вместо
создания новых.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Легковес**, а именно --- из
каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--index-ts .anchor} **index.ts:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * Легковес хранит общую часть состояния (также называемую внутренним
 * состоянием), которая принадлежит нескольким реальным бизнес-объектам.
 * Легковес принимает оставшуюся часть состояния (внешнее состояние, уникальное
 * для каждого объекта) через его параметры метода.
 */
class Flyweight {
    private sharedState: any;

    constructor(sharedState: any) {
        this.sharedState = sharedState;
    }

    public operation(uniqueState): void {
        const s = JSON.stringify(this.sharedState);
        const u = JSON.stringify(uniqueState);
        console.log(`Flyweight: Displaying shared (${s}) and unique (${u}) state.`);
    }
}

/**
 * Фабрика Легковесов создает объекты-Легковесы и управляет ими. Она
 * обеспечивает правильное разделение легковесов. Когда клиент запрашивает
 * легковес, фабрика либо возвращает существующий экземпляр, либо создает новый,
 * если он ещё не существует.
 */
class FlyweightFactory {
    private flyweights: {[key: string]: Flyweight} = &lt;any&gt;{};

    constructor(initialFlyweights: string[][]) {
        for (const state of initialFlyweights) {
            this.flyweights[this.getKey(state)] = new Flyweight(state);
        }
    }

    /**
     * Возвращает хеш строки Легковеса для данного состояния.
     */
    private getKey(state: string[]): string {
        return state.join(&#39;_&#39;);
    }

    /**
     * Возвращает существующий Легковес с заданным состоянием или создает новый.
     */
    public getFlyweight(sharedState: string[]): Flyweight {
        const key = this.getKey(sharedState);

        if (!(key in this.flyweights)) {
            console.log(&#39;FlyweightFactory: Can\&#39;t find a flyweight, creating new one.&#39;);
            this.flyweights[key] = new Flyweight(sharedState);
        } else {
            console.log(&#39;FlyweightFactory: Reusing existing flyweight.&#39;);
        }

        return this.flyweights[key];
    }

    public listFlyweights(): void {
        const count = Object.keys(this.flyweights).length;
        console.log(`\nFlyweightFactory: I have ${count} flyweights:`);
        for (const key in this.flyweights) {
            console.log(key);
        }
    }
}

/**
 * Клиентский код обычно создает кучу предварительно заполненных легковесов на
 * этапе инициализации приложения.
 */
const factory = new FlyweightFactory([
    [&#39;Chevrolet&#39;, &#39;Camaro2018&#39;, &#39;pink&#39;],
    [&#39;Mercedes Benz&#39;, &#39;C300&#39;, &#39;black&#39;],
    [&#39;Mercedes Benz&#39;, &#39;C500&#39;, &#39;red&#39;],
    [&#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;],
    [&#39;BMW&#39;, &#39;X6&#39;, &#39;white&#39;],
    // ...
]);
factory.listFlyweights();

// ...

function addCarToPoliceDatabase(
    ff: FlyweightFactory, plates: string, owner: string,
    brand: string, model: string, color: string,
) {
    console.log(&#39;\nClient: Adding a car to database.&#39;);
    const flyweight = ff.getFlyweight([brand, model, color]);

    // Клиентский код либо сохраняет, либо вычисляет внешнее состояние и
    // передает его методам легковеса.
    flyweight.operation([plates, owner]);
}

addCarToPoliceDatabase(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;M5&#39;, &#39;red&#39;);

addCarToPoliceDatabase(factory, &#39;CL234IR&#39;, &#39;James Doe&#39;, &#39;BMW&#39;, &#39;X1&#39;, &#39;red&#39;);

factory.listFlyweights();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>FlyweightFactory: I have 5 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white

Client: Adding a car to database.
FlyweightFactory: Reusing existing flyweight.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;M5&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

Client: Adding a car to database.
FlyweightFactory: Can&#39;t find a flyweight, creating new one.
Flyweight: Displaying shared ([&quot;BMW&quot;,&quot;X1&quot;,&quot;red&quot;]) and unique ([&quot;CL234IR&quot;,&quot;James Doe&quot;]) state.

FlyweightFactory: I have 6 flyweights:
Chevrolet_Camaro2018_pink
Mercedes Benz_C300_black
Mercedes Benz_C500_red
BMW_M5_red
BMW_X6_white
BMW_X1_red</code></pre>
</figure>

::: next
#### Читаем дальше

[Заместитель на TypeScript []{.fa
.fa-arrow-right}](../../proxy/typescript/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Фасад на
TypeScript](../../facade/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Легковес** на других языках программирования

[![Легковес на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Легковес на C#"){.prog-lang-link}
[![Легковес на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Легковес на C++"){.prog-lang-link}
[![Легковес на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Легковес на Go"){.prog-lang-link}
[![Легковес на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Легковес на Java"){.prog-lang-link}
[![Легковес на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Легковес на PHP"){.prog-lang-link}
[![Легковес на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Легковес на Python"){.prog-lang-link}
[![Легковес на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Легковес на Ruby"){.prog-lang-link}
[![Легковес на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Легковес на Rust"){.prog-lang-link}
[![Легковес на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Легковес на Swift"){.prog-lang-link}
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
