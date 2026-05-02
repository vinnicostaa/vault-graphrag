:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Wzorce
projektowe](../../../design-patterns.html){.type} /
[Obserwator](../../observer.html){.type} /
[TypeScript](../../typescript.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Obserwator](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Obserwator** w języku TypeScript {#obserwator-w-języku-typescript .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
**Obserwator** to behawioralny wzorzec projektowy pozwalający obiektom
powiadamiać inne obiekty o zmianach swojego stanu.

Obserwator daje możliwość subskrypcji lub zrezygnowania z subskrypcji
zdarzeń dowolnego obiektu implementującego interfejs subskrybenta.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Dowiedz się więcej o Obserwator ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Nawigacja
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Przykład koncepcyjny](#example-0)
:::

::: en-file
 [index](#example-0--index-ts)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Złożoność:** []{.fa-stars}

**Popularność:** []{.fa-stars}

**Przykłady użycia:** Wzorzec Obserwator jest dość powszechny w kodzie
TypeScript, szczególnie w komponentach graficznego interfejsu
użytkownika. Pozwala reagować na zdarzenia dotyczące innych obiektów bez
konieczności sprzęgania z klasami tych obiektów.

**Identyfikacja:** Wzorzec Obserwator można poznać po obecności metod
służących subskrypcji, które przechowują obiekty w strukturze listy i po
wywołaniach metod aktualizacji obiektów z tej listy.
:::::::::::

[]{#example-0}

## Przykład koncepcyjny {#example-0-title}

Poniższy przykład ilustruje strukturę wzorca **Obserwator** ze
szczególnym naciskiem na następujące kwestie:

-   Z jakich składa się klas?
-   Jakie role pełnią te klasy?
-   W jaki sposób elementy wzorca są ze sobą powiązane?

#### []{#example-0--index-ts .anchor} **index.ts:** Przykład koncepcyjny

<figure class="code">
<pre class="code" lang="typescript"><code>/**
 * The Subject interface declares a set of methods for managing subscribers.
 */
interface Subject {
    // Attach an observer to the subject.
    attach(observer: Observer): void;

    // Detach an observer from the subject.
    detach(observer: Observer): void;

    // Notify all observers about an event.
    notify(): void;
}

/**
 * The Subject owns some important state and notifies observers when the state
 * changes.
 */
class ConcreteSubject implements Subject {
    /**
     * @type {number} For the sake of simplicity, the Subject&#39;s state, essential
     * to all subscribers, is stored in this variable.
     */
    public state: number;

    /**
     * @type {Observer[]} List of subscribers. In real life, the list of
     * subscribers can be stored more comprehensively (categorized by event
     * type, etc.).
     */
    private observers: Observer[] = [];

    /**
     * The subscription management methods.
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
     * Trigger an update in each subscriber.
     */
    public notify(): void {
        console.log(&#39;Subject: Notifying observers...&#39;);
        for (const observer of this.observers) {
            observer.update(this);
        }
    }

    /**
     * Usually, the subscription logic is only a fraction of what a Subject can
     * really do. Subjects commonly hold some important business logic, that
     * triggers a notification method whenever something important is about to
     * happen (or after it).
     */
    public someBusinessLogic(): void {
        console.log(&#39;\nSubject: I\&#39;m doing something important.&#39;);
        this.state = Math.floor(Math.random() * (10 + 1));

        console.log(`Subject: My state has just changed to: ${this.state}`);
        this.notify();
    }
}

/**
 * The Observer interface declares the update method, used by subjects.
 */
interface Observer {
    // Receive update from subject.
    update(subject: Subject): void;
}

/**
 * Concrete Observers react to the updates issued by the Subject they had been
 * attached to.
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
 * The client code.
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

#### []{#example-0--Output-txt .anchor} **Output.txt:** Wynik działania

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
#### Czytaj dalej

[Stan w języku TypeScript []{.fa
.fa-arrow-right}](../../state/typescript/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Wróć

[[]{.fa .fa-arrow-left} Pamiątka w języku
TypeScript](../../memento/typescript/example.html){.btn .btn-default
rel="prev"}
:::

## **Obserwator** w innych językach

[![Obserwator w języku
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Obserwator w języku C#"){.prog-lang-link}
[![Obserwator w języku
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Obserwator w języku C++"){.prog-lang-link}
[![Obserwator w języku
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Obserwator w języku Go"){.prog-lang-link}
[![Obserwator w języku
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Obserwator w języku Java"){.prog-lang-link}
[![Obserwator w języku
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Obserwator w języku PHP"){.prog-lang-link}
[![Obserwator w języku
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Obserwator w języku Python"){.prog-lang-link}
[![Obserwator w języku
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Obserwator w języku Ruby"){.prog-lang-link}
[![Obserwator w języku
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Obserwator w języku Rust"){.prog-lang-link}
[![Obserwator w języku
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Obserwator w języku Swift"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pl" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archiwum\
przykładów kodu](../../book.html){.btn .btn-secondary .btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Kupując eBook **Wzorce projektowe. Nowoczesny podręcznik** zyskujesz
dostęp do archiwum zawierającego mnóstwo szczegółowych przykładów, które
można od razu wkleić do swojego preferowanego zintegrowanego środowiska
programistycznego.

[ Dowiedz się więcej...](../../book.html){.btn .btn-secondary
.btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
