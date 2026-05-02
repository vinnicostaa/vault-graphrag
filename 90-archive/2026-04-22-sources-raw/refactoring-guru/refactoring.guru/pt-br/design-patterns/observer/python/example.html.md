:::::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Padrões de
Projeto](../../../design-patterns.html){.type} /
[Observer](../../observer.html){.type} /
[Python](../../python.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observer](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observer** em Python {#observer-em-python .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
O **Observer** é um padrão de projeto comportamental que permite que um
objeto notifique outros objetos sobre alterações em seu estado.

O padrão Observer fornece uma maneira de assinar e cancelar a assinatura
desses eventos para qualquer objeto que implemente uma interface de
assinante.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ Saiba mais sobre o Observer ](../../observer.html){.btn .btn-lg
.btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navegação
:::

::: en-intro
 [Introdução](#)
:::

::: en-intro
 [Exemplo conceitual](#example-0)
:::

::: en-file
 [main](#example-0--main-py)
:::

::: en-file
 [Output](#example-0--Output-txt)
:::
::::::::

**Complexidade:** []{.fa-stars}

**Popularidade:** []{.fa-stars}

**Exemplos de uso:** O padrão Observer é bastante comum no código
Python, especialmente nos componentes da interface de usuário. Ele
fornece uma maneira de reagir a eventos que acontecem em outros objetos
sem acoplamento às suas classes.

**Identificação:** O padrão pode ser reconhecido por métodos de
assinatura, que armazenam objetos em uma lista e por chamadas para o
método de atualização emitido para objetos nessa lista.
:::::::::::

[]{#example-0}

## Exemplo conceitual {#example-0-title}

Este exemplo ilustra a estrutura do padrão de projeto **Observer**. Ele
se concentra em responder a estas perguntas:

-   De quais classes ele consiste?
-   Quais papéis essas classes desempenham?
-   De que maneira os elementos do padrão estão relacionados?

#### []{#example-0--main-py .anchor} **main.py:** Exemplo conceitual

<figure class="code">
<pre class="code" lang="python"><code>from __future__ import annotations
from abc import ABC, abstractmethod
from random import randrange
from typing import List


class Subject(ABC):
    &quot;&quot;&quot;
    The Subject interface declares a set of methods for managing subscribers.
    &quot;&quot;&quot;

    @abstractmethod
    def attach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Attach an observer to the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def detach(self, observer: Observer) -&gt; None:
        &quot;&quot;&quot;
        Detach an observer from the subject.
        &quot;&quot;&quot;
        pass

    @abstractmethod
    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Notify all observers about an event.
        &quot;&quot;&quot;
        pass


class ConcreteSubject(Subject):
    &quot;&quot;&quot;
    The Subject owns some important state and notifies observers when the state
    changes.
    &quot;&quot;&quot;

    _state: int = None
    &quot;&quot;&quot;
    For the sake of simplicity, the Subject&#39;s state, essential to all
    subscribers, is stored in this variable.
    &quot;&quot;&quot;

    _observers: List[Observer] = []
    &quot;&quot;&quot;
    List of subscribers. In real life, the list of subscribers can be stored
    more comprehensively (categorized by event type, etc.).
    &quot;&quot;&quot;

    def attach(self, observer: Observer) -&gt; None:
        print(&quot;Subject: Attached an observer.&quot;)
        self._observers.append(observer)

    def detach(self, observer: Observer) -&gt; None:
        self._observers.remove(observer)

    &quot;&quot;&quot;
    The subscription management methods.
    &quot;&quot;&quot;

    def notify(self) -&gt; None:
        &quot;&quot;&quot;
        Trigger an update in each subscriber.
        &quot;&quot;&quot;

        print(&quot;Subject: Notifying observers...&quot;)
        for observer in self._observers:
            observer.update(self)

    def some_business_logic(self) -&gt; None:
        &quot;&quot;&quot;
        Usually, the subscription logic is only a fraction of what a Subject can
        really do. Subjects commonly hold some important business logic, that
        triggers a notification method whenever something important is about to
        happen (or after it).
        &quot;&quot;&quot;

        print(&quot;\nSubject: I&#39;m doing something important.&quot;)
        self._state = randrange(0, 10)

        print(f&quot;Subject: My state has just changed to: {self._state}&quot;)
        self.notify()


class Observer(ABC):
    &quot;&quot;&quot;
    The Observer interface declares the update method, used by subjects.
    &quot;&quot;&quot;

    @abstractmethod
    def update(self, subject: Subject) -&gt; None:
        &quot;&quot;&quot;
        Receive update from subject.
        &quot;&quot;&quot;
        pass


&quot;&quot;&quot;
Concrete Observers react to the updates issued by the Subject they had been
attached to.
&quot;&quot;&quot;


class ConcreteObserverA(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state &lt; 3:
            print(&quot;ConcreteObserverA: Reacted to the event&quot;)


class ConcreteObserverB(Observer):
    def update(self, subject: Subject) -&gt; None:
        if subject._state == 0 or subject._state &gt;= 2:
            print(&quot;ConcreteObserverB: Reacted to the event&quot;)


if __name__ == &quot;__main__&quot;:
    # The client code.

    subject = ConcreteSubject()

    observer_a = ConcreteObserverA()
    subject.attach(observer_a)

    observer_b = ConcreteObserverB()
    subject.attach(observer_b)

    subject.some_business_logic()
    subject.some_business_logic()

    subject.detach(observer_a)

    subject.some_business_logic()</code></pre>
</figure>

#### []{#example-0--Output-txt .anchor} **Output.txt:** Resultados da execução

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 5
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 0
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event</code></pre>
</figure>

::: next
#### Leia a seguir

[State em Python []{.fa
.fa-arrow-right}](../../state/python/example.html){.btn .btn-primary
rel="next"}
:::

::: prev
#### Voltar

[[]{.fa .fa-arrow-left} Memento em
Python](../../memento/python/example.html){.btn .btn-default rel="prev"}
:::

## **Observer** em outras linguagens

[![Observer em
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observer em C#"){.prog-lang-link}
[![Observer em
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observer em C++"){.prog-lang-link}
[![Observer em
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observer em Go"){.prog-lang-link}
[![Observer em
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observer em Java"){.prog-lang-link}
[![Observer em
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observer em PHP"){.prog-lang-link}
[![Observer em
Ruby](../../../../images/patterns/icons/ruby3275.svg?id=b065b718c914bf8e960ef731600be1eb){width="53"
height="53"
loading="lazy"}](../ruby/example.html "Observer em Ruby"){.prog-lang-link}
[![Observer em
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observer em Rust"){.prog-lang-link}
[![Observer em
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observer em Swift"){.prog-lang-link}
[![Observer em
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observer em TypeScript"){.prog-lang-link}
:::::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-pt-br" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Arquivo com exemplos de código](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Compre o eBook **Mergulho nos Padrões de Projeto** e tenha acesso ao
arquivo com dúzias de exemplos detalhados que podem ser abertos
diretamente em seu IDE.

[ Saiba mais...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::::
::::::::::::::::::::::::
