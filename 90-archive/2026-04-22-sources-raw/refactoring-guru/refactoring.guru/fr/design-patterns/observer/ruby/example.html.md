:::::::::::::::::::::: {.main-content .top-content .center-content role="main" page_class=""}
::::::::::::::::::::: {.main-content-container .center-content-container}
::::::::::::::: {.pattern .pattern-example .page .text}
::: breadcrumb
[](../../../../index.html){.home} / [Patrons de
conception](../../../design-patterns.html){.type} /
[Observateur](../../observer.html){.type} /
[Ruby](../../ruby.html){.type}
:::

:::: pattern-example-title-block
::: pattern-example-title-block-image
![Observateur](../../../../images/patterns/cards/observer-minifa2e.png?id=fd2081ab1cff29c60b499bcf6a62786a){srcset="/images/patterns/cards/observer-mini-2x.png?id=f205b0655572ac8e4636691c0e0debfd 2x"}
:::

# **Observateur** en Ruby {#observateur-en-ruby .pattern-example-title-block-title}
::::

::::::::::: pattern-example-body
::: pattern-example-brief
L'**Observateur** est un patron de conception comportemental qui permet
à certains objets d'envoyer des notifications concernant leur état à
d'autres objets.

Ce patron fournit la possibilité aux objets qui implémentent une
interface de souscription, de s'inscrire et de se désinscrire de ces
événements.
:::

::: {style="text-align:center; margin-bottom: 40px;"}
[ En savoir plus sur la patron Observateur ](../../observer.html){.btn
.btn-lg .btn-primary}
:::

:::::::: {.sidebar-navigation .with-scroll}
::: en-title
Navigation
:::

::: en-intro
 [Intro](#)
:::

::: en-intro
 [Exemple conceptuel](#example-0)
:::

::: en-file
 [main](#example-0--main-rb)
:::

::: en-file
 [output](#example-0--output-txt)
:::
::::::::

**Complexité :** []{.fa-stars}

**Popularité :** []{.fa-stars}

**Exemples d'utilisation :** L'observateur est assez répandu en Ruby,
surtout dans les composants GUI. Il fournit une manière de réagir aux
événements qui se produisent chez d'autres objets sans se coupler à
leurs classes.

**Identification :** Ce patron peut être reconnu dans les méthodes de
souscription qui stockent des objets dans une liste et par les appels
des objets de cette liste à la méthode update.
:::::::::::

[]{#example-0}

## Exemple conceptuel {#example-0-title}

Dans cet exemple, nous allons voir la structure de l'**Observateur**.
Nous allons répondre aux questions suivantes :

-   Que contiennent les classes ?
-   Quels rôles jouent-elles ?
-   Comment les éléments du patron sont-ils reliés ?

#### []{#example-0--main-rb .anchor} **main.rb:** Exemple conceptuel

<figure class="code">
<pre class="code" lang="ruby"><code># The Subject interface declares a set of methods for managing subscribers.
class Subject
  # Attach an observer to the subject.
  def attach(observer)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Detach an observer from the subject.
  def detach(observer)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end

  # Notify all observers about an event.
  def notify
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# The Subject owns some important state and notifies observers when the state
# changes.
class ConcreteSubject &lt; Subject
  # For the sake of simplicity, the Subject&#39;s state, essential to all
  # subscribers, is stored in this variable.
  attr_accessor :state

  # @!attribute observers
  # @return [Array&lt;Observer&gt;] attr_accessor :observers private :observers

  def initialize
    @observers = []
  end

  # List of subscribers. In real life, the list of subscribers can be stored
  # more comprehensively (categorized by event type, etc.).

  # @param [Observer] observer
  def attach(observer)
    puts &#39;Subject: Attached an observer.&#39;
    @observers &lt;&lt; observer
  end

  # @param [Observer] observer
  def detach(observer)
    @observers.delete(observer)
  end

  # The subscription management methods.

  # Trigger an update in each subscriber.
  def notify
    puts &#39;Subject: Notifying observers...&#39;
    @observers.each { |observer| observer.update(self) }
  end

  # Usually, the subscription logic is only a fraction of what a Subject can
  # really do. Subjects commonly hold some important business logic, that
  # triggers a notification method whenever something important is about to
  # happen (or after it).
  def some_business_logic
    puts &quot;\nSubject: I&#39;m doing something important.&quot;
    @state = rand(0..10)

    puts &quot;Subject: My state has just changed to: #{@state}&quot;
    notify
  end
end

# The Observer interface declares the update method, used by subjects.
class Observer
  # Receive update from subject.
  def update(_subject)
    raise NotImplementedError, &quot;#{self.class} has not implemented method &#39;#{__method__}&#39;&quot;
  end
end

# Concrete Observers react to the updates issued by the Subject they had been
# attached to.

class ConcreteObserverA &lt; Observer
  # @param [Subject] subject
  def update(subject)
    puts &#39;ConcreteObserverA: Reacted to the event&#39; if subject.state &lt; 3
  end
end

class ConcreteObserverB &lt; Observer
  # @param [Subject] subject
  def update(subject)
    return unless subject.state.zero? || subject.state &gt;= 2

    puts &#39;ConcreteObserverB: Reacted to the event&#39;
  end
end

# The client code.

subject = ConcreteSubject.new

observer_a = ConcreteObserverA.new
subject.attach(observer_a)

observer_b = ConcreteObserverB.new
subject.attach(observer_b)

subject.some_business_logic
subject.some_business_logic

subject.detach(observer_a)

subject.some_business_logic</code></pre>
</figure>

#### []{#example-0--output-txt .anchor} **output.txt:** Résultat de l'exécution

<figure class="code">
<pre class="code" lang="output"><code>Subject: Attached an observer.
Subject: Attached an observer.

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 1
Subject: Notifying observers...
ConcreteObserverA: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 10
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event

Subject: I&#39;m doing something important.
Subject: My state has just changed to: 2
Subject: Notifying observers...
ConcreteObserverB: Reacted to the event</code></pre>
</figure>

## **Observateur** dans les autres langues

[![Observateur en
C#](../../../../images/patterns/icons/csharp8c8a.svg?id=da64592defc6e86d57c39c66e9de3e58){width="53"
height="53"
loading="lazy"}](../csharp/example.html "Observateur en C#"){.prog-lang-link}
[![Observateur en
C++](../../../../images/patterns/icons/cpp9770.svg?id=f7782ed8b8666246bfcc3f8fefc3b858){width="53"
height="53"
loading="lazy"}](../cpp/example.html "Observateur en C++"){.prog-lang-link}
[![Observateur en
Go](../../../../images/patterns/icons/go3287.svg?id=1a89927eb99b1ea3fde7701d97970aca){width="53"
height="53"
loading="lazy"}](../go/example.html "Observateur en Go"){.prog-lang-link}
[![Observateur en
Java](../../../../images/patterns/icons/java9ca9.svg?id=e6d87e2dca08c953fe3acd1275ed4f4e){width="53"
height="53"
loading="lazy"}](../java/example.html "Observateur en Java"){.prog-lang-link}
[![Observateur en
PHP](../../../../images/patterns/icons/phpb8be.svg?id=be1906eb26b71ec1d3b93720d6156618){width="53"
height="53"
loading="lazy"}](../php/example.html "Observateur en PHP"){.prog-lang-link}
[![Observateur en
Python](../../../../images/patterns/icons/python25f6.svg?id=6d815d43c0f7050a1151b43e51569c9f){width="53"
height="53"
loading="lazy"}](../python/example.html "Observateur en Python"){.prog-lang-link}
[![Observateur en
Rust](../../../../images/patterns/icons/rusta244.svg?id=1f5698a4b5ae23fe79413511747e4a87){width="53"
height="53"
loading="lazy"}](../rust/example.html "Observateur en Rust"){.prog-lang-link}
[![Observateur en
Swift](../../../../images/patterns/icons/swift68dd.svg?id=0b716c2d52ec3a48fbe91ac031070c1d){width="53"
height="53"
loading="lazy"}](../swift/example.html "Observateur en Swift"){.prog-lang-link}
[![Observateur en
TypeScript](../../../../images/patterns/icons/typescript325e.svg?id=2239d0f16cb703540c205dd8cb0c0cb7){width="53"
height="53"
loading="lazy"}](../typescript/example.html "Observateur en TypeScript"){.prog-lang-link}
:::::::::::::::

::::::: {.prom .banner-sidebar .banner .banner-striped .banner-examples .banner-removable .banner-removable-patterns data-id="DP: Examples-IDE" creative-id="examples-sidebar-fr" position="sidebar"}
:::::: {.banner-inner style="font-size: 14px; line-height: 21px;"}
::: banner-examples-img
[![](../../../../images/patterns/banners/examples-ide91ca.png?id=3115b4b548fb96b75974e2de8f4f49bc){width="215"
height="158" loading="lazy"
srcset="/images/patterns/banners/examples-ide-2x.png?id=93c007a6d157b97c28eb213f59d716ae 2x"}](../../book.html)
:::

::: banner-examples-fade
[ Archive avec exemples](../../book.html){.btn .btn-secondary
.btn-block}
:::

::: {.banner-examples-text style="margin-top: 1rem;"}
Achetez l'ebook **Plongée au cœur des patrons de conception** et obtenez
l'accès à une archive comprenant des dizaines d'exemples détaillés qui
peuvent être ouverts dans votre environnement de développement.

[ En savoir plus...](../../book.html){.btn .btn-secondary .btn-block}
:::
::::::
:::::::
:::::::::::::::::::::
::::::::::::::::::::::
