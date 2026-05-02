:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Паттерны
проектирования](../../../design-patterns.html){.type} /
[Наблюдатель](../../observer.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Наблюдатель](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Наблюдатель** на TypeScript {#наблюдатель-на-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Наблюдатель** --- это поведенческий паттерн, который позволяет
объектам оповещать другие объекты об изменениях своего состояния.

При этом наблюдатели могут свободно подписываться и отписываться от этих
оповещений.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Подробней о паттерне Наблюдатель ](../../observer.html){.btn .btn-lg
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

**Применимость:** Наблюдатель можно часто встретить в TypeScript коде,
особенно там, где применяется событийная модель отношений между
компонентами. Наблюдатель позволяет отдельным компонентам реагировать на
события, происходящие в других компонентах.

**Признаки применения паттерна:** Наблюдатель можно определить по
механизму подписки и методам оповещения, которые вызывают компоненты
программы.
:::::::::::

[]{#example-0}

## Концептуальный пример {#example-0-title}

Этот пример показывает структуру паттерна **Наблюдатель**, а именно ---
из каких классов он состоит, какие роли эти классы выполняют и как они
взаимодействуют друг с другом.

#### []{#example-0--index-ts .anchor} **index.ts:** Пример структуры паттерна

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * Интерфейс издателя объявляет набор методов для управлениями подписчиками.
 */
interface Subject {
    // Присоединяет наблюдателя к издателю.
    attach(observer: Observer): void;

    // Отсоединяет наблюдателя от издателя.
    detach(observer: Observer): void;

    // Уведомляет всех наблюдателей о событии.
    notify(): void;
}

/**
 * Издатель владеет некоторым важным состоянием и оповещает наблюдателей о его
 * изменениях.
 */
class ConcreteSubject implements Subject {
    /**
     * @type {number} Для удобства в этой переменной хранится состояние
     * Издателя, необходимое всем подписчикам.
     */
    public state: number;

    /**
     * @type {Observer[]} Список подписчиков. В реальной жизни список
     * подписчиков может храниться в более подробном виде (классифицируется по
     * типу события и т.д.)
     */
    private observers: Observer[] = [];

    /**
     * Методы управления подпиской.
     */
    public attach(observer: Observer): void {
        const isExist = this.observers.includes(observer);
        if (isExist) {
            return console.log(&#39;Subject: Observer has been attached already.&#39;);
        }

        console.log(&#39;Subject: Attached an observer.&#39;);
        this.observers.push(observer);
    }

    public detach(observer: Observer): void {
        const observerIndex = this.observers.indexOf(observer);
        if (observerIndex === -1) {
            return console.log(&#39;Subject: Nonexistent observer.&#39;);
        }

        this.observers.splice(observerIndex, 1);
        console.log(&#39;Subject: Detached an observer.&#39;);
    }

    /**
     * Запуск обновления в каждом подписчике.
     */
    public notify(): void {
        console.log(&#39;Subject: Notifying observers...&#39;);
        for (const observer of this.observers) {
            observer.update(this);
        }
    }

    /**
     * Обычно логика подписки – только часть того, что делает Издатель. Издатели
     * часто содержат некоторую важную бизнес-логику, которая запускает метод
     * уведомления всякий раз, когда должно произойти что-то важное (или после
     * этого).
     */
    public someBusinessLogic(): void {
        console.log(&#39;\nSubject: I\&#39;m doing something important.&#39;);
        this.state = Math.floor(Math.random() * (10 + 1));

        console.log(`Subject: My state has just changed to: ${this.state}`);
        this.notify();
    }
}

/**
 * Интерфейс Наблюдателя объявляет метод уведомления, который издатели
 * используют для оповещения своих подписчиков.
 */
interface Observer {
    // Получить обновление от субъекта.
    update(subject: Subject): void;
}

/**
 * Конкретные Наблюдатели реагируют на обновления, выпущенные Издателем, к
 * которому они прикреплены.
 */
class ConcreteObserverA implements Observer {
    public update(subject: Subject): void {
        if (subject instanceof ConcreteSubject &amp;&amp; subject.state &lt; 3) {
            console.log(&#39;ConcreteObserverA: Reacted to the event.&#39;);
        }
    }
}

class ConcreteObserverB implements Observer {
    public update(subject: Subject): void {
        if (subject instanceof ConcreteSubject &amp;&amp; (subject.state === 0 || subject.state &gt;= 2)) {
            console.log(&#39;ConcreteObserverB: Reacted to the event.&#39;);
        }
    }
}

/**
 * Клиентский код.
 */

const subject = new ConcreteSubject();

const observer1 = new ConcreteObserverA();
subject.attach(observer1);

const observer2 = new ConcreteObserverB();
subject.attach(observer2);

subject.someBusinessLogic();
subject.someBusinessLogic();

subject.detach(observer2);

subject.someBusinessLogic();</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Результат выполнения

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 6
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 1
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event.
Subject: Detached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 5
Subject: Notifying observers...</code></pre>
</figure>

::: next
#### Читаем дальше

[Состояние на TypeScript []{.fa
.fa-arrow-right}](../../state/typescript/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Вернуться назад

[[]{.fa .fa-arrow-left} Снимок на
TypeScript](../../memento/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Наблюдатель** на других языках программирования

[![Наблюдатель на
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Наблюдатель на C#"){.prog-lang-link}
[![Наблюдатель на
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Наблюдатель на C++"){.prog-lang-link}
[![Наблюдатель на
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Наблюдатель на Go"){.prog-lang-link}
[![Наблюдатель на
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Наблюдатель на Java"){.prog-lang-link}
[![Наблюдатель на
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Наблюдатель на PHP"){.prog-lang-link}
[![Наблюдатель на
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Наблюдатель на Python"){.prog-lang-link}
[![Наблюдатель на
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Наблюдатель на Ruby"){.prog-lang-link}
[![Наблюдатель на
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Наблюдатель на Rust"){.prog-lang-link}
[![Наблюдатель на
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Наблюдатель на Swift"){.prog-lang-link}
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
